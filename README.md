# Task Explorer

Tree-driven autonomous task exploration for Codex-style agents.

Original author: **sunyixiao**  
Original repository: <https://github.com/sunyixiao/task-explorer>

`task-explorer` is a skill that pushes an agent to treat a task as a search tree instead of a linear checklist. It generates multiple routes, scores active frontier leaves, chooses the most promising affordable shortest path, backtracks after failure, and keeps the user updated with the current exploration tree.

## What This Skill Solves

Many agents are too conservative:

- they stop at planning
- they commit too early to one path
- they do not compare alternatives explicitly
- they fail to show the user how the search is evolving

This skill fixes that by turning task execution into a branching search process with visible scoring, frontier management, and failure handling.
It also distinguishes competing `OR` routes from mandatory `AND` subtasks so sibling solution branches are not incorrectly merged into one composite path.

## Core Idea

The task is modeled as an exploration tree:

- Root node: the final user objective
- First layer: multiple materially different solution routes
- Leaf nodes: branches that can be tried next
- Frontier: all active leaves that still might reach the goal

At each step, the agent selects the frontier leaf that best balances:

- success likelihood
- remaining path length
- execution cost
- information gain
- reversibility and safety

If a branch fails, it is removed from the active frontier. The agent then selects the next best leaf. If the frontier is exhausted, the agent moves back up the tree and checks whether credible new branches can still be invented. If not, it reports that the task is unresolved under the current evidence and constraints.

The tree is meant to be shown proactively.
The agent should render it in the first substantive response after initial branching instead of waiting for the user to ask.

## Default Scoring

The default rubric uses:

`score = 4p + 2s + r - 2d - c`

Where:

- `p`: success likelihood
- `s`: information gain
- `r`: reversibility and safety
- `d`: remaining distance
- `c`: immediate cost

The score is not presented as proof. It is a ranking aid for comparing frontier leaves consistently.

## How to Use

Place this folder in your Codex skills directory and invoke it explicitly:

```text
$task-explorer Explore this repository as a search tree, keep the frontier visible, and finish with a verified conclusion.
```

Chinese usage also fits naturally:

```text
用 $task-explorer 把这个任务当成探索树来做，持续展示前沿、当前叶子、失败回溯和最终结论。
```

## OpenClaw Quick Start

OpenClaw uses AgentSkills-compatible skill folders, so this repository can be installed as a normal OpenClaw skill. See the official Skills docs: <https://docs.openclaw.ai/tools/skills>.

OpenClaw loads skills from:

- `~/.openclaw/skills`
- `<workspace>/skills`

If the same skill name exists in both places, the workspace copy wins.

### Option 1: Shared install for all local OpenClaw agents

PowerShell example:

```powershell
New-Item -ItemType Directory -Force "$HOME\.openclaw\skills" | Out-Null
git clone https://github.com/sunyixiao/task-explorer.git "$HOME\.openclaw\skills\task-explorer"
```

This makes the skill visible to OpenClaw agents on the same machine.

### Option 2: Workspace-only install

From your OpenClaw workspace root:

```powershell
New-Item -ItemType Directory -Force ".\skills" | Out-Null
git clone https://github.com/sunyixiao/task-explorer.git ".\skills\task-explorer"
```

Use this when you want the skill to apply only to one workspace or one agent setup.

### After installing

Start a new OpenClaw session before testing.

OpenClaw snapshots eligible skills when a session starts, so a new session is the safest way to pick up a newly installed skill. If the skills watcher is enabled, changes to `SKILL.md` can also refresh during a session, but relying on a fresh session is more predictable.

### Invoking in OpenClaw

Example prompts:

```text
Use $task-explorer to explore this repository as a search tree, keep the frontier visible, and stop only after verification or frontier exhaustion.
```

```text
用 $task-explorer 把这个问题当成探索树来做，持续展示当前前沿、当前选中叶子、失败回溯和最终结论。
```

## Good Prompts

- "Explore this bug as a search tree and keep showing the tree."
- "Do not stop at analysis. Branch, test, backtrack, and close the loop."
- "Try several routes, choose the most promising shortest path, and keep going until one works."
- "Keep searching for new branches if the current frontier fails, and say clearly if no credible route remains."

## Example Output

```text
探索树
[0] 根目标: 修复启动慢并给出已验证结论 [活跃]
|- [A] 复现并测量基线 [完成] p=3 s=3 r=3 d=1 c=1 score=14
|  \- [A2] 分析 import 路径耗时 [活跃] p=3 s=3 r=3 d=1 c=1 score=15 <- 当前选中
|- [B] 检查配置加载路径 [活跃] p=2 s=2 r=3 d=2 c=1 score=10
\- [C] 检查 IO 或网络等待 [活跃] p=1 s=2 r=2 d=2 c=2 score=4

当前前沿: A2, B, C
当前选中叶子: A2
选择原因: A2 相比 B 和 C，成功概率更高，剩余路径更短，且当前尝试成本可接受。
下一步动作: 运行 import-time profiler 并记录最慢模块。
```

## Repository Structure

```text
task-explorer/
├─ SKILL.md
├─ agents/
│  └─ openai.yaml
└─ references/
   ├─ and-or-semantics.md
   ├─ tree-output-format.md
   ├─ scoring-and-update-template.md
   └─ chinese-tree-output-format.md
```

## Key Files

- `SKILL.md`: the runtime behavior and decision rules for the skill
- `agents/openai.yaml`: UI-facing metadata
- `references/tree-output-format.md`: compact English tree display format
- `references/scoring-and-update-template.md`: scoring rubric and reusable update templates
- `references/chinese-tree-output-format.md`: Chinese-facing tree display format
- `references/and-or-semantics.md`: strict route-vs-subtask semantics

## Design Principles

- Prefer exploration over premature certainty
- Compare branches explicitly instead of implicitly
- Keep `OR` routes independent and label `AND` decompositions explicitly
- Keep the frontier visible to the user
- Backtrack quickly after falsification
- Admit when no credible branch remains

## Notes

- This repository is human-facing. The actual skill logic lives in `SKILL.md`.
- The README is for GitHub visitors; the skill runtime does not depend on it.
- OpenClaw installation details in this README follow the official Skills documentation: <https://docs.openclaw.ai/tools/skills>.


