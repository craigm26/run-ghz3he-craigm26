# run: ghz3_he — hardware-efficient GHZ-3 via Euler-angle CZ sandwich

**Problem:** `ghz3_he` (state_prep) — prepare the 3-qubit GHZ state under the
hardware-efficient brief: native gates restricted to `{rz, rx, cz}`, linear
coupling `[0-1, 1-2]`, `max_depth 12`, `max_two_qubit_gates 2`. The reference
pins these constraints host-side, so a bundle cannot loosen them.

**Design.** Started from the textbook GHZ recipe (`h q0; cx 0,1; cx 1,2`) and
transpiled it into the native set by hand, verifying every step numerically
against the harness's own `sim.py` rather than trusting a remembered identity:

1. `H = Rz(π/2) · Rx(π/2) · Rz(π/2)` up to a global phase of `−i` — confirmed
   by computing both matrices and checking their ratio is a scalar multiple
   of the identity (global phase is invisible to fidelity, so this is exact
   for our purposes).
2. `CX(c, t) = H(t) · CZ(c, t) · H(t)` — the standard CZ-sandwich identity,
   applied with `t` as the actual target qubit each time.

Substituting (1) into (2) gives 7 native ops per CX (3 + 1 + 3) and 3 for the
leading `H(q0)`: **17 ops, 2 CZ gates, depth 11** (one under the 12 budget).
The two `H(q2)`-branch rotations schedule in parallel with the `q0`/`q1`
branch (they don't share a qubit until the second CZ), which is what keeps
depth at 11 instead of a naive serial 17.

**Verified result** (`judge_verify.py`, exit 0):

| check | value |
|---|---|
| fidelity | 1.0 (recomputed by the judge from the circuit alone) |
| structure | depth 11 ≤ 12, two_qubit_gates 2 ≤ 2, all native |
| performance | 1.0 ≥ threshold 0.99, ≥ classical baseline 0.5 |
| two-qubit cost | cost-adjusted fidelity 0.9 (= 1.0 − 2×0.05) — still clears baseline |

**Provenance:** design + write-up by Claude (Sonnet 5), directed by the
repo's own operator, as an independent proof this problem's discovery loop
works end-to-end for a first-time contributor — no `--org` flag, minted
under a personal GitHub account, discovered by the `quantum-harness-run`
topic like any other stranger's run.

Re-verify locally:

```
git clone https://github.com/craigm26/run-ghz3he-craigm26
cd run-ghz3he-craigm26
python3 bench/quantum-judge/judge_verify.py bench/quantum-judge/quantum-proof-ghz3he-craigm26.json
```
