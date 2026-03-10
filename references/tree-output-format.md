# Tree Output Format

Use this format when showing the exploration tree to the user.
Keep it compact, current, and easy to compare.

## Required Elements

Always show:

- root objective
- node type for every expanded parent (`OR` or `AND`)
- active frontier leaves
- currently selected leaf
- status changes since the last update
- short reason for the current selection

## Recommended Node Fields

Use these compact fields in the tree:

- `type`: `OR` or `AND`
- `p`: estimated success likelihood
- `d`: estimated remaining steps
- `c`: next-step cost
- `status`: `active`, `running`, `failed`, `blocked`, `pruned`, or `done`

## Compact Tree Template

```text
Exploration Tree
[0] Root: fix slow startup and land a verified fix [active]
|- [A][OR] Reproduce and measure baseline [done] p=0.85 d=1 c=1
|  |- [A1][OR] Compare cold and warm start [failed] p=0.20 d=2 c=1
|  \- [A2][OR] Profile import path [active] p=0.70 d=1 c=2 <- selected
|- [B][OR] Inspect config loading path [active] p=0.55 d=2 c=1
\- [C][OR] Check IO or network waits [active] p=0.45 d=2 c=2

Frontier: A2, B, C
Selected leaf: A2
Why selected: highest current chance of short affordable completion
Last change: A1 failed because the measured gap was negligible
```

## Update Rules

Update the tree after:

- initial branching
- the first substantive reply after branching, even if the user did not ask for the tree
- selecting a new leaf
- expanding a leaf into children
- marking a leaf failed, blocked, pruned, or done
- completing the task
- concluding that no credible new branch is currently available

If the tree gets large, show only the relevant subtree plus a frontier summary.
Do not dump the full history every time.
If the frontier is exhausted, say so explicitly and state whether higher-level re-expansion found any credible new branch.
If a node is blocked, also state whether the blocker is `local`, `route`, or `global`, and whether substitute children were created.

## Status Meanings

- `active`: eligible to be selected next
- `running`: currently being executed
- `blocked`: cannot proceed until an external blocker changes
- `failed`: evidence falsified this branch
- `pruned`: still possible in theory, but dominated by better branches
- `done`: this node's goal was achieved

## Selection Commentary

When choosing a leaf, explain the choice in one line using frontier comparison.
Good examples:

- "A2 beats B and C because it has similar cost but shorter remaining distance."
- "B1 wins because its probability is much higher even though it costs one extra step."
- "C becomes the best leaf after A2 failed and B was blocked by missing credentials."

When no branch remains credible, say so directly.
Example:

- "The frontier is exhausted. I checked higher-level alternatives and did not find a credible new branch under the current constraints."

When a blocker is only method-specific, say so directly.
Example:

- "C1 is blocked only as one method. I expanded C2 and C3 as substitute children under the same route."

## OR and AND Rules

Apply these display rules:

- If siblings are competing solutions, the parent must be shown as `OR`.
- If siblings are mandatory substeps, the parent must be shown as `AND`.
- Never show sibling `OR` nodes as if they must all succeed together.
- If a chosen route needs mandatory subtasks, insert an explicit `AND` node beneath that route before listing those subtasks.

Bad pattern:

```text
[C][OR] Solve the task
|- [C1] Prepare events
\- [C2] Fetch market data
```

and then treating `C1 + C2` as jointly required to satisfy `C`.

Correct pattern:

```text
[C][OR] Solve via known-event route
\- [Cx][AND] Execute route C
   |- [Cx1] Prepare events
   \- [Cx2] Fetch market data
```
