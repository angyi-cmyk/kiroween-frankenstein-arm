# Requirements — RoboArm NL (Kiroween 2026, Frankenstein track)

## Introduction

RoboArm NL is a single-page static web app: the user types one English
sentence describing a grasping action ("pick up the red cube and place it
on the shelf") and a 3D simulated robot arm executes it in the browser.
It combines robot-arm kinematics simulation, natural-language instruction
understanding, and Web 3D visualisation in one app, built end-to-end with
Kiro spec-driven development. Track: Frankenstein. Deadline: submission
closes 2026-12-04 14:00 PT.

## Glossary

- **FK (forward kinematics):** computing the gripper (end-effector) pose
  from the joint angles.
- **IK (inverse kinematics):** computing the joint angles needed to put the
  gripper at a target pose. Must be a real analytic solve, never a canned
  animation.
- **DOF:** degrees of freedom. The arm has 4: base yaw (J0), shoulder
  pitch (J1), elbow pitch (J2), wrist pitch (J3).
- **NL command:** a natural-language instruction typed by the user.
- **EE:** end-effector (the gripper tip).

## Requirements

### Requirement 1: 3D scene

**User Story:** As a user, I want to see a 3D robot arm on a workbench
with graspable objects, so I can watch it execute my commands.

#### Acceptance Criteria

1. WHEN the page loads, THE APP SHALL render a 3D scene containing a
   workbench table, a 4-joint articulated arm, a raised shelf, and at
   least 4 graspable objects of distinct colors and shapes.
2. WHEN the page loads, THE APP SHALL show the scene from a clear default
   camera angle, and the user can orbit/zoom the camera.
3. WHEN the page loads, THE APP SHALL render without console errors and
   reach an interactive frame rate on a normal laptop.

### Requirement 2: Forward kinematics and live readouts

**User Story:** As a user, I want to see the arm's joint angles and
end-effector pose update live, so I can tell the motion is really
computed, not a canned animation.

#### Acceptance Criteria

1. WHEN the arm moves, THE APP SHALL display all four joint angles in
   degrees, updating every frame.
2. WHEN the arm moves, THE APP SHALL display the end-effector position
   (x, y, z) in metres, updating every frame.
3. WHEN the arm moves, THE APP SHALL display the gripper state
   (open/closed) and the object currently held, if any.
4. THE APP SHALL compute the end-effector pose from the joint angles
   with a pure FK function, and the displayed pose SHALL match the 3D
   scene-graph pose within 1 mm.

### Requirement 3: Inverse kinematics

**User Story:** As a user, I want the arm to reach the object I named,
so the grasp actually happens at the right place.

#### Acceptance Criteria

1. WHEN a target pose is requested, THE APP SHALL solve the joint
   angles analytically (base yaw from the target bearing, planar two-link
   IK for shoulder/elbow, wrist pitch from the desired gripper
   orientation).
2. WHEN a target is outside the arm's reach or outside a joint limit,
   THE APP SHALL report it as unreachable in plain language and SHALL
   NOT move the arm to a wrong pose.
3. THE IK SOLUTION SHALL place the gripper within 5 mm of the requested
   target for every reachable object and for the shelf.

### Requirement 4: Natural-language command parsing

**User Story:** As a user, I want to type one English sentence and have
the arm understand which object to pick up and where to put it.

#### Acceptance Criteria

1. THE APP SHALL parse commands of the forms "pick up the red cube",
   "grab the blue cylinder", "pick up the red cube and place it on the
   shelf", "move the green cube to the table", and "place it on the
   shelf" (where "it" means the held or last-mentioned object).
2. THE APP SHALL recognise colors red, blue, green, yellow, orange and
   shapes cube/box/block, cylinder, sphere/ball.
3. THE APP SHALL recognise the targets "shelf" and "table".
4. THE APP SHALL recognise "reset" and "home" as a command to return
   the arm to its home pose and release any held object.
5. WHEN a command cannot be understood, THE APP SHALL say so in plain
   language, name what it did understand (if anything), and show the
   supported command shapes.
6. THE APP SHALL state clearly in the interface that commands are
   English only.

### Requirement 5: Pick–transport–place execution

**User Story:** As a user, I want the arm to visibly grasp the object,
carry it, and set it down, so the whole action is believable.

#### Acceptance Criteria

1. WHEN a pick command is accepted, THE APP SHALL move the gripper to
   an approach point above the object, descend to it, close the gripper,
   and lift the object clear of the table.
2. WHEN an object is grasped, THE APP SHALL attach it to the gripper so
   it moves with the arm, and release it only when the gripper opens.
3. WHEN a place command is accepted, THE APP SHALL carry the held object
   to the target, lower it onto the surface, open the gripper, and
   retreat to a safe pose.
4. WHEN a motion is running, THE APP SHALL show a status line naming
   the current phase (e.g. "Moving to red cube…", "Grasping…",
   "Placing on shelf…").
5. WHEN a new command arrives while the arm is busy, THE APP SHALL
   queue it and run it after the current action finishes.
6. THE APP SHALL provide a Stop button that halts motion safely and a
   Reset button that returns the arm home.

### Requirement 6: User interface

**User Story:** As a user, I want a clean interface with examples, so I
can try the app in one click without reading a manual.

#### Acceptance Criteria

1. THE APP SHALL provide a text input and a Run button for commands.
2. THE APP SHALL provide at least three one-click example commands.
3. THE APP SHALL list the graspable objects with their colors and
   shapes, and show short usage instructions.
4. THE APP SHALL be usable at desktop width and remain readable at a
   narrow (mobile) width.

### Requirement 7: Static deployment

**User Story:** As a developer, I want the app to be one static page I
can host on GitHub Pages, so the submission has a working public link.

#### Acceptance Criteria

1. THE APP SHALL be a single `index.html` (plus repo docs) with no
   build step and no server code.
2. THE APP SHALL load Three.js from a public CDN with a pinned version
   and SHALL work when served from GitHub Pages.
3. THE APP SHALL NOT require any API key, login, or payment.

### Requirement 8: Kiro usage (contest rule)

**User Story:** As a contestant, I want the work to be genuinely built
with Kiro, so the entry is eligible.

#### Acceptance Criteria

1. THE REPO SHALL contain `.kiro/specs/robot-arm-nl-3d/` with
   `requirements.md`, `design.md`, and `tasks.md` that describe the
   app as actually built.
2. THE IMPLEMENTATION SHALL be produced through the Kiro CLI agent
   (`kiro-cli chat --no-interactive`) driven by the spec's tasks, and
   the agent session log SHALL be kept in the workspace.
3. THE REPO README SHALL describe how Kiro was used.
