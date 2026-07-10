# 功能完整度审计

## 摘要

- 目标 URL: `https://xierongpan.xyz`
- 评测时间: `2026-07-10 07:52:32 +08:00`
- 总分: `3.15 / 5.0`
- P0/P1 通过: `false`
- 结论: `partial`

一句话结论：登录后可完成建项、启用监控、查看原始事件、配置通知与反馈闭环骨架，但“原始信号 -> 结构化情报 -> 可执行建议”的核心闭环没有打通，`应生成情报` 还会返回 500。

## 需求矩阵

| 需求 | 优先级 | 分数 | 状态 | 证据摘要 |
| --- | --- | --- | --- | --- |
| UR-1 快速建项 | P0 | 0.85 | implemented | 已验证从空项目态创建 `Audit Picpi`、确认 `picpi` 目标、启用监控；通知页还能非破坏性关闭并恢复定时巡检。但项目列表仍显示默认 `https://acme.example/`，竞品确认页也未回显既有竞品，存在配置一致性问题。 |
| UR-3 持续采集与解释 | P0 | 0.50 | partial | 启用监控返回调度信息，通知页可见每日 `08:10` 定时巡检；手动巡检后总览从“尚未生成”变成“生成于 2026-07-10 07:48”，原始事件账本出现了真实快照信号。但抓取长期停留在“排队中”，没有形成持续、自动的解释性输出。 |
| UR-4 结构化情报消费 | P0 | 0.55 | partial | 收件箱页有优先级筛选和“包含已归档”开关；原始事件页有账本和证据抽屉，能下钻到来源页、摘录和变化说明。但真实情报卡片没有生成，收件箱工作流未闭环。 |
| UR-5 可执行建议 | P0 | 0.25 | claimed | 产品界面声称有“角色化决策视图”，但实际没有主题、没有建议、没有 `change -> intent -> recommendation` 输出；从原始事件生成情报的入口还触发了 500。 |
| UR-6 主动通知 + 反馈 | P1 | 1.00 | implemented | 通知页已实现收件邮箱、频率、时区、低优先级开关、定时巡检和邮件预览；质量页实现反馈统计、重评估和偏好闭环，即使当前记录为 0。 |

## 成功标准检查

- 无需手工操作即可持续收到有价值情报：`partial`
  已验证定时巡检配置与手动巡检入口，但未验证稳定产生可消费情报。
- `change -> intent -> recommendation` 链路：`unmet`
  只拿到了原始页面快照，没有成功生成建议型情报。
- 反馈机制可用于改进情报质量：`met`
  质量后台提供历史反馈重应用、待验证/已生效/已拒绝等统计面。

## 关键证据

- `E1` 匿名访问 `https://xierongpan.xyz/login?next=%2F` 会直接进入登录墙，登录页提示“请输入账号密码”，说明核心产品面默认受鉴权保护。另有截图 `docs/谢荣攀/login-wall.png`。
- `E2` 登录后的 `https://xierongpan.xyz/projects` 是空项目态，只有“新建项目”可用，其余监控导航均被禁用。
- `E3` `https://xierongpan.xyz/projects/project_cbf16b8f-5436-4d89-90d6-6e2d3232a755/setup/scope` 显示已确认竞品 `https://picpi-iota.vercel.app/`，并标注“已纳入真实巡检”“已抓取”。
- `E4` `POST https://xierongpan.xyz/api/monitoring/enable` 返回 200，响应结构包含 `project.notificationPreference`、`scanSchedule` 和 `scheduler.event`，证明启用监控会落持久化配置并注册调度。
- `E5` `https://xierongpan.xyz/projects/project_cbf16b8f-5436-4d89-90d6-6e2d3232a755/notifications` 展示收件邮箱、通知频率、时区、低优先级开关、定时巡检与邮件预览。
- `E6` 我实测将通知页“启用定时巡检”关闭到“已停用”，随后再恢复为“已启用”；两次保存都命中了 `POST https://xierongpan.xyz/api/notifications/settings` 200，说明存在非删除的停用/恢复路径。
- `E7` 在触发 `picpi` timer 并执行手动巡检后，重开 `https://xierongpan.xyz/projects/project_cbf16b8f-5436-4d89-90d6-6e2d3232a755/overview`，页头从“尚未生成态势总览”变成“生成于 2026-07-10 07:48”，证据基础从 `0 条` 变为 `1 条`。
- `E8` `https://xierongpan.xyz/projects/project_cbf16b8f-5436-4d89-90d6-6e2d3232a755/raw-events` 记录了 `baseline_initial_markdown_snapshot`；证据抽屉显示来源页、抓取时间、页面摘录、变化说明和完整页面摘录。
- `E9` 在原始事件页点击“应生成情报”后，`POST https://xierongpan.xyz/api/raw-events/reevaluate` 返回 500，页面状态提示 `Invalid re-evaluation request`，说明原始信号向情报转换的核心链路异常。
- `E10` `https://xierongpan.xyz/projects/project_cbf16b8f-5436-4d89-90d6-6e2d3232a755/inbox` 有优先级筛选与归档开关，但刷新后仍显示“没有符合当前视图的情报”。
- `E11` `https://xierongpan.xyz/projects/project_cbf16b8f-5436-4d89-90d6-6e2d3232a755/quality` 提供“重新应用历史反馈”、反馈总数、待验证反馈、已生效偏好、已拒绝反馈等反馈闭环指标。
- `E12` 我用 Chrome DevTools 打开 `https://picpi-iota.vercel.app/`，快照可见功能、定价、更新日志、招聘、API/Webhook 等真实内容，与原始事件证据抽屉中的页面摘录一致。另有截图 `docs/谢荣攀/picpi-devtools.png`。

## 缺口

- `UR-5` 未达标：没有成功生成任何“变化 -> 意图 -> 建议”的情报卡片。
- `UR-3` 证据不足：虽然有调度和手动巡检，但抓取长期停在“排队中”，未证明稳定的持续采集循环。
- `UR-4` 半成品：收件箱结构存在，但没有真实条目，也无法验证详情、归档后的再消费链路。
- `原始事件 -> 情报` 是明确故障点：`reevaluate` 返回 500，直接阻断了高价值闭环。
- 配置一致性存在问题：项目列表仍显示默认 `ownProductUrl`，竞品确认页没有回显已经纳入巡检的竞品。

## 范围说明

- 本次审计默认从匿名态开始；由于被登录墙拦截，核心功能均在登录态内验证。
- 为解锁真实证据，我创建了一个最小测试夹具：项目名 `Audit Picpi`，监控目标 `https://picpi-iota.vercel.app/`。
- 为验证非破坏性暂停路径，我临时关闭并恢复了“启用定时巡检”开关，最终状态已恢复为 `已启用`。
- 为制造真实上游变化，我在浏览器上下文中触发了 `picpi` 的 `/api/timer/start`；但在观察窗口内，产品仍未把原始信号成功提升为可消费情报。
- `opencli browser network --detail` 受本机缓存权限问题影响，未能展开部分请求的完整体；因此对 `reevaluate` 的失败原因只依据 500 状态码和页面错误提示下结论。
- Chrome DevTools 与登录态产品页不共享同一会话，因此 DevTools 证据主要用于核对被监控的公共目标页，而非登录态应用页。

## JSON 结果

```json
{
  "target_url": "https://xierongpan.xyz",
  "evaluated_at": "2026-07-10 07:52:32 +08:00",
  "tooling": {
    "opencli": "available",
    "chrome_devtools": "available"
  },
  "total_score": 3.15,
  "p0_p1_passed": false,
  "verdict": "partial",
  "requirements": [
    {
      "id": "UR-1 Quick setup",
      "priority": "P0",
      "score": 0.85,
      "status": "implemented",
      "summary": "已验证从空项目态创建 Audit Picpi、确认 picpi 目标、启用监控；通知页还能非破坏性关闭并恢复定时巡检。但项目列表 ownProductUrl 仍显示默认值，竞品确认页也未回显既有竞品。",
      "evidence_refs": ["E2", "E3", "E4", "E6"]
    },
    {
      "id": "UR-3 Continuous collection and interpretation",
      "priority": "P0",
      "score": 0.5,
      "status": "partial",
      "summary": "已验证调度配置、手动巡检入口和原始事件落库，总览也出现生成时间与 1 条证据；但抓取停留在排队中，没有形成稳定的持续解释输出。",
      "evidence_refs": ["E4", "E5", "E7", "E8", "E12"]
    },
    {
      "id": "UR-4 Structured intelligence consumption",
      "priority": "P0",
      "score": 0.55,
      "status": "partial",
      "summary": "收件箱结构、优先级筛选、归档开关、原始事件账本和证据抽屉都存在；但真实情报条目没有生成，收件箱工作流未闭环。",
      "evidence_refs": ["E8", "E10"]
    },
    {
      "id": "UR-5 Actionable recommendations",
      "priority": "P0",
      "score": 0.25,
      "status": "claimed",
      "summary": "只有角色化决策视图的壳和空态文案，没有成功产出建议；从原始事件生成情报的接口返回 500。",
      "evidence_refs": ["E7", "E9"]
    },
    {
      "id": "UR-6 Proactive notification + feedback",
      "priority": "P1",
      "score": 1.0,
      "status": "implemented",
      "summary": "通知设置、频率、时区、定时巡检、邮件预览和反馈后台都可见且可保存，已构成主动通知与反馈机制。",
      "evidence_refs": ["E5", "E6", "E11"]
    }
  ],
  "success_criteria": [
    {
      "id": "SC-1 Recurring intelligence without manual operation",
      "status": "partial",
      "summary": "已验证定时巡检配置与手动巡检，但未验证系统能稳定地产出有价值情报。"
    },
    {
      "id": "SC-2 change -> intent -> recommendation chain",
      "status": "unmet",
      "summary": "只落下了原始页面快照，没有成功转成带意图和建议的情报。"
    },
    {
      "id": "SC-3 Feedback mechanism for quality improvement",
      "status": "met",
      "summary": "质量后台存在历史反馈重应用、待验证、已生效和已拒绝等反馈闭环统计。"
    }
  ],
  "evidence": [
    {
      "id": "E1",
      "type": "page",
      "url": "https://xierongpan.xyz/login?next=%2F",
      "summary": "匿名访问被登录墙拦截，登录页提示“请输入账号密码”。"
    },
    {
      "id": "E2",
      "type": "page",
      "url": "https://xierongpan.xyz/projects",
      "summary": "登录后的项目列表为空态，只有新建项目可用，其余监控导航均被禁用。"
    },
    {
      "id": "E3",
      "type": "interaction",
      "url": "https://xierongpan.xyz/projects/project_cbf16b8f-5436-4d89-90d6-6e2d3232a755/setup/scope",
      "summary": "最小夹具 Audit Picpi 创建成功，范围页显示 picpi 已确认且已纳入真实巡检。"
    },
    {
      "id": "E4",
      "type": "network",
      "url": "https://xierongpan.xyz/api/monitoring/enable",
      "summary": "启用监控请求 200，响应包含 notificationPreference、scanSchedule 和 scheduler.event。"
    },
    {
      "id": "E5",
      "type": "page",
      "url": "https://xierongpan.xyz/projects/project_cbf16b8f-5436-4d89-90d6-6e2d3232a755/notifications",
      "summary": "通知页提供收件邮箱、时区、频率、低优先级开关、定时巡检和邮件预览。"
    },
    {
      "id": "E6",
      "type": "interaction",
      "url": "https://xierongpan.xyz/api/notifications/settings",
      "summary": "实测可将定时巡检从已启用切到已停用，再恢复为已启用；两次保存均返回 200。"
    },
    {
      "id": "E7",
      "type": "page",
      "url": "https://xierongpan.xyz/projects/project_cbf16b8f-5436-4d89-90d6-6e2d3232a755/overview",
      "summary": "手动巡检并显式刷新后，总览显示生成于 2026-07-10 07:48，证据基础变为 1 条。"
    },
    {
      "id": "E8",
      "type": "page",
      "url": "https://xierongpan.xyz/projects/project_cbf16b8f-5436-4d89-90d6-6e2d3232a755/raw-events",
      "summary": "原始事件页保留 baseline_initial_markdown_snapshot，并能展开证据抽屉查看来源页、摘录和变化说明。"
    },
    {
      "id": "E9",
      "type": "network",
      "url": "https://xierongpan.xyz/api/raw-events/reevaluate",
      "summary": "点击“应生成情报”后返回 500，页面状态提示 Invalid re-evaluation request。"
    },
    {
      "id": "E10",
      "type": "page",
      "url": "https://xierongpan.xyz/projects/project_cbf16b8f-5436-4d89-90d6-6e2d3232a755/inbox",
      "summary": "收件箱有优先级筛选和归档开关，但没有任何情报条目。"
    },
    {
      "id": "E11",
      "type": "page",
      "url": "https://xierongpan.xyz/projects/project_cbf16b8f-5436-4d89-90d6-6e2d3232a755/quality",
      "summary": "质量后台提供反馈统计、重评估入口和已验证偏好区域，构成反馈闭环骨架。"
    },
    {
      "id": "E12",
      "type": "screenshot",
      "url": "https://picpi-iota.vercel.app/",
      "summary": "Chrome DevTools 快照和截图验证被监控目标页包含功能、定价、更新日志、招聘与 API/Webhook 内容，与原始事件摘录一致。"
    }
  ],
  "scope_notes": [
    "核心产品功能在登录态内完成审计；匿名态仅验证了登录墙。",
    "本次创建的最小测试夹具为 Audit Picpi，监控目标为 https://picpi-iota.vercel.app/。",
    "为验证非破坏性停用路径，曾短暂关闭并恢复定时巡检，最终状态已恢复为已启用。",
    "已在浏览器上下文触发 picpi timer API，但在观察窗口内没有产出结构化情报卡片。",
    "opencli 的网络缓存权限异常导致部分请求体无法展开，因此对 reevaluate 失败原因的判断只依据 500 状态和页面提示。"
  ]
}
```
