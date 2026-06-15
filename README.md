# AI301 Open Source Capstone

## Project

tgstation/tgstation

## Selected Issue

**Issue:** #66224 - Can't wrench down chemistry equipment onto icebox snow without laying down rods and floor tiles first

**Issue Link:** https://github.com/tgstation/tgstation/issues/66224

## Why I Chose This Issue

I chose this issue because it appears well-scoped for a first open source contribution. The bug has been discussed by multiple contributors, and the issue thread identifies a likely root cause involving snow, ash, and asteroid turfs being changed to `/turf/open/misc`.

This issue also has a clear gameplay impact: chemistry and plumbing equipment cannot be secured on certain terrain types without first placing rods and floor tiles, which makes areas like Icebox snow, Lavaland, and TramStation chemistry more difficult to use.

## Initial Understanding

Plumbing and chemistry equipment can no longer be secured on certain terrain types because the wrenching logic checks whether the object is located on a floor turf.

The relevant logic appears to be in:

* `code/game/objects/objs.dm`
* `code/__DEFINES/is_helpers.dm`

The current helper macro:

```dm
#define isfloorturf(A) (istype(A, /turf/open/floor))
```

only treats `/turf/open/floor` as floor. However, snow, ash, asteroid, and similar terrain types are now under `/turf/open/misc` as of PR #65504. Because of this, the existing `can_be_unfasten_wrench()` logic rejects those turfs when trying to wrench down plumbing or chemistry equipment.

## Reproduction Process

### Environment Setup

I am setting up the tgstation development environment locally using my fork of the repository.

Fork: `<PASTE YOUR FORK LINK HERE>`

Working branch: `<PASTE YOUR BRANCH LINK HERE>`

Setup steps:

1. Forked `tgstation/tgstation`.
2. Cloned my fork locally.
3. Created a working branch named `fix-chemistry-equipment-misc-turfs`.
4. Started reviewing the relevant wrenching logic in `code/game/objects/objs.dm`.
5. Started tracing the turf helper macro in `code/__DEFINES/is_helpers.dm`.

Any setup issues encountered:

* `<Write any setup issue here, or write "No major setup issue yet.">`

### Steps to Reproduce

Based on the issue description, the bug can be reproduced with these steps:

1. Start a local tgstation test environment.
2. Load or enter a map area with Icebox snow, Lavaland ash, or asteroid turf.
3. Use a plumbing constructor to place chemistry or plumbing equipment on that turf.
4. Attempt to wrench the equipment down.
5. Expected behavior: the equipment should be allowed to secure onto the valid terrain.
6. Actual behavior: the equipment cannot be secured and shows a message similar to:
   `"needs to be on the floor to be secured!"`

### Reproduction Evidence

The issue discussion identifies the relevant failing check in `can_be_unfasten_wrench()`:

```dm
if(!(isfloorturf(loc) || isindestructiblefloor(loc)) && !anchored)
    to_chat(user, span_warning("[src] needs to be on the floor to be secured!"))
    return FAILED_UNFASTEN
```

The problem appears to happen because `isfloorturf(loc)` returns false for `/turf/open/misc` terrain such as snow, ash, and asteroid turf.

Branch link: `<PASTE YOUR BRANCH LINK HERE>`

## Solution Approach

### Understand

The bug happens because chemistry and plumbing equipment use the generic wrenching logic for securing objects. That logic only allows objects to be secured on `/turf/open/floor` or indestructible floor turfs. Since snow, ash, asteroid, and similar turfs are now categorized as `/turf/open/misc`, they fail the floor check even though gameplay expects this equipment to be secured there.

### Match

The existing code uses helper macros such as `isfloorturf()` and `isindestructiblefloor()` to decide whether an object is on a valid surface. I will look for similar construction, anchoring, or machine-placement logic to see how the codebase handles non-standard but valid turfs.

### Plan

1. Inspect `can_be_unfasten_wrench()` in `code/game/objects/objs.dm`.
2. Inspect turf helper definitions in `code/__DEFINES/is_helpers.dm`.
3. Search for existing helper procs or macros that classify snow, ash, asteroid, or misc turfs as valid construction surfaces.
4. Avoid broadly changing `isfloorturf()` unless that is clearly safe, because many systems may depend on its current meaning.
5. Prefer a targeted fix for plumbing or chemistry equipment if the broader helper change could affect unrelated behavior.
6. Add or update tests if the project has relevant tests for object anchoring, wrenching, plumbing equipment, or turf validation.
7. Manually verify the fix by repeating the reproduction steps on snow, ash, or asteroid turf.

### Review

Before opening a pull request, I will review tgstation's contribution guidelines and check that my change follows the project's style, naming, and testing expectations.

### Evaluate

I will verify the fix by:

* Confirming equipment can be wrenched down on Icebox snow or similar `/turf/open/misc` terrain.
* Confirming the original warning no longer appears for valid terrain.
* Confirming equipment still cannot be secured on invalid surfaces.
* Running any relevant tests or validation commands available in the project.
