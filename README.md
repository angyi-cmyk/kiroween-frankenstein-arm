# RoboArm NL

**Type a sentence. Watch the arm do it.**

A single-page static web app built for the [Kiroween 2026](https://kiroween.devpost.com/) hackathon
(Frankenstein track). You type one plain-English grasping instruction — "pick up the red
cube and place it on the shelf" — and a simulated 4-DOF robot arm executes it in the
browser. Joint angles and end-effector pose are computed from real forward/inverse
kinematics, not a canned animation.

---

## Live demo

> **GitHub Pages:** `https://angyi-cmyk.github.io/kiroween-frankenstein-arm/`
> *(Live once the repo is pushed and GitHub Pages is enabled on the `main` branch.)*

---

## Run locally

No build step, no server, no API keys required.

```bash
git clone https://github.com/angyi-cmyk/kiroween-frankenstein-arm.git
cd kiroween-frankenstein-arm
python3 -m http.server 8080
# then open http://localhost:8080 in your browser
```

Three.js r160 is loaded from the unpkg CDN via an import map — the browser needs internet
access to fetch it on first load (or use a local cache).

---

## Controls

| Action | How |
|--------|-----|
| Orbit camera | Click and drag in the 3D view |
| Zoom | Scroll wheel |
| Pan | Right-click drag |
| Run a command | Type in the input box and press **Run** or Enter |
| One-click examples | Click any chip under "Examples" |
| Stop motion | **Stop** button |
| Return arm home | **Reset** button |

---

## Command syntax

Commands are **English only**. Supported forms:

```
pick up the <color> <shape>
grab the <color> <shape>
take / lift / get the <color> <shape>
pick up the <color> <shape> and place it on the <target>
move the <color> <shape> to the <target>
place / put the <color> <shape> on the <target>
place it on the <target>        ← "it" = held or last-mentioned object
reset  /  home  /  go home
```

**Colors:** red, blue, green, yellow, orange  
**Shapes:** cube / box / block, cylinder, sphere / ball  
**Targets:** shelf, table

---

## How Kiro was used

This project was built end-to-end with **[Kiro](https://kiro.dev)** spec-driven development
using the `kiro-cli` agent:

1. **Spec authoring** — The `.kiro/specs/robot-arm-nl-3d/` folder contains three documents
   written iteratively with Kiro:
   - `requirements.md` — user stories and acceptance criteria
   - `design.md` — scene layout, arm geometry, FK/IK math, NL grammar, sequencer, UI layout
   - `tasks.md` — ordered implementation tasks referencing the requirements

2. **Implementation** — The agent (`kiro-cli chat --no-interactive`) was given the task list
   and implemented tasks 1–8 in order, following the spec exactly. The agent session log is
   kept in `build-log-kiro-impl.txt`.

3. **Verification** — Each iteration was verified with a local HTTP server and headless
   Chromium (Playwright): zero console errors, in-page self-test (FK ≤1 mm, IK ≤5 mm,
   10 parser cases), and two end-to-end commands confirmed by object position checks.

---

## Architecture

Everything lives in `index.html` — one `<script type="module">` block containing:

| Section | What it does |
|---------|-------------|
| Scene setup | Three.js renderer, camera, OrbitControls, lights, table, floor grid, shelf, arm |
| Arm hierarchy | `baseGroup → shoulderGroup → elbowGroup → wristGroup → gripperGroup` |
| `forwardKinematics(j)` | Pure function: joint angles → gripper tip position |
| `inverseKinematics(P, φ)` | Analytic: bearing → yaw, wrist-centre offset, 2-link planar IK |
| `ikFeasible(P)` | Scans gripper pitch for a valid IK solution closest to straight-down |
| `parseCommand(text, state)` | Regex grammar: pick/place forms, pronoun resolution, reset |
| `buildPickSteps / buildPlaceSteps` | Waypoint sequences with IK at each point |
| Animation loop | `requestAnimationFrame`, smoothstep interpolation, object re-parenting |
| Self-test | FK vs scene graph (≤1 mm), IK round-trip (≤5 mm), 10 parser cases |

---

## Spec notes

Two prose points in `design.md` are inconsistent with the spec's own IK math and
reachability claim; the app satisfies the math (the binding contract):

1. **Pitch joint axis is local X, not "local Z".** With base yaw about world Y, the arm's
   reach direction is local +Z. Rotating about local X tilts the arm toward the table;
   rotating about Z would tilt it sideways and never reach any object.

2. **Grasp tip height is 0.30 m, not 0.24 m** ("object top + 0.02"). At 0.24 m, the green
   cube and yellow sphere are infeasible for every gripper pitch under the spec's joint
   limits (by ~2–3°), contradicting the design's own "all targets reachable" claim and the
   listed example commands. At 0.30 m all five objects are reachable at the exact spec
   limits.

---

## Credits

- **Liangyi Nie** — concept, spec, implementation direction
- **Kiro** (kiro-cli agent) — spec-driven implementation
- **Three.js r160** — 3D rendering ([threejs.org](https://threejs.org))

---

## License

MIT © 2026 Liangyi Nie — see [LICENSE](LICENSE).
