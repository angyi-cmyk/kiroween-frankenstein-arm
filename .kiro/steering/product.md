# Product steering — RoboArm NL

## What this is

A one-sentence robot arm: the user types a plain-English grasping
instruction and a simulated 3D arm does it. Built for the Kiroween
2026 hackathon, Frankenstein track — the "monster" is the combination
of natural-language understanding, real robot kinematics, and Web 3D
in a single static page.

## Tone and quality bar

- The app must really work. A visitor should be able to type a command
  and watch the arm pick the object up and put it down — no canned
  animation, no fake readouts. Joint angles and end-effector pose are
  computed from real FK/IK and shown live.
- Clean and calm UI. Dark theme, one clear input, a few example chips,
  live numbers. Nothing flashy that distracts from the arm.
- English commands only, stated plainly in the UI. Do not pretend to
  support languages the parser does not handle.

## Hard constraints

- Single static page, no build step, no server, no keys, no payment.
- Three.js from a pinned public CDN version.
- Never invent identity, address, or residence information. Never touch
  passwords, keys, or payment details.
- The `.kiro` directory is part of the product: specs must describe the
  app as actually built, not as wished.
