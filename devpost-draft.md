# Devpost submission draft — RoboArm NL (Kiroween 2026, Frankenstein track)

> DRAFT — do not paste into Devpost until the repo is public on GitHub,
> GitHub Pages is live, and the demo video is uploaded to YouTube.
> Fields marked [FILL IN] need the user (account steps).

- **Project name:** RoboArm NL
- **Tagline:** Type a sentence. Watch the arm do it.
- **Track:** Frankenstein
- **Repo:** https://github.com/angyi-cmyk/kiroween-frankenstein-arm
- **Live demo:** https://angyi-cmyk.github.io/kiroween-frankenstein-arm/
- **Demo video:** [FILL IN — YouTube link after upload]
  (file: `~/workspace/kiroween/demo-video/roboarm-nl-demo.mp4`, 45 seconds,
  1280×800, screen recording of the live app running two real commands)

---

## Project description (paste into the Devpost "Description" field)

### Inspiration

Industrial robot arms are programmed in joint angles and waypoints — a
language only engineers speak. We wanted the opposite: tell the arm what
you want in one plain sentence, and watch it figure out the rest. The
Frankenstein track asked for a mash-up, so we fused three things that
don't usually live together: a robot-arm kinematics simulator, a
natural-language command parser, and a Web 3D visualisation — in a single
static web page.

### What it does

RoboArm NL is a single-page web app. A simulated 4-DOF robot arm
(base yaw, shoulder, elbow, wrist) stands at a worktable with five
graspable objects (red cube, blue cylinder, green cube, yellow sphere,
orange cylinder) and a shelf.

You type one English sentence — for example
"pick up the red cube and place it on the shelf" — and the arm executes
it in 3D: it solves the inverse kinematics for the target, moves to the
object, closes the gripper, carries the object, and sets it down. Other
supported forms: "grab the blue cylinder", "move the green cube to the
table", "place it on the shelf" (the pronoun resolves to the held or
last-mentioned object), "reset" / "home".

Nothing is a canned animation. Every motion is computed live:

- **Forward kinematics** — a pure function from joint angles to gripper
  tip position, checked against the scene graph every run (agreement
  ≤ 1 mm).
- **Inverse kinematics** — analytic: bearing → base yaw, wrist-centre
  offset, two-link planar IK (elbow-up), with a per-target gripper-pitch
  scan that finds the feasible pitch closest to straight down.
- **Live readouts** — joint angles J0–J3, end-effector x/y/z, gripper
  state and held object, updated every frame.
- **In-page self-test** — 20 checks (FK vs scene graph, IK round-trip on
  all five objects plus the shelf, 10 parser cases) run on load and
  report pass/fail in the UI.

The page also has one-click example commands, orbit/zoom/pan camera,
Stop and Reset, and a "Commands are in English only" note.

### How we built it

Built end-to-end with **Kiro** spec-driven development, using the
`kiro-cli` agent:

1. **Spec first.** `.kiro/specs/robot-arm-nl-3d/` holds `requirements.md`
   (user stories + acceptance criteria), `design.md` (scene layout, arm
   geometry, FK/IK math, NL grammar, sequencer, UI layout) and
   `tasks.md` (ordered implementation tasks), plus
   `.kiro/steering/product.md` (product principles). The spec was the
   contract — the agent implemented tasks 1–8 in order and the spec was
   only corrected when its own IK math proved two first-draft points
   wrong (pitch axis and grasp height; both corrections are documented
   in `design.md`'s Revisions section and in the README).
2. **Implementation.** One static `index.html`, Three.js r160 from the
   unpkg CDN, no build step, no server, no API keys — so it deploys
   straight to GitHub Pages.
3. **Verification.** Headless Chromium (Playwright): zero console
   errors, 20/20 self-tests, and end-to-end commands confirmed by
   measured object positions — "pick up the red cube and place it on
   the shelf" ends with the red cube at exactly (0.90, 0.56, 0.30) m,
   on the shelf.

### Challenges we ran into

- **The spec contradicted its own math.** The first design draft said
  pitch joints rotate about local Z and grasps happen at "object top +
  0.02". The agent's numeric checks showed both break the spec's
  reachability claim: Z-pitch tilts the arm sideways, and at 0.24 m two
  of the five objects are unreachable for every gripper pitch. We
  corrected the spec to the as-built truth (local X pitch, 0.30 m
  grasp tip height) and documented why, instead of quietly shipping
  code that disagreed with its spec.
- **A literal straight-down grasp is impossible** within the wrist
  joint limit (it needs J3 ≈ 150–158° vs a 120° limit), so grasps use
  the feasible pitch closest to straight down — down-and-slightly-forward.
- **Headless rendering is slow** (software rasteriser), so the demo
  video was recorded headless and sped up 2×, which matches the
  real-time pacing in a normal browser.

### Accomplishments that we're proud of

- Real kinematics, not keyframed motion: FK matches the rendered scene
  graph to 0.00 mm and IK round-trips to 0.00 mm on every target.
- A natural-language interface that actually resolves pronouns and
  reports unknown objects instead of guessing.
- A spec that survived contact with its own implementation — and was
  corrected in writing when it was wrong.

### What we learned

Spec-driven development only works if the spec is treated as a living
contract: writing the math down first caught two real design errors
before they became bugs, and the agent's feasibility scans were what
caught them.

### What's next

- Chinese-language commands (the parser is English-only today).
- More objects, obstacles, and multi-step instructions ("stack the red
  cube on the green cube").
- A second arm for hand-off tasks.

---

## How Kiro was used (also covered above; keep as the "Built with Kiro" note)

This project was built with Kiro spec-driven development from start to
finish: requirements, design and tasks were authored as a Kiro spec
(`.kiro/specs/robot-arm-nl-3d/`), steering in `.kiro/steering/`, and the
`kiro-cli` agent implemented the full task list (session log kept in the
repo as `build-log-kiro-impl.txt`). Kiro's agent also ran the numeric
verification that found and corrected two design errors in the spec.
The repository root contains the `.kiro` directory, as required.

---

## Submission checklist (for the user)

- [ ] Push the repo to GitHub as public `angyi-cmyk/kiroween-frankenstein-arm`
      (needs `gh auth login` — device flow — or a personal access token)
- [ ] Enable GitHub Pages on the `main` branch; open the live link and
      confirm the 3D scene renders
- [ ] Upload `demo-video/roboarm-nl-demo.mp4` to YouTube as **public**,
      paste the link above
- [ ] Submit on Devpost (Frankenstein track) with the description above
      before **2026-12-04 14:00 PT (Beijing 2026-12-05 06:00)**
