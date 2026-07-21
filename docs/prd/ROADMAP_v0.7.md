# KPi 实施路线图（ROADMAP）

> **文档版本**：0.7-draft  
> **文档日期**：2026-07-21  
> **对应 PRD**：`PRD_v0.7.md`  
> **路线原则**：**OMP-first → KPi CLI → KPi Core → 双 Runtime → 必要时受控 Fork**  
> **SBTD 基线**：`KunoLu/640-skills main @ 340f9dd4dc7a92e8b91c31e111de9a8de06cef36`  
> **说明**：P0～P3 是按依赖顺序排列的实施优先级，不是并行开发泳道，也不是缺陷等级。  
> **0.7 更新重点**：将项目规则拆为根 `AGENTS.md` 跨 Agent 事实层与 `.omp/AGENTS.md` OMP/KPi Adapter；增加 `@../AGENTS.md` 导入、双项目 Managed Block、Monorepo nearest-native 检测、三目标上游同步，以及由统一 Command Registry 生成的 `/sbtd help [command]`。

---

## 1. 路线总览

| 阶段 | 建议版本 | 核心路线 | 主要交付物 | 不做 |
|---|---|---|---|---|
| **P0** | `v0.1-alpha` | OMP-hosted SBTD MVP | Plugin、三层 AGENTS、`/sbtd help/on/off`、核心 Gate 与 Report | 不 Fork、不重写 Provider/Tool |
| **P1** | `v0.1-beta` → `v0.1` | KPi CLI 产品化 | `kpi`、Onboard/Provider/Session/Report CLI | 不抽离全部 Core |
| **P2** | `v0.2` | KPi Core 抽离 | Workflow/Rule/Policy/Validation/Provider/Core、TS Onboard | 不承诺双 Runtime 完整等价 |
| **P3** | `v0.3` | OMP/Pi 双 Runtime | `runtime-omp`、`runtime-pi`、兼容套件、Fork Gate | 未通过 Gate 不 Fork |

依赖关系：

```text
P0 OMP Plugin
  ↓
P1 KPi CLI
  ↓
P2 KPi Core
  ↓
P3 OMP/Pi Dual Runtime
  ↓
Controlled Fork（仅在必要时）
```

---

# P0 — OMP-hosted SBTD MVP

## 2. P0 目标

以最小的底层自研成本，把 `640-skills` 中最关键的 SBTD 行为转换为 OMP 可执行插件，验证：

1. `/sbtd on` 是否能稳定改变后续每轮的工作流；
2. Book Gate、BDD、Trellis、GitNexus、Validation 与 Report 能否被结构化执行；
3. Dormant Rule 是否能减少模型偏航；
4. Session/Compaction/Resume 后状态是否连续；
5. SBTD 是否提升真实 Coding Task 的正确性，而不是只增加仪式。

## 3. P0 产品形态

```text
npm package: @kpi/omp-sbtd

OMP
  └── plugin
      ├── command registry + /sbtd help
      ├── extensions/hooks
      ├── tools
      ├── rules
      ├── state
      ├── report renderer
      └── pinned read-only SBTD kit
          ├── catalog/schema
          ├── onboard.py
          ├── Global / Root Project / OMP Project Adapter templates
          ├── bundled Skills
          └── stable source metadata/assets
```

P0 直接使用 OMP 的：

- Agent Loop；
- Provider 与 Model Roles；
- Tool Registry；
- TUI；
- Session；
- Compaction；
- Approval；
- LSP/DAP/Search/Edit；
- TTSR/Hook/Extension 能力；
- Plugin Manifest 和项目级 Override。

### P0-A：三层 AGENTS 与 Mode Contract（必须先完成）

```text
$PI_CODING_AGENT_DIR/AGENTS.md
  默认 ~/.omp/agent/AGENTS.md

<project-root>/AGENTS.md
  跨 Agent 项目事实

<project-root>/.omp/AGENTS.md
  OMP/KPi Adapter + @../AGENTS.md
```

范围：

- Platform-aware Global Path Resolver；
- Root Project Facts Template；
- OMP Project Adapter Template；
- `/sbtd on` = `enforced`；
- `/sbtd off` = `advisory`；
- 三类 Managed Block 与用户内容保留；
- `@../AGENTS.md` Import Validation；
- Context Bridge、Provider Shadow 和 nearest-native Detection；
- Onboard 同时处理根 AGENTS 与 `.omp/AGENTS.md`；
- 旧 `~/.codex/AGENTS.md` 仅作为迁移输入，不自动删除。

P0-A 退出标准：

- normal Onboard 写入正确 OMP Global Path；
- 根 Project AGENTS 和 OMP Adapter 均写入正确路径；
- `/sbtd off` 停止自动 Gate，但保留 Root Facts/Always-on；
- `.omp/AGENTS.md` 正确导入根文件；
- `exists/discovered/loaded/effective/shadowedBy/imports` 可审计；
- 未知 Existing AGENTS 不被无确认覆盖。

### P0-B：上游 AGENTS 三目标转换与同步（P0 发布前完成）

```text
640-skills pinned snapshot
  → Markdown Section Parse
  → agents-section-map.yaml
  → Global / Root Project / OMP Adapter Generator
  → Plugin Rule/Workflow/Skill Changes
  → Sync Report
  → Conformance Tests
```

范围：

- `upstream.lock.json`；
- `agents-section-map.yaml`；
- Transform Version；
- `AGENTS.global.md`；
- `AGENTS.project-root.md`；
- `AGENTS.project-omp.md`；
- Source/Generated Digests；
- Added/Changed/Removed/Unmapped Section 报告；
- Unmapped Section Release Blocker；
- 三类 Managed Block Update；
- Old-template Digest Migration；
- User-owned Content Preservation。

P0-B 退出标准：

- 上游两个 AGENTS 模板所有 Section 均有唯一 Owner 或显式 Split Targets；
- Generated 三类 AGENTS 与 Plugin/Skills 无语义漂移；
- 上游新增未知 Section 使 CI 失败；
- Plugin 更新后的 `/sbtd onboard plan/reset` 可安全同步三类 Managed Block。

## 4. P0 交付物

### P0-E1：插件骨架

交付：

- `@kpi/omp-sbtd` 包；
- OMP Plugin Manifest；
- Feature/Settings Schema；
- 固定版本、只读的 SBTD Kit Bootstrap Source；
- Plugin ↔ Kit Revision 映射；
- 插件安装、启用、禁用、Doctor 文档；
- 支持固定 OMP 版本范围。

安装原则：

- 不定义会写用户/项目环境的 Package `postinstall`；
- 不复制 AGENTS、Skills 或模板；
- 不安装 Trellis、GitNexus、RTK、Java、Maestro、Playwright；
- 不配置 MCP、Provider、Model Role；
- 仅允许 OMP 自身 Plugin Registry/Lock/Cache 变化。

验收：

- `omp plugin install` 或等价流程可安装；
- 安装前后环境 Diff 证明没有非 Plugin 管理写入；
- Plugin Doctor 返回明确状态；
- Kit Revision 与文件摘要可校验；
- 插件禁用后不改变 OMP 默认行为；
- 不修改 OMP Core。

### P0-E2：Slash Commands

实现：

```text
/sbtd help [command]
/sbtd on
/sbtd off
/sbtd status
/sbtd route
/sbtd doctor
/sbtd onboard status
/sbtd onboard plan
/sbtd onboard init
/sbtd onboard reset
/sbtd onboard init-projects
/sbtd setup
/sbtd report
/sbtd strict
/sbtd relaxed
```

验收：

- `/sbtd help` 在所有环境状态和 Runtime Mode 下可用；
- `/sbtd help <nested command>` 支持如 `onboard init`；
- Help 不调用模型、不创建 Agent Turn、不修改 Session；
- `/sbtd on` 创建/恢复 Session State 并执行只读 Preflight；
- 未完成 normal Onboard 时返回 `needs-onboard` 或 `degraded`，不自动安装；
- `/sbtd onboard plan` 零写入；
- Apply 类命令显示计划并获得确认；
- `/sbtd off` 切为 `advisory`，停止自动分类、Book Gates、自动 Skill 路由和 SBTD Delivery Gate；不删除 AGENTS、Skills、Tools 或历史报告；
- `/sbtd status` 显示 Environment Mode、Runtime Mode、Stage、Route、Book Gates、三层 AGENTS、Tool Evidence、Validation、Provider；
- `strict/relaxed` 不得关闭 Hard Gates；
- 命令在 Resume 后保持一致。

### P0-E2A：首次使用与 Onboard UX

P0 标准链：

```text
OMP Provider/Login 已配置
  → omp plugin install @kpi/omp-sbtd
  → 新 Session / Reload Plugin
  → /sbtd help
  → /sbtd doctor
  → /sbtd onboard plan
  → 用户确认
  → /sbtd onboard init|reset|init-projects
  → 新 Session / 完整资源 Reload
  → /sbtd on
```

要求：

- P0 不依赖尚未发布的 `kpi` CLI；
- Plugin Install 和 Onboard 是两个独立事务；
- `/sbtd setup` 是 Plan Wizard，不直接 Apply；
- Plan 显示 Kit Revision、Source Policy、Global/Root/OMP Adapter 三类目标路径、Managed Digests、备份和可选项；
- Onboard 完成后，如果 Runtime 不能完整 Reload AGENTS/Skills/MCP，必须要求新 Session；
- `/sbtd on` 不得把 “configured” 当成 “callable”。

验收：

- Fresh OMP + Plugin 可稳定进入 `needs-onboard`；
- 完成 Onboard 并新建 Session 后进入 `managed`；
- 无根 Project AGENTS 时进入 `degraded`；
- 无 OMP Project Adapter 时进入 `needs-onboard` 或 `degraded`；
- 用户取消 Plan 后无任何环境变化；
- 同一 Plan 重复执行保持现有 Onboard 幂等和回滚语义。

### P0-E2B：Command Registry 与 `/sbtd help`

实现统一 `SbtdCommandSpec`：

```text
path / aliases
category
summary
usage
examples
mutates
requiresConfirmation
availableWhen
```

同一 Registry 生成：

- 命令解析；
- `/sbtd help`；
- 未知命令建议；
- PRD/README 命令表；
- P1 Shell Completion；
- Command Contract Tests。

验收：

- 所有公开命令都有 Summary、Usage 和至少一个 Example；
- 写操作明确标记 `mutates=yes` 和 `confirmation=required`；
- Registry 与实际 Handler 一一对应；
- 删除或新增命令时，Help Snapshot 和文档测试同步变化；
- 帮助输出不依赖 LLM 文案生成。

### P0-E3：Session State 与 Event Bridge

挂接优先级：

```text
session_start
before_agent_start
context
tool_call
tool_result
turn_start
turn_end
session_before_compact / session.compacting
session_switch / branch / tree
session_shutdown
message_update / ttsr_triggered（可用时）
```

状态：

```text
enabled
mode: enforced | advisory
environmentMode: managed | needs-onboard | degraded | blocked
stage
classification
route
activeSkills
bookGates
toolEvidence
validation
provider
lessons
decisions
agents:
  global
  projectRoot
  ompProjectAdapter
  effectiveNativePath
  imports
  shadowedBy
```

验收：

- Compaction 前后状态不重置；
- Branch/Switch 后按 Session 重建；
- State 使用非 LLM Persistent Entry；
- `enforced/advisory`、三层 AGENTS 状态和 Import/Shadow 信息可恢复；
- 关键变化同时产生可读 UI 状态。

### P0-E4：SBTD 分类与 Route

实现最小 Route：

```text
small-direct-change
bugfix
bdd-user-visible-change
trellis-managed-task
legacy-safe-change
refactoring-pass
data-design-risk
web-runtime-diagnostics
web-e2e-regression
mobile-e2e
release-readiness
review
```

分类输入：

- 用户请求；
- Repo Facts；
- AGENTS；
- Trellis；
- Diff/Task；
- 现有 Feature/Test；
- 风险 Predicate。

验收：

- 小型文档/配置任务不会自动进入完整 Trellis；
- 用户可见变化必定标记 BDD；
- Existing Production Code 标记 Refactoring Gate；
- Existing Behavior Bug 标记 Legacy Gate；
- Data/Async/Cross-service 标记 DDIA Gate；
- Production Path 标记 Release Gate；
- Route 输出包含客观原因。

### P0-E5：Book Gate Plan

实现 5 个 Gate：

1. DDD Boundary Review；
2. DDIA Data Design Review；
3. Legacy Change Safety Review；
4. Refactoring Review；
5. Release Readiness Review。

验收：

- 每个开发任务输出 Gate Plan；
- Gate State 符合 `planned/running/passed/blocked/not-required`；
- Reviewer Status 与 Gate State 分离；
- `grill-with-docs` 完成后强制 DDD 二次审核；
- Legacy/Refactoring `safety-seam-only` 回路可执行；
- Release Gate 必须位于验证之后；
- Required Gate 未通过时阻断相应阶段。

### P0-E6：SBTD Kit Loader

加载：

- Plugin 内固定版本、只读的 SBTD Kit Bootstrap Source；
- OMP Global AGENTS、根 Project AGENTS、OMP Project Adapter 与 Deep AGENTS；
- 15 Bundled Skills；
- 14 External Skills；
- Lessons 短入口与命中 Topic；
- Trellis Workflow/Spec/Task；
- BDD Feature；
- Project Validation；
- Knowledge Base P1.1 说明。

策略：

- 安装 Plugin 时不把 Kit 内容复制到用户或项目目录；
- `/sbtd onboard` 才把固定 Kit 作为显式安装源；
- 不在每次 `/sbtd on` 时联网拉取仓库 `main`；
- 常驻只放 Route、Hard Boundary、Status Contract；
- Skill 内容按需；
- 不默认读取全部 Lessons；
- 缺失 Skill 标记 Blocked 或按规则降级。

验收：

- Context 使用可观察；
- 重复 Turn 不重复注入完整 Skill；
- Skill 版本和来源进入 Doctor；
- Legacy `kuno-workflow-onboard-skills` 不作为别名保留。

### P0-E6A：三层 AGENTS Contract Bridge

实现：

- OMP Global Path Resolver：`$PI_CODING_AGENT_DIR/AGENTS.md`，fallback `~/.omp/agent/AGENTS.md`；
- Root Project Resolver：`<project-root>/AGENTS.md`；
- OMP Project Adapter Resolver：`<project-root>/.omp/AGENTS.md`；
- `@../AGENTS.md` Import Validation；
- Global/Root/Adapter/Deep AGENTS 发现；
- Root Facts Context Bridge 与内容摘要去重；
- OMP Provider Shadow/nearest-native Detection；
- `enforced/advisory` Runtime Marker；
- 三类 Managed Block Parse/Replace；
- 用户内容保留；
- 旧已知模板迁移；
- 未知 Existing File 的 `merge-required`；
- 旧 `--skip-project-agents` → `--skip-project-root-agents` 弃用映射。

验收：

- Global Path 支持 `PI_CODING_AGENT_DIR`；
- 根和 `.omp/AGENTS.md` 均写入正确位置；
- Adapter 必须导入根 AGENTS；
- `.omp/AGENTS.md` 或其他 Provider Shadow 状态可见；
- 更近 Workspace Adapter 被识别，不静默忽略根 Adapter；
- `/sbtd on` 激活 Conditional SBTD；
- `/sbtd off` 保留 Root Facts/Always-on；
- 三类 Managed Block 外内容字节级保留；
- Marker/Import 损坏时安全阻断。

### P0-E6B：Upstream AGENTS 三目标同步流水线

实现：

```text
upstream.lock.json
agents-section-map.yaml
transform-version.json
generated/agents/omp/AGENTS.global.md
generated/agents/omp/AGENTS.project-root.md
generated/agents/omp/AGENTS.project-omp.md
sync-report.json
sync-report.md
```

Section Owner：

```text
omp-global-agents
project-root-agents
omp-project-agents
plugin-rule
plugin-workflow
skill:<name>
onboard
ignored-with-reason
```

同步流程：

1. 固定 `640-skills` Commit/Tag；
2. 比较 AGENTS/Catalog/Skills；
3. Markdown AST 按 Heading/Section Digest 识别；
4. 应用 Owner 或显式 Split Targets；
5. 生成三类 AGENTS；
6. 输出 Plugin Rule/Skill 影响清单；
7. 运行 Golden/Conformance Tests；
8. Unmapped Section 阻断 Release；
9. 发布 Plugin/Kit；
10. 用户通过 `/sbtd onboard plan/reset` 更新三类 Managed Block。

验收：

- 上游每个 Section 唯一映射或显式 Split；
- Removed/Renamed Section 有迁移结果；
- 未映射新增内容使 CI 失败；
- Sync Report 包含 Source Commit、Digest、Owner、Transform Version 和三个 Generated Digest；
- Installed 三类 Managed Block 可从旧 Kit 安全更新；
- 不在 `/sbtd on` 时联网拉取 `main`。

### P0-E7：Rule/Policy Gate

P0 规则：

- 禁止普通任务自动 `trellis init`；
- BDD 缺失阻断可见行为交付；
- Required Book Gate 未通过阻断编辑/交付；
- RTK 报告风险；
- Report Artifact Gate；
- Mock/Contract/App-mocked 不得冒充 Full-stack；
- GitNexus 强证据；
- Maestro Java 17+；
- Dependency/Global Install Approval；
- Secret Path Guard；
- Release Gate。

验收：

- Tool Call 可 Block；
- Rule Match 可 Remind/Interrupt；
- Block Reason 对用户可见；
- Rule 在 Compaction 后仍有效；
- Rule 不误匹配代码块、路径或示例文本；
- 可通过项目配置禁用 Optional Rule，Hard Rule 不能静默关闭。

### P0-E8：Validation 与 Report

实现：

- Focused → Affected → Full；
- Project-defined Commands；
- RTK/Native Gate；
- BDD Language/Trace；
- Playwright/Maestro/API Formal Report；
- Evidence 状态；
- Final Full Rerun；
- Structured JSON + Chinese Markdown Report。

验收：

- 报告文件 mtime/size/content 可验证；
- stdout-only 不满足 Formal Gate；
- 同 stem 中文 Markdown；
- E2E Mode 不升级；
- Dirty Revision 不作为 PR Head 证据；
- Failed/Blocked/Skipped/Not-needed 可区分。

### P0-E9：Onboard Python Bridge

支持调用：

```text
check
check-projects
plan
init
reset
init-projects
check-agent-cli
install-agent-cli
install-external-skills
```

P0 原则：

- Plugin 内固定 Kit 所带的 `onboard.py` 仍是 P0 Source of Truth；OMP Platform Adapter 解析 Global、Root Project 与 OMP Project Adapter 三类目标；
- `/sbtd onboard` 是 P0 用户入口，`kpi onboard` 属于 P1；
- Plugin Install/Load 不调用 Onboard；
- 写操作必须 `plan` 后确认；
- `--yes` 语义不被插件重新解释；
- 多项目 JSON 完整透传；
- 不重写安装事务。

验收：

- 可定位 Plugin 内固定 Kit 的脚本，并可识别已安装 Global Skill 的兼容来源；
- 可显示 Plugin/Kit Revision、Global/Root/OMP Adapter 三类路径、Import/Shadow 状态、Managed Digests 与 Plan；
- 可处理 Python 不存在；
- Exit Code 与 Trellis Aggregate Status 正确；
- 不吞掉 Rollback Path；
- Project-only 不触碰 Global State。

### P0-E9A：Onboard/Runtime 工具生命周期

实现并独立记录：

```text
installed
configured
callable
project-ready
current/fresh
blocked reason
```

#### Plugin 安装与 Onboard 的写入矩阵

| 目标 | Plugin Install | `/sbtd on` | normal Onboard | project-only |
|---|---|---|---|---|
| OMP Plugin/Kit | 安装 | 加载 | 不重复安装 | 不涉及 |
| OMP Global AGENTS/Skills/Tools/MCP | 不写 | 只读检测 | Plan+确认后处理 | 禁止处理 |
| 根 Project AGENTS | 不写 | 读取项目事实 | Managed Block 逐项目处理 | Managed Block 逐项目处理 |
| OMP Project Adapter | 不写 | 读取 Mode/Import Contract | Managed Block 逐项目处理 | Managed Block 逐项目处理 |
| Project `.gitignore` / `.trellis/` | 不写 | 只读检测 | 逐项目处理 | 逐项目处理 |
| GitNexus Index/Hook | 不写 | 检测 | 默认不自动创建 | 不涉及 |
| Provider/Login/Model | 不改 | 只读报告 | 不修改 | 不涉及 |

#### Trellis

- Onboard 可在确认条件下初始化；
- 普通 Runtime 只检测，不自动 `trellis init`；
- 加载 `workflow.md`、Task/Spec、Bootstrap 状态；
- 小任务/非 Trellis 项目继续 Native Route。

#### GitNexus

- Onboard 安装 CLI、可选配置用户级 OMP MCP；
- Runtime 必须同时确认 MCP callable、Index、Branch 与 Freshness；
- Stale 刷新失败时降级 Advisory，并回到源码/Diff/测试。

#### Playwright/Maestro/MCP

- Playwright 只按项目适用性安装到 devDependency；
- Maestro 区分 Java、CLI、MCP、设备和 App Artifact；
- MCP 配置存在不等于当前 Session 可调用；
- Auth/Token 不进入项目和报告。

验收：

- 所有组合状态都有 Fixture；
- Project-only 不触碰 Global Tool/Skill/MCP；
- Tool 不可用时只报告 `blocked/skipped/not-needed`，不伪造通过。

### P0-E10：Provider Delegation

P0 不自研 Provider Gateway，使用 OMP：

- BYOK；
- OMP OAuth/Plan/Local；
- Model Roles；
- Fallback；
- Custom OpenAI-compatible Provider。

KPi 插件只记录：

```text
provider
model
role
auth mode
availability
fallback used
```

验收：

- 不读取 OMP 凭据文件；
- 不把 Key 注入 LLM Context；
- Provider 不可用时给出明确 Blocker；
- Codex/Claude/Kimi Subscription 仅由 Runtime/官方 CLI 管理；
- DeepSeek 通过兼容 Provider 配置验证。

### P0-E11：测试矩阵

必须覆盖：

1. 小型 Docs/Config；
2. 新用户可见 Feature；
3. Existing Behavior Bug；
4. Existing Production Code 修改；
5. Data/Schema/Async 变更；
6. Production API/Job/Integration；
7. Web E2E；
8. Mobile E2E Blocked/Available；
9. Cross-repo Contract-only；
10. Session Resume/Compaction；
11. Multi-project Onboard；
12. Provider Unavailable/Fallback；
13. Global + Root + OMP Adapter + Plugin Managed Mode；
14. 缺 OMP Global AGENTS → `needs-onboard`；
15. 缺 OMP Project Adapter → `needs-onboard/degraded`；
16. 缺根 Project AGENTS → `degraded`；
17. `/sbtd on` Enforced 与 `/sbtd off` Advisory 对照；
18. `/sbtd off` 后 Root Facts/Always-on 仍有效；
19. `.omp/AGENTS.md` 的 `@../AGENTS.md` 导入；
20. 同层 Native Adapter 对根独立 AGENTS 的预期 Shadow；
21. 更近 Workspace `.omp/AGENTS.md` nearest-native Detection；
22. Managed Block 外用户内容保持；
23. Old-template Digest 迁移与未知文件 `merge-required`；
24. Upstream 新增 Unmapped Section 阻断 CI；
25. Plugin Update → Plan → Reset → New Session 的三文件同步闭环；
26. Plugin Install 环境 Diff 为零非 Plugin 写入；
27. `/sbtd onboard plan` 零写入；
28. Onboard 后未 Reload 时不得声称新 Skills/MCP callable；
29. `/sbtd help` 全命令 Snapshot；
30. `/sbtd help onboard init` 嵌套命令；
31. Help 零模型调用、零 Session Mutation；
32. Registry/Handler/Help/Docs 一致性；
33. Unknown Command 的候选和 Help 提示。

## 5. P0 退出标准

P0 完成必须满足：

- 核心场景端到端通过；
- 无已知 Hard Gate 静默绕过；
- 无敏感凭据落盘/日志泄露；
- OMP 升级兼容测试可自动运行；
- `/sbtd help`、Parser、Handler、文档和测试共享同一 Registry；
- OMP Global、根 Project、OMP Project Adapter 三层 Contract 通过；
- `.omp/AGENTS.md` 导入根事实并正确处理 nearest-native；
- Plugin Install/Load 与 `/sbtd on` 通过零环境写入测试；
- `/sbtd onboard` 是 P0 唯一产品化 Onboard 入口；
- Fresh Install 的 `needs-onboard`、完整配置后的 `managed`、缺根事实的 `degraded` 可重复；
- `/sbtd on/off` 的 `enforced/advisory` 语义可验证；
- 上游三目标转换/同步流水线通过，所有 Section 已映射且无语义漂移；
- Onboard 与 Runtime 对 Trellis/GitNexus/MCP 的职责没有交叉写入；
- 已收集足够数据决定哪些逻辑进入 KPi Core。

## 6. P0 明确不做

- 独立 KPi Provider Gateway；
- 完整 TypeScript Onboard 重写；
- Pi Runtime；
- OMP Fork；
- 自研 LSP/DAP/Search；
- 云端 Evidence Store；
- 默认 Channel Spawn；
- 删除或废弃全局/项目 AGENTS；
- 在 Package `postinstall`、Plugin Load 或 `/sbtd on` 时静默安装、初始化或改写 AGENTS、Skills、External Tools、MCP 或项目；
- 使用 P1 的 `kpi onboard` 作为 P0 必需流程。

---

# P1 — KPi CLI 产品化

## 7. P1 目标

让用户无需理解 OMP Plugin 安装细节即可使用 KPi：

```text
kpi
  → 检查 Runtime
  → 检查/安装 Plugin
  → 加载 SBTD Kit
  → 默认开启 SBTD
  → 启动 OMP Session
```

P1 建立品牌、配置、安装、Doctor、Onboard、Provider、Session 和 Report 的产品边界。

## 8. P1 交付物

### P1-E1：独立 CLI

命令：

```text
kpi
kpi run
kpi plan
kpi review
kpi validate
kpi doctor
```

要求：

- TypeScript ESM；
- Node.js LTS；
- 显式 Command Table；
- Unknown Input 才进入 Run；
- Management Reserved Words Guard；
- Exit Code 稳定；
- JSON Output 可选。

### P1-E2：默认 SBTD 体验

- `kpi` 与 `kpi run` 默认启用；
- `--sbtd=off|on`；
- 项目配置可覆盖；
- 直接 OMP 仍需 `/sbtd on`；
- 首屏显示 Runtime、Provider、SBTD Mode、Project、Trellis、Kit Version。

### P1-E3：Runtime Bootstrap

实现：

```text
kpi runtime doctor
kpi runtime install omp
kpi runtime update omp
```

要求：

- 检查 OMP 版本；
- 检查 Plugin Version；
- 检查兼容区间；
- 不静默升级；
- 安装/升级需确认；
- 支持本地开发 Link。

### P1-E4：Onboard CLI

```text
kpi onboard check
kpi onboard check-projects
kpi onboard plan
kpi onboard init
kpi onboard reset
kpi onboard init-projects
```

要求：

- 默认 `--platform omp`；
- 保留 `codex/claude/kimi/omp`；
- 多项目绝对路径；
- `--json`；
- Plan/Confirm；
- 逐项目输出；
- Project-only 硬隔离；
- 传递 External Skill Source Policy；
- 传递 Trellis Username/Platform；
- 处理 `bootstrap-required`。

### P1-E4A：AGENTS 与 Workflow Doctor

```text
kpi doctor
kpi rules doctor
kpi kit doctor
```

必须显示：

- OMP Global、根 Project、OMP Project Adapter 三类路径、是否加载、Managed Digest、模板/Kit 版本；
- `@../AGENTS.md` Import、Provider Shadow、nearest-native Adapter 与 Deep AGENTS 层级；
- Plugin 是否加载；
- Environment Mode：managed/needs-onboard/degraded/blocked；
- Runtime Mode：enforced/advisory；
- Required Skills；
- Trellis/GitNexus/MCP 的 installed/configured/callable/project-ready；
- AGENTS 与 Kit Machine Rules 的冲突；
- 建议的 Onboard 修复命令，但未经确认不执行写入。

### P1-E5：Kit/Skill/Tool 管理

```text
kpi kit list|install|doctor
kpi skills list|doctor
kpi tools check|install|doctor
kpi rules list|doctor
```

要求：

- Kuno/SBTD Kit Version 可见；
- Catalog/Schema 校验；
- Bundle/External 来源可见；
- Stable Set 与 Revision 可见；
- Tool Availability 与 Installed 分离。

### P1-E6：Provider UX

```text
kpi provider list
kpi provider doctor
kpi provider add
kpi provider login
kpi provider role
```

P1 仍委托 OMP，但提供统一 UX：

- BYOK 引导；
- Runtime OAuth/Plan 状态；
- Official CLI Delegation 状态；
- DeepSeek Compatible 配置；
- Role Mapping；
- Fallback 状态；
- Secret Reference。

`login` 只允许调用 Runtime/官方 CLI 提供的登录流程，不处理 Token。

### P1-E7：Config

建议层级：

```text
~/.kpi/config.jsonc
<project>/.kpi/config.jsonc
environment
CLI flags
```

优先级：

```text
CLI > Project > User > Defaults
```

配置：

- Runtime；
- SBTD Mode；
- Kit；
- Policy；
- Model Roles；
- Provider References；
- Rules；
- Reports；
- Lessons；
- Onboard Defaults。

### P1-E8：Session 与 Report

```text
kpi session list|show|export
kpi report latest|export
```

要求：

- 不暴露敏感 Prompt/Credential；
- JSON/Markdown；
- 报告引用真实 Artifact；
- 可从 OMP Session 重建；
- 支持 Resume Selector；
- 支持工作流摘要。

### P1-E9：发行与完成度

- npm 安装；
- macOS/Linux/Windows；
- Shell Completions（从 Command Registry 生成）；
- Upgrade/Uninstall；
- `kpi doctor --json`；
- Smoke Test；
- 安装方法 CI；
- License/NOTICE 传播。

## 9. P1 退出标准

- 新用户只安装/运行 KPi 即可完成首个 SBTD Task；
- KPi 能准确修复或引导 Runtime/Plugin/Kit 缺失；
- Onboard 完整兼容当前 Python 接口；
- Provider Doctor 不误报订阅/Key 可用；
- CLI 管理命令不会被发送给 LLM；
- 所有平台安装方式有 Smoke Test；
- `v0.1` 使用文档完整。

## 10. P1 明确不做

- 将所有 OMP Provider 代码复制进 KPi；
- 移除 Python；
- Pi Runtime；
- 组织级 Evidence Store；
- 默认自动安装可选 MCP/Playwright/Maestro；
- OMP Fork。

---

# P2 — KPi Core 抽离

## 11. P2 目标

把 P0/P1 已验证的行为从 OMP 插件中抽离为 Runtime 无关 TypeScript Core，使：

- OMP 只是 Adapter；
- Workflow 状态可 Headless 测试；
- Onboard、Provider、Rule、Validation 有稳定 API；
- Pi Runtime 可在 P3 接入；
- SBTD Kit 可独立版本化。

## 12. P2 包结构

```text
packages/
  core/
  command-registry/
  workflow-engine/
  rule-engine/
  policy/
  validation/
  context/
  provider-gateway/
  session/
  reporting/
  kit-registry/
  skill-registry/
  onboard/
  runtime-omp/
  kuno-workflow-kit/
```

## 13. P2 交付物

### P2-E1：Core Event Contract

统一事件：

```text
session.start
session.resume
session.compacting
session.compacted
turn.start
turn.end
context.request
tool.before
tool.after
workflow.transition
rule.triggered
validation.updated
provider.changed
session.end
```

Runtime Adapter 负责翻译，Core 不使用 OMP 私有 Event 类型。

### P2-E2：Workflow Engine

- State Machine；
- Route Registry；
- SBTD Classifier；
- Book Gate Planner；
- Decision Log；
- Transition Guard；
- Headless Replay；
- Versioned State Migration。

### P2-E3：Rule Engine

- Text/Tool/Result/Workflow Condition；
- Remind/Interrupt/Block；
- Repeat Policy；
- Scope/Glob；
- Rule Priority；
- Project Override；
- Compaction Persistence；
- False-positive Test Corpus。

不要求 P2 自研 OMP 等价的 Token-level TTSR；Adapter 可使用宿主能力，Core 提供语义。

### P2-E4：Policy Engine

- Path Guard；
- Command Classifier；
- Approval Gate；
- Network Policy；
- Secret Redactor；
- Install/Deploy/Migration Policy；
- Sandbox Profile；
- Audit Record。

### P2-E5：Validation Engine

- Command Discovery；
- Project-native Selection；
- RTK Policy；
- Report Artifact Guard；
- BDD Trace；
- Web/Mobile/API；
- Evidence State；
- Final Report Schema；
- Artifact Sanitization。

### P2-E6：Context Engine

- AGENTS Hierarchy；
- Repo Map；
- Lazy Skill；
- Lessons Index/Topic；
- Trellis Artifacts；
- BDD Feature；
- Context Budget；
- Compaction Preserve Data；
- Internal URI：

```text
skill://
workflow://
rule://
trellis://
bdd://
report://
lesson://
session://
```

### P2-E7：Kuno Workflow Kit

将 `640-skills` 转为版本化 Kit：

```text
kpi-kit.json
catalog.json
prompts/
agents/
skills/
rules/
workflows/
schemas/
assets/
licenses/
```

要求：

- 保留源 License/NOTICE；
- Snapshot/Revision 可追踪；
- Schema 校验；
- Bundled/External 分离；
- Upstream/Stable Source Policy；
- Legacy Migration；
- Runtime Target Mapping。

### P2-E7A：三目标 AGENTS Renderer、Section Mapping 与 Conformance

实现：

- Runtime-specific Global Path Adapter；
- 根 Project Facts Renderer；
- Runtime-specific Project Adapter Renderer；
- Markdown AST Section Owner Mapping；
- `@` Import Contract；
- 三类 Managed Block Merge；
- User-owned Content Preservation；
- Source/Generated Digests；
- Semantic Drift Test；
- Template Version Migration；
- Shadow/nearest-native Contract；
- `enforced/advisory` Mode Contract。

验收：

- AGENTS、Plugin/Core Gate 和 Skills 来自同一版本化 Kit；
- 文本规则变更必须伴随 Owner/机器规则/测试评估；
- 未映射 Section 阻断发布；
- 三类 Managed Block 外内容不被覆盖；
- OMP、Pi 使用各自 Global/Project Adapter，但共享根 Project Facts Contract；
- Runtime Adapter 能表达 Import/Inheritance 或提供等价 Context Bridge；
- 项目事实仍以根 Project AGENTS、深层规则和结构化项目配置为准。

### P2-E8：Onboard TypeScript 迁移

步骤：

1. 固定 Python JSON Golden Fixtures；
2. TS 实现 Read-only `check/plan`；
3. 双跑并 Diff；
4. 迁移 Project-only；
5. 迁移 Template Operations；
6. 迁移 External Skill Transaction；
7. 迁移 Agent CLI/Tool Installer；
8. 切换默认；
9. Python 保留 Fallback；
10. 达到稳定后再决定移除。

等价范围：

- Path Canonicalization；
- Multi-project；
- Catalog Validation；
- `.gitignore` BOM/幂等；
- Backup；
- Rollback；
- Legacy Identity；
- Source Fallback；
- Trellis Aggregate Exit；
- `--yes`；
- JSON Shape。

### P2-E9：Provider Gateway

接口：

```ts
interface ProviderAdapter {}
interface CredentialSource {}
interface ModelCatalog {}
interface RuntimeDelegation {}
interface RoleRouter {}
```

原生支持：

- OpenAI；
- Anthropic；
- OpenAI-compatible；
- Anthropic-compatible；
- DeepSeek Preset；
- Local/Gateway；
- Role Mapping；
- Fallback；
- Capability Probe；
- Dynamic Model Catalog。

订阅支持方式：

- Codex：官方 CLI Runtime Delegation；
- Claude：官方 CLI Delegation（受合规开关约束）；
- Kimi：官方 CLI 或订阅 API Key；
- OMP OAuth/Plan：仅 `runtime-omp` 管理。

### P2-E10：Knowledge P1.1 Adapter

- Decision；
- Ingest；
- Smoke；
- Revision Set；
- Artifact Manifest；
- Checksum；
- Metrics；
- Evidence State；
- 输出进入 KPi Report；
- 不提前实现 P2 Publication。

### P2-E11：Compatibility Fixtures

建立跨层 Fixture：

- SBTD Classification；
- Gate Plans；
- Rule Matches；
- Tool Evidence；
- Onboard JSON；
- Validation Artifacts；
- Provider Doctor；
- Session Resume；
- Report Snapshot。

## 14. P2 退出标准

- Core 包不依赖 OMP/Pi 私有模块；
- `runtime-omp` 通过 Adapter 接入；
- Headless Replay 可重放关键 Task；
- TS Onboard 与 Python Golden Fixture 等价；
- BYOK Gateway 可独立工作；
- Provider/Secret 测试无泄露；
- SBTD Kit 可独立发布与升级；
- P0/P1 用户可无破坏迁移。

## 15. P2 明确不做

- 保证 Pi 与 OMP 全工具等价；
- 直接复制 OMP Native Core；
- 无法律审核的第三方订阅 OAuth；
- 默认多 Agent；
- 无必要 Fork。

---

# P3 — 双 Runtime 与 Fork-last

## 16. P3 目标

让同一 KPi Core 与 SBTD Kit 可运行于：

```text
runtime-omp
runtime-pi
```

用户可选择：

```bash
kpi run --runtime omp
kpi run --runtime pi
```

Runtime 差异通过 Capability Matrix 显式呈现，而不是隐藏。

## 17. P3 交付物

### P3-E1：Runtime Contract v1

接口至少包含：

```text
startSession
restoreSession
injectContext
registerTools
registerCommands
subscribeEvents
requestApproval
abortTurn
continueTurn
persistState
compact
shutdown
capabilities
```

### P3-E2：runtime-omp 标准化

将 P0 Plugin Host 适配为正式 Adapter：

- Plugin/Extension/Hook；
- TTSR；
- Model Roles；
- Tool Surface；
- Session；
- Approval；
- Compaction；
- UI；
- Provider Delegation。

### P3-E3：runtime-pi

实现：

- Pi Extension；
- `before_agent_start`/等价 Context Injection；
- Tool Bridge；
- Session State；
- Command/Prompt Template；
- Start Prompt Fallback；
- Capability Detection；
- Policy Wrapper；
- Validation/Report；
- 缺失 TTSR 时的 Turn-level Rule 降级；
- 加载/渲染 Pi 对应的 AGENTS/SYSTEM/Skill/Prompt 资源；
- 保持项目 AGENTS 作为项目事实层，必要时由 Runtime Adapter 转换为等价上下文。

### P3-E4：Runtime Capability Matrix

示例：

| 能力 | OMP | Pi | KPi 行为 |
|---|---|---|---|
| Slash Command | 原生 | Extension/Prompt | Adapter |
| Token-level Interrupt | TTSR | 视版本/扩展能力 | OMP 原生；Pi 降级 |
| LSP/DAP | 丰富 | 可选 | Capability Gate |
| Provider Plan/OAuth | 丰富 | 视 Runtime | Provider/Runtime Doctor |
| Session State | 原生 | 原生/扩展 | 统一序列化 |
| Approval | 原生 | KPi Policy Wrapper | 统一语义 |
| Subagents | 原生 Task | 可扩展 | Channel Bridge |
| Custom UI | 丰富 | TUI Extension | 最小公共集 |

### P3-E5：双 Runtime Compatibility Suite

核心场景在两 Runtime 执行：

- SBTD On；
- BDD Gate；
- Book Gates；
- Tool Block；
- Validation Report；
- Session Resume；
- Provider Unavailable；
- Onboard；
- Lessons；
- Channel Preflight。

比较：

- Stage；
- Route；
- Gate；
- Tool Evidence；
- Final Status；
- Report Schema。

允许 UI/Tool Detail 不同，不允许语义静默不同。

### P3-E6：可选外部 CLI Runtime

在不扩大 P3 核心范围的前提下评估：

```text
runtime-codex-cli
runtime-claude-cli
runtime-kimi-cli
```

定位为 **Delegation Adapter**，不是偷取订阅凭据。每个 Adapter 必须：

- 调用官方 CLI；
- 由用户完成官方登录；
- 不解析私有 Token；
- 有版本与 Terms Gate；
- 能力不足时明确降级。

### P3-E7：Channel/Subagent Bridge

- Trellis Channel 仍是协作事实源；
- OMP Task/Subagent 与 Pi Extension Worker 通过统一 Worker Contract；
- Preflight；
- 单 Writer；
- 单 Validation Controller；
- Worktree Isolation；
- Typed Output；
- Cleanup；
- Cost/Concurrency Guard。

### P3-E8：Sandbox 与 Native 优化（可选）

仅在 Profile 数据证明需要时：

- Container/Sandbox；
- Workspace Isolation；
- Native Repo Map；
- AST Search/Edit；
- Secret Scan；
- Report Scanner。

不以追平 OMP Native Core 为目标。

### P3-E9：Evidence P2（可选独立里程碑）

评估：

- Evidence Store；
- PR Check；
- Head SHA Invalidation；
- Retention；
- Quarantine；
- Publication；
- Gate；
- Organization Policy。

该工作可独立于 Runtime 双适配推进，不应阻塞核心 v0.3。

### P3-E10：Fork Decision ADR

如果发现 OMP 缺口，必须输出：

```text
Problem
Reproduction
Why Plugin/Extension/Hook/Tool/SDK/RPC fails
Upstream proposal
Minimal patch
Merge strategy
Security/update ownership
Release cost
Exit strategy
Decision
```

Fork 仅允许：

- 关键 Hard Gate 无法实现；
- 上游扩展点不可得；
- 最小 Patch 可维护；
- 有长期 Owner；
- 有自动 Upstream Sync/Compatibility Suite。

## 18. P3 退出标准

- OMP/Pi 两 Runtime 通过语义兼容测试；
- Runtime 缺口可见、可降级；
- KPi Core 和 Kit 不依赖宿主；
- Provider/Credential 跨 Runtime 保持安全；
- Channel/Session/Report 可重建；
- Fork 结论有证据；
- 未通过 Gate 时保持无 Fork。

---

## 19. 跨阶段关键路径

```text
OMP Plugin Event Bridge
  → SBTD State
  → Classifier/Book Gates
  → Rule/Validation
  → P0 实测
  → KPi CLI
  → Config/Onboard/Provider UX
  → Core Contracts
  → TS Onboard / Provider Gateway
  → Runtime Adapter v1
  → Pi Adapter
  → Compatibility Suite
  → Fork Decision
```

任何阶段不得跳过前置：

- 未验证 P0，不应提前重写全部 Core；
- 未有 CLI 产品边界，不应让用户直接依赖内部插件路径；
- 未抽离 Core，不应同时维护两个 Runtime；
- 未有 Compatibility Suite，不应 Fork。

---

## 20. 质量门禁

### 20.1 每阶段必须通过

```text
Type Check
Lint
Unit Tests
Integration Tests
CLI Help/Version Smoke
Install Smoke
Secret Scan
License/NOTICE Check
Documentation Check
Compatibility Check
```

### 20.2 SBTD 专项测试

- Classification Golden Cases；
- Book Gate Predicate Cases；
- Rule False Positive/Negative；
- BDD Language；
- BDD Trace；
- RTK Report Gate；
- Report Staleness；
- E2E Mode；
- GitNexus Evidence；
- Trellis Init Boundary；
- Multi-project Aggregation；
- External Skill Rollback；
- Session Compaction；
- Provider Credential Redaction；
- Global/Root/OMP Adapter Hierarchy、Import、Shadow/nearest-native 与 Degraded Mode；
- `enforced/advisory` Conformance；
- `/sbtd help` Registry/Handler/Docs Snapshot；
- Trellis Onboard-vs-Runtime Boundary；
- GitNexus MCP/Index/Freshness Matrix；
- MCP Configured-vs-Callable Matrix。

### 20.3 Release Blockers

以下任一项阻断 Release：

- Hard Gate 可被静默跳过；
- 报告陈旧仍显示 Passed；
- 非 Full-stack 被显示为 Full-stack；
- 凭据进入日志/报告/Context；
- Project-only 修改 Global State；
- External Skill Transaction 无法恢复且不报告；
- Resume 丢失 Gate/Stage；
- 管理命令被发送给模型；
- 未声明 Runtime/Provider 降级；
- P0 Runtime 静默删除/覆盖 AGENTS，或破坏 Managed Block 外内容；
- `.omp/AGENTS.md` 缺少/错误导入根 Project AGENTS；
- `/sbtd help` 与实际命令 Handler 漂移；
- Project-only 修改 Global AGENTS/Skills/Tools/MCP；
- GitNexus 仅凭 CLI/目录即声称 MCP 影响分析已完成；
- License/NOTICE 缺失。

---

## 21. Provider 路线

| 阶段 | Provider 目标 |
|---|---|
| P0 | 继承 OMP；记录 Provider/Role/Auth/Fallback；不触碰凭据 |
| P1 | 统一 `kpi provider` UX；Doctor；Secret Reference；官方 Login 委托 |
| P2 | 独立 BYOK Gateway；OpenAI/Anthropic/Compatible/DeepSeek；Role/Fallback |
| P3 | 跨 Runtime Provider Contract；可选 Codex/Claude/Kimi CLI Delegation |

合规检查持续要求：

- Codex ChatGPT 登录只由官方 CLI/Runtime 管理；
- Claude Subscription 不内嵌第三方 OAuth，不代理订阅凭据；
- Kimi 使用官方 CLI 或官方提供的 API Key；
- DeepSeek 使用官方兼容 API；
- Model Catalog 动态，不硬编码已弃用模型；
- 凭据永不进入项目、报告和 LLM Context。

---

## 22. Onboard 迁移路线

| 阶段 | 实现 |
|---|---|
| P0-A | 调用 `onboard.py` + OMP Platform Adapter；Global 写 `$PI_CODING_AGENT_DIR/AGENTS.md`，项目写根 `AGENTS.md` 与 `.omp/AGENTS.md` |
| P0-B | 建立三目标 Section Mapping、Generated Templates、Managed Blocks 和 Sync Report |
| P1 | `kpi onboard` 封装参数、确认、输出、路径发现、Import/Shadow Doctor |
| P2-A | TypeScript `check/plan` 双跑 |
| P2-B | Project-only、Root/Adapter Template Operations |
| P2-C | External Skill Transaction/Stable Fallback |
| P2-D | Agent CLI/Tool Installers |
| P2-E | TS 默认、Python Fallback |
| P3 | Runtime-specific Project Adapter，评估移除 Python并保留旧版本迁移工具 |

迁移必须保持：

- Catalog Schema、Multi-project、`--yes`、Project-only；
- Backup、Rollback、Legacy Identity、Stable Digest/License；
- Trellis Exit Code、JSON Contract、`.gitignore` BOM/幂等；
- OMP Global `PI_CODING_AGENT_DIR` 路径；
- 根 `<project-root>/AGENTS.md`；
- `<project-root>/.omp/AGENTS.md` 与 `@../AGENTS.md`；
- 三类 Managed Block 和块外内容保留；
- 旧 `--skip-project-agents` 的弃用映射；
- Section Mapping、三个 Generated Digests 和 Unmapped Release Gate；
- `/sbtd on/off` 的 `enforced/advisory`；
- `/sbtd help` Registry Contract；
- Trellis Onboard 初始化例外；
- MCP 用户级 Scope；
- Project-only Global State 隔离。

## 23. 兼容性与升级策略

### 23.1 OMP

- Pin 支持区间；
- CI 测试最低/推荐/最新兼容版；
- Plugin API Feature Detection；
- 升级前 Compatibility Suite；
- 不自动修改用户 OMP 配置；
- Breaking Change 先提示和迁移。

### 23.2 `640-skills`

- Kit 记录 Commit/Tag；
- 可选择 Floating Main、Pinned Tag、Vendored Snapshot；
- Catalog/Schema 版本迁移；
- Bundled/External Skill Diff；
- License/NOTICE 校验；
- Runtime Gate Contract 变更需要 Changelog。

### 23.3 Pi

- P3 以前不作为生产默认；
- Adapter 只使用公开 API；
- 启动 Context 注入存在 Fallback；
- 缺少 Token-level Rule 时明确能力降级；
- 不为追求形式一致而绕过 Pi 安全边界。

---

## 24. 里程碑交付清单

### P0

- [ ] OMP Plugin Skeleton
- [ ] Zero-mutation Plugin Install
- [ ] Pinned Read-only SBTD Kit
- [ ] `/sbtd` Commands
- [ ] Command Registry + `/sbtd help [command]`
- [ ] `/sbtd onboard` Commands/Setup Wizard
- [ ] Session State
- [ ] Classifier/Routes
- [ ] Book Gate Plan
- [ ] Kit Loader
- [ ] OMP Native Global Path Resolver
- [ ] Root Project Facts Contract
- [ ] OMP Project Adapter + `@../AGENTS.md`
- [ ] Mode-aware AGENTS (`enforced/advisory`)
- [ ] Managed Block Merge/Preservation
- [ ] Shadow Detection
- [ ] Upstream AGENTS 三目标 Section Mapping（P0-B）
- [ ] AGENTS Sync Report/Release Gate
- [ ] Rules/Policy
- [ ] Validation/Report
- [ ] Python Onboard Bridge
- [ ] Onboard/Runtime Tool Lifecycle
- [ ] Provider Delegation
- [ ] P0 Test Matrix
- [ ] P0 User Guide

### P1

- [ ] `kpi` CLI
- [ ] Default SBTD On
- [ ] Runtime Bootstrap
- [ ] `kpi onboard`
- [ ] AGENTS/Workflow Doctor
- [ ] Kit/Skill/Tool/Rule Doctor
- [ ] Provider UX
- [ ] Config Layering
- [ ] Session/Report Export
- [ ] Completion
- [ ] Cross-platform Install Tests
- [ ] v0.1 Documentation

### P2

- [ ] Core Event Contract
- [ ] Workflow Engine
- [ ] Rule Engine
- [ ] Policy Engine
- [ ] Validation Engine
- [ ] Context Engine
- [ ] Kuno Workflow Kit
- [ ] AGENTS Renderer/Conformance
- [ ] TS Onboard
- [ ] Provider Gateway
- [ ] Knowledge P1.1 Adapter
- [ ] Runtime OMP Adapter
- [ ] Migration Guide

### P3

- [ ] Runtime Contract v1
- [ ] Runtime OMP Final
- [ ] Runtime Pi
- [ ] Capability Matrix
- [ ] Compatibility Suite
- [ ] CLI Delegation Feasibility
- [ ] Channel/Subagent Bridge
- [ ] Sandbox/Native Profiling
- [ ] Evidence P2 Decision
- [ ] Fork ADR
- [ ] v0.3 Migration/Release Guide

---

## 25. Definition of Done

一个阶段只有在以下条件全部满足时才算完成：

1. 功能代码、Schema、文档和测试同步；
2. 用户可见行为有 BDD 或明确 not-needed；
3. Required Book Gates 已通过；
4. 项目原生验证已执行；
5. 正式报告满足 Artifact Gate；
6. 失败、跳过、阻塞和剩余风险已列出；
7. Provider/Credential 无泄露；
8. 安装、升级、回滚路径可验证；
9. License/NOTICE 完整；
10. 下一阶段依赖已形成稳定 Contract。

---

## 26. 最终路线结论

```text
P0-A：固定 OMP Global、根 Project Facts、OMP Project Adapter 和 Mode-aware Contract
P0-B：建立上游 AGENTS 三目标 Section Mapping、生成模板、Managed Blocks 与同步流水线
P1：用 KPi CLI 建立独立产品入口
P2：抽离 Runtime 无关 KPi Core
P3：支持 OMP/Pi 双 Runtime
Fork：仅在插件、SDK、RPC 均无法实现关键门禁时进行
```

该路线优先验证 SBTD 的真实价值，同时避免过早承担 OMP Fork、Provider 重造、Native Tool 和双 Runtime 的维护成本。
