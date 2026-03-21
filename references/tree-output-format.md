# Tree Output Format

Use this format when showing the exploration tree to the user.
Keep it compact, current, and easy to compare.

## Required Elements

Always show:

- root objective
- root objective at the top
- explicit `AND` or `OR` logic rows where structure matters
- active frontier leaves
- current work set
- status changes since the last update
- short reason for the current selection

## Recommended Node Fields

Use these compact fields on goal leaves when helpful:

- `p`: estimated success likelihood
- `d`: estimated remaining steps
- `c`: next-step cost
- `status`: `active`, `running`, `failed`, `blocked`, `pruned`, or `done`

## Compact Tree Template

```text
Exploration Tree
[A] Fix slow startup and land a verified fix [active]
\- OR
   |- [B] Use import profiling route [active] p=0.70 d=1 c=2
   |- [C] Inspect config-loading route [active] p=0.55 d=2 c=1 <- selected
   \- [D] Check IO or network waits [active] p=0.45 d=2 c=2

Frontier: B, C, D
Work set: C
Why selected: C has the best current balance of likelihood, distance, and cost
Last change: initial OR branching completed
```

For mandatory decomposition:

```text
Exploration Tree
[A] Complete the analysis [active]
\- OR
   |- [B] Use known-event route [active]
   |  \- AND
   |     |- [B1] Prepare event list [running]
   |     \- [B2] Fetch market data [running]
   \- [C] Use broad factor-analysis route [active]

Frontier: B1, B2, C
Work set: B1, B2
Why selected: B1 and B2 are independent mandatory children under the chosen AND decomposition, and two workers are available
Last change: route B expanded into mandatory parallel subtasks
```

## Update Rules

Update the tree after:

- initial branching
- the first substantive reply after branching, even if the user did not ask for the tree
- selecting a new leaf
- selecting multiple leaves for parallel execution
- expanding a leaf into children
- marking a leaf failed, blocked, pruned, or done
- completing the task
- concluding that no credible new branch is currently available

If the tree gets large, show only the relevant subtree plus a frontier summary.
Do not dump the full history every time.
If the frontier is exhausted, say so explicitly and state whether higher-level re-expansion found any credible new branch.
If a node is blocked, also state whether the blocker is `local`, `route`, or `global`, and whether substitute children were created.
Do not default to left-to-right flow arrows.

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

When multiple workers are used, explain the work set.
Examples:

- "B1 and B2 run together because they are independent AND children and both are required."
- "C and D also run in parallel because two extra workers are free and speculative OR exploration is worth the cost."

## OR and AND Rules

Apply these display rules:

- show the final objective at the top
- if children are competing solutions, show `OR` beneath the parent
- if children are mandatory substeps, show `AND` beneath the parent
- never show sibling `OR` nodes as if they must all succeed together
- `AND` children may run in parallel
- `OR` children may also run in parallel when worker capacity allows and the speculative value is high enough

Bad pattern:

```text
[OR] -> A
 /     \
[AND]   D
/   \
B    C
```

This is hard to read because the goal is on the right and the tree behaves like a sideways flowchart.

Correct pattern:

```text
[A] Final objective
\- OR
   |- AND
   |  |- [B]
   |  \- [C]
   \- [D]
```
