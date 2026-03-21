---
name: task-explorer
description: "Tree-driven autonomous task exploration and execution for unfamiliar codebases, bugs, workflows, datasets, and product questions. Use when Codex should proactively explore a task as a top-down AND/OR goal tree, generate multiple solution branches, score and rank frontier leaves by success likelihood, remaining path length, execution cost, and information gain, distinguish OR alternatives from AND mandatory subtasks, orient parent-child structure by dependency, expand AND nodes until ready and waiting children are explicit, parallelize eligible AND or OR children when enough agent capacity exists, backtrack after failed branches, check whether new branches can still be invented when the frontier is exhausted, and keep the user updated with the current exploration tree without waiting to be asked. Typical triggers include requests to explore independently, think through several routes, avoid stopping at planning, show the search tree, or keep trying alternatives until one succeeds or no credible route remains."
---

# Task Explorer

## Overview

Treat the task as a top-down AND/OR goal tree instead of a single linear plan.
Keep the final objective at the top, place logical composition beneath it, maintain an explicit frontier of ready work leaves, and expand, parallelize, or backtrack until the task is finished or a real external blocker remains.

## Core Model

Use these meanings consistently:

- Root goal: the final user objective at the top of the tree
- Goal node: a concrete goal, hypothesis, or action-bearing subgoal
- Logic node: an explicit `AND` or `OR` composition row beneath a goal
- Frontier: all ready goal leaves that can be advanced now
- Work set: one or more frontier leaves selected for the current execution cycle

Use `OR` only for alternatives.
Use `AND` only for mandatory decomposition.
Do not invent extra `Route` wrapper nodes; the tree should be readable directly as goals plus `AND/OR` logic.

Draw the tree vertically:

- parent goal on top
- `AND` or `OR` beneath the parent when needed
- children below the logic node

Do not draw the final goal on the right as a flowchart target.

Track these fields for every goal node that can be evaluated or worked on:

- `id`: short stable label such as `A`, `A2`, `B1`
- `goal`: what this node is trying to prove or achieve
- `status`: `candidate`, `ready`, `running`, `waiting`, `blocked`, `failed`, `pruned`, or `done`
- `evidence`: the main fact supporting or weakening the node
- `p`: estimated chance that this branch can still reach the final objective
- `d`: estimated remaining steps from this node to completion
- `c`: immediate trial cost for the next action on this node
- `next`: the next concrete action if this node is selected
- `blocker_scope`: optional, one of `local`, `route`, or `global` when a branch is blocked

Use coarse estimates when precision would be fake.
Low, medium, high is acceptable, but keep the scale consistent within one task.

Logic nodes are structural.
Score and schedule goal leaves, not bare `AND/OR` operators.

## Branch Integrity Rules

Apply these rules strictly:

1. Siblings under an `OR` logic node are competing alternatives.
2. Every child under an `OR` logic node must be sufficient to satisfy the parent on its own if it succeeds.
3. Do not mark an `OR` parent complete by combining progress from multiple siblings.
4. Siblings under an `AND` logic node are all required to satisfy the parent.
5. If several steps must all happen, insert an explicit `AND` logic node before those child steps.
6. If several alternatives exist, insert an explicit `OR` logic node before those child steps.
7. If several alternatives share a common prerequisite, lift that prerequisite above them or represent it as a separate `AND` decomposition, but keep the `OR` siblings independent.

Read [references/and-or-semantics.md](references/and-or-semantics.md) for concrete good and bad examples.

## Dependency Orientation Rules

Apply these rules before accepting a tree shape:

1. If node `X` depends on outputs, evidence, or completion from nodes `Y` and `Z`, then `X` must sit above that dependency set, not beside it.
2. Do not place a synthesis, summary, comparison, validation, or conclusion node as a sibling of the prerequisites it consumes.
3. Siblings should be peers at the same abstraction level:
   - alternative ways to satisfy the same parent under `OR`, or
   - jointly required children under `AND`
4. If a node mostly exists to combine or judge the results of other nodes, it should be the parent goal whose children provide those inputs.
5. If a child cannot even be meaningfully evaluated until another sibling finishes, the structure is probably wrong; move the dependent child downward beneath the prerequisite parent.

Sanity check:

- If `D2` needs results from `B` and `C`, then `D2` should not be a sibling of `B` and `C`.
- Either `D` is the parent whose `AND` children include `B`, `C`, and the remaining dependent work, or the dependent work should be merged upward into the real parent goal.

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
These first-layer branches normally sit under an `OR` logic node beneath the root goal.

### 3. Initialize the frontier

For each first-layer leaf, estimate `p`, `d`, and `c`, then mark all viable leaves as `ready`.
Keep the frontier small enough that it can still be compared deliberately.

When an `AND` parent is introduced, continue decomposing until each child is classified as either:

- `ready`: can be worked on now
- `waiting`: structurally relevant, but cannot yet start because a true prerequisite is unfinished

Do not leave a decomposable `AND` node as one opaque pending block if some of its descendants are already `ready`.

## Parallel Scheduling

Logical structure and execution parallelism are different things.

- `AND` or `OR` tells you the success condition.
- available agent capacity tells you how many leaves can be worked in parallel.

Let `k` be the number of independent workers you can afford right now.

- If `k = 1`, select one frontier leaf.
- If `k > 1`, select a work set of up to `k` compatible frontier leaves.

Parallelism rules:

- children under `AND` may be run in parallel when they are independent and all required
- children under `OR` may also be run in parallel when agent capacity is high enough and speculative exploration is worth the cost
- when one `OR` child succeeds, stop, prune, or deprioritize the other siblings unless more evidence is still needed
- do not parallelize destructive or strongly conflicting actions without explicit reason

## Select the Next Work Set

Pick the frontier leaf or work set that appears most likely to reach the goal by the shortest affordable route.
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

When `k > 1`, fill the work set with the best compatible leaves after choosing the top candidate.
Compatibility matters:

- `AND` siblings are usually compatible
- `OR` siblings are compatible only when speculative parallelism is worth the resource cost
- two leaves that depend on the same single mutable resource may be incompatible

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

1. Build or refresh the frontier.
2. Determine available worker capacity `k`.
3. Select the best work set.
4. Execute one or more high-signal actions on those leaves.
5. Interpret each result immediately.
6. If a leaf advanced but did not finish the task, add an explicit `AND` or `OR` logic node if needed, then expand its children and classify each descendant as `ready` or `waiting`.
7. If a leaf completed the task, propagate success upward according to the enclosing `AND/OR` logic.
8. If a leaf was falsified, mark it `failed`, record why, and remove it from the active frontier.
9. If a leaf is temporarily impossible to try, classify the blocker before deciding what to do next.

Repeat the same rule at every new depth.
Every selected leaf in the current work set becomes the root of a local subproblem, and its children inherit the same scheduling rules.

For `OR` logic nodes:

- one successful child is enough to satisfy the parent
- other siblings become `pruned`, `failed`, `canceled`, or still untested alternatives
- children may run in parallel if resources allow, but success still comes from any one sufficient child
- do not merge sibling outputs into one composite answer

For `AND` logic nodes:

- all required children must complete to satisfy the parent
- children may run in parallel if they are independent and resources allow
- children that are structurally required but not yet startable should be shown as `waiting`, not hidden
- failure of a mandatory child threatens the whole parent unless a replacement child is invented inside that same decomposition
- these child steps are mandatory parts of one decomposition, not alternatives

## Blocker Handling

Do not treat every blocker as terminal.
First classify the blocker:

- `local`: a method-specific issue such as a broken API, changed library function, rate limit, missing one credential, bad parser, or inaccessible site
- `route`: the chosen route still matters, but its current decomposition is blocked and must be re-expanded inside the route
- `global`: a true external stop such as unavailable required credentials, unavailable target system, destructive action needing approval, or missing business decision

Apply these rules:

1. A `local` blocker does not end the parent route.
2. A `local` blocker must trigger at least one new alternative child if any credible substitute method exists.
3. Prefer adding substitute siblings under the same `OR` parent or replacement children inside the same `AND` route before escalating upward.
4. A data-source failure is evidence against one method, not automatically against the whole analytical route.
5. Only call the frontier exhausted after checking whether substitute methods can be invented at the current level or one level up.

When a branch hits a `local` blocker, explicitly try alternatives such as:

- different data source
- different library or endpoint
- cached or local snapshot
- broader proxy variable or indirect evidence
- manual extraction if cost is acceptable

Read [references/blocker-re-expansion.md](references/blocker-re-expansion.md) for examples of correct and incorrect blocker handling.

## Backtrack and Re-expand

Do not keep hammering a failed branch without new evidence.
When a leaf fails:

- record the failure reason
- prune the leaf from active consideration
- select the next best remaining frontier leaf

When a leaf is blocked:

- record the blocker and its scope
- if the blocker is `local`, immediately ask whether substitute child nodes can be invented under the same parent
- if substitutes exist, add them and keep the blocked node as one failed method, not as the end of the route
- if no substitutes exist locally, move one level up and re-expand the ancestor
- only preserve a node as `blocked` when further progress truly depends on an external change

If all current leaves fail, become blocked, or look dominated:

1. Inspect higher-level ancestors for unexplored alternatives.
2. Check whether new sibling branches can be invented where uncertainty is still high, especially after method-specific blockers.
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
- newly exposed `waiting` children when they matter to understanding the plan
- the selected work set
- the reason the selected leaf or work set won
- failures that were pruned
- the relevant `AND/OR` logic above the selected nodes

The default display style is a top-down tree with the final objective at the top.
Do not default to left-to-right arrows or put the goal on the far right.

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
- Keep the final goal at the top of the tree.
- Treat `AND/OR` as logic structure, and treat parallelism as scheduling based on available workers.
- Orient parent-child structure by real dependency, not by informal project phases.
- For `AND` goals, keep decomposing until `ready` and `waiting` children are explicit whenever that decomposition is discoverable.
- If resources allow, consider parallel exploration of both `AND` children and selected `OR` alternatives.
- Treat broken tools, changed APIs, parser failures, and missing one data source as prompts to invent substitute children before declaring the route blocked.
- Do not conclude "unresolved" until substitute methods were attempted or explicitly rejected as not credible.

## Anti-Patterns

Do not:

- collapse too early into one linear plan
- branch endlessly without selecting a winner
- keep retrying a falsified leaf without new evidence
- hide the search tree from the user
- wait for the user to request the tree before showing it
- draw the final goal on the far right as a flowchart target by default
- call a branch "best" without comparing it to the current frontier
- score leaves implicitly without exposing the comparison
- merge sibling `OR` routes into one combined solution path
- decompose a chosen route into mandatory steps without labeling that parent as `AND`
- invent extra `Route` wrapper nodes when `AND/OR` logic already expresses the structure
- place a result/synthesis/conclusion node beside the prerequisites it depends on
- leave an `AND` node opaque when some descendants are already `ready`
- stop at planning when the next experiment is already clear
- stop at a blocked child without expanding substitute methods
- treat "API changed", "search key missing", or "library call failed" as the end of the route when other evidence paths still exist

## Example Requests

This skill is a good fit for prompts like:

- "Explore this task as a search tree and keep showing the tree."
- "Try several routes, choose the most promising shortest path, and keep going until one works."
- "Do not stop at analysis. Branch, test, backtrack, and close the loop."
- "Keep searching for new branches if the current frontier fails, and say clearly if no credible route remains."
