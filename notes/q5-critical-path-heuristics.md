# Q5 — Critical-Path Heuristics: hᵐ⁺¹ Dominates hᵐ

> Topics: h^max, the h^m family, regression, dominance · Handnote pp. 21–25

## Question

*"h^{m+1} dominates h^m" — justify this statement for the critical-path heuristic family.* Consider the STRIPS task:

```text
Facts:   D = at-dock, C1 = have-crate1, C2 = have-crate2, L = loaded, W = at-warehouse
Initial: {D}          Goal: {W, L}

Actions (cost 1):
  fetch-crate1:  pre {D}         add {C1}
  fetch-crate2:  pre {D}         add {C2}
  load:          pre {C1, C2}    add {L}
  drive:         pre {D}         add {W}    del {D}
```

1. Compute h¹(s0) (= h^max), showing the fixpoint computation fact by fact.
2. Compute h²(s0) for the goal pair {L, W} with the h^m recursion, and compare with h¹(s0). Does h² recover the optimal plan cost?
3. Explain why h^{m+1}(s) ≥ h^m(s) always holds.

**The key conflict:** both fetches need D, and drive destroys D. So the real plan must do both fetches before driving: `fetch1, fetch2, load, drive`, giving **h\* = 4**.

## The h^m equations

```text
h^m(B) = 0                                         if B ⊆ s
h^m(B) = min_{a : regr(B,a) defined} [ c(a) + h^m(regr(B, a)) ]      if |B| ≤ m
h^m(B) = max_{B' ⊆ B, |B'| = m} h^m(B')            if |B| > m

regr(B, a) = (B \ add(a)) ∪ pre(a),  defined only if add(a) ∩ B ≠ ∅ and del(a) ∩ B = ∅
```

`h^m(s) = h^m(G)`. For m = 1 this is exactly h^max.

---

## (i) h¹(s0): fixpoint iteration

| Fact | Iter 0 | Iter 1 | Iter 2 | Iter 3 | How the value is obtained |
|---|---:|---:|---:|---:|---|
| D | 0 | 0 | 0 | 0 | True in s0 |
| C1 | ∞ | 1 | 1 | 1 | fetch-crate1: 1 + h¹(D) = 1 |
| C2 | ∞ | 1 | 1 | 1 | fetch-crate2: 1 + h¹(D) = 1 |
| W | ∞ | 1 | 1 | 1 | drive: 1 + h¹(D) = 1 |
| L | ∞ | ∞ | 2 | 2 | load: 1 + max(h¹(C1), h¹(C2)) = 1 + 1 = 2 |

Iteration 3 equals iteration 2, so the fixpoint is reached.

```text
h¹(s0) = max(h¹(W), h¹(L)) = max(1, 2) = 2
```

**Why h¹ is so low:** at `load`, max(C1, C2) = 1 pretends that one fetch provides both crates; at the goal, max(W, L) pretends the drive costs nothing on top of the loading chain.

## (ii) h²(s0): all needed sets of size ≤ 2

| Set B | Possible last action → regression | h² |
|---|---|---:|
| {D} | true in s0 | 0 |
| {C1}, {C2}, {W} | fetch1 / fetch2 / drive → {D} | 1 |
| {C1, D} | fetch1 → {D} (fetch does not delete D) | 1 |
| {C2, D} | fetch2 → {D} | 1 |
| {W, D} | drive adds W but deletes D, so it cannot be last; no action adds D | ∞ |
| {C1, C2} | fetch1 → {C2, D} = 1, so 2; fetch2 → {C1, D} = 1, so 2 | 2 |
| {W, C1} | drive → {C1, D} = 1, so 2; fetch1 → {W, D} = ∞ | 2 |
| {W, C2} | drive → {C2, D} = 1, so 2 | 2 |
| {L, D} | load → {C1, C2, D} (size 3) = max({C1,C2} = 2, {C1,D} = 1, {C2,D} = 1) = 2, so 1 + 2 = 3 | 3 |
| {L} | load → {C1, C2} = 2, so 3 | 3 |
| {L, C1}, {L, C2} | load → {C1, C2} = 2, so 3 (fetch last → {L, D} = 3, so 4) | 3 |

**Goal {L, W}:**

| Last action | Regression | Value | Total |
|---|---|---|---:|
| load | {W, C1, C2} (size 3) | max({W,C1} = 2, {W,C2} = 2, {C1,C2} = 2) = 2 | 1 + 2 = **3** |
| drive | {L, D} | 3 | 1 + 3 = 4 |

```text
h²(s0) = min(3, 4) = 3          h¹ = 2  ≤  h² = 3  ≤  h* = 4
```

**Does h² recover h\*?** No. h² sees that the two crates need two separate fetches ({C1, C2} = 2), but {W, C1, C2} is a triple; h² splits it into pairs and never sees that both fetches must happen before the drive.

### h³(s0) = 4: triples recover the optimum

```text
{C1, C2, D}: fetch1 → {C2, D} = 1, so 2
{W, C1, C2}: drive  → {C1, C2, D} = 2, so 3
             fetch1 → {W, C2, D} = ∞ (W and D can never hold together)
{L, D}:      load   → {C1, C2, D} = 2, so 3
Goal {L, W}: load   → {W, C1, C2} = 3, so 4
             drive  → {L, D} = 3, so 4
h³(s0) = 4 = h*  ✓
```

## (iii) Why h^{m+1}(s) ≥ h^m(s) always holds

**Lemma (monotonicity):** for every m, if B' ⊆ B then h^m(B') ≤ h^m(B). Reaching all of B also reaches all of B', so a larger set can never be cheaper. (Formally, by induction over the fixpoint iterations: any regression that is legal for B gives a superset of a legal regression for B'.)

**Proof of dominance**, by induction over the fixpoint iterations (both start from the same 0 / ∞ values):

1. For sets of size ≤ m, h^m and h^{m+1} use the same regression rule, `min_a [c(a) + h(regr(B, a))]`.
2. The only difference is how a regressed set R with |R| > m is valued. h^m uses `max_{R' ⊆ R, |R'| = m} h^m(R')`. h^{m+1} uses R's own value if |R| = m + 1, or the max over its (m+1)-subsets if |R| > m + 1. Either way, every m-subset R' lies inside a set that h^{m+1} evaluates, so by the lemma `h^{m+1}(R) ≥ max_{R'} h^{m+1}(R')`, and by the induction hypothesis this is `≥ max_{R'} h^m(R')`.
3. So in every iteration, each value h^{m+1} computes is ≥ the corresponding h^m value, and this carries over to the fixpoint: **h^{m+1}(s) ≥ h^m(s)** for every s.

In words: h^m assumes the most expensive m-subset alone determines the cost. h^{m+1} can see interactions among m + 1 facts, which can only add cost. Since every h^m is admissible:

```text
h¹ = h^max ≤ h² ≤ … ≤ h^m ≤ h*,   and h^m = h* once m ≥ the number of facts
In this task:  2 ≤ 3 ≤ 4 = 4
```

(The inequality is ≥, not >: consecutive h^m values are often equal.)

## Answer

- **(i)** D = 0; C1 = C2 = W = 1 (one action from D); L = 1 + max(1, 1) = 2; the fixpoint is reached at iteration 2. **h¹(s0) = max(W, L) = 2.** (h* = 4: fetch1, fetch2, load, drive.)
- **(ii)** h²: {C1,C2} = 2, {W,C1} = {W,C2} = 2, {W,D} = ∞ (drive deletes D), {L,D} = 3. Goal {L, W}: load → {W,C1,C2} = 2, so 3; drive → {L,D} = 3, so 4. **h²(s0) = 3.** So h¹ = 2 ≤ h² = 3 < h* = 4: h² does not recover h*; triples are needed (h³ = 4).
- **(iii)** h^m and h^{m+1} share the regression rule for sets of size ≤ m; for larger sets h^{m+1} evaluates (m+1)-subsets, which contain the m-subsets h^m uses. By monotonicity and induction over the fixpoint iterations, h^{m+1}(s) ≥ h^m(s) for all s, giving h¹ ≤ h² ≤ … ≤ h*.
