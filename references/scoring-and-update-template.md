# Scoring and Update Template

Use this file when the task needs an explicit frontier comparison or a stable tree update format.

## Default 0..3 Scale

Apply these coarse values unless the task needs a custom scale.

### Success likelihood `p`

- `3`: strong evidence this route can reach the goal
- `2`: plausible but still uncertain
- `1`: weak support or heavy assumptions
- `0`: effectively disproven

### Information gain `s`

- `3`: one step will eliminate major uncertainty
- `2`: useful evidence but not decisive
- `1`: minor learning only
- `0`: mostly administrative or redundant

### Reversibility and safety `r`

- `3`: easy to undo, low blast radius
- `2`: manageable risk
- `1`: annoying or costly to reverse
- `0`: destructive, risky, or hard to undo

### Remaining distance `d`

- `3`: many steps still needed
- `2`: medium path remaining
- `1`: short path remaining
- `0`: this step likely finishes the task

### Immediate cost `c`

- `3`: expensive in time, compute, or coordination
- `2`: moderate cost
- `1`: cheap
- `0`: nearly free

## Default Score

Use:

`score = 4p + 2s + r - 2d - c`

This weighting intentionally prefers:

- branches that are likely to work
- steps that collapse uncertainty quickly
- short cheap paths
- safer reversible actions

Do not treat the score as proof.
Use it as a disciplined ranking aid.

## Frontier Table Template

```text
Frontier Table
ID   Goal                              p  s  r  d  c  Score  Status   Next
A2   Profile import path               3  3  3  1  1  15     active   run import-time profiler
B1   Inspect config loading            2  2  3  2  1  10     active   trace config reads
C1   Check external IO waits           1  2  2  2  2   4     active   disable network calls locally
```

Always select the highest-scoring credible leaf.
If the table hides an important nuance, explain it in one line below the table.

## Tree Update Template

```text
Exploration Update
Root objective: <goal>

Current tree
[0] Root: <goal> [active]
|- [A] <branch> [status]
|  \- [A2] <leaf> [active] p=3 s=3 r=3 d=1 c=1 score=15 <- selected
\- [B] <branch> [active] p=2 s=2 r=3 d=2 c=1 score=10

Frontier: A2, B
Selected leaf: A2
Why selected: <one-line comparison against the main alternatives>
Next action: <concrete next step>
Last result: <what just happened>
If this fails: <next best frontier leaf or re-expansion plan>
```

## Failure and Exhaustion Template

Use this when a leaf fails or the frontier is nearly exhausted:

```text
Branch result
Failed leaf: <id>
Failure reason: <evidence that falsified or blocked it>
Effect on frontier: <what was removed or downgraded>
Next selected leaf: <id or none>
Higher-level re-expansion: <what new branch search was attempted>
Conclusion: <either the next route or that no credible branch remains under current constraints>
```

## When to Stop

Stop searching when one of these is true:

- a branch is verified and the task is complete
- every credible branch is blocked by an external dependency
- the frontier is exhausted and higher-level re-expansion did not produce a credible new branch

When stopping without success, say that the task is unresolved under current evidence and constraints.
