# Tree Output Format

Use this format when showing the exploration tree to the user.
Keep it compact, current, and easy to compare.

## Required Elements

Always show:

- root objective
- active frontier leaves
- currently selected leaf
- status changes since the last update
- short reason for the current selection

## Recommended Node Fields

Use these compact fields in the tree:

- `p`: estimated success likelihood
- `d`: estimated remaining steps
- `c`: next-step cost
- `status`: `active`, `running`, `failed`, `blocked`, `pruned`, or `done`

## Compact Tree Template

```text
Exploration Tree
[0] Root: fix slow startup and land a verified fix [active]
|- [A] Reproduce and measure baseline [done] p=0.85 d=1 c=1
|  |- [A1] Compare cold and warm start [failed] p=0.20 d=2 c=1
|  \- [A2] Profile import path [active] p=0.70 d=1 c=2 <- selected
|- [B] Inspect config loading path [active] p=0.55 d=2 c=1
\- [C] Check IO or network waits [active] p=0.45 d=2 c=2

Frontier: A2, B, C
Selected leaf: A2
Why selected: highest current chance of short affordable completion
Last change: A1 failed because the measured gap was negligible
```

## Update Rules

Update the tree after:

- initial branching
- selecting a new leaf
- expanding a leaf into children
- marking a leaf failed, blocked, pruned, or done
- completing the task
- concluding that no credible new branch is currently available

If the tree gets large, show only the relevant subtree plus a frontier summary.
Do not dump the full history every time.
If the frontier is exhausted, say so explicitly and state whether higher-level re-expansion found any credible new branch.

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
