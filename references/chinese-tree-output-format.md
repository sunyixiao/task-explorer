# Chinese Tree Output Format

When the user is communicating in Chinese, prefer this user-facing format.
Keep node IDs such as `A`, `A2`, `B1` and score fields such as `p`, `s`, `r`, `d`, `c`, `score` in ASCII so branches remain easy to compare.

## Recommended Chinese Labels

Use these labels by default:

- `Exploration Tree` -> `探索树`
- `Root objective` -> `根目标`
- `Current tree` -> `当前树`
- `Frontier` -> `当前前沿`
- `Selected leaf` -> `当前选中叶子`
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
[0] 根目标: 修复启动慢并给出已验证结论 [活跃]
|- [A] 复现并测量基线 [完成] p=3 s=3 r=3 d=1 c=1 score=14
|  |- [A1] 对比冷启动与热启动 [失败] p=1 s=2 r=3 d=2 c=1 score=4
|  \- [A2] 分析 import 路径耗时 [活跃] p=3 s=3 r=3 d=1 c=1 score=15 <- 当前选中
|- [B] 检查配置加载路径 [活跃] p=2 s=2 r=3 d=2 c=1 score=10
\- [C] 检查 IO 或网络等待 [活跃] p=1 s=2 r=2 d=2 c=2 score=4

当前前沿: A2, B, C
当前选中叶子: A2
选择原因: A2 相比 B 和 C，成功概率更高，剩余路径更短，且当前尝试成本可接受。
下一步动作: 运行 import-time profiler 并记录最慢模块。
最新结果: A1 已排除，因为冷启动与热启动差异不足以解释问题。
若失败则: 转向 B；如果 B 也失败，再回到更高层检查是否还能发明新分支。
```

## Chinese Frontier Table Template

```text
前沿表
ID   目标                             p  s  r  d  c  score  状态    下一步
A2   分析 import 路径耗时             3  3  3  1  1  15     活跃    运行 import-time profiler
B1   检查配置加载                     2  2  3  2  1  10     活跃    跟踪配置读取路径
C1   检查外部 IO 等待                 1  2  2  2  2   4     活跃    本地禁用网络调用后复测
```

Keep the table narrow.
If more explanation is needed, add one short line below it instead of widening every cell.

## Chinese Failure and Exhaustion Template

```text
分支结果
失败叶子: A2
失败原因: import profiler 结果显示主要耗时不在 import 阶段，因此该路径被证伪。
对前沿的影响: A2 从活跃前沿移除，B 成为当前最优候选。
下一选中叶子: B
高层重扩展: 暂不需要；当前仍有可信前沿。
结论: 继续沿 B 分支探索。
```

When the frontier is exhausted, state it plainly:

```text
分支结果
失败叶子: B2
失败原因: 关键假设已被证伪，且当前环境缺少进一步验证所需条件。
对前沿的影响: 当前前沿已耗尽。
下一选中叶子: 无
高层重扩展: 已回到更高层检查替代方案，但未找到新的可信分支。
结论: 在当前证据与约束下，任务暂时无法继续推进。
```

## Style Rules

- Prefer short Chinese sentences over verbose narration.
- Explain why the selected leaf beat the main alternatives.
- Distinguish confirmed facts from estimates.
- If no credible branch remains, say so directly instead of implying that more searching will surely help.

