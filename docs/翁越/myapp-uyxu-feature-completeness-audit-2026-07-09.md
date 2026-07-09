# 功能完整度审计

## 摘要

- 目标 URL: `https://myapp-uyxu.vercel.app/dashboard`
- 评测时间: `2026-07-09T13:50:29Z`
- 总分: `4.30 / 5.0`
- P0/P1 是否通过: `true`
- 结论: `pass`

## 需求矩阵

| 需求 | 优先级 | 分数 | 状态 | 证据摘要 |
| --- | --- | --- | --- | --- |
| UR-1 快速配置 | P0 | 0.80 | partial | 在登录态下，审计通过真实产品 UI 创建了 `picpi`，观察到 `POST /api/competitors` 返回 `201`，随后竞对列表持久化显示 `picpi` 和 `Audit Example`；但未发现非破坏性的停止监控入口，如需停用很可能只能删除竞对。 |
| UR-3 持续采集与解读 | P0 | 0.75 | partial | 创建 `picpi` 并触发上游 demo timer 后，`/dashboard/snapshots/8` 确实出现了 1 条真实记录，`/api/snapshots?competitor_id=8&limit=20` 也返回非空数组；但产品没有暴露监控间隔配置，且在 `picpi` 每 10 秒变化的情况下，本次只观察到 1 条变化内容，持续采集强度证据不足。 |
| UR-4 结构化情报消费 | P0 | 1.00 | implemented | Dashboard 收集箱具备归档状态标签、竞对筛选、重要度筛选、可选择列表和详情跳转，历史页与详情页也都可用。 |
| UR-5 可执行建议 | P0 | 1.00 | implemented | 快照详情 `552` 的 AI 解读区包含完整的 `变化 -> 意图 -> 行动 -> 理由` 链路。 |
| UR-6 主动通知 + 反馈 | P1 | 0.75 | partial | 设置页存在已开启的邮件通知开关，且由 `/api/user/settings` 支撑；但未发现投递历史，也未发现产品内的反馈机制。 |

## 成功标准检查

- 无需人工操作即可持续获得情报: `partial`
- 存在 `变化 -> 意图 -> 建议` 链路: `met`
- 存在用于提升质量的反馈机制: `unmet`

## 关键证据

- `https://myapp-uyxu.vercel.app/dashboard`
  匿名态下通过 DevTools 打开会重定向到 `/login`，且受保护 API 返回 `401`，说明匿名用户无法验证核心监控流程。
- `https://myapp-uyxu.vercel.app/dashboard/competitors`
  登录态下存在真实的竞对管理界面。审计在这里创建了 `picpi`，之后确认 `picpi` 和 `Audit Example` 都被持久化保存；同时未发现非破坏性的停止监控入口，如需停用很可能只能删除竞对。
- `https://myapp-uyxu.vercel.app/api/competitors`
  创建过程中，产品发起了 `POST /api/competitors` 并返回 `201`，随后 `GET /api/competitors` 返回 `array(2)`。
- `https://picpi-iota.vercel.app/api/timer/start`
  审计调用了 `picpi` demo timer，参数为 `{"interval":10}`，用于安全地产生上游变化证据。
- `https://myapp-uyxu.vercel.app/dashboard`
  重新验证后，首页直接显示 `共 1 条`、`picpi`、`2026/7/9 21:45:18`、`促销活动`，以及截断后的 AI 摘要。
- `https://myapp-uyxu.vercel.app/api/snapshots?archived=false&page=1&pageSize=100`
  Dashboard 数据请求返回 `array(1)`，字段包括 `id`、`competitorId`、`crawledAt`、`changeType`、`summary`、`importance`、`archivedAt` 和 `readAt`。
- `https://myapp-uyxu.vercel.app/dashboard/snapshots/8`
  `picpi` 的历史页已经展示 1 条真实快照记录。
- `https://myapp-uyxu.vercel.app/api/snapshots?competitor_id=8&limit=20`
  历史页请求返回 `array(1)`，在引入上游变化后不再是空状态。
- `https://myapp-uyxu.vercel.app/dashboard/settings`
  设置页只暴露了邮件通知开关，没有暴露监控频率或抓取间隔设置。
- `https://myapp-uyxu.vercel.app/dashboard/snapshots/8/552`
  详情页展示了 `AI 解读`，并明确包含 `【变更】`、`【意图】`、`【行动】`、`【理由】`。
- `https://myapp-uyxu.vercel.app/api/snapshots/552`
  详情请求返回了完整快照元数据和 `htmlContent`，说明产品保存了抓取结果，而不只是静态卡片。
- `https://myapp-uyxu.vercel.app/dashboard/settings`
  设置页显示 `启动邮件通知` 为开启状态，`/api/user/settings` 也返回了 `emailNotifyEnabled: true`。

## 主要缺口

- 在 dashboard、历史页、详情页和设置页中，都没有发现点赞/点踩、质量纠错、反馈表单等产品内反馈入口。
- 通知能力目前只有配置层证据，本次审计没有验证真实邮件投递或投递日志。
- 尽管 `picpi` 上游 demo 已按 10 秒频率持续变化，但本次产品侧仅观察到 1 条变化记录，无法强证明监控循环已稳定覆盖该变化频率。
- 产品未发现非破坏性的停止监控能力；如需停用很可能只能删除竞对，目标生命周期管理不完整，且历史监控内容可能因此失去查看入口。
- 产品没有向用户暴露监控间隔时间设置，因此无法验证用户是否能控制采集周期。

## 范围说明

- 匿名态证据和登录态证据已分开处理。核心产品验证依赖浏览器中已存在的 `test@163.com` 登录态。
- 审计执行了低风险产品变更：创建 `Audit Example`、创建 `picpi`，以及触发 `picpi` 上游 demo timer API 以生成真实变化证据。
- 审计刻意避免了删除竞对或修改账户级配置等破坏性操作，仅观察了已有通知开关状态。
- 早期看到的 dashboard 与历史页空状态，已被后续 `picpi` 变化产生后的重新验证结果替换。

## JSON 结果

```json
{
  "target_url": "https://myapp-uyxu.vercel.app/dashboard",
  "evaluated_at": "2026-07-09T13:50:29Z",
  "tooling": {
    "opencli": "用于登录态浏览、交互和网络取证。",
    "chrome_devtools": "用于验证匿名态重定向和受保护 API 行为。"
  },
  "total_score": 4.3,
  "p0_p1_passed": true,
  "verdict": "pass",
  "requirements": [
    {
      "id": "UR-1 Quick setup",
      "priority": "P0",
      "score": 0.8,
      "status": "partial",
      "summary": "审计通过真实 UI 创建了 picpi，观察到 POST /api/competitors 返回 201，并确认该竞对已持久化保存；但未发现非破坏性的停止监控入口，如需停用很可能只能删除竞对。",
      "evidence_refs": ["e2", "e3"]
    },
    {
      "id": "UR-3 Continuous collection and interpretation",
      "priority": "P0",
      "score": 0.75,
      "status": "partial",
      "summary": "在 picpi 上游 demo 发生变化后，产品产出了一条包含时间戳、变化类型、存储内容和 AI 解读的真实快照；但未暴露监控间隔设置，且本次只观察到 1 条变化记录。",
      "evidence_refs": ["e4", "e7", "e8", "e9", "e11"]
    },
    {
      "id": "UR-4 Structured intelligence consumption",
      "priority": "P0",
      "score": 1,
      "status": "implemented",
      "summary": "Dashboard 收集箱包含筛选器、归档状态标签、可选择行，以及跳转到历史页和详情页的入口，且对应快照 API 非空。",
      "evidence_refs": ["e5", "e6", "e7", "e8"]
    },
    {
      "id": "UR-5 Actionable recommendations",
      "priority": "P0",
      "score": 1,
      "status": "implemented",
      "summary": "快照详情 552 在 AI 解读区明确展示了变化、意图、行动和理由。",
      "evidence_refs": ["e10", "e11"]
    },
    {
      "id": "UR-6 Proactive notification + feedback",
      "priority": "P1",
      "score": 0.75,
      "status": "partial",
      "summary": "邮件通知已配置并持久化，但未找到投递证据，也未发现情报质量反馈机制。",
      "evidence_refs": ["e12"]
    }
  ],
  "success_criteria": [
    {
      "id": "recurring_intelligence_without_manual_operation",
      "status": "partial",
      "summary": "在上游 picpi demo timer 引入变化后，产品自动展示出了一条真实 picpi 变化，但仅有单条记录，持续性证据偏弱。"
    },
    {
      "id": "change_intent_recommendation_chain",
      "status": "met",
      "summary": "快照详情 552 包含明确的变化、意图、行动和理由文本。"
    },
    {
      "id": "feedback_mechanism_for_quality_improvement",
      "status": "unmet",
      "summary": "未发现用于评价或纠正情报质量的反馈机制。"
    }
  ],
  "evidence": [
    {
      "id": "e1",
      "type": "network",
      "url": "https://myapp-uyxu.vercel.app/dashboard",
      "summary": "匿名验证时会重定向到 /login，且 /api/auth/me、/api/competitors、/api/snapshots 等受保护 API 返回 401。"
    },
    {
      "id": "e2",
      "type": "interaction",
      "url": "https://myapp-uyxu.vercel.app/dashboard/competitors",
      "summary": "审计通过产品内竞对表单创建了 picpi，随后在列表中确认它已持久化；同时未发现非破坏性的停止监控入口，如需停用很可能只能删除竞对。"
    },
    {
      "id": "e3",
      "type": "network",
      "url": "https://myapp-uyxu.vercel.app/api/competitors",
      "summary": "创建流程发起了 POST /api/competitors 并返回 201，随后 GET /api/competitors 返回 array(2)。"
    },
    {
      "id": "e4",
      "type": "interaction",
      "url": "https://picpi-iota.vercel.app/api/timer/start",
      "summary": "审计触发了间隔 10 秒的 picpi demo timer，以安全地产生上游变化。"
    },
    {
      "id": "e5",
      "type": "page",
      "url": "https://myapp-uyxu.vercel.app/dashboard",
      "summary": "Dashboard 收集箱展示了 1 条 picpi 真实记录，包含时间、标题、重要度和截断后的 AI 摘要。"
    },
    {
      "id": "e6",
      "type": "network",
      "url": "https://myapp-uyxu.vercel.app/api/snapshots?archived=false&page=1&pageSize=100",
      "summary": "Dashboard 快照请求返回 array(1)，字段包括 crawledAt、changeType、summary、importance、archivedAt 和 readAt。"
    },
    {
      "id": "e7",
      "type": "page",
      "url": "https://myapp-uyxu.vercel.app/dashboard/snapshots/8",
      "summary": "picpi 历史页展示了 1 条时间为 2026/7/9 21:45:18 的真实快照记录。"
    },
    {
      "id": "e8",
      "type": "network",
      "url": "https://myapp-uyxu.vercel.app/api/snapshots?competitor_id=8&limit=20",
      "summary": "竞对历史请求返回 array(1)，重新验证后替代了早先的空状态。"
    },
    {
      "id": "e9",
      "type": "page",
      "url": "https://myapp-uyxu.vercel.app/dashboard/settings",
      "summary": "设置页仅显示邮件通知开关，没有暴露监控频率或抓取间隔设置。"
    },
    {
      "id": "e10",
      "type": "page",
      "url": "https://myapp-uyxu.vercel.app/dashboard/snapshots/8/552",
      "summary": "详情页为记录 552 渲染了包含变化、意图、行动和理由的 AI 解读。"
    },
    {
      "id": "e11",
      "type": "network",
      "url": "https://myapp-uyxu.vercel.app/api/snapshots/552",
      "summary": "详情 API 返回了完整存储快照内容，包括 htmlContent、summary、importance 和 readAt。"
    },
    {
      "id": "e12",
      "type": "network",
      "url": "https://myapp-uyxu.vercel.app/api/user/settings",
      "summary": "设置 API 返回了持久化用户设置，其中包括 emailNotifyEnabled=true。"
    }
  ],
  "scope_notes": [
    "匿名态证据受登录保护限制，因此实现评分依赖浏览器中已存在的登录会话。",
    "审计创建了两个低风险夹具：Audit Example 和 picpi，并使用 picpi 进行真实上游变化验证。",
    "审计触发了 picpi demo timer API 以制造真实变化，然后重新验证了 dashboard、竞对历史页和快照详情页。",
    "未执行破坏性操作。没有删除竞对，也没有修改账户级通知设置。"
  ]
}
```
