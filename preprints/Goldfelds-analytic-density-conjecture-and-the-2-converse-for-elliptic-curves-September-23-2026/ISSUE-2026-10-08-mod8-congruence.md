# Reported issue: the mod-8 congruence in the proof of Lemma `pw:split-witness` fails numerically

**Manuscript:** *Goldfeld's analytic density conjecture and the 2-converse for elliptic curves* (September 23, 2026)
**Location:** `build/sections/08-rational-converse.tex`, lines 541–600, equation `pw:split-mod8`, inside the proof of Lemma `pw:split-witness` (lines 470–523).
**Reported:** 2026-10-08. This repository has issues disabled, so the report is filed as a pull request adding this note; no manuscript text is changed.

## The claim

Under the lemma's hypotheses (`E[2]` rational, odd functional sign, `k < 0` a unit of a fixed even symbol square at `2N` and prime to the fixed support and to the ramification support of `E[8]`, `K = Q(√k)`, `S_0 ⊇ {2N, primes of k, class-group generators}`), and under the branch assumption made at lines 511–523 ("equality never occurs", giving `v(a(D)) ≥ ω(D)+1` for every nonempty allowed all-K-split `D`), the proof asserts

> `a_p(E^{(k)}) ≡ p + 1 (mod 8)` for every allowed good `p` split in `K`  (eq. `pw:split-mod8`, line 541),

and closes (lines 598–600): "No individual total filter condition on `p` was needed: its matched prime `p′` makes `pp′` lie on the filter, and the square-index terms on the right come from 1."

## Why it cannot hold as stated

For full rational 2-torsion `y² = (x−e₁)(x−e₂)(x−e₃)`, the congruence `a_p ≡ p+1 (mod 8)` is equivalent to `8 | #E^{(k)}(F_p)`, i.e. to some 2-torsion point being 2-divisible over `F_p`, i.e. to two of the Legendre symbols `((e_i−e_j)/p)` at a common vertex being `+1`. The hypotheses require `k` prime to the primes dividing the differences `e_i−e_j` (they divide `2N`), so `Q(√k)` is linearly disjoint from the fields `Q(√(e_i−e_j))`, and by Chebotarev the condition "`p` split in `K`" is independent of the three symbols. Unless two differences at a common vertex are already rational squares (rational 4-torsion), a positive proportion of K-split primes has no vertex with two `+1` symbols, and the congruence fails for all of them.

For the congruent-number curves `y² = x³ − n²x` (CM by `Z[i]`) this is immediate: `a_p = 0` for every `p ≡ 3 (mod 4)`, so the congruence forces `p ≡ 7 (mod 8)`, yet half of the K-split primes `p ≡ 3 (mod 4)` are `≡ 3 (mod 8)`.

## Reproduction (PARI/GP, self-contained)

```gp
\\ E: y^2 = x^3 - 25x (n = 5, conductor 800, rank 1, root number -1, E[2] rational)
\\ k = -31: square at 2N, prime to 2*5, L(E^{(k)},1) != 0;  K = Q(sqrt(-31)), S_0 = {2, 5, 31}
E  = ellinit([0,0,0,-25,0]);  k = -31;  Ek = ellinit(elltwist(E, k));  S0 = [2,5,31];
tested = 0; fails = 0; wit = [];
forprime(p = 3, 3000,
  if (p == 31, next);
  if (kronecker(k, p) != 1, next);     \\ p split in K
  if (vecsearch(S0, p), next);         \\ p outside S_0
  a = ellap(Ek, p);  tested++;
  if ((a - (p+1)) % 8 != 0, fails++; if (#wit < 8, wit = concat(wit, [[p, a]]))));
print(ellrootno(E), "  ", lfun(lfuncreate(Ek), 1));
print("tested = ", tested, "  fails = ", fails);  print(wit);
```

Output (PARI 2.17 via cypari2):

```
-1  3.36973969456359
tested = 207  fails = 113
[[19, 0], [59, 0], [67, 0], [97, -18], [107, 0], [113, 14], [131, 0], [157, -22]]
```

E.g. `p = 19`: `a_p(E^{(k)}) = 0`, `p+1 = 20 ≡ 4 (mod 8)`. Across 23 `(E, k)` pairs (12 curves of the shape `y² = x(x−a)(x−b)`, odd sign, `k` the minimum-depth admissible representative with `|k| ≤ 200`), 2325 of 4807 K-split primes `p ≤ 3000` fail, every pair at roughly half of its primes; every verdict was cross-checked against the independent halving criterion with 0 mismatches.

## What this does and does not show

- If "allowed" is read more strongly — `p` a square at every place of `S_0` — the congruence holds (251/251 in our sample) but trivially: every `e_i − e_j` is then a square mod `p`, all three 2-torsion points halve, and `8 | #E^{(k)}(F_p)` without any trace computation. The proof's closing sentence rules this reading out.
- The congruence is derived only under the branch assumption "equality never occurs" (lines 511–523). For 7 of the 23 pairs we found an equality witness `D` (e.g. `n = 5, k = −31`: `D = 329`, `√D·L(E^{(kD)},1)/L(E^{(k)},1) = 16` exactly), so for those pairs the other branch applies and the failure is vacuous. For 4 pairs (`n = 6, k = −47`; `n = 7, k = −31`; `n = 13, k = −79`; `n = 31, k = −79`) no equality witness was found among admissible `D ≤ 600` with `ω(D) ≤ 2`, and the congruence still fails; the search is finite, so this does not exclude a larger witness. Either the branch is always empty (then the lemma's conclusion may survive, but the written argument for that branch and the "no filter condition" remark do not), or the branch is nonempty and the proof of the lemma has a gap.
- "`K` chosen nonexceptional" is not defined at the cited lines, so that hypothesis could not be checked.
- This is a report about one step of one proof. It is not a counterexample to Theorem 1.1 of this manuscript or to the results that cite it.
