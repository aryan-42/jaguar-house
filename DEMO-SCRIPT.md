# Jaguar House — Demo Script

**Audience:** Internal stakeholders, HR leadership, or innovation/digital team
**Runtime:** ~10–12 minutes end-to-end
**URL:** `http://localhost:3000` (or your live Vercel URL)
**Admin:** `http://localhost:3000/admin`

---

## Before You Start

- Open the app in a browser at full screen (F11 or Presentation Mode)
- Make sure the tab has audio permission ready (you'll arm it once inside)
- Have the admin dashboard open in a second tab ready to switch to
- Close DevTools — keep the console hidden
- If demoing remotely, share your full screen, not just the window

---

## Act 1 — The Landing (1–2 min)

**What to say:**

> "This is the first thing a new joiner sees when they open their onboarding link on day one."

Let the hero animation settle for a moment before continuing.

> "We wanted it to feel less like a portal and more like an arrival. The language, the visuals, the pace — it's all designed around the Jaguar brand tone from the very first second."

**Point out:**
- The "Launch the Drive" call-to-action
- The ambient background motion (subtle, not distracting)
- The tagline framing the experience as a journey, not a form

> "There's no username and password wall, no onboarding checklist in a PDF. You click the button and you're already in it."

**Click "Launch the Drive"** to move to the cockpit.

---

## Act 2 — The Cockpit Form (2 min)

**What to say:**

> "This is the cockpit. It's where the new joiner sets up their profile before they take the wheel."

Fill in the form live. Suggested values:
- **Name:** Alex Mercer
- **Employee ID:** EMP-24001
- **Role:** Customer Experience Specialist
- **Region:** Jaguar House London
- **Manager:** Sofia Hartmann
- **Start Date:** today's date
- **Focus Track:** Retail Experience

> "Notice the focus track — this isn't decorative. It actually changes the content inside some of the modules. A retail specialist sees different scenarios to someone in operations or aftersales."

**Point out the employee import panel** (top right):
> "In a live deployment, this pulls from your HRIS. The field auto-fills from Workday or SuccessFactors — the new joiner doesn't have to type a thing."

**Click "Begin the Drive"** to enter the campus.

---

## Act 3 — The Campus Drive (2–3 min)

**What to say:**

> "Now they're in Jaguar House. This is a purpose-built virtual district — there are roads, intersections, a raised bridge section, six checkpoint zones. The new joiner drives using W/A/S/D or the arrow keys."

**Drive around briefly** — show the road network, the checkpoint markers.

> "Each gold beacon you can see is a checkpoint — one for each onboarding module. They unlock in sequence, so there's no skipping ahead. You have to complete HR before you drive to Safety, and so on."

**Show the HUD:**
> "The heads-up display in the corner shows the employee name, current checkpoint status, and their XP progress. It's a small thing, but it means the new joiner always knows exactly where they are."

**Arm the audio** using the HUD audio button:
> "The audio is opt-in — browsers require user interaction before playing sound. Once armed, you get a procedural V8 engine soundscape that reacts to your speed. It's subtle, but it shifts the feel of the experience."

**Drive to Checkpoint 1** (Reception Gallery — the nearest marker).

---

## Act 4 — Checkpoint 1: HR Introduction (3 min)

**What to say:**

> "When the car gets close to a checkpoint, it triggers automatically. Watch the zone change as we approach — the lighting shifts, the zone label appears on the boards."

**Enter the checkpoint zone** to open the overlay.

> "The module opens as an overlay on top of the world. You can still see the campus behind it — it never fully disconnects from the physical space."

**Walk through the steps:**

1. **Welcome brief** — read the intro copy aloud briefly:
   > "Day one welcome — straightforward, no jargon. Getting the badge, sorting access, meeting the key contact."

2. **Choice step** — show the branching question:
   > "This is where it gets interesting. It's not multiple-choice trivia — it's a real decision. The answer they pick is logged and feeds into the HR dashboard."

3. **Day-one checklist** — show the interactive checklist:
   > "These aren't just checkboxes. Each one unlocks when you interact with it, and progress is saved continuously. If the new joiner closes the browser and comes back tomorrow, they pick up exactly where they left off."

4. **Complete the module** — confirm and close.

> "Completion plays a brief animation and XP is awarded. They unlock the next checkpoint and can keep driving."

---

## Act 5 — The Admin Dashboard (2 min)

**Switch to the admin tab** (`/admin`).

> "Now let me show you what this looks like from the HR side."

If there are no sessions yet, navigate to `/admin/login` and log in with the configured credentials.

**Walk through the dashboard:**

- **Session count** — total joiners who have started
- **Completion rate** — percentage who finished all six modules
- **Decision posture chart** — shows how many people made considered vs. risk-adjacent choices in the branching scenarios
- **Focus track breakdown** — which tracks are most common
- **Checkpoint drop-off** — which module has the highest abandonment rate

> "This is all live, from real session data. Nothing is manually entered. Every time a new joiner drives through a module, the system captures their progress and decisions automatically."

**Show the CSV export:**
> "One click gives you a filtered export. You can slice by track, region, status, or decision tone — then take it straight into your reporting tools."

---

## Wrap Up (1 min)

> "So what you've seen today is a fully working prototype — not a mockup, not a Figma file. A real Next.js application with a 3D world, session persistence, a backend database, and an admin dashboard."

> "The path from here to production is clear: connect it to your HRIS, replace the procedural world with real campus imagery, expand the checkpoint modules into longer learning journeys, and integrate with your LMS for tracked completions."

> "The onboarding experience is the first impression an employer makes on a new hire. This is our proposal for what that first impression looks like."

---

## Handling Common Questions

**"What happens if someone closes the browser halfway through?"**
> The session is automatically synced to a backend database. When they reopen the link, they're prompted to resume. Progress is fully restored — no module needs to be repeated.

**"Can the content be customised per department / region?"**
> Yes — the focus-track system handles this today. Extending it to departments, regions, or business units is a configuration change, not a rebuild.

**"How does this connect to our existing HRIS?"**
> The employee directory has a configurable HTTP provider. You point it at your Workday or SuccessFactors API endpoint and the cockpit pulls records directly. No manual data entry.

**"Who maintains the module content?"**
> Right now the content lives in TypeScript files — a developer would update it. In production, you'd connect a headless CMS (Contentful, Sanity) so HR can update copy without touching code.

**"What about mobile / tablet?"**
> The current prototype is desktop-optimised for the driving mechanic. A mobile-first version would need a different interaction model — tap-to-drive rather than keyboard — which is a known next step.

---

## Keyboard Shortcuts (for reference during demo)

| Key | Action |
|-----|--------|
| `W` / `↑` | Accelerate |
| `S` / `↓` | Brake / Reverse |
| `A` / `←` | Steer left |
| `D` / `→` | Steer right |
| `H` | Toggle HUD |

---

*Good luck tomorrow — the app speaks for itself.*
