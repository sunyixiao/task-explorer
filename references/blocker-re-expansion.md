# Blocker Re-expansion

Use this file when a branch becomes blocked and you need to decide whether to stop, backtrack, or invent replacement children.

## Core Rule

A blocker is not automatically the end of a route.

First ask:

1. Is this blocker specific to one method?
2. Is the route still valid if a different method is used?
3. Does progress now depend on a true external condition?

Only the third case is a real external blocker.

## Blocker Scope

- `local`: only one method is broken
- `route`: the current route needs internal redesign
- `global`: the task cannot continue without external change

Examples of `local` blockers:

- one API key is missing but other sources exist
- a library function changed
- a site layout changed and the scraper broke
- one parser fails on one format
- one vendor endpoint rate-limits requests

Examples of `global` blockers:

- the only required credential is unavailable
- the only target system is offline and no substitute evidence exists
- a destructive action requires approval
- a business choice from the user is required

## Required Behavior

When a leaf hits a `local` blocker:

1. Mark the current method as blocked or failed.
2. Generate substitute child nodes under the same parent if the route is still credible.
3. Score those new children against the frontier.
4. Continue with the best surviving leaf.

Do not present the route as exhausted until substitute children were considered.

## Good Pattern

```text
探索树
[0][OR] 研究中东冲突对A股影响 [活跃]
\- [C][OR] 用已知事件路线分析 [活跃]
   \- [Cx][AND] 执行路线 C [活跃]
      |- [Cx1][OR] 用 akshare 原油数据 [失败]
      |- [Cx2][OR] 改用其他公开原油源 [活跃]
      \- [Cx3][OR] 改用中东冲突指数作为代理变量 [活跃]
```

Meaning:

- `Cx1` failed as one method
- route `C` is still alive
- the tree keeps moving through `Cx2` or `Cx3`

## Bad Pattern

```text
探索树
[0] 根目标: 研究中东地区突发新闻对A股大盘影响 [完成]
|- [A][OR] 扩展伊朗事件到更多中东国家事件 [完成]
|- [B][OR] 找已有的中东事件对A股影响研究报告 [阻塞]
|  \- 阻塞原因: 缺少搜索 API key，无法找到研究报告
\- [C][OR] 用更广泛的中东冲突指数来分析 [阻塞]
   \- 阻塞原因: akshare 原油数据接口变更，无法获取数据
```

Why this is bad:

- `B` stopped at one search method instead of spawning substitutes
- `C` stopped at one data source instead of spawning substitutes
- the tree exposed blockers but did not use them to invent new children

## Better Rewrite of the Same Situation

```text
探索树
[0] 根目标: 研究中东地区突发新闻对A股大盘影响 [活跃]
|- [A][OR] 扩展伊朗事件到更多中东国家事件 [完成]
|- [B][OR] 找已有研究或替代证据 [活跃]
|  |- [B1][OR] 用搜索 API 找研究报告 [阻塞] blocker_scope=local
|  |- [B2][OR] 用普通网页搜索和学术站点找报告 [活跃]
|  \- [B3][OR] 找券商研报和新闻评论作为替代证据 [活跃]
\- [C][OR] 用更广泛的中东冲突代理变量来分析 [活跃]
   |- [C1][OR] 用 akshare 原油数据 [阻塞] blocker_scope=local
   |- [C2][OR] 改用其他公开原油源 [活跃]
   \- [C3][OR] 改用战争/地缘风险指数做代理变量 [活跃]
```

## Decision Test

Before writing `blocked`, ask:

- Is the blocker about this method or about the whole route?
- If I were forced to keep pursuing this route, what alternative child would I create next?

If you can answer the second question with a credible method, do not stop the route.
