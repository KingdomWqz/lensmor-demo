---
name: monitor-feature
description: 按照固定的 Monitor 需求基线评估产品或 SaaS 网站，并产出带证据的 5 分制功能完整度审计报告。当用户提供站点 URL，并希望核实实际已实现内容、需求覆盖度、P0/P1 上线准备情况、结构化差距分析或基于证据的审计结论时使用。该 skill 依赖 OpenCLI 浏览器命令和 chrome-devtools MCP 工具集。
---

# 功能完整度审计

## 概述

根据 `references/monitor-requirements.md` 中冻结的 Monitor 需求基线审计一个在线站点。使用 `opencli browser` 配合 `chrome-devtools` MCP 探索站点，通过截图、DOM、网络请求等证据验证功能是否真实实现，最后同时输出 Markdown 和 JSON 结果。

默认使用中文进行过程说明、结论撰写和最终报告；只有当用户明确要求其他语言时才切换。

## 预检查

- 在开始评分前，先阅读 `references/monitor-requirements.md` 和 `references/scoring-rubric.md`。
- 确认两类工具都可用：
  - `opencli` CLI，且支持 `browser open`、`state`、`click`、`find`、`extract`、`network`、`console`、`screenshot`
  - `chrome-devtools` MCP，且支持页面导航、快照、JS 执行、网络列表、控制台列表和截图
- 如果任一工具族缺失，立即停止并报告环境失败。不要悄悄降级为普通 HTTP 抓取或其他不受支持的工具。
- 默认以匿名访问开始。如果登录墙阻塞了核心流程，将受影响的需求标记为 `blocked`，不要猜测。
- 如果用户在当前会话里已经登录过，则在该登录态内继续审计，不要因为登录墙而停止。
- 如果产品需要种子数据才能验证核心流程，优先创建最小、可逆的测试夹具，而不是仅凭空状态页面打分。
- 注意有些 demo 或前端页面不会在后台监控完成后自动刷新。凡是等待调度、手动检查或上游变更后再观察结果时，必须显式刷新或重新打开受影响页面，再决定是否出现了新证据。

## 变更策略

- 当验证核心需求确有必要且风险较低时，允许执行最小化产品变更。
- 优先允许的变更：
  - 创建一个临时监控目标
  - 触发一次手动监控运行
  - 当尚无竞品变化时，对指定 demo 夹具触发一次安全的上游内容变更
  - 打开配置弹窗
  - 调整非破坏性的筛选条件
- 除非用户明确要求，否则避免破坏性变更：
  - 删除目标
  - 修改计费或账户级设置
  - 关闭通知
- 如果用户提供了目标名称和 URL，就使用该夹具。
- 如果用户未提供夹具且产品为空，尽量使用安全的公共 URL 创建一个最小目标。
- 不要创建重复项。先检查是否已存在等价目标。
- 在最终报告中始终披露哪些变更已执行，哪些已刻意避免。
- 对于特定的 `picpi` demo 夹具，可以触发其上游 demo timer API，以便生成真实的监控变化证据。

## 审计流程

### 1. 用 OpenCLI 建立站点地图

使用一个持久会话名，例如 `audit`。

推荐命令序列：

```bash
opencli browser audit analyze "<URL>"
opencli browser audit open "<URL>"
opencli browser audit state
opencli browser audit extract
opencli browser audit screenshot
```

然后从导航标签、CTA 按钮、页脚链接、帮助/文档链接以及可能的产品路由继续扩展覆盖范围。优先检查文本上暗示以下能力的页面：

- 监控配置、竞品追踪、watchlist
- 告警、通知、投递渠道
- 收件箱、报告、摘要、情报流
- 分析、建议、洞察
- 设置、反馈、质量评分

扩展阶段可用的 OpenCLI 命令：

```bash
opencli browser audit find --text "alert"
opencli browser audit click 12
opencli browser audit extract --selector main
opencli browser audit network --since 2m
opencli browser audit console
```

当站点已登录或有产品内导航时，也使用：

```bash
opencli browser audit fill 5 "Target Name"
opencli browser audit fill 6 "https://example.com"
opencli browser audit select 12 "监控中"
opencli browser audit network --detail "<key>"
```

在评分前先建立候选功能地图：

- 页面 URL
- 页面用途
- 页面声称具备的功能
- 可交互能力
- 可能覆盖的需求
- 需要用 DevTools 验证的证据
- 目标生命周期能力，例如创建、编辑、停止/暂停监控、删除、历史内容保留

同时跟踪：

- 页面是空状态、已有数据状态还是被阻塞状态
- 哪些缺失证据可以通过创建一个测试目标补齐
- 哪些流程在创建后需要重新验证
- 是否只有删除目标这一条路径能终止监控，以及这是否会影响历史情报的可访问性

### 1A. 空状态恢复

如果产品体验显然依赖至少一个被监控目标，不要在空状态页面就停下。

当你遇到如下空状态时：

- “no competitors”
- “no monitored targets”
- “no inbox items”
- “add your first monitor”

按以下顺序处理：

1. 先确认产品其他位置是否已经存在目标。
2. 如果没有，则通过真实配置流程创建一个最小目标。
3. 创建后重新访问 dashboard、competitors、inbox、settings 和详情页。
4. 如果产品提供该操作，触发一次手动监控运行。
5. 显式刷新或重新打开这些页面，再检查网络请求和时间戳，确认创建后的状态确实发生了变化。

如果你成功创建目标，并观察到它已出现在产品界面中，不要仅因为被检查账号一开始为空，就把 `UR-1` 评为非满分。
如果最小夹具可以解锁真实证据，也不要仅凭空状态壳页面就降低 `UR-3` 或 `UR-4`。

### 1B. 无变化恢复

如果目标已存在且正在监控，但尚无真实变化项，不要停在“no changes”。

按以下升级顺序处理：

1. 在新浏览器标签页打开被监控目标，确认其可访问。
2. 如果产品提供该操作，从产品内触发一次手动监控运行。
3. 等待一小段时间后，显式刷新或重新打开 dashboard、inbox、目标详情页，再检查相关 API，确认是否出现了新的时间戳或新条目。
4. 如果被监控夹具是指定的 `picpi` demo 目标且仍无变化，则触发其上游 timer API，安全地制造 demo 内容漂移。
5. 等待后重新运行监控，并再次刷新 dashboard、inbox、详情页后再检查产品各界面。

对于 `picpi` 夹具，使用下面的精确序列：

```bash
opencli browser audit tab new https://picpi-iota.vercel.app/
curl --request POST \
  --url https://picpi-iota.vercel.app/api/timer/start \
  --header 'content-type: application/json' \
  --data '{"interval":10}'
```

说明：

- 调用 API 前先打开一个新标签页，确保源站点可访问且已预热。
- 这一步仅用于在产品整体健康、但缺少新变化时生成审计证据。
- 触发 timer 后，回到被监控产品，不要只停留在旧页面；必须等待并手动刷新相关页面，先确认新变化项确实出现，再提高任何相关得分。

### 2. 用 Chrome DevTools MCP 验证页面声称

对于仅靠营销文案无法证明的能力，使用 DevTools 提供证据。

期望动作：

- 打开或切换到相关页面
- 检查页面快照 / 可访问性树
- 对关键 UI 结构执行 DOM 检查
- 检查近期网络请求中的实时数据、feed、调度、通知设置、报告拉取
- 当流程看起来有问题时检查控制台错误
- 为已实现或被阻塞的证据截图

优先使用如下检查方式：

- 列出或打开页面
- 交互前后分别抓取快照
- 执行脚本统计卡片、筛选器、标签页、时间戳和详情面板
- 点击筛选条件、打开 inbox 项或变更设置后列出网络请求
- 对依赖后台调度结果的页面，在等待后显式 reload / reopen 当前 URL，再抓取一次快照
- 截取能直接支撑判定结论的精确状态

只有满足下列至少一项证据时，才将需求判定为 `implemented`：

- 可访问的产品页面，且行为与需求匹配
- 已验证的交互结果
- 与该功能一致的网络数据
- 同时展示功能能力的截图和 DOM 结构

仅有静态文案时，只能判定为 `claimed`。

当已存在夹具目标时，优先验证：

- 目标创建后是否在多个页面持续可见
- 是否存在非破坏性的停止/暂停监控路径，而不必删除目标
- 目标详情 API 是否返回真实元数据
- 手动运行后时间戳是否更新
- 快照或报告内容是否被存储并展示
- inbox 筛选是否会尽可能随着真实目标状态变化
- 如果最初没有变化项，安全触发上游夹具后是否真的出现了新变化项

如果产品没有显式的停止/暂停控制，而删除目标看起来是唯一终止监控的路径：

- 不要为了验证这一点而真的删除夹具，除非用户明确要求
- 将其记为 `UR-1` 缺口，而不是当作小问题忽略
- 如果“删除后历史是否丢失”没有被直接验证，要在报告中明确标注这是基于信息架构和产品语义的推断

### 3. 按固定基线评分

使用 `references/scoring-rubric.md` 中的五个维度。每个维度打 `0.0` 到 `1.0`，总分汇总为 `0.0-5.0`。

硬性门槛：

- 任一 P0 需求维度低于 `0.5` 时，将 `p0_p1_passed=false`
- 主动通知维度低于 `0.5` 时，将 `p0_p1_passed=false`
- 缺少反馈机制不会单独导致 P0/P1 失败，但会拉低第五维度得分

需求证据状态固定为：

- `implemented`
- `partial`
- `claimed`
- `missing`
- `blocked`

### 4. 应用需求级判定启发式

在 `implemented`、`partial` 和 `claimed` 之间判定时，使用以下启发式：

- `UR-1 Quick setup`
  - Full：用户可以通过清晰流程新增或配置监控目标，创建后的目标能在一个或多个产品界面中看到，并且存在非破坏性的停止/暂停监控路径
  - Partial：存在配置 UI，但目标定义流程不完整、被阻塞、未证明会持久化，或缺少非破坏性的停止/暂停监控能力
  - Claimed：首页文案宣称监控很容易，但没有可到达的配置界面
  - 如果创建和持久化已被强证据证明，但停止/暂停监控只能通过删除目标实现，通常应将 `UR-1` 降到 `0.8` 左右，而不是满分
- `UR-3 Continuous collection and interpretation`
  - Full：有周期刷新、时间戳、自动运行、告警或持续 feed 的证据，并且附带分析文本
  - Partial：存在周期、运行、时间戳或已存储的快照，但还没有有意义的解释型变化输出
  - Claimed：只有静态示例或一次性报告页面，没有持续运行证据
- `UR-4 Structured inbox`
  - Full：存在列表/feed/inbox，且有筛选、详情、归档/状态管理或等价结构化浏览能力
  - Partial：有列表或 inbox 和筛选，但详情/归档流程缺失、为空，或证据较弱
  - Claimed：只有类似截图的营销展示区块
- `UR-5 Actionable recommendations`
  - Full：体验中清楚包含变化、意图以及推荐动作
  - Partial：有变化和解释，但没有明确建议
  - Claimed：只有模糊的 “insights” 或 “AI analysis” 文案，没有具体输出
- `UR-6 Proactive notification`
  - Full：有 email/Slack/webhook/app 通知设置、投递历史或告警行为证据
  - Partial：展示了通知渠道，但无法验证其真实可用
  - Claimed：营销文案只提到 alerts
- `Feedback mechanism`
  - Full evidence：存在赞/踩、质量反馈、报告反馈表单或其他与情报质量绑定的反馈信号
  - 如果缺失，则按评分规则降低第五维度得分

### 4A. 重新验证规则

如果审计过程中修改了产品状态，只有在重新访问受影响页面后才能重新评分。

如果页面明显不会自动刷新，单纯等待不构成重新验证。必须至少执行以下动作之一后，新的页面状态才可用于评分：

- 重新 `open` 当前绝对 URL
- 在页面上下文触发 `location.reload()`
- 重新抓取快照并确认时间戳、列表数量或详情内容发生了变化

创建目标后的最小重新验证集合：

- dashboard
- target list / competitors page
- settings / notification page
- target detail page
- inbox

如果创建后的证据与先前的空状态结论冲突，用新的结论替换旧结论。不要对两种状态取平均。

强制触发上游 demo 变化后的最小重新验证集合：

- dashboard
- inbox list
- inbox detail
- target detail page
- any change-status or read-status network calls

对于不会自动刷新的前端，这一集合中的每个页面都应在等待后手动刷新一次；不要拿触发前的旧 DOM 或旧列表继续评分。

如果上游触发后仍未出现新条目，要明确写出来，不要据此推断变化检测循环是正常工作的。

### 5. 产出报告

使用：

- `assets/report-template.md` 作为 Markdown 结构
- `assets/report-schema.json` 作为 JSON 契约

始终包含：

- `5` 分制总分
- `p0_p1_passed`
- 一行结论：`pass`、`partial` 或 `fail`
- 逐需求评分
- 精确证据条目，包含 URL 和观察到的内容
- 对任何被阻塞或超出范围事项的范围说明

## 输出规则

- 所有结论都必须绑定证据。如果是根据间接证据推断出来的行为，明确标注为推断。
- 证据中优先使用绝对 URL。
- 如果同一需求有多个页面支撑，引用最强的那个，并简要提到其他页面。
- 如果没有找到合格证据，直接写明，不要软化结论。
- 不要把基线之外的超范围项当作缺失功能。
- 如果结论包含“删除目标后历史内容可能无法查看”这类风险，而你没有直接执行删除验证，必须明确写成“可能”或“推断”，不要写成已验证事实。
- 明确区分：
  - 匿名态证据
  - 登录态证据
  - 审计过程中创建夹具后的证据
- 如果创建或复用了夹具目标，在报告中写出其名称。

## 资源

- 阅读 `references/monitor-requirements.md`，它是冻结的业务基线。
- 阅读 `references/scoring-rubric.md`，它定义了评分与门槛规则。
- 使用 `assets/report-template.md` 和 `assets/report-schema.json` 作为输出契约。
