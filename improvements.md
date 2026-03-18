# Jaguar House Onboarding — Improvement Audit

Based on a full code review of the running prototype. Findings are grouped by priority, with specific file references and concrete fixes.

---

## Priority 1 — Bugs & Inconsistencies

### 1.1 BMW naming still in environment variables and admin auth

`.env.example` and `.env.local` still use `BMW_ADMIN_USERNAME`, `BMW_ADMIN_PASSWORD`, `BMW_ADMIN_SESSION_SECRET`, and `BMW_EMPLOYEE_DIRECTORY_*`. The admin auth module (`lib/admin-auth.ts`) reads these. After the Jaguar rebrand, these should be renamed to brand-neutral keys like `ADMIN_USERNAME`, `ADMIN_PASSWORD`, etc.

**Files:** `.env.example`, `.env.local`, `lib/admin-auth.ts`

### 1.2 Cockpit form still uses BMW-era blue accent colors

The cockpit form (`components/screens/cockpit-form.tsx`) uses hardcoded blue tones (`#7fd8ff`, `#8dbfff`, `#2c72ff`, `#7bd5ff`, `#91a5c0`, `#0a101a`) that clash with the Jaguar green/ivory/champagne palette established in `globals.css` and `brand.ts`. Same issue in the completion screen with blue radial gradients.

**Files:** `components/screens/cockpit-form.tsx` (lines 16, 46, 55, 66, 68, 73, 78-79, 86, 93-94), `components/screens/completion-screen.tsx` (line 73), `components/world/checkpoint-overlay.tsx` (step badges still use `#7fdfff`, `#6ef2bf`)

### 1.3 Hardcoded brand copy outside brand config

The cockpit form contains `"Configure the Jaguar House handover."` directly in JSX (line 57) rather than pulling from `brandTheme`. The driving stage loading text also interpolates brand labels but several other strings are hardcoded. This means a future rebrand would require another full sweep.

**Files:** `components/screens/cockpit-form.tsx`, `components/screens/driving-stage.tsx`, `components/world/driving-hud.tsx`

### 1.4 Duplicate `formatPercent` utility

`formatPercent` is defined in both `lib/utils.ts` (line 31) and `app/admin/page.tsx` (line 24). The admin page should import from utils.

**Files:** `lib/utils.ts`, `app/admin/page.tsx`

---

## Priority 2 — Architecture & Maintainability

### 2.1 Monster files need decomposition

Several files are too large for comfortable maintenance:

| File | Lines | Problem |
|------|-------|---------|
| `lib/module-experiences.ts` | 891 | All 6 modules' step content defined inline |
| `components/world/campus-world.tsx` | 835 | Physics, collision, keyboard input, camera, environment, route guide all in one file |
| `components/world/checkpoint-overlay.tsx` | 747 | 7 different step-type renderers plus the overlay shell |
| `components/world/realistic-district-scene.tsx` | 480 | Building, tree, road, water rendering in one component |

**Suggested splits:**

- `campus-world.tsx` → extract `vehicle-controller.ts` (physics/collision/keyboard), `chase-camera.ts` (camera logic), `route-guide.tsx` (the guide strip), `environment.tsx` (checkpoint placement)
- `checkpoint-overlay.tsx` → extract each step type (`ChoiceStep`, `ScenarioStep`, `ChecklistStep`, `HotspotStep`, `SequenceStep`, `ConfirmStep`, `BriefStep`) into `components/world/steps/`
- `module-experiences.ts` → move content to JSON files under `data/modules/`, import and type-check at build time
- `realistic-district-scene.tsx` → split into `district-buildings.tsx`, `district-vegetation.tsx`, `district-roads.tsx`, `district-water.tsx`

### 2.2 Type duplication between campus-world and drivable-vehicle

`VehicleMotionState` in `drivable-vehicle.tsx` (line 11) is structurally identical to `MotionState` in `campus-world.tsx` (line 56). Extract into `lib/types.ts`.

### 2.3 State management surface area is wide

The `OnboardingContextValue` type has 24 fields. The provider file manages local storage, remote sync, reducer dispatch, session hydration, and resume logic in a single context. Consider splitting into:

- `OnboardingStateProvider` — pure reducer + local persistence
- `SessionSyncProvider` — remote session creation, fetching, syncing
- `OnboardingSelectors` — derived values (progress, nextModule, allModulesComplete)

### 2.4 No error boundaries around 3D canvas

If the Three.js canvas crashes (WebGL context lost, shader compilation failure, unsupported GPU), the entire app goes white. Wrap the `<Canvas>` in a React error boundary that shows a static fallback with the HUD and a "Resume" button.

**File:** `components/screens/driving-stage.tsx`

---

## Priority 3 — Performance

### 3.1 Three.js object allocation inside useFrame

In `campus-world.tsx`, the `RouteGuide` component creates `new THREE.Color(sceneProfile.route)` on every frame (line 179). This should be pre-allocated in a ref and only updated when `sceneProfile` changes.

### 3.2 Road surface raycasting is expensive

`sampleDriveSurface` fires a full raycaster against all road meshes every frame. Consider:
- Using a spatial hash or grid-based height lookup pre-computed from the road geometry
- Caching the last road segment and only re-raycasting when the vehicle crosses a segment boundary
- Reducing raycast frequency to every 2-3 frames and interpolating between samples

### 3.3 Building window meshes are heavily duplicated

Each `BuildingMass` creates individual mesh nodes per window row per face. With dozens of buildings, this is hundreds of draw calls. Use `InstancedMesh` for window strips, or bake window textures.

### 3.4 No `React.memo` on checkpoint zone setpieces

`CheckpointZoneSetpiece` receives `module`, `status`, and `highlighted` props but isn't memoized. Since there are 6 of them rendered every frame in the scene, they should be wrapped in `React.memo` with a shallow comparison.

### 3.5 Canvas has no performance tuning

The `<Canvas>` in `campus-world.tsx` uses default settings. Consider adding:
```tsx
<Canvas
  dpr={[1, 1.5]}          // cap pixel ratio for high-DPI screens
  performance={{ min: 0.5 }} // allow frame rate adaptation
  shadows="soft"           // or remove shadows entirely on low-end
  gl={{ antialias: true, powerPreference: 'high-performance' }}
>
```

---

## Priority 4 — UX & Accessibility

### 4.1 Almost zero accessibility attributes

Across all components, there are only ~27 instances of `aria-*`, `role=`, or `tabIndex`. The checkpoint overlay, HUD, and form are all heavily interactive but lack:
- `aria-label` on icon-only buttons (audio toggle, minimize, copy link)
- `role="dialog"` and `aria-modal="true"` on the checkpoint overlay
- Focus trapping inside the overlay when it opens
- `aria-live="polite"` on the progress/mission status in the HUD
- Keyboard navigation for the checkpoint step interactions (choice cards, checklist items)

### 4.2 No mobile fallback for 3D experience

The README mentions "desktop-first optimization for 3D driving experience" and suggests showing a message on mobile. Currently there's no such gate — the 3D canvas loads on mobile too, likely performing poorly. Add a viewport check that swaps the drive stage for a simplified click-through checkpoint navigator on small screens.

### 4.3 No loading progress for GLTF assets

The vehicle GLTF models can take several seconds to load. The current feedback is a text string ("Staging vehicle showcase..."). Add a progress bar using Three.js's `LoadingManager` or `useProgress` from drei.

### 4.4 Keyboard controls can conflict with browser

W/S/Arrow keys can scroll the page or trigger browser shortcuts. The keyboard event handlers don't call `event.preventDefault()`, which means on some browsers the arrow keys will also scroll the background.

**File:** `campus-world.tsx` (lines 462-474)

### 4.5 No skip/fast-travel option

Users who've partially completed modules and resume can't fast-travel to the next checkpoint. They must physically drive there. Add a "Navigate to next checkpoint" button in the HUD that teleports the vehicle.

---

## Priority 5 — Polish & Production Readiness

### 5.1 .env.local committed to git with real credentials

`.env.local` contains `BMW_ADMIN_PASSWORD=campus-control-2026` and a session secret. While `.gitignore` excludes `.env.local`, the file exists in the working tree. Verify it's not in git history with `git log --all -- .env.local`.

### 5.2 next.config.ts is empty

No configuration for image optimization, webpack customization, or header security. Consider adding:
- CSP headers
- `images.remotePatterns` if external images are used
- `serverExternalPackages: ['better-sqlite3']` to avoid bundling issues

### 5.3 No SEO/OG metadata

`layout.tsx` sets `title` and `description` from brandTheme but has no OpenGraph images, Twitter card meta, or favicon configuration beyond the SVG icon.

### 5.4 Test coverage is minimal

4 test files covering admin auth, employee directory, admin report, and team personalization. No tests for:
- Module experience step completion logic
- State sanitization (`onboarding-state.ts`)
- Session persistence round-trip
- Checkpoint status resolution
- Module interaction flows

### 5.5 Unused 3D scene components still in tree

`london-city-scene.tsx`, `campus-headquarters.tsx`, and `four-cylinder-landmark.tsx` appear to be remnants from the BMW era. If they're not rendered anywhere in the active flow, remove them to reduce bundle size.

### 5.6 CSS class naming inconsistency

Some styles are defined as CSS classes in `globals.css` (`.panel-glass`, `.editorial-shell`, `.blue-glow`) while others are inline Tailwind with hardcoded hex values. The codebase uses both approaches inconsistently. Consolidate either toward CSS custom properties consumed by Tailwind or toward the brand theme config.

---

## Quick Wins (can be done in under an hour each)

1. Rename `.env` variables from `BMW_*` to neutral names
2. Add `event.preventDefault()` to keyboard handlers in campus-world
3. Wrap Canvas in an error boundary
4. Delete unused scene components (london-city, campus-hq, four-cylinder)
5. Extract `formatPercent` duplication
6. Add `dpr` and `performance` props to Canvas
7. Add `React.memo` to `CheckpointZoneSetpiece` and `CheckpointBanner`
8. Pre-allocate THREE.Color objects in RouteGuide
9. Add `role="dialog"` and focus management to checkpoint overlay
10. Add a viewport gate that shows a mobile notice before loading the 3D canvas
