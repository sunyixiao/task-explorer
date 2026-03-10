---
name: task-explorer
description: "Tree-driven autonomous task exploration and execution for unfamiliar codebases, bugs, workflows, datasets, and product questions. Use when Codex should proactively explore a task as a branching search tree, generate multiple solution routes, score and rank frontier leaves by success likelihood, remaining path length, execution cost, and information gain, distinguish OR route nodes from AND subtask nodes, backtrack after failed branches, check whether new branches can still be invented when the frontier is exhausted, and keep the user updated with the current exploration tree without waiting to be asked. Typical triggers include requests to explore independently, think through several routes, avoid stopping at planning, show the search tree, or keep trying alternatives until one succeeds or no credible route remains."
---

# Task Explorer

## Overview

Treat the task as an AND/OR search tree instead of a single linear plan.
Maintain an explicit frontier of active leaf nodes, always pursue the most promising affordable shortest route, and keep expanding or backtracking until the task is finished or a real external blocker remains.

## Core Model

Use these meanings consistently:

- Root node: the user objective
- Internal node: a decision, hypothesis, or partial subgoal
- Leaf node: a branch that can be advanced next
- Frontier: all non-failed leaves that still might reach the goal

Every non-root node must also declare a node type:

- `OR`: children are alternative self-sufficient routes to satisfy the parent
- `AND`: children are mandatory substeps inside one chosen route

Default to `OR` when branching the search.
Use `AND` only when the parent truly requires multiple child steps to be completed together.

Track these fields for every active or recently failed node:

- `id`: short stable label such as `A`, `A2`, `B1`
- `type`: `OR` or `AND`
- `goal`: what this node is trying to prove or achieve
- `status`: `candidate`, `active`, `running`, `blocked`, `failed`, `pruned`, or `done`
- `evidence`: the main fact supporting or weakening the node
- `p`: estimated chance that this branch can still reach the final objective
- `d`: estimated remaining steps from this node to completion
- `c`: immediate trial cost for the next action on this node
- `next`: the next concrete action if this node is selected

Use coarse estimates when precision would be fake.
Low, medium, high is acceptable, but keep the scale consistent within one task.

## Branch Integrity Rules

Apply these rules strictly:

1. Siblings under an `OR` parent are competing routes, not shared parts of one route.
2. Every `OR` child must be able to satisfy the parent on its own if it succeeds.
3. Do not mark an `OR` parent complete by combining progress from multiple siblings.
4. If a chosen route requires several mandatory actions, insert or relabel an explicit `AND` execution node before adding those child steps.
5. If several competing routes share a common prerequisite, lift that prerequisite above them or represent it as a separate `AND` node, but keep the sibling routes independent.

Read [references/and-or-semantics.md](references/and-or-semantics.md) for concrete good and bad examples.

## Start the Tree

### 1. Build the root

Rewrite the request into:

- Objective
- Success signal
- Constraints
- Known blockers
- Unknowns that can likely be discovered without asking the user

### 2. Create first-layer branches

Generate two to five materially different routes from the root.
Do not create branches that only restate the same idea with different wording.
Good first-layer branches often differ by evidence source, intervention type, or diagnostic strategy.
These first-layer branches are `OR` routes unless the task is obviously a required checklist.

### 3. Initialize the frontier

For each first-layer leaf, estimate `p`, `d`, and `c`, then mark all viable leaves as `active`.
Keep the frontier small enough that it can still be compared deliberately.

## Select the Next Leaf

Pick the leaf that appears most likely to reach the goal by the shortest affordable route.
This is the default selection rule for every level of the tree.

Evaluate each frontier leaf on:

- `p`: success likelihood
- `d`: remaining path length
- `c`: next-step execution cost
- information gain if the step succeeds or fails
- reversibility and blast radius

Use this heuristic:

1. Prefer higher `p`.
2. Prefer shorter `d`.
3. Prefer lower `c` when `p` and `d` are close.
4. Break ties with higher information gain and safer reversibility.

If a quick numeric aid helps, use a rough priority like `p / (1 + d + c)`.
Do not pretend the score is exact; it is only a ranking aid.

### Default Scoring Rubric

Use this default rubric unless the task clearly needs a different one.
Normalize each field to `0..3`:

- `p`: success likelihood
- `s`: information gain from trying the next step
- `r`: reversibility and safety
- `d`: remaining path length
- `c`: immediate execution cost

Then compute:

`score = 4p + 2s + r - 2d - c`

Interpretation:

- high `p` matters most
- high-signal cheap steps are favored
- long paths are penalized
- irreversible or risky steps lose tie-break strength

Use tie-breakers in this order:

1. higher `p`
2. lower `d`
3. lower `c`
4. higher `s`
5. higher `r`

Read [references/scoring-and-update-template.md](references/scoring-and-update-template.md) for default scales and an update template.

## Explore Recursively

Run this loop until the root objective is satisfied or a real blocker remains:

1. Select the best frontier leaf.
2. Execute the smallest high-signal action that advances or tests that leaf.
3. Interpret the result immediately.
4. If the leaf advanced but did not finish the task, choose whether the next children are competing `OR` routes or mandatory `AND` subtasks, label them explicitly, then rescore the frontier.
5. If the leaf completed the task, verify the result and stop.
6. If the leaf was falsified, mark it `failed`, record why, and remove it from the active frontier.
7. If the leaf is temporarily impossible to try, mark it `blocked` and continue elsewhere.

Repeat the same rule at every new depth.
Every selected leaf becomes the root of a local subproblem, and its children compete the same way.

For `OR` parents:

- one successful child is enough to satisfy the parent
- other siblings become `pruned`, `failed`, or still untested alternatives
- do not merge sibling outputs into one composite answer unless the parent is converted to `AND`

For `AND` parents:

- all required children must complete to satisfy the parent
- failure of a mandatory child threatens the whole route unless a replacement child is invented inside that same `AND` decomposition
- these child steps are execution structure inside one route, not alternative routes to satisfy the original parent

## Backtrack and Re-expand

Do not keep hammering a failed branch without new evidence.
When a leaf fails:

- record the failure reason
- prune the leaf from active consideration
- select the next best remaining frontier leaf

If all current leaves fail, become blocked, or look dominated:

1. Inspect higher-level ancestors for unexplored alternatives.
2. Check whether new sibling branches can be invented where uncertainty is still high.
3. Re-open the root only if lower-level re-expansion is weak or exhausted.
4. If no credible new branches can be invented, report that the task remains unresolved under current evidence and constraints.

The tree is not static, but do not assume it can always grow.
Search for new routes whenever the current frontier no longer contains a credible path to completion, and explicitly acknowledge when that search fails.

## Output the Tree Continuously

Show the user the exploration tree throughout the task instead of hiding it.
Do not wait for the user to ask for the tree.
The first substantive response after creating the initial frontier must already include the current exploration tree.
After each meaningful change, update:

- the current tree or relevant subtree
- the active frontier
- the selected leaf
- the reason that leaf won
- failures that were pruned
- whether the selected parent is `OR` or `AND`

Match the user's language in user-facing updates.
If the conversation is in Chinese, keep node IDs and score fields compact as ASCII, but present the tree labels, commentary, and conclusion in Chinese.

Read [references/tree-output-format.md](references/tree-output-format.md) for the compact display format.
Read [references/scoring-and-update-template.md](references/scoring-and-update-template.md) for the frontier table and status update template.
Read [references/chinese-tree-output-format.md](references/chinese-tree-output-format.md) when the user is communicating in Chinese.

## Decision Rules

- Continue without asking the user when the answer can be discovered locally.
- Ask only when blocked by missing credentials, unavailable systems, destructive action, or an external business choice.
- Prefer a cheap decisive test over a large speculative rewrite.
- Keep two to five live branches in normal use; branch wider only when the task is unusually ambiguous.
- Prune dominated branches when another leaf clearly subsumes them.
- Implement once one branch has enough evidence, then verify hard.
- Match the user's language while preserving stable node IDs and comparable score fields.
- Default to showing the tree proactively instead of waiting for a user prompt.
- Preserve route independence under `OR` nodes.

## Anti-Patterns

Do not:

- collapse too early into one linear plan
- branch endlessly without selecting a winner
- keep retrying a falsified leaf without new evidence
- hide the search tree from the user
- wait for the user to request the tree before showing it
- call a branch "best" without comparing it to the current frontier
- score leaves implicitly without exposing the comparison
- merge sibling `OR` routes into one combined solution path
- decompose a chosen route into mandatory steps without labeling that parent as `AND`
- stop at planning when the next experiment is already clear

## Example Requests

This skill is a good fit for prompts like:

- "Explore this task as a search tree and keep showing the tree."
- "Try several routes, choose the most promising shortest path, and keep going until one works."
- "Do not stop at analysis. Branch, test, backtrack, and close the loop."
- "Keep searching for new branches if the current frontier fails, and say clearly if no credible route remains."
