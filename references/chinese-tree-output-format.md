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
- `running` -> `执行中`
- `blocked` -> `阻塞`
- `failed` -> `失败`
- `pruned` -> `剪枝`
- `done` -> `完成`

You may keep the raw English status token in brackets if consistency matters more than readability.

## Chinese Tree Template

```text
探索树
[A] 修复启动慢并给出已验证结论 [活跃]
\- OR
   |- [B] 走 import 分析路线 [活跃] p=3 d=1 c=1
   |- [C] 检查配置加载路线 [活跃] p=2 d=2 c=1 <- 当前选中
   \- [D] 检查 IO 或网络等待 [活跃] p=1 d=2 c=2

当前前沿: B, C, D
当前工作集: C
选择原因: C 当前在成功概率、剩余路径和尝试成本之间最平衡。
下一步动作: 跟踪配置读取路径。
最新结果: 已完成初始 OR 分解。
```

如果是必做分解，应该这样画：

```text
探索树
[A] 完成分析 [活跃]
\- OR
   |- [B] 走已知事件路线 [活跃]
   |  \- AND
   |     |- [B1] 整理事件列表 [执行中]
   |     \- [B2] 获取市场数据 [执行中]
   \- [C] 走宽口径因子路线 [活跃]

当前前沿: B1, B2, C
当前工作集: B1, B2
选择原因: B1 和 B2 是同一 AND 结构下相互独立且都必须完成的子任务，当前有两个可用 agent。
最新结果: 已把 B 路线展开成必做并行子任务。
```

## Chinese Frontier Table Template

```text
前沿表
ID   目标                               p  s  r  d  c  score  状态    下一步
B1   整理事件列表                       3  2  3  1  1  14     活跃    汇总候选事件
B2   获取市场数据                       3  3  2  1  1  15     活跃    拉取价格序列
C    走宽口径因子路线                   2  2  3  2  1  10     活跃    建立替代分析法
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

## Style Rules

- Prefer short Chinese sentences over verbose narration.
- Explain why the current work set beat the main alternatives.
- Distinguish confirmed facts from estimates.
- If no credible branch remains, say so directly instead of implying that more searching will surely help.



