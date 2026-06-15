# AI301 Open Source Capstone

## Project

**Repository:** tgstation/tgstation
**Project Link:** https://github.com/tgstation/tgstation

## Selected Issue

**Issue:** #66224 - Can't wrench down chemistry equipment onto icebox snow without laying down rods and floor tiles first
**Issue Link:** https://github.com/tgstation/tgstation/issues/66224

## Why I Chose This Issue

I chose this issue because it appears well-scoped for a first open source contribution. The issue has already been discussed by multiple contributors, and the discussion identifies a likely root cause involving snow, ash, asteroid, and similar turfs being classified as `/turf/open/misc`.

This issue also has a clear gameplay impact. Chemistry and plumbing equipment cannot be secured on certain terrain types unless the player first places rods and floor tiles, which makes areas like Icebox snow, Lavaland, and TramStation chemistry more difficult to use.

## Initial Understanding

Plumbing and chemistry equipment can no longer be secured on certain terrain types because the generic wrenching logic checks whether the object is located on a floor turf.

The relevant logic appears to be in:

* `code/game/objects/objs.dm`
* `code/__DEFINES/is_helpers.dm`

The current helper macro appears to only treat `/turf/open/floor` as floor:

```dm
#define isfloorturf(A) (istype(A, /turf/open/floor))
```

However, snow, ash, asteroid, and similar terrain types are now under `/turf/open/misc`. Because of this, the existing wrenching logic rejects those turfs when trying to secure plumbing or chemistry equipment.

## Reproduction Process

### Environment Setup

I reviewed the tgstation repository README to understand the setup process.

The README says that building directly in DreamMaker is deprecated and may produce errors. The recommended setup path is to use the provided build scripts:

* `bin/server.cmd` for the quick build-and-host setup
* `bin/build.cmd` for the longer build process

I forked the repository and cloned my fork locally.

**Fork:**  https://github.com/junphuc/tgstation

**Working Branch:** https://github.com/junphuc/tgstation/tree/fix-chemistry-equipment-misc-turfs

Local setup steps:

1. Forked `tgstation/tgstation`.
2. Cloned my fork locally.
3. Created a working branch named `fix-chemistry-equipment-misc-turfs`.
4. Reviewed the README setup instructions.
5. Started setup using the tgstation build scripts from the `bin/` folder.
6. Started reviewing the likely relevant files:

   * `code/game/objects/objs.dm`
   * `code/__DEFINES/is_helpers.dm`

Setup notes:

* The README points to `bin/server.cmd` as the quick setup path.
* The README points to `bin/build.cmd` as the longer build path.
* The README warns that building directly in DreamMaker is deprecated.
* Current setup status: I have started local setup and identified the relevant build scripts and code files for this issue.

### Steps to Reproduce

Based on issue #66224, the expected reproduction process is:

1. Start a local tgstation server using the repository setup scripts.
2. Enter a test environment with Icebox snow, Lavaland ash, or asteroid turf.
3. Use a plumbing constructor to place chemistry or plumbing equipment on that turf.
4. Try to wrench the equipment down.
5. Expected behavior: the equipment should be secured onto the terrain.
6. Actual behavior: the equipment cannot be secured and shows a warning similar to:
   `"[equipment] needs to be on the floor to be secured!"`

### Reproduction Evidence

The issue discussion points to the likely failing logic in:

* `code/game/objects/objs.dm`
* `code/__DEFINES/is_helpers.dm`

The relevant wrenching logic is in `can_be_unfasten_wrench()`:

```dm
if(!(isfloorturf(loc) || isindestructiblefloor(loc)) && !anchored)
    to_chat(user, span_warning("[src] needs to be on the floor to be secured!"))
    return FAILED_UNFASTEN
```

The likely problem is that `isfloorturf(loc)` returns false for `/turf/open/misc` terrain such as snow, ash, and asteroid turf.

**Branch Link:** https://github.com/junphuc/tgstation/tree/fix-chemistry-equipment-misc-turfs

## Solution Approach

### Understand

The bug happens because chemistry and plumbing equipment use generic wrenching logic when being secured. That logic only allows objects to be secured on `/turf/open/floor` or indestructible floor turfs.

Since snow, ash, asteroid, and similar turfs are now categorized as `/turf/open/misc`, they fail the current floor check even though gameplay expects this equipment to be securable there.

### Match

The existing code uses helper macros such as `isfloorturf()` and `isindestructiblefloor()` to decide whether an object is on a valid surface.

I will look for similar construction, anchoring, machine-placement, or plumbing logic to see how tgstation handles non-standard but valid terrain types elsewhere in the codebase.

### Plan

1. Inspect `can_be_unfasten_wrench()` in `code/game/objects/objs.dm`.
2. Inspect turf helper definitions in `code/__DEFINES/is_helpers.dm`.
3. Search for existing helpers or procs that classify snow, ash, asteroid, or misc turfs as valid construction surfaces.
4. Avoid broadly changing `isfloorturf()` unless that is clearly safe, because many systems may depend on its current meaning.
5. Prefer the narrowest safe fix, likely by allowing relevant misc turfs for this equipment or this wrenching case without changing unrelated floor behavior.
6. Add or update tests if the project has relevant tests for object anchoring, wrenching, plumbing equipment, or turf validation.
7. Manually verify the fix by repeating the reproduction steps on Icebox snow, Lavaland ash, or asteroid turf.

### Review

Before opening a pull request, I will review tgstation's contributor guidelines and check that my change follows the project's style, naming, and testing expectations.

I will also make sure the commit and pull request explain:

* What behavior was broken
* Why it happened
* What code path was changed
* How I tested the fix

### Evaluate

I will verify the fix by:

* Confirming equipment can be wrenched down on Icebox snow or similar `/turf/open/misc` terrain.
* Confirming the original warning no longer appears for valid terrain.
* Confirming equipment still cannot be secured on invalid surfaces.
* Running any relevant tests or validation commands available in the project.
* Repeating the original reproduction steps after the fix.

## Current Status

* Issue selected: Yes
* Repository forked: Yes
* Working branch created: Yes
* Local setup started: Yes
* Reproduction process documented: Yes
* Implementation plan written: Yes
* Pull request opened: Not yet

## Next Step

My next step is to finish local reproduction, confirm the exact code path responsible for the failed wrenching behavior, and then implement the narrowest safe fix for this issue.
