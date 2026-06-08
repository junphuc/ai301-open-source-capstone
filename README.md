# AI301 Open Source Capstone

## Project
tgstation

## Selected Issue
#66224 - Can't wrench down chemistry equipment onto icebox snow without laying down rods and floor tiles first

Issue Link:
https://github.com/tgstation/tgstation/issues/66224

## Why I Chose This Issue
This issue appears well-scoped for a first contribution. The bug has been reproduced by multiple contributors, and the discussion identifies a likely root cause involving the transition of snow/ash/asteroid turfs to `/turf/open/misc`.

## Initial Understanding
Plumbing and chemistry equipment can no longer be secured on certain terrain types because the wrenching logic checks for `/turf/open/floor`, while these terrain types are now categorized differently.

## Planned Next Steps
- Set up the tgstation development environment
- Reproduce the issue locally
- Identify the exact code path responsible
- Propose and implement a fix
- Submit a pull request
