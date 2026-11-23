# Tasks — RoboArm NL (Kiroween 2026, Frankenstein track)

Work in `/home/hatch/workspace/kiroween`. Implement in order. Do not
change the specs to fit the code — if a spec point is infeasible, say
so in the task's notes and keep the spec as the contract. Commit
progress with git as you go (local commits; pushing needs GitHub auth
which is not available in this environment).

- [ ] 1. Scaffold the page and scene
  - Create `index.html` with import map pinning `three@0.160.0` from
    unpkg and `three/addons/` → `examples/jsm/`.
  - Set up renderer (antialias, shadows), perspective camera with a
    clear default view, OrbitControls, hemisphere + directional light
    with shadows, fog-free background.
  - Build the table, floor grid, and shelf per design.md positions.
  - Add the 5 objects from the design table with distinct colors and
    labels (small floating name sprites or a legend in the UI).
  - _Requirements: 1.1, 1.2, 1.3, 7.1, 7.2_

- [ ] 2. Build the arm model and FK
  - Build the 4-joint hierarchy from design.md (base yaw J0, shoulder
    J1, elbow J2, wrist J3; L1=0.75, L2=0.65, L3=0.25), with a
    two-finger gripper.
  - Implement `forwardKinematics(joints)` as a pure function and the
    live readouts (joint angles in degrees, EE x/y/z, gripper state,
    held object).
  - Implement the home pose (J0=0°, J1=35°, J2=−70°, J3=35°).
  - Verify FK vs scene-graph tip position agrees within 1 mm.
  - _Requirements: 2.1, 2.2, 2.3, 2.4_

- [ ] 3. Implement analytic IK
  - Implement `inverseKinematics(target, gripPitch)` exactly as in
    design.md (yaw from bearing, wrist-centre offset by L3, two-link
    elbow-up solve, joint-limit checks, null when unreachable).
  - Verify IK round-trip (FK(IK(P)) within 5 mm of P) for all 5
    objects and the shelf drop point.
  - _Requirements: 3.1, 3.2, 3.3_

- [ ] 4. Implement the NL command parser
  - Implement the grammar from design.md: pick/grab forms, pick-and-
    place in one sentence, move/place forms, "place it…" with pronoun
    resolution, reset/home.
  - Vocabulary: red/blue/green/yellow/orange; cube/box/block,
    cylinder, sphere/ball; targets shelf/table.
  - Return structured commands `{ action, object, target }` plus a
    clear parse-error message naming supported shapes.
  - _Requirements: 4.1, 4.2, 4.3, 4.4, 4.5, 4.6_

- [ ] 5. Implement the pick–transport–place sequencer
  - Waypoint builder: approach → descend → close → lift → transit →
    descend → open → retreat, with IK at each waypoint and smoothstep
    joint interpolation scaled by joint delta.
  - Gripper open/close animation; attach the grasped object to the
    gripper while carried; snap it onto the target surface on release.
  - Status line per phase; command queue; Stop and Reset buttons.
  - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5, 5.6_

- [ ] 6. Build the UI
  - Header, command input + Run, at least 3 example chips, object
    legend, short instructions, English-only note, status line.
  - Readout bar (J0–J3, EE position, gripper, held object) updating
    every frame; self-test line (see task 7).
  - Dark clean styling, responsive under 760 px width.
  - _Requirements: 6.1, 6.2, 6.3, 6.4_

- [ ] 7. In-page self-test + local verification
  - On load, run: FK vs scene graph (≤1 mm), IK round-trip on all
    targets (≤5 mm), parser cases for every grammar form plus one bad
    input. Show pass/fail in the UI self-test line and log to console.
  - Serve with `python3 -m http.server`, load in headless Chromium,
    confirm zero console errors and take a screenshot of the rendered
    scene.
  - Drive two example commands end-to-end headlessly and confirm the
    object ends on the target.
  - _Requirements: 1.3, 2.4, 3.3, 4.5_

- [ ] 8. Repo docs
  - `README.md`: what the app is, live demo link placeholder (GitHub
    Pages URL), run locally, controls, how Kiro was used (specs +
    kiro-cli agent implementation), credits.
  - `LICENSE`: MIT (Copyright 2026 Liangyi Nie).
  - `.gitignore`: node_modules, .DS_Store, editor files. Must NOT
    ignore `.kiro`.
  - _Requirements: 8.1, 8.3, 7.3_
