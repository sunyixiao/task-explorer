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
B    Build trusted evaluation          3  2  3  1  1  14     ready    fix eval suite and metrics
C    Find effective route              3  3  2  1  1  15     ready    expand optimization candidates
D1   Define result-log template        2  2  3  1  1  10     ready    create comparison format
D2   Produce before/after comparison   1  1  3  2  1   5     waiting  wait for B/C outputs
```

Always select the highest-scoring credible leaf.
If the table hides an important nuance, explain it in one line below the table.
When using multiple workers, convert the top compatible leaves into the current work set.
Explain whether each chosen item belongs to an `AND` decomposition or is a speculative `OR` alternative.

## Tree Update Template

```text
Exploration Update
Root objective: <goal>

Current tree
[A] <goal> [ready]
\- [D] <dependent parent goal> [ready]
   \- AND
      |- [B] <prerequisite goal> [ready] p=3 s=3 r=3 d=1 c=1 score=15
      |- [C] <prerequisite goal> [ready] p=2 s=2 r=3 d=2 c=1 score=10
      \- [D1] <startable conclusion child> [ready] p=2 s=2 r=3 d=1 c=1 score=10

Frontier: B, C, D1
Work set: B, D1
Why selected: <one-line explanation that respects dependency and readiness>
Next action(s): <one or more concrete next steps>
Last result: <what just happened>
If this fails: <next best frontier leaf or re-expansion plan>
```

The very first substantive reply after initial branching should already use this structure or the compact tree format.

For mandatory decomposition:

```text
Exploration Update
Root objective: <goal>

Current tree
[A] <goal> [active]
\- OR
   |- [B] <route> [active]
   |  \- AND
   |     |- [B1] <subtask> [active] p=3 s=2 r=3 d=1 c=1 score=14
   |     \- [B2] <subtask> [active] p=3 s=3 r=2 d=1 c=1 score=15
   \- [C] <alternative> [active] p=2 s=2 r=3 d=2 c=1 score=10

Frontier: B1, B2, C
Work set: B1, B2
Why selected: B1 and B2 are independent mandatory children under the same AND decomposition, and two workers are available
Next action(s): <parallel actions for B1 and B2>
Last result: <what just happened>
If this fails: <replacement child, alternate route, or re-expansion plan>
```

## Failure and Exhaustion Template

Use this when a leaf fails or the frontier is nearly exhausted:

```text
Branch result
Failed leaf: <id>
Failure reason: <evidence that falsified or blocked it>
Effect on frontier: <what was removed or downgraded>
Next work set: <id, ids, or none>
Higher-level re-expansion: <what new branch search was attempted>
Conclusion: <either the next route or that no credible branch remains under current constraints>
```

## OR and AND Reminder

- `OR` children are alternative routes. One successful child can satisfy the parent.
- `AND` children are mandatory steps. All required children must complete.
- If a goal needs mandatory work items, insert an explicit `AND` node under that goal.
- Never combine sibling `OR` routes into one composite success path.
- Both `AND` and `OR` children may run in parallel when enough workers are available, but the logical success condition does not change.
- If a node depends on other nodes' outputs, place it above them and decompose it until `ready` and `waiting` children are visible.

## When to Stop

Stop searching when one of these is true:

- a branch is verified and the task is complete
- every credible branch is blocked by an external dependency
- the frontier is exhausted and higher-level re-expansion did not produce a credible new branch

When stopping without success, say that the task is unresolved under current evidence and constraints.
