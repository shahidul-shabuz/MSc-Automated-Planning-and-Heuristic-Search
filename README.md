# Automated Planning and Heuristic Search — MSc Worked Solutions

Step-by-step solutions to the problem set for **CSE 6109: Automated Planning and Heuristic Search** (M.Sc. in CSE, DUET): state-space search, A*, PDDL, and the four families of planning heuristics (delete relaxation, critical path, abstraction and landmarks).

| | |
|---|---|
| **Course** | CSE 6109: Automated Planning and Heuristic Search — M.Sc. in CSE, Dhaka University of Engineering & Technology (DUET) |
| **Semester** | 2nd semester |
| **Course teacher** | [Dr. Sabah Binte Noor](https://profile.duet.ac.bd/u/sabah), Associate Professor, Department of Computer Science and Engineering, DUET |
| **Grade** | Result pending (expected 4 November 2026) |

## Contents

| Q | Topic | Key result |
|---|---|---|
| [Q1](notes/q1-uninformed-vs-informed-search.md) | Uninformed vs. informed search: BFS and DFS on a grid | Both find a length-4 path (8 vs. 6 expansions); BFS is optimal for unit costs, DFS is not |
| [Q2](notes/q2-astar-and-greedy-best-first.md) | Consistent ⇒ admissible; A* vs. greedy best-first | A* finds S→A→C→G, cost 8; h verified admissible and consistent on all 12 directed edges |
| [Q3](notes/q3-pddl-delivery-robot.md) | PDDL domain and problem files; plan validation | 2-step optimal plan, proved state by state; goal-count heuristic shown admissible |
| [Q4](notes/q4-abstraction-heuristics.md) | Identity vs. perfect abstraction; orthogonal abstractions | h* = 5; non-orthogonal sum 3 + 3 = 6 is inadmissible, orthogonal sum 3 + 1 = 4 is admissible |
| [Q5](notes/q5-critical-path-heuristics.md) | Critical-path heuristics: h^{m+1} dominates h^m | h¹ = 2 ≤ h² = 3 ≤ h³ = 4 = h*, with a proof of dominance |
| [Q6](notes/q6-delete-relaxation.md) | Goal-aware vs. safe; relaxed planning graph; h^max, h^add | h^max = 3 ≤ h⁺ = 5 ≤ h* = 7; h^add = 6 double-counts dr(B,C) |
| [Q7](notes/q7-landmark-heuristics.md) | Fact landmarks, disjunctive action landmarks, LM-cut | Fact landmarks give only 2; five disjoint action landmarks give h^LM-cut = 5 = h* |
| 📘 | [**Complete handnote (PDF)**](handnote/CSE6109-handnote.pdf) | All seven questions with diagrams, including the transition-system and landmark-cut figures |

Each solution states the question, the definitions it relies on, the full working, and an **Answer** section.

## Highlights

- **Heuristic hierarchy in numbers.** On one logistics task: h^OA = 2 ≤ h^max = 3 ≤ h⁺ = 5 ≤ h* = 7, while the inadmissible h^add = 6 (Q6).
- **Dominance of h^m, computed and proved.** h¹, h² and h³ are calculated set by set on the same task, showing exactly which conflict (both fetches must precede the drive) each level can see (Q5).
- **When adding heuristics breaks admissibility.** Abstractions that share the move actions sum to 6 > h* = 5; making them orthogonal, or cost-partitioning the moves, restores admissibility (Q4). The same overcounting is shown for overlapping landmarks (Q7).
- **Landmarks beyond single facts.** Fact landmarks give h = 2 on a two-branch problem; disjunctive action landmarks, extracted both by delete relaxation and by LM-cut, give the exact value 5 (Q7).

## Course project: LLM-guided evolutionary search for Blocksworld

For the course assignment *Build a Simple LLM Planning Pipeline*, I built an evolutionary planner for Blocksworld in which candidate plans are scored by a simulator and bred by crossover and mutation. The LLM-guided variant differs from the baseline in exactly one line: each generation, the two best plans are sent to an LLM, which proposes new candidate plans. The fitness function, `−(10 × missing goal facts + 1 × illegal actions + 0.1 × plan length)`, ranks any plan that reaches the goal above every plan that does not, then prefers shorter plans. The LLM never decides what is correct: every plan it proposes is executed and scored by the simulator, and its prompt includes the simulator's feedback (which step was illegal, which precondition was missing, which goal facts remain).

| Problem | Shortest plan | Baseline (no LLM) | LLM-guided |
|---|---:|---|---|
| P1: three blocks, build A on B on C | 4 | 2/3 runs solved | 2/3 runs solved |
| P2: Sussman anomaly | 6 | 0/3 | 1/3 (optimal, 6 actions) |
| P3: four-block tower inverted | 8 | 0/3 | 1/3 (optimal, 8 actions) |
| Candidate plans tested per run | | 1,640 | 140 |

The baseline never solved the two harder problems: on P2 its best fitness stalls at −10.2 from generation 10 onward, trapped by the Sussman anomaly, where making one goal true buries the block needed for the other. The LLM-guided search solved both, while testing about a tenth as many candidate plans. The report is candid about the limits: three runs per cell is a small sample; 50 of 168 API requests (30%) failed on the free-tier quota, so those generations silently fell back to the baseline; and the LLM's success on the famous Sussman anomaly may reflect recall rather than reasoning, which is why the unnamed P3 was included.

## Paper presentation

As part of the course, I presented:

> A. Murray, D. Dervovic, A. Pozanco, M. Cashmore, "GenePlan: Evolving Better Generalized PDDL Plans Using Large Language Models," in *Proceedings of the Thirty-Sixth International Conference on Automated Planning and Scheduling (ICAPS 2026)*.

GenePlan casts generalized planning as an optimization problem: an LLM-assisted evolutionary loop evolves interpretable Python planners that minimize plan length across many problem instances of a PDDL domain. Across eight benchmark domains it reaches an average SAT score of 0.91, close to state-of-the-art planners (0.93) and well above chain-of-thought prompting (0.64), while solving new instances in about half a second each.

## A note on preparation

These solutions were prepared from my course notes with the help of an AI assistant (Claude) and reviewed by me; all numerical results were checked independently. Lecture slides and other course materials are not included, as they belong to their authors.

## Author

**Md Shahidul Islam Shabuz** — Assistant Professor, Dept. of CSE, RSTU; M.Sc. candidate, DUET
[Website](https://shahidul-shabuz.github.io/) · [Google Scholar](https://scholar.google.com/citations?user=VDtX90sAAAAJ&hl=en) · [ORCID](https://orcid.org/0009-0008-8271-888X)

## License

[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)
