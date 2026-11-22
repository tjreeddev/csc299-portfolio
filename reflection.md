# Reflection

Project developed further: **Albert Cat Climber** (https://github.com/tjreeddev/albert-cat-climber).

## 1. In the project you developed further, what did a fresh session get wrong or ask about that an earlier round had already settled? What was missing from the repository that would have prevented it?

I started the Round 5 session (2026-11-22) by asking it to "Start Round 4." The session had to check `spec.md` section 10 and commit `83587e7` to find that Round 4 (the large kitchen, R21–R26) was already done. It then asked whether the new work should be renamed Round 5 starting at R27. Numbering had been settled by Round 4, but no file stated it directly:

- `problem.md` still listed only R1–R8. Rounds 2–7 were added there only afterwards, in `6616675`.
- No handoff file existed, so nothing said "Round 4 is done; next is Round 5, starting at R27." The `handoff-round-*.md` files were reconstructed after development, also in `6616675`.

A handoff file written at the end of each round, plus `problem.md` kept up to date with each round's requirements, would have prevented the question.

## 2. In the round that changed an earlier requirement, what broke? Did the tests catch it, and does the decision record explain the change to someone who was not there?

Round 6 (`c542338`, hazards R39–R46) qualified R1/R2: for 18 updates after a hit, the knockback controls left/right and jump is ignored.

**What broke:** the R19 drawer-timing checks in `tests/test_drawer_route.py`. Hazards H5–H8 sit on the cabinets used for the drawer jumps, so a knockback changed what those simulations measured.

**Did the tests catch it?** Partly. The session switched the `jump_between` helper to a hazard-free level *before* the hazards existed (`tests/test_drawer_route.py:101`, `tests/conftest.py:73`). Only afterwards did it confirm that, with the original helper, one R19 test fails once hazards exist. The test would have caught the problem, but the change went in pre-emptively.

**Does the decision record explain it?** Only half of it. `docs/decisions/0006-hazards-knock-back-instead-of-instant-loss.md:15` explains the R1/R2 change, calling the loss of control "a documented qualification of R1/R2." No decision record mentions the hazard-free test level. That reasoning is only in the commit message, `handoff-round-6.md` and the `conftest.py` docstring. Also, every decision record was written retrospectively on 2026-11-22 (`docs/decisions/README.md`), not during Round 6. Someone who wasn't there would understand the knockback rule, but not why the R19 tests run on a different level.

## 3. Name one test that caught a real problem, or one test the assistant wrote that checked nothing useful. Quote it.

The R19 drawer-timing tests in `tests/test_drawer_route.py`, `test_r19_jumping_onto_each_drawer_depends_on_timing` (line 128) and `test_r19_jumping_from_each_drawer_to_the_next_stop_depends_on_timing` (line 137), end with:

```python
results = {jump_between(i - 1, i, n) for n in range(0, cycle_frames(d), SAMPLE_STEP)}
assert results == {True, False}, f"drawer at y={d.y}: timing made no difference ({results})"
```

Once the Round 6 hazards existed, one of these failed on the real level. A hazard on a route cabinet could knock Albert off during a timed drawer jump, so the R19 promise that "timing matters" no longer measured timing alone. The fix was to run these simulations on `hazard_free_level()`.

## 4. Choose one commit you would now write differently, and say how.

`c542338`, "Add Round 6 kitchen hazards with warning cycles and knockback (R39-R46)", is one commit of 20 files and +1,442 lines. It holds the spec (`spec.md` §12), 112 new tests, the implementation, and two test fixes approved mid-round. `handoff-round-6.md` notes "There is no separate earlier requirements commit for this round."

I would split it the way Round 1 was split (`e88c8bf` → `a1c7e71` → `fbc5fa6`):

1. R39–R46 added to `problem.md` and `spec.md`
2. the failing tests, with the "110 failed, 408 passed" result in the message
3. the implementation
4. the approved R44 test fixes on their own

The history would then show tests-first instead of only claiming it.

## 5. Choose one function in the project you developed further that you did not read while building it. Without an assistant open, read it and write what you think it does, in two or three sentences. Then ask an assistant, and say what you got right and wrong.

**Function:** `hit_area()` in `albert/hazards.py`

I correctly understood that `hit_area()` calculates the dangerous area of an active hazard. The assistant helped me understand that each hazard uses a different hitbox and that the drip's hitbox moves downward as the hazard cycle progresses. I understood the main purpose of the function, but not all of the timing and position calculations.
