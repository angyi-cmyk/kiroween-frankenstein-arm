# Design — RoboArm NL (Kiroween 2026, Frankenstein track)

## Overview

One static page (`index.html`). Three.js r160 loaded from the unpkg CDN
through an import map (pinned version `0.160.0`); `OrbitControls` from
`three/addons/`. All app code lives in one `<script type="module">` in
the page: scene setup, arm model, kinematics, NL parser, animation
sequencer, and UI wiring. No build step, no server, no keys.

## Scene layout (units = metres, y up, table top at y = 0)

- Table: 3.4 × 0.12 × 2.2 box, top at y = 0, centred at the origin.
  A subtle grid texture on the floor around the table.
- Arm base: column at (0, 0, −0.75), base plate radius 0.16, column
  height 0.18. Shoulder pivot S = (0, 0.18, −0.75).
- Shelf: raised platform centred at (0.90, 0, 0.30), top surface at
  y = 0.45, footprint 0.70 × 0.45. Its front edge faces the arm.
- Graspable objects (all sitting on the table, y = 0):
  | id | shape | color | size | position (x, z) |
  |----|-------|-------|------|-----------------|
  | red-cube | cube | #d23b2e | 0.22 | (0.55, 0.15) |
  | blue-cylinder | cylinder | #2e6fd2 | r 0.11, h 0.22 | (−0.45, 0.35) |
  | green-cube | cube | #2ea44f | 0.22 | (−0.75, −0.15) |
  | yellow-sphere | sphere | #e0a800 | r 0.11 | (0.85, −0.35) |
  | orange-cylinder | cylinder | #e07b00 | r 0.11, h 0.22 | (0.10, 0.60) |
- Reach check (shoulder at S, max reach L1+L2+L3 = 1.65): the farthest
  target is the orange cylinder at horizontal distance 1.35 and the
  shelf drop point (0.90, 0.56, 0.30) at 3D distance ≈ 1.43. All
  targets are inside the reachable sphere with margin. The app still
  verifies reachability at run time (see IK) and refuses unreachable
  targets instead of stretching.

## Arm model (4 DOF)

Link lengths: L1 (shoulder→elbow) = 0.75, L2 (elbow→wrist) = 0.65,
L3 (wrist→gripper tip) = 0.25. Joint limits:
J0 yaw ±170°, J1 shoulder −10°…+120°, J2 elbow −150°…0° (elbow-up
configuration, negative = bent), J3 wrist −120°…+120°.

Scene-graph structure (links extend along local +Y, pitch joints rotate
about local Z, base yaw about world Y):

```
baseGroup (position 0,0,−0.75; rotation.y = J0)
 └ shoulderGroup (position 0,0.18,0; rotation.z = J1)
    └ upperArm mesh (box 0.13×0.75×0.13, centred y=+0.375)
    └ elbowGroup (position 0,0.75,0; rotation.z = J2)
       └ forearm mesh (box 0.11×0.65×0.11, centred y=+0.325)
       └ wristGroup (position 0,0.65,0; rotation.z = J3)
          └ wrist mesh + gripperGroup (position 0,0.25,0)
             ├ fingerL / fingerR (translate ±x to open/close)
```

Home pose: J0 = 0°, J1 = 35°, J2 = −70°, J3 = 35° (gripper pointing
forward-down, parked above the table centre).

## Forward kinematics

`forwardKinematics(j)` is a pure function of the four joint angles
(in radians) returning the gripper-tip position and the gripper
direction, computed with the same rotations as the scene graph:

- yaw J0 rotates the arm plane: plane direction u = (sin J0, 0, cos J0).
- In the plane, with shoulder at S: elbow E = S + L1·(sin a1, cos a1)
  where a1 is J1 measured from vertical; wrist W = E + L2·(sin(a1+a2),
  cos(a1+a2)); tip T = W + L3·(sin(a1+a2+a3), cos(a1+a2+a3)).
- The 3D position is S + (in-plane x)·u + (in-plane y)·(0,1,0).

The readouts call this function every frame. A dev check compares it
against the gripper's scene-graph world position; the two must agree
within 1 mm.

## Inverse kinematics (analytic)

Input: target tip position P (world) and desired gripper pitch
`gripPitch` (angle of the gripper axis from vertical in the arm plane;
−90° means pointing straight down for a grasp).

1. dx = P.x − S.x, dz = P.z − S.z, r = hypot(dx, dz).
   J0 = atan2(dx, dz), clamped to ±170°. (If r < 0.05 the yaw is
   left unchanged.)
2. Wrist centre: the tip is L3 beyond the wrist along the gripper
   direction, so the wrist must reach W = P − L3·g, where g is the
   unit gripper direction in the plane (g = (sin φ, cos φ) with
   φ = gripPitch measured from vertical, in plane coordinates).
   In plane coordinates: rw = r − L3·sin φ, yw = (P.y − S.y) − L3·cos φ.
3. Two-link IK on (rw, yw) with links L1, L2 (elbow-up):
   D = (rw² + yw² − L1² − L2²) / (2·L1·L2), clamped to [−1, 1];
   J2 = −acos(D) (negative = elbow bent up).
   J1 = atan2(rw, yw) − atan2(L2·sin J2, L1 + L2·cos J2).
   J3 = φ − J1 − J2.
4. Reachability: if hypot(rw, yw) > L1 + L2 − 0.001, or any joint is
   outside its limits after clamping by more than 0.5°, return null
   (unreachable) and let the UI report it.
5. All angles are converted to degrees only for display.

Grasp heights: approach point = target + 0.28 in y; grasp point =
object top + 0.02 (cube: centre y + 0.11; cylinder: top y + 0.02;
sphere: centre y + 0.11). Gripper points straight down (φ = −90°…
i.e. gripper axis vertical) for grasps; for transit the wrist keeps
the last orientation.

## Natural-language parser

Lowercase, trim, strip trailing punctuation. Grammar (regex-based,
checked in order):

1. `^(pick up|grab|take|lift|get) the (\w+) (\w+)( and (place|put) it on the (\w+))?$`
   → pick object, optionally place on target.
2. `^(place|put|move) the (\w+) (\w+) (on|onto|to) (the )?(\w+)$`
   → pick object then place on target.
3. `^(place|put) it (on|onto) (the )?(\w+)$` → place held object.
4. `^(reset|home|go home)$` → home + release.
5. `^(pick up|grab) the (\w+) (\w+)$` → covered by (1).

Vocabulary: colors red/blue/green/yellow/orange; shapes
cube|box|block → cube, cylinder → cylinder, sphere|ball → sphere;
targets shelf|table. "it" resolves to the held object, else the last
mentioned object. Anything else → parse error with the supported
shapes listed. Matching is by colour+shape pair against the live
object list, so a moved object is still found at its new position.

## Animation sequencer

A parsed command becomes a list of steps; each step is
`{ joints: [J0..J3] (radians), gripper: open|closed, duration, label }`.
Waypoints for pick(obj): approach (IK above obj, +0.28 y) →
descend (IK at grasp point) → close gripper (0.4 s) → lift (IK +0.30 y).
Waypoints for place(target): transit (IK above target at safe height
0.55) → descend (IK at drop point: shelf top + object half-height,
or table + half-height) → open gripper (0.4 s) → retreat (IK +0.30 y).
Joint motion interpolates with a smoothstep easing; duration scales
with the largest joint delta (0.5–1.6 s per move). The held object is
re-parented to the gripper group while carried (keeping its world
transform), and re-parented back to the scene on release, snapped to
the target surface. A command queue runs steps in order; Stop clears
the queue and freezes motion; Reset homes the arm and releases any
held object in place.

## UI layout

- Header: title "RoboArm NL", subtitle "Type a sentence. Watch the
  arm do it." plus a "Commands are in English" note.
- Left panel (320 px): command input + Run button; example chips
  ("Pick up the red cube and place it on the shelf", "Grab the blue
  cylinder", "Move the green cube to the table", "Place it on the
  shelf"); object legend (color dots + names); short instructions;
  status line (current phase or error).
- Main area: the 3D canvas (OrbitControls: rotate/zoom/pan).
- Readout bar under the canvas: J0–J3 in degrees, EE position
  (x, y, z) in metres, gripper open/closed, held object.
- Buttons: Stop, Reset. Style: dark, clean, system font stack,
  responsive (panel stacks above the canvas under 760 px width).

## File layout

```
index.html            # the whole app (import map + module script)
README.md             # demo link, run locally, how Kiro was used
LICENSE               # MIT
.gitignore            # node_modules, .DS_Store, editor files — NOT .kiro
.kiro/
  specs/robot-arm-nl-3d/{requirements,design,tasks}.md
  steering/product.md
demo-video/           # demo mp4 (added in the video step, not in v1)
```

## Verification plan

1. Serve locally (`python3 -m http.server`), load in headless Chromium,
   confirm no console errors and the canvas renders (screenshot).
2. In-page self-test on load: FK function vs scene-graph tip position
   (≤1 mm), IK round-trip on all 5 objects and the shelf (≤5 mm),
   parser unit cases (all grammar forms + one bad input). Results are
   printed to the console and shown in a small "self-test" line in the
   UI; a failed check blocks the "ready" state.
3. Drive two example commands end-to-end in the headless browser and
   confirm the object ends up on the target (position check).
