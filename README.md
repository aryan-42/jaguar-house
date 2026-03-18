# Jaguar House — Gamified Onboarding Prototype

A premium onboarding web app prototype built with Next.js, TypeScript, Tailwind, Framer Motion, and React Three Fiber.

The experience is structured as a cinematic employee arrival flow:

1. Landing hero with luxury automotive framing
2. Cockpit-style onboarding form
3. Transition into a simplified 3D campus
4. Keyboard-driven checkpoint navigation for onboarding modules
5. Completion handover screen after all modules are finished

## Stack

- Next.js App Router
- TypeScript
- Tailwind CSS
- Framer Motion
- React Three Fiber
- Local state via React context and reducer

## Features

- Dark premium UI with warm gold and champagne accents
- Cinematic screen transitions and staged handoff into the 3D world
- Browser-friendly arcade driving with third-person chase camera
- Road-constrained driving over a purpose-built realistic district road network
- A procedural premium district world with landscaped boulevards, bridge sections, plazas, skyline massing, and checkpoint courts
- Interactive checkpoint modules with guided steps, branching scenarios, inspections, ordered-response exercises, validation gates, and persistent progress
- Checkpoint-specific scene states with zone lighting shifts, target halos, route guidance, and mission-context HUD copy
- Focus-track-specific onboarding variants for retail, events, operations, aftersales, and digital support routes
- Manager-aware and team-aware route personalization with assigned buddy, first-week cadence, and handover planning
- Completion handover summarizes decision posture from branching scenarios, not just module completion
- Checkpoint zones include animated signage, contextual screen content, and modular setpiece props around each stop
- Read-only `/admin` dashboard for session volume, completion, decision posture, focus-track breakdowns, and checkpoint drop-off
- Protected `/admin` access with environment-backed credentials, signed HTTP-only sessions, and logout support
- Admin filters and CSV export for track, region, status, decision tone, and search-based session review
- Employee directory import flow with local sample data and an HTTP provider hook for future HRIS integration
- Soft collision rails and barrier volumes keep the campus route controlled
- Checkpoint setpieces carry physical collision volumes so the vehicle cannot drive through module structures
- Sequential checkpoint unlock logic
- HUD for mission, controls, employee profile, and progress
- Six-checkpoint onboarding experience covering HR, safety, training, compliance, team structure, and benefits
- Hybrid persistence: local restore plus backend session sync with shareable resume links
- Lightweight session auth layered on top of access codes with per-session resume keys
- Opt-in procedural premium audio driven by vehicle speed, mission zones, and checkpoint events
- Real GLTF showcase asset in the campus with fallback rendering
- GLTF-driven player vehicle with live wheel steering/spin and procedural fallback
- Included basis transcoders, vehicle assets, and built-in fallbacks — no extra asset setup is required
- Architecture ready for future replacement with higher-fidelity models, environments, and backend data

## Getting Started

Install dependencies and start the dev server:

```bash
npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

Create a local environment file before using the protected admin flow:

```bash
cp .env.example .env.local
```

Required environment variables:

- `ADMIN_USERNAME`
- `ADMIN_PASSWORD`
- `ADMIN_SESSION_SECRET`

Optional employee directory integration variables:

- `EMPLOYEE_DIRECTORY_PROVIDER=local|http`
- `EMPLOYEE_DIRECTORY_URL`
- `EMPLOYEE_DIRECTORY_TOKEN`

## Driving Controls

- `W` or `ArrowUp`: accelerate
- `S` or `ArrowDown`: brake / reverse
- `A` or `ArrowLeft`: steer left
- `D` or `ArrowRight`: steer right

The car follows a shared district road mesh that includes approach roads, a raised bridge segment, connected checkpoint courts, and landscaped city blocks.
Off-road moves are rejected when no valid drivable surface exists underneath, so the onboarding path stays attached to the built world.

## Persistence

- Local state is stored in `localStorage` under `jaguar-house-onboarding:v4`.
- A prototype backend stores synced sessions in `data/onboarding-sessions.db`.
- The drive flow creates a cloud session, assigns an access code, and keeps progress synced as checkpoints complete.
- New cloud sessions also receive a separate resume key used to authorize restore and sync actions.
- Resume works through a secure `?session=ACCESSCODE&auth=RESUMEKEY` link or by entering both the access code and resume key on the landing screen.
- Legacy sessions created before the auth upgrade continue to work by treating the access code as the resume key until they are recreated.
- If a legacy `data/onboarding-sessions.json` file exists from the previous prototype revision, it is migrated into SQLite on first access.
- Resetting the experience clears the local route state while keeping the audio preference; backend session records remain available for explicit resume links.

## Employee Directory

- `GET /api/employee-directory?identifier=EMP-24002` returns a single employee record for cockpit import.
- `GET /api/employee-directory?query=London` returns a filtered search slice for the cockpit search panel.
- The default `local` provider reads `data/employee-directory.json`.
- Set `EMPLOYEE_DIRECTORY_PROVIDER=http` plus `EMPLOYEE_DIRECTORY_URL` to point the same flow at a real HRIS gateway later.
- Imported records hydrate `employeeId`, `fullName`, `role`, `region`, `manager`, `startDate`, and `focusTrack`.

## Audio

- Audio is opt-in because browsers gate `AudioContext` behind user interaction.
- Use the HUD button to arm audio, then toggle it on or off during the drive stage.
- The sound layer is procedural: no external audio files are required.
- The current soundscape layers engine harmonics, road texture, wind rush, cabin rumble, zone-specific ambient beds, arrival stingers, mild distortion, and reverb-backed checkpoint cues.

## Onboarding Flow Logic

- Only one new checkpoint is unlocked at a time.
- Completed checkpoints remain visible and reviewable.
- Future checkpoints stay dimmed and inactive.
- After all six modules are complete, the final onboarding handover button appears.

## 3D World

- The drive world uses a fully procedural district layout defined in `lib/realistic-district-layout.ts` and rendered through `components/world/realistic-district-scene.tsx`.
- The district includes 18 road segments, landscaped boulevards, a raised bridge, plazas, skyline massing, and six checkpoint courts — all generated in code with no external asset dependencies.
- Six in-world checkpoints are anchored to the road network, each with its own zone lighting, scene profile, and setpiece props.
- The player vehicle can use a GLTF/GLB asset via `lib/car-concept.ts`; if the asset fails or is absent the scene falls back to a built-in mesh so the prototype always renders.
- Basis transcoders are bundled in `public/assets/basis/`.
- The world is designed to be swap-ready: higher-fidelity environments and production assets can replace the procedural layer without changing the onboarding state flow.

## Backend Session API

- `POST /api/onboarding-sessions` creates a new synced onboarding session from the current state.
- `GET /api/onboarding-sessions/[sessionKey]` restores a session by UUID or short access code and requires the `x-onboarding-auth` header.
- `PUT /api/onboarding-sessions/[sessionKey]` updates an existing session with the latest onboarding state and requires the `x-onboarding-auth` header.
- The current implementation uses a local SQLite database through `better-sqlite3` for prototype simplicity.

## Admin Dashboard

- `GET /admin` renders a server-side reporting dashboard directly from the SQLite session store.
- `/admin` is protected by a signed HTTP-only session cookie and redirects to `/admin/login` when unauthenticated.
- `POST /api/admin-auth/login` authenticates against the configured environment credentials.
- `POST /api/admin-auth/logout` clears the admin session cookie.
- The dashboard summarizes total sessions, completion rate, average progress, decision posture, track and region breakdowns, checkpoint completion rates, and the most recent session activity.
- Filters support free-text query plus track, region, status, and decision-tone slicing on the server.
- `GET /admin/export` downloads the currently filtered session slice as CSV.
- Sessions with recorded risk decisions or stalled progress are surfaced in an attention queue.

## Validation

- `npm run typecheck` runs the TypeScript-only validation pass.
- `npm run test` runs the Node test suite for auth, employee directory, reporting, and personalization logic.
- `npm run build` verifies the Next.js production build.
- `npm run ci:check` runs the full local CI sequence: typecheck, tests, and production build.
- `.github/workflows/ci.yml` runs the same validation path on pushes and pull requests.

## Future Upgrade Points

- Replace procedural district architecture with production assets for higher visual fidelity
- Persist onboarding progress with an API or database
- Add audio design, engine feedback, or controller support
- Add analytics hooks for HR funnel tracking
- Expand each checkpoint into richer workflows or embedded training content
- Add checkpoint-specific world audio and more advanced scenario types
- Add write actions and deeper drill-down views to the admin reporting deck
- Replace the sample employee directory with a real HRIS/LMS/calendar integration layer

## Notes

- Styling is intentionally premium and dark, with CSS variables defined in `app/globals.css`.
- The world logic is isolated so the environment can be upgraded without changing the onboarding state flow.
