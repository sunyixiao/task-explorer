# Chinese Tree Output Format

When the user is communicating in Chinese, prefer this user-facing format.
Keep node IDs such as `A`, `A2`, `B1` and score fields such as `p`, `s`, `r`, `d`, `c`, `score` in ASCII so branches remain easy to compare.
Also keep node types as ASCII `OR` and `AND`.
目标要放在最上面，自上而下展开，不要把最终目标画在最右边。

## Recommended Chinese Labels

Use these labels by default:

- `Exploration Tree` -> `探索树`
- `Root objective` -> `根目标`
- `Node type` -> `节点类型`
- `Current tree` -> `当前树`
- `Frontier` -> `当前前沿`
- `Selected leaf` -> `当前选中叶子`
- `Work set` -> `当前工作集`
- `Ready` -> `就绪`
- `Waiting` -> `等待`
- `Why selected` -> `选择原因`
- `Next action` -> `下一步动作`
- `Last result` -> `最新结果`
- `If this fails` -> `若失败则`
- `Branch result` -> `分支结果`
- `Failure reason` -> `失败原因`
- `Effect on frontier` -> `对前沿的影响`
- `Higher-level re-expansion` -> `高层重扩展`
- `Conclusion` -> `结论`

## Suggested Status Translation

For user-facing Chinese output, these mappings are usually the clearest:

- `active` -> `活跃`
- `ready` -> `就绪`
- `running` -> `执行中`
- `waiting` -> `等待`
- `blocked` -> `阻塞`
- `failed` -> `失败`
- `pruned` -> `剪枝`
- `done` -> `完成`

You may keep the raw English status token in brackets if consistency matters more than readability.

## Chinese Tree Template

```text
探索树
[A] 修复启动慢并给出已验证结论 [就绪]
\- OR
   |- [B] 走 import 分析路线 [就绪] p=3 d=1 c=1
   |- [C] 检查配置加载路线 [就绪] p=2 d=2 c=1 <- 当前选中
   \- [D] 检查 IO 或网络等待 [就绪] p=1 d=2 c=2

当前前沿: B, C, D
当前工作集: C
选择原因: C 当前在成功概率、剩余路径和尝试成本之间最平衡。
下一步动作: 跟踪配置读取路径。
最新结果: 已完成初始 OR 分解。
```

如果是必做分解，应该这样画：

```text
探索树
[A] 完成分析 [就绪]
\- [D] 形成可验证结论 [就绪]
   \- AND
      |- [B] 建立可信评测框架 [就绪]
      |- [C] 找到有效提升路线 [就绪]
      |- [D1] 定义结果记录模板 [就绪]
      |- [D2] 输出 before/after 对比 [等待]
      \- [D3] 做最小消融并写结论 [等待]

当前前沿: B, C, D1
当前工作集: B, C, D1
选择原因: D 依赖 B 与 C 的产出，所以 D 在上、B/C 在下；同时 D1 已经就绪，而 D2/D3 仍需等待上游结果。
最新结果: 已把 D 展开为就绪与等待子节点。
```

## Chinese Frontier Table Template

```text
前沿表
ID   目标                               p  s  r  d  c  score  状态    下一步
B    建立可信评测框架                   3  2  3  1  1  14     就绪    固定评测集与指标
C    找到有效提升路线                   3  3  2  1  1  15     就绪    展开候选优化分支
D1   定义结果记录模板                   2  2  3  1  1  10     就绪    建立实验日志与对比格式
D2   输出 before/after 对比             1  1  3  2  1   5     等待    等待 B/C 产出
```

Keep the table narrow.
If more explanation is needed, add one short line below it instead of widening every cell.

## Chinese Failure and Exhaustion Template

```text
分支结果
失败叶子: A2
失败原因: import profiler 结果显示主要耗时不在 import 阶段，因此该路径被证伪。
对前沿的影响: A2 从活跃前沿移除，B 成为当前最优候选。
下一工作集: B
高层重扩展: 暂不需要；当前仍有可信前沿。
结论: 继续沿 B 分支探索。
```

When the frontier is exhausted, state it plainly:

```text
分支结果
失败叶子: B2
失败原因: 关键假设已被证伪，且当前环境缺少进一步验证所需条件。
对前沿的影响: 当前前沿已耗尽。
下一工作集: 无
高层重扩展: 已回到更高层检查替代方案，但未找到新的可信分支。
结论: 在当前证据与约束下，任务暂时无法继续推进。
```

## Parallel Work-Set Pattern

如果有多个 agent，可同时派发多个叶子节点。

```text
当前前沿: B1, B2, C
当前工作集: B1, B2, C
并行原因:
- B1 与 B2 属于同一 AND 结构，彼此独立且都必须完成
- C 属于 OR 备选路线；当前 agent 充足，允许做投机并行探索
```

如果 agent 不够，就只选最优的一部分，不改变树的逻辑关系。

## Blocked-but-Continue Pattern

如果阻塞只是某个方法挂了，而不是整条路线失效，要这样表达：

```text
分支结果
阻塞叶子: C1
阻塞范围: local
阻塞原因: akshare 原油接口变更，当前方法失效。
对前沿的影响: C1 暂停，已在同一父节点下扩展 C2、C3 作为替代方法。
下一工作集: C2
高层重扩展: 暂不需要；当前路线仍有可信替代子节点。
结论: 继续沿 C2 分支探索，而不是把 C 整条路线判死。
```

只有当阻塞属于真正外部条件，且没有可信替代路线时，才允许把该父路线长期保留为 `blocked`。

## OR and AND Rules

这部分最重要，必须严格执行：

- `OR` 父节点的兄弟节点代表互斥的候选路线。
- `OR` 父节点只需要一个子节点成功即可满足。
- `OR` 父节点的兄弟节点在 agent 足够时也可以并行探索，但仍然是替代关系，不是共同完成关系。
- 不能把多个 `OR` 兄弟节点的部分进展拼起来当成父节点成功。
- 如果某条已选路线内部需要多个必做步骤，先插入一个显式 `AND` 节点，再把这些步骤挂在它下面。
- `AND` 子节点在彼此独立时可以并行执行，而且它们必须全部完成。
- 如果一个节点依赖别的节点产出，它应在这些前置节点的上方，而不是与它们平级。
- 如果 `AND` 父节点里有些子任务现在能做、有些暂时不能做，就要明确标出 `就绪` 和 `等待`，不能整块挂起。

错误示例：

```text
[C][OR] 已知事件路线
|- [C1] 整理事件
\- [C2] 获取数据
```

然后把 `C1 + C2` 共同当成 `C` 的完成条件。

正确示例：

```text
[A] 完成分析
\- OR
   |- AND
   |  |- [B] 整理事件
   |  \- [C] 获取数据
   \- [D] 使用另一条独立路线完成
```

这里目标 `A` 在最上面，树是自上而下展开的。

依赖方向错误示例：

```text
[A] 完成分析
\- AND
   |- [B] 建立评测框架
   |- [C] 找到提升路线
   \- [D] 写结论
```

如果 `D` 依赖 `B` 和 `C` 的结果，这个结构就是错的。

依赖方向正确示例：

```text
[A] 完成分析
\- [D] 写结论
   \- AND
      |- [B] 建立评测框架
      |- [C] 找到提升路线
      |- [D1] 定义记录模板
      \- [D2] 汇总实验结论
```

## Style Rules

- Prefer short Chinese sentences over verbose narration.
- Explain why the current work set beat the main alternatives.
- Distinguish confirmed facts from estimates.
- If no credible branch remains, say so directly instead of implying that more searching will surely help.




