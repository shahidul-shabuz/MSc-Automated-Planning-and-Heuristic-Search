# Q6 — Goal-Aware vs. Safe Heuristics; Delete Relaxation (h^max, h^add)

> Topics: heuristic properties, relaxed planning graph, h^max, h^add, h⁺, only-adds relaxation · Handnote pp. 26–29

## Question

*What is the relationship between goal-aware and safe heuristics?* A logistics task:

```text
Locations (left to right): A — B — C — D
Initial I:  t(B), p(A)            (truck at B, package at A)
Goal G:     t(C), p(D)            (truck at C, package at D)

Actions (cost 1):
  dr(X,Y)  drive the truck between adjacent locations
  lo(X)    load the package (truck and package both at X)
  ul(X)    unload the package (truck at X, package in the truck)
```

1. Compute the heuristic value of the initial state using delete relaxation (h^add or h^max over the relaxed planning graph; state which and why). Show the relaxed planning graph layer by layer.
2. Is an "only-adds" relaxation, which drops only the delete lists but keeps all preconditions, more informative than the standard delete relaxation? Justify your answer.

## Heuristic properties

- **Safe:** `h(s) = ∞ ⇒ h*(s) = ∞`. It never declares a solvable state a dead end.
- **Goal-aware:** `h(s) = 0` for every goal state s.
- **Admissible:** `h(s) ≤ h*(s)`. **Consistent:** `h(s) ≤ c(a) + h(s⟦a⟧)`.

### Relationship between goal-aware and safe

They are **independent**: neither implies the other.

- `h(s) = 1` for all s is **safe** (never ∞) but **not goal-aware** (h = 1 on goal states).
- `h(s) = 0` on goal states and ∞ elsewhere is **goal-aware** but **not safe** (solvable non-goal states get ∞).

Links to other properties:

- **Admissible ⇒ safe and goal-aware.** For a goal state, h(s) ≤ h*(s) = 0; and if h(s) = ∞ then h*(s) ≥ ∞.
- **Goal-aware + consistent ⇒ admissible.**
- **Goal-aware + safe does not imply admissible.** Example: h = 0 on goals and 100 elsewhere.

---

## (i) Relaxed planning graph

In the delete relaxation, facts are never removed, so each fact layer contains all earlier facts. Below, p(T) means "package in the truck".

| Layer | New facts | Actions applicable for the first time |
|---|---|---|
| F0 | t(B), p(A) | — |
| A0 | | dr(B,A), dr(B,C) |
| F1 | + t(A), t(C) | |
| A1 | | lo(A), dr(A,B), dr(C,B), dr(C,D) |
| F2 | + p(T), t(D) | |
| A2 | | ul(A), ul(B), ul(C), ul(D), dr(D,C) |
| F3 | + p(B), p(C), **p(D)** | G = {t(C), p(D)} ⊆ F3 → stop |

### Fact costs: h^max and h^add

```text
h^max(p) = min_{a achieves p} [ c(a) + max_{q ∈ pre(a)} h(q) ]
h^add(p) = min_{a achieves p} [ c(a) + Σ_{q ∈ pre(a)} h(q) ]
```

| Fact | Best achiever | Preconditions | h^max | h^add |
|---|---|---|---:|---:|
| t(B), p(A) | — (initial) | — | 0 | 0 |
| t(A) | dr(B,A) | t(B) | 1 | 1 |
| t(C) | dr(B,C) | t(B) | 1 | 1 |
| t(D) | dr(C,D) | t(C) | 2 | 2 |
| p(T) | lo(A) | t(A), p(A) | 1 + max(1, 0) = 2 | 1 + (1 + 0) = 2 |
| p(D) | ul(D) | t(D), p(T) | 1 + max(2, 2) = 3 | 1 + (2 + 2) = 5 |
| **Goal {t(C), p(D)}** | | | **max(1, 3) = 3** | **1 + 5 = 6** |

**Comparison with exact values:**

```text
Optimal relaxed plan:  dr(B,A), lo(A), dr(B,C), dr(C,D), ul(D)                    h⁺(I) = 5
Real optimal plan:     dr(B,A), lo(A), dr(A,B), dr(B,C), dr(C,D), ul(D), dr(D,C)  h*(I) = 7

h^max = 3  ≤  h⁺ = 5  ≤  h* = 7        while   h^add = 6 > h⁺ = 5
```

**Which one to use, and why: h^max.** It is admissible (h^max ≤ h⁺ ≤ h*), so A* remains optimal. h^add is more informative but **not admissible**, because it adds up the costs of subgoals that share actions. Here `dr(B,C)` is counted twice: once in t(C) = 1, and again inside t(D) = 2, which p(D) needs. That is why h^add = 6 > h⁺ = 5. It happens to stay below h* = 7 here, but in general h^add can overestimate h*, so it suits greedy search rather than optimal A*.

## (ii) Only-adds vs. delete relaxation

**No, it is not more informative.** As defined in the question, the relaxation "drops only the delete lists but keeps all preconditions". That is exactly the standard delete relaxation, so it gives the same value h⁺(I) = 5 and rules out exactly the same plans; there is nothing extra it could rule out.

The usual **only-adds relaxation** (from the lecture) drops the preconditions as well as the deletes: only the add lists matter. That makes it **less** informative:

```text
h^OA(I) = 2   via ⟨dr(B,C), ul(D)⟩
```

One action adds t(C) and one adds p(D), and no single action adds both. This "plan" unloads at D even though the truck is not at D and the package is not in the truck.

The delete relaxation still enforces preconditions, so it correctly rules out such impossible plans; only-adds cannot. Therefore:

```text
h^OA = 2  ≤  h⁺ = 5  ≤  h* = 7
```

The delete relaxation dominates only-adds, not the other way round: the more a relaxation drops, the lower its value.

## Answer

- **Goal-aware vs. safe:** independent. h ≡ 1 is safe but not goal-aware; h = 0 on goals and ∞ elsewhere is goal-aware but not safe. Admissible implies both; goal-aware + consistent implies admissible.
- **(i)** RPG: F0 {t(B), p(A)} → A0 dr(B,A), dr(B,C) → F1 + t(A), t(C) → A1 lo(A), dr(C,D), … → F2 + p(T), t(D) → A2 ul(D), … → F3 + p(D), and G ⊆ F3. Costs t(A) = t(C) = 1, t(D) = 2, p(T) = 2, p(D) = 3 (max) or 5 (add). **h^max(I) = 3, h^add(I) = 6.** h^max is used because it is admissible (3 ≤ h⁺ = 5 ≤ h* = 7); h^add double-counts dr(B,C) and is inadmissible in general.
- **(ii)** No. As described (deletes dropped, preconditions kept), it is identical to the delete relaxation and rules out nothing extra. The usual only-adds relaxation also drops preconditions and gives h^OA(I) = 2 via ⟨dr(B,C), ul(D)⟩, allowing an unload at D without the package; the delete relaxation rules this out, so it dominates only-adds.
