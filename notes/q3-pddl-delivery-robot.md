# Q3 — PDDL: Coffee-Delivery Robot

> Topics: PDDL, STRIPS semantics, plan validation, admissible heuristics · Handnote pp. 12–15

## Question

*"A delivery robot is in the kitchen holding nothing. There is a coffee cup on the kitchen table. The goal is for the robot to be in the office holding the coffee cup."*

1. Write the domain file (types, predicates, and STRIPS actions `move(?from ?to)`, `pick-up(?obj ?loc)`, `put-down(?obj ?loc)` with explicit `:precondition` and `:effect`) and the problem file (`:objects`, `:init`, `:goal`).
2. Give a valid plan and prove it correct by showing the state after each action.
3. Propose an admissible heuristic and justify why it never overestimates.

## Background

PDDL separates a planning task into two files: the **domain** (the general rules: types, predicates and action schemas) and the **problem** (one concrete situation: objects, initial state and goal). Grounding both gives a STRIPS task `Π = ⟨P, A, c, I, G⟩`. Every atom not listed in `:init` is false (closed-world assumption).

**STRIPS semantics:**

```text
a is applicable in s  ⇔  pre(a) ⊆ s
s⟦a⟧ = (s \ del(a)) ∪ add(a)
s is a goal state     ⇔  G ⊆ s
```

---

## (i) Domain file

```lisp
(define (domain delivery)
  (:requirements :strips :typing)
  (:types location item)
  (:predicates
    (robot-at ?l - location)
    (obj-at ?o - item ?l - location)
    (holding ?o - item)
    (hand-empty))

  (:action move
    :parameters (?from - location ?to - location)
    :precondition (robot-at ?from)
    :effect (and (robot-at ?to)
                 (not (robot-at ?from))))

  (:action pick-up
    :parameters (?obj - item ?loc - location)
    :precondition (and (robot-at ?loc) (obj-at ?obj ?loc) (hand-empty))
    :effect (and (holding ?obj)
                 (not (obj-at ?obj ?loc))
                 (not (hand-empty))))

  (:action put-down
    :parameters (?obj - item ?loc - location)
    :precondition (and (robot-at ?loc) (holding ?obj))
    :effect (and (obj-at ?obj ?loc)
                 (hand-empty)
                 (not (holding ?obj))))
)
```

## (i) Problem file

```lisp
(define (problem bring-coffee)
  (:domain delivery)
  (:objects kitchen office - location
            cup - item)
  (:init (robot-at kitchen)
         (obj-at cup kitchen)        ; the cup is on the kitchen table
         (hand-empty))
  (:goal (and (robot-at office)
              (holding cup)))
)
```

## (ii) Plan and proof of correctness

**Plan:** `π = ⟨pick-up(cup, kitchen), move(kitchen, office)⟩`

| Step | Action | pre ⊆ s? | del | add | Resulting state |
|---|---|---|---|---|---|
| s0 | — | — | — | — | {robot-at kitchen, obj-at cup kitchen, hand-empty} |
| 1 | pick-up(cup, kitchen) | robot-at kitchen ✓, obj-at cup kitchen ✓, hand-empty ✓ | obj-at cup kitchen, hand-empty | holding cup | s1 = {robot-at kitchen, holding cup} |
| 2 | move(kitchen, office) | robot-at kitchen ✓ | robot-at kitchen | robot-at office | s2 = {robot-at office, holding cup} |

**Goal check:** `G = {robot-at office, holding cup} ⊆ s2` ✓. So π is a valid plan, with cost 2.

- **Why 2 is optimal:** both goal atoms are false in s0. `holding cup` can only be added by pick-up, and `robot-at office` only by move. These are different actions, so every plan needs at least 2 actions.
- **Why the order matters:** if the robot moves first, it is in the office while the cup is in the kitchen, so pick-up is no longer applicable.

## (iii) An admissible heuristic: goal count

```text
h(s) = |G \ s|        (the number of goal atoms not yet true in s)
```

**Why it never overestimates here:** look at the add lists. `move` adds only `robot-at(?to)`, at most one goal atom. `pick-up` adds only `holding(?obj)`, at most one goal atom. `put-down` adds no goal atom. So every action makes at most one missing goal atom true, and with unit costs, reaching the goal needs at least `|G \ s|` actions. Therefore `h(s) ≤ h*(s)` for every state s. It is also goal-aware: h = 0 exactly when G ⊆ s.

| State | h(s) | h*(s) |
|---|---:|---:|
| s0 | 2 | 2 |
| s1 | 1 | 1 |
| s2 | 0 | 0 |

**Caution:** goal count is **not** admissible in general. It works here only because no action adds two goal atoms at once and every action costs 1.

## Answer

- **(i)** Domain and problem files as above: types `location` and `item`; predicates `robot-at`, `obj-at`, `holding`, `hand-empty`; actions `move`, `pick-up`, `put-down`. Init {robot-at kitchen, obj-at cup kitchen, hand-empty}; goal {robot-at office, holding cup}.
- **(ii)** π = ⟨pick-up(cup, kitchen), move(kitchen, office)⟩ reaches s2 = {robot-at office, holding cup} ⊇ G, so it is valid, and it is optimal at cost 2.
- **(iii)** `h(s) = |G \ s|` is admissible (and goal-aware) for this task because every action adds at most one goal atom and costs 1.
