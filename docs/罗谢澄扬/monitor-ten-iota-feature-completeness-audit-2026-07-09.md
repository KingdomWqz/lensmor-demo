# 功能完整度审计

## 摘要

- 目标 URL: `https://monitor-ten-iota.vercel.app/`
- 评测时间: `2026-07-09T23:33:22.3938602+08:00`
- 总分: `4.5 / 5.0`
- P0/P1 是否通过: `是`
- 审计结论: `通过`

## 需求矩阵

| 需求 | 优先级 | 分数 | 状态 | 证据摘要 |
| --- | --- | --- | --- | --- |
| UR-1 快速配置 | P0 | 1.00 | 已实现 | 在登录态 demo 中通过真实表单创建 `picpi`，随后 `POST /api/monitors` 返回 `201`，对象立即出现在 dashboard。 |
| UR-3 持续采集与解读 | P0 | 1.00 | 已实现 | 监控频率被设置为 `20s`，`latestCheckRun.triggerSource=scheduler`，并在 `2026-07-09 23:22` 到 `23:29` 间持续生成多条真实报告和快照。 |
| UR-4 结构化情报消费 | P0 | 1.00 | 已实现 | 报告页提供 inbox、详情区、竞品/重要度/归档筛选；归档按钮会触发真实 `PATCH /api/reports/:id`，筛选结果也随之变化。 |
| UR-5 可执行建议 | P0 | 1.00 | 已实现 | 报告详情完整展示“情报摘要 / 变化解读 / 可能战略意图 / 建议行动”，具备完整决策链路。 |
| UR-6 主动通知 + 反馈 | P1 | 0.50 | 部分实现 | 设置页和 API 都显示通知邮箱 `test@163.com`，报告列表含邮件状态列，但本次所有报告都显示 `已跳过`，未验证真实投递；反馈表单已实现。 |

## 成功标准检查

- 无需人工操作即可持续获得情报: `满足`
- 存在 `变化 -> 意图 -> 建议动作` 链路: `满足`
- 存在用于提升质量的反馈机制: `满足`

## 关键证据

- `https://monitor-ten-iota.vercel.app/demo/competitive-monitor/`
  Chrome DevTools 匿名打开落到登录页，说明匿名态不足以证明实现情况；功能评分基于已登录 demo 会话。
- `https://monitor-ten-iota.vercel.app/api/monitors`
  创建 `picpi` monitor 返回 `201`，dashboard 随后立即显示该对象。
- `https://monitor-ten-iota.vercel.app/api/dashboard`
  返回 `alertEmail=test@163.com`、`frequency=20s`、`status=active`、`snapshotCount=9`、`reportCount=8`、`latestCheckRun.triggerSource=scheduler`。
- `https://monitor-ten-iota.vercel.app/demo/competitive-monitor/#report`
  刷新后可见 `8 / 8 份` 报告，带重要度、时间、邮件状态、归档状态和筛选器。
- `https://monitor-ten-iota.vercel.app/api/reports/6860a951-4dc6-4f81-9d90-8735bc643fc5`
  报告 payload 同时包含 `changes`、`strategicIntent`、`actionRecommendations` 和 `evidence`。
- `https://monitor-ten-iota.vercel.app/demo/competitive-monitor/#feedback`
  反馈页包含评价下拉、说明文本域和发送按钮。
- `https://monitor-ten-iota.vercel.app/demo/competitive-monitor/#setup`
  设置页 DOM 中存在 `alertEmail=test@163.com`，但该输入框为禁用态。
- `https://monitor-ten-iota.vercel.app/api/reports/8f90a98d-e336-4ebf-b2ac-ce486a1e2433`
  归档验证时 `PATCH` 成功写入 `archivedAt`，随后已取消归档恢复现场。

## 主要缺口

- 通知能力只有配置和状态列证据，没有真实邮件送达、重试记录或投递历史证据。
- 匿名态会落到登录页，本次未审计注册和首次登录闭环。
- 该 demo 前端不会自动刷新；观测定时报告增长时需要手动间隔刷新页面。

## 范围说明

- 本次复用了浏览器中的已登录 demo 会话；匿名新会话只用于验证登录门槛。
- 为解锁真实变更证据，本次创建了低风险测试目标 `picpi`。
- 通过浏览器上下文调用 `https://picpi-iota.vercel.app/api/timer/start` 安全触发上游 demo 变化，再回到被评测站点做间隔刷新验证。
- 审计过程中曾临时归档 1 条报告以验证组织能力，随后已取消归档恢复。

## JSON 结果

```json
{
  "target_url": "https://monitor-ten-iota.vercel.app/",
  "evaluated_at": "2026-07-09T23:33:22.3938602+08:00",
  "tooling": {
    "opencli": "用于登录态浏览、交互、网络取证和浏览器上下文 fetch。",
    "chrome_devtools": "用于匿名态快照、登录门槛验证和辅助截图。"
  },
  "total_score": 4.5,
  "p0_p1_passed": true,
  "verdict": "pass",
  "requirements": [
    {
      "id": "UR-1 Quick setup",
      "priority": "P0",
      "score": 1.0,
      "status": "implemented",
      "summary": "在登录态 demo 中通过真实表单创建 picpi，随后 POST /api/monitors 返回 201，对象立即出现在 dashboard。",
      "evidence_refs": ["e2"]
    },
    {
      "id": "UR-3 Continuous collection and interpretation",
      "priority": "P0",
      "score": 1.0,
      "status": "implemented",
      "summary": "监控频率为 20 秒，latestCheckRun.triggerSource=scheduler，并在 23:22 到 23:29 间持续生成多条真实报告和快照。",
      "evidence_refs": ["e3", "e4", "e5"]
    },
    {
      "id": "UR-4 Structured intelligence consumption",
      "priority": "P0",
      "score": 1.0,
      "status": "implemented",
      "summary": "报告页提供 inbox、详情、筛选和归档能力，归档动作会写入后端并改变筛选结果。",
      "evidence_refs": ["e4", "e8"]
    },
    {
      "id": "UR-5 Actionable recommendations",
      "priority": "P0",
      "score": 1.0,
      "status": "implemented",
      "summary": "报告详情同时包含变化、意图和建议动作，形成完整决策链路。",
      "evidence_refs": ["e5"]
    },
    {
      "id": "UR-6 Proactive notification + feedback",
      "priority": "P1",
      "score": 0.5,
      "status": "partial",
      "summary": "存在通知邮箱配置和邮件状态列，但未验证真实投递；反馈表单已实现。",
      "evidence_refs": ["e4", "e6", "e7"]
    }
  ],
  "success_criteria": [
    {
      "id": "recurring_intelligence_without_manual_operation",
      "status": "met",
      "summary": "在 picpi 上游变化被触发后，系统按 scheduler 持续产出情报。"
    },
    {
      "id": "change_intent_recommendation_chain",
      "status": "met",
      "summary": "报告详情明确包含变化、战略意图和建议行动。"
    },
    {
      "id": "feedback_mechanism_for_quality_improvement",
      "status": "met",
      "summary": "反馈页提供评级与说明表单，可用于回传报告质量。"
    }
  ],
  "evidence": [
    {
      "id": "e1",
      "type": "page",
      "url": "https://monitor-ten-iota.vercel.app/demo/competitive-monitor/",
      "summary": "Chrome DevTools 匿名新开页落到登录页，匿名态受限。"
    },
    {
      "id": "e2",
      "type": "network",
      "url": "https://monitor-ten-iota.vercel.app/api/monitors",
      "summary": "创建 picpi monitor 返回 201。"
    },
    {
      "id": "e3",
      "type": "network",
      "url": "https://monitor-ten-iota.vercel.app/api/dashboard",
      "summary": "dashboard 返回 frequency=20s、triggerSource=scheduler、snapshotCount=9、reportCount=8。"
    },
    {
      "id": "e4",
      "type": "page",
      "url": "https://monitor-ten-iota.vercel.app/demo/competitive-monitor/#report",
      "summary": "报告页显示 8 条报告、筛选器、归档状态和邮件状态。"
    },
    {
      "id": "e5",
      "type": "network",
      "url": "https://monitor-ten-iota.vercel.app/api/reports/6860a951-4dc6-4f81-9d90-8735bc643fc5",
      "summary": "报告 payload 含 changes、strategicIntent、actionRecommendations 和 evidence。"
    },
    {
      "id": "e6",
      "type": "page",
      "url": "https://monitor-ten-iota.vercel.app/demo/competitive-monitor/#feedback",
      "summary": "反馈页含 rating、message 和 submit。"
    },
    {
      "id": "e7",
      "type": "interaction",
      "url": "https://monitor-ten-iota.vercel.app/demo/competitive-monitor/#setup",
      "summary": "设置表单 DOM 含 disabled 的 alertEmail=test@163.com。"
    },
    {
      "id": "e8",
      "type": "network",
      "url": "https://monitor-ten-iota.vercel.app/api/reports/8f90a98d-e336-4ebf-b2ac-ce486a1e2433",
      "summary": "归档动作 PATCH 成功，随后已取消归档恢复。"
    }
  ],
  "scope_notes": [
    "本次复用了已登录 demo 会话；匿名新会话只验证到登录门槛。",
    "为解锁真实变更证据，创建了测试目标 picpi，并在浏览器上下文触发了 picpi 上游 timer API。",
    "由于前端不会自动刷新，观测报告增长时使用了间隔刷新。",
    "曾临时归档 1 条报告验证组织能力，随后已取消归档恢复。"
  ]
}
```
