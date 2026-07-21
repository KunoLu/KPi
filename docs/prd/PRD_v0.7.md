# KPi 产品需求文档（PRD）

> **文档版本**：0.7-draft  
> **文档日期**：2026-07-21  
> **产品名称**：KPi  
> **产品形态**：CLI Coding Agent / OMP 插件 / 可替换 Runtime 的 SBTD 工作流平台  
> **推荐路线**：**OMP-first → KPi CLI → KPi Core → 双 Runtime → 必要时受控 Fork**  
> **SBTD 基线来源**：`KunoLu/640-skills` 默认分支 `main`，评估快照 `340f9dd4dc7a92e8b91c31e111de9a8de06cef36`  
> **说明**：本文中的 P0～P3 表示实施优先级和架构成熟阶段，不表示缺陷严重级别。  
> **0.7 更新重点**：将项目规则正式拆为根 `<project-root>/AGENTS.md` 跨 Agent 项目事实层与 `<project-root>/.omp/AGENTS.md` OMP/KPi 运行时适配层；增加双文件 Managed Block、`@../AGENTS.md` 导入、Monorepo/Shadowing 规则、上游三目标 Section Mapping，以及由统一命令注册表生成的 `/sbtd help [command]`。

---

## 1. 执行摘要

KPi 是一个以 **SBTD（SDD + BDD + TDD + DDD）工程工作流**为核心的独立 CLI Coding Agent。其差异化重点不是重新实现模型调用、终端编辑器、代码搜索、LSP、DAP 或通用 Agent Loop，而是把已经沉淀在 `640-skills` 中的 SBTD Workflow Onboard、Trellis 生命周期、BDD 行为规格、Book Gate Plan、GitNexus 强证据判断、项目原生验证、Web/Mobile E2E、Evidence、Lessons 与 Channel 协作规则，转换成可执行、可审计、可扩展的产品能力。

产品采用四阶段路线：

| 阶段 | 路线 | 核心结果 |
|---|---|---|
| **P0** | OMP 插件优先 | 通过 `@kpi/omp-sbtd` 和 `/sbtd on` 快速验证 SBTD 工作流，不 Fork OMP |
| **P1** | KPi CLI 产品外壳 | 用户统一使用 `kpi`；内部启动 OMP、加载插件和 SBTD Kit，默认启用 SBTD |
| **P2** | 抽离 KPi Core | 将分类、Workflow、Rule、Policy、Validation、Context、Provider、Session 抽离为 Runtime 无关核心 |
| **P3** | OMP/Pi 双 Runtime | 同时支持 `runtime-omp` 与 `runtime-pi`；插件/SDK/RPC 均无法满足关键门禁时，才评估受控 Fork |

核心决策：

1. **P0 不直接 Fork OMP。**
2. **KPi 的长期核心资产是 Workflow 与工程门禁，不是某个 Runtime。**
3. **OMP 是 P0/P1 的默认宿主，Pi 是 P3 的第二 Runtime。**
4. **TypeScript 是主语言；Node.js LTS 是 KPi Core/CLI 默认运行时。**
5. **现有 `onboard.py` 在 P0/P1 作为权威兼容层保留，P2 再以契约测试驱动迁移到 TypeScript。**
6. **Provider 同时支持 BYOK 与合规的订阅/官方 CLI 委托，但 KPi 不读取、复制或代理浏览器 Cookie、私有刷新令牌和隐藏订阅凭据。**
7. **SBTD 的强制门禁必须进入状态机、Rule Engine 和 Validation Engine，而不是只依靠长 Prompt。**
8. **P0 继续安装并加载全局与项目 `AGENTS.md`；插件不会在 P0 直接取代它们。**
9. **OMP 插件是可执行控制面，`AGENTS.md` 是声明式规则/项目事实/跨 Agent 兼容层，Skills 是专项流程，Onboard 是环境与安装管理器。**
10. **Trellis、GitNexus、Playwright、Maestro 和 MCP 由 Onboard 负责安装/配置，由 Runtime 依据当前 Session 强证据决定是否启用。**
11. **`omp plugin install @kpi/omp-sbtd` 只安装插件代码及经过测试、固定版本的只读 SBTD Kit 快照；不得通过 `postinstall` 静默写入 AGENTS、安装全局工具、修改项目或配置 MCP。**
12. **P0 的环境初始化入口是插件内的 `/sbtd onboard ...`；独立 `kpi onboard ...` 属于 P1，不能出现在 P0 的必需使用链中。**
13. **`/sbtd on` 首先执行只读 Preflight。环境未完成 Onboard 时进入 `needs-onboard` 或 `degraded`，只给出计划与恢复路径，不自动安装。**

14. **OMP 目标平台的 Global AGENTS 固定写入 `$PI_CODING_AGENT_DIR/AGENTS.md`；未设置该变量时写入 `~/.omp/agent/AGENTS.md`。不再把 `~/.codex/AGENTS.md` 作为 OMP 默认目标。**
15. **项目规则采用双层结构：根 `<project-root>/AGENTS.md` 保存跨 Agent 项目事实，`<project-root>/.omp/AGENTS.md` 保存 OMP/KPi 专用的 Mode Contract 与 SBTD Runtime Adapter。**
16. **`<project-root>/.omp/AGENTS.md` 通过 `@../AGENTS.md` 导入根项目事实；OMP 原生 Context 优先级更高，因此该文件是 OMP Session 的项目级有效入口。**
17. **OMP Global AGENTS 与 `.omp/AGENTS.md` 必须是 Mode-aware 模板：Always-on 基线和项目事实始终有效；SBTD 自动编排只在 Plugin 注入 `mode=enforced` 时启用。**
18. **`/sbtd off` 是当前 Session 的 Soft Off/Advisory Mode：停止 Plugin 自动分类、Book Gate、自动 Skill 路由和 SBTD 交付门禁，但不卸载 AGENTS、Skills、Tools 或撤销 Onboard。**
19. **`640-skills` 后续更新通过“固定上游快照 + Section Mapping + 三目标 AGENTS 生成 + Managed Block + Conformance Test”同步；任何未映射的新上游 Section 都必须阻断自动发布并进入人工评审。**
20. **P0 增加 `/sbtd help [command]`；帮助内容、命令解析、错误提示、文档和测试必须来自同一份确定性 Command Registry，不调用模型、不修改 Session。**

---

## 2. 背景与问题

### 2.1 当前工作流资产

`640-skills` 当前主流程已收敛为：

```text
Codex / OMP
  + GitNexus
  + Trellis
  + Chrome DevTools MCP
  + Playwright
  + Maestro
  + SBTD / Book Gates / Evidence / Lessons
```

其基本执行链为：

```text
读取 Lessons 短入口
  → 澄清需求与 SBTD 判断
  → Trellis / GitNexus / Skills 按强证据启用
  → 生成 Book Gate Plan
  → 规格、实现或配置修改
  → 项目原生验证
  → BDD / Web / Mobile / Release 补充验证
  → 最终报告状态、跳过原因、证据与剩余风险
  → 必要时记录 Durable Lesson
```

该仓库已不再只是若干 Prompt，而是包含：

- 自包含 `sbtd-workflow-onboard` Skill；
- `catalog.json` 与 Draft 2020-12 Schema；
- 多项目 `plan / init / reset / init-projects`；
- 全局与项目级 AGENTS 模板；
- 15 个 bundled Skills；
- 14 个 required external Skills；
- Trellis / GitNexus / Playwright / Maestro / MCP 检测与配置边界；
- External Skill upstream/stable fallback 与事务回滚；
- BDD Knowledge Ingest P1.1；
- 报告、Evidence、Revision、Environment Alignment 等状态契约；
- Book Gate Plan 与 5 类客观开发门禁；
- 分层 Lessons；
- Channel 多 Agent 预检、所有权和清理规则。

### 2.2 直接使用 Skills 的局限

仅依赖 AGENTS 和 Skills 存在以下问题：

1. 模型可能在长任务或 Context Compaction 后遗忘硬规则。
2. “工具可用性”容易被候选目录、安装提示或历史状态误判。
3. `rtk`、报告文件、mock/full-stack、Trellis 初始化等边界需要执行时拦截。
4. Book Gate Plan、验证状态、Evidence 状态和跳过原因缺乏统一结构化状态。
5. 不同 Agent 平台对 Skill、Hook、MCP、Session 和 Context 注入的支持不同。
6. Provider、订阅、BYOK、模型角色、回退链与凭据安全需要产品化统一入口。
7. 多项目 Onboard 与 Runtime 工作流没有统一的 CLI 用户体验。

### 2.3 为什么 P0 选择 OMP

OMP 已具备适合作为 MVP 宿主的基础设施：

- Pi 派生的 Agent Loop 和 TUI；
- 丰富 Provider、BYOK、OAuth/Plan/Local 模型接入；
- Model Role 与 Fallback；
- 插件、Feature、Extension、Hook、Command 和 Tool 注册能力；
- TTSR 类休眠规则与流式中断；
- LSP、DAP、Browser、Search、Git、Subagent、Memory；
- Session、Compaction、Resume 与项目级覆盖；
- 跨平台发行能力。

因此，P0 应验证“**SBTD 工作流是否显著提升真实 Coding Agent 的工程可靠性**”，而不是先重造通用 Agent 基础设施。

---

## 3. 产品愿景与定位

### 3.1 愿景

让 Coding Agent 不仅“能改代码”，而且能够：

- 依据项目事实判断应采用的工程方法；
- 把用户可见行为、设计风险、遗留风险、数据风险与发布风险转化为明确门禁；
- 仅在有强证据时启用工具；
- 形成可复核的规格、测试、报告和 Evidence；
- 在 Runtime、模型和 Provider 变化时保持工作流稳定。

### 3.2 一句话定位

> **KPi 是一个基于 OMP 快速落地、逐步抽离为可替换 Runtime 的 SBTD-first CLI Coding Agent。**

### 3.3 核心价值

| 价值 | 说明 |
|---|---|
| Workflow-first | 工程工作流是核心，模型和 Runtime 可替换 |
| Evidence-driven | 不把安装、候选能力或 stdout 当作已通过证据 |
| Risk-driven | 不对小任务强制仪式；命中客观风险后启用门禁 |
| Behavior SOT | 用户可见行为以持久 `.feature` 为行为事实源 |
| Runtime-agnostic | 长期支持 OMP、Pi，并可扩展其他外部 Runtime |
| Safe by default | 命令、路径、网络、依赖安装、密钥和发布操作受控 |
| Auditable | 路由、门禁、工具状态、验证、报告、跳过和风险均可重建 |

---

## 4. 产品目标与非目标

### 4.1 产品目标

1. 提供 `/sbtd on`，在 OMP Session 内启用完整 SBTD 工作流。
2. 提供 `/sbtd help [command]`，确定性输出所有 SBTD 命令的用途、写入风险、前提和示例。
3. 提供独立 `kpi` CLI，并在通过 KPi 启动时默认启用 SBTD。
4. 保留并执行 `640-skills` 的实际门禁，而不是做简化版概念演示。
5. 支持 normal `init/reset` 与 project-only `init-projects`。
6. 支持多项目绝对路径、逐项目状态、幂等写入和事务回滚。
7. 支持 BYOK、Runtime 管理的 OAuth/Plan、合规的官方 CLI 订阅委托。
8. 支持 Codex、Claude Code、Kimi Code 与 DeepSeek 的明确接入路径和安全边界。
9. 提供统一 Workflow、Rule、Policy、Validation、Context、Command Registry、Provider 与 Session 抽象。
10. 在 P3 支持 OMP 与 Pi 双 Runtime。
11. 只有当可观测证据证明插件/SDK/RPC 无法实现关键门禁时，才进入 Fork 评估。

### 4.2 非目标

P0/P1 不包含：

- 重写 OMP 的 LSP、DAP、Browser、Search、Hashline、Native Shell；
- 自研通用多模型 Agent Loop；
- 默认 Fork OMP；
- 在 Package `postinstall`、Plugin Load 或 `/sbtd on` 时静默写入 AGENTS、安装 Skills/Tools、修改项目或配置 MCP；
- 静默安装所有可选工具；
- 在 P0 使用尚未实现的 `kpi onboard` 作为必需入口；
- 自动绕过 Trellis 或工具的安全保护；
- 把 mock-backed、contract-backed、smoke-only 报告为 full-stack；
- 读取浏览器 Cookie、私有 OAuth Refresh Token、订阅客户端隐藏存储；
- 默认启用多 Agent/Channel；
- 为没有项目证据的小任务强制完整 SDD/BDD/TDD/DDD 仪式；
- P0 即完成云端 Evidence Store、PR Gate 和组织级治理。

---

## 5. 目标用户与核心场景

### 5.1 目标用户

1. **个人开发者**：希望把自己的工程工作流固化为稳定 Coding Agent。
2. **小型研发团队**：需要统一规格、BDD、验证、报告和风险审查。
3. **多仓产品团队**：前后端、Web、Mobile、服务与知识库分仓。
4. **高风险业务团队**：Auth、Billing、Notification、Job、Queue、Data Pipeline、External Integration。
5. **Agent Workflow 维护者**：需要维护 Skills、Rules、Provider、Runtime 和版本兼容性。

### 5.2 核心使用场景

| 场景 | 预期行为 |
|---|---|
| 小型低风险修改 | 最小直接修改 + 项目原生验证，不强制 Trellis |
| 用户可见功能 | 先完成持久 BDD 场景，再实现和追踪测试 |
| 用户可见 Bug | 正确行为场景 → 失败回归测试 → 最小修复 |
| 遗留代码修改 | Legacy Gate → 必要安全网/Seam → Refactoring Gate |
| 数据与跨服务变更 | DDIA Gate 在设计稳定和实现前通过 |
| 生产路径变更 | 所有测试门禁后执行 Release Readiness Gate |
| 中大型需求 | Trellis 生命周期、PRD/design/implement、to-spec/to-tickets |
| Web 回归 | Playwright CLI 正式报告 + 同 stem 中文 Markdown |
| Mobile/Hybrid | Maestro Flow、Java 17+、正式报告和环境事实 |
| 跨仓链路 | 明确 complete/contract-only/environment-only/missing |
| 多 Agent Review | 先 Channel Preflight，用户确认后才能 Spawn |
| 知识库读取 | 固定目标 Ref 到精确 SHA，只读 Knowledge Ingest |
| 多项目初始化 | 逐项目 AGENTS、gitignore、Trellis、Playwright/React Bits 状态 |

---

## 6. 产品设计原则

1. **项目事实优先**：代码、配置、测试、README、CI、AGENTS、Trellis 产物优先于通用假设。
2. **强证据启用工具**：安装目录或 Marketplace 展示不是 callable tool 的证据。
3. **常驻规则最小化**：常驻上下文只保留路由、硬边界和输出契约；细节通过 Skill 延迟加载。
4. **硬门禁进入代码**：可判定的规则必须进入 Rule/Policy/Validation Engine。
5. **不扩大任务范围**：Book Gate 可以明确“不需要重构”，不能为了执行 Skill 而创造工作。
6. **状态可恢复**：Session Resume、Branch、Compaction、Handoff 后恢复 SBTD 状态。
7. **证据不可冒充**：诊断、探索、mock、contract、smoke 与 full-stack 必须严格区分。
8. **安全优先**：网络、安装、破坏性操作、密钥、发布和生产数据必须显式受控。
9. **插件优先、Fork 最后**：所有 Fork 决策必须有缺口证据、成本评估和 ADR。
10. **Runtime 可替换**：KPi Core 不依赖 OMP 私有类型。

---

## 7. P0～P3 推荐路线

### 7.1 阶段总览

| 优先级 | 推荐路线 | 产品形态 | 主要依赖 | 退出条件 |
|---|---|---|---|---|
| **P0** | 先做 OMP 插件 | `@kpi/omp-sbtd` | OMP Plugin/Extension/Hook | `/sbtd on` 可完整跑通核心 SBTD 闭环 |
| **P1** | 再做 KPi CLI 外壳 | `kpi` + OMP Runtime | P0 插件、`onboard.py` | 用户无需直接管理 OMP 插件即可使用 KPi |
| **P2** | 抽离独立 KPi Core | Runtime 无关 TypeScript Packages | P0/P1 实测数据 | 核心逻辑不依赖 OMP，Python 迁移具备契约等价性 |
| **P3** | 支持 OMP/Pi 双 Runtime | `runtime-omp` + `runtime-pi` | KPi Core | 同一 Workflow Contract 在两 Runtime 通过兼容套件 |

### 7.2 与语义版本的建议映射

| 阶段 | 建议版本 |
|---|---|
| P0 | `v0.1-alpha`：OMP-hosted SBTD 插件 |
| P1 | `v0.1-beta` → `v0.1`：KPi CLI 产品化 |
| P2 | `v0.2`：KPi Core 抽离 |
| P3 | `v0.3`：双 Runtime 与 Fork Decision Gate |

---

## 8. 产品体验

### 8.1 两种入口

#### 直接 OMP 入口

```text
omp
/sbtd on
```

- 不改变 OMP 默认行为；
- SBTD 由用户显式启用；
- 适合现有 OMP 用户和 P0 验证。

#### KPi 入口

```bash
kpi
kpi run "修复登录跳转后返回 404 的问题"
```

- KPi 启动的 Session 默认 `sbtd.enabled=true`；
- 默认 `route=auto`；
- 用户可通过 `--sbtd=off` 或 `/sbtd off` 关闭；
- KPi CLI 内部选择 Runtime；该入口从 P1 开始提供，P1 默认 Runtime 为 OMP。

### 8.2 Slash Commands

| 命令 | 说明 | 示例 |
|---|---|---|
| `/sbtd help [command]` | 显示所有 SBTD 命令，或指定命令的用途、前提、副作用和示例；不调用模型、不修改状态 | `/sbtd help onboard init` |
| `/sbtd on` | 将当前 Session 切换为 `enforced`，启用自动 SBTD 分类、路由、Book Gates、Rule/Validation Enforcement | `/sbtd on` |
| `/sbtd off` | 将当前 Session 切换为 `advisory`，停止 Plugin 自动编排；已加载的 AGENTS、Skills 和 Always-on 安全边界仍保留 | `/sbtd off` |
| `/sbtd status` | 查看环境模式、Runtime Mode、阶段、路由、Book Gates、Skills、工具和验证状态 | `/sbtd status` |
| `/sbtd route` | 查看或在受控范围内覆盖自动路由 | `/sbtd route bugfix` |
| `/sbtd doctor` | 只读检查 Runtime、三层 AGENTS、Kit、Skills、Trellis、GitNexus、Playwright、Maestro、MCP、Provider | `/sbtd doctor` |
| `/sbtd onboard status` | 查看 Onboard 完成度、Kit 版本、全局与逐项目配置状态 | `/sbtd onboard status` |
| `/sbtd onboard plan` | 生成机器可读和人类可读的安装/更新计划，不执行写入 | `/sbtd onboard plan` |
| `/sbtd onboard init` | 用户确认计划后执行 normal Onboard | `/sbtd onboard init` |
| `/sbtd onboard reset` | 用户确认计划后按现有安全契约更新/重置 | `/sbtd onboard reset` |
| `/sbtd onboard init-projects` | 仅处理项目资产，不修改任何全局工具、Skills、Global AGENTS 或 MCP | `/sbtd onboard init-projects` |
| `/sbtd setup` | `/sbtd onboard plan` 的交互式向导别名，不直接 Apply | `/sbtd setup` |
| `/sbtd report` | 生成当前结构化 Workflow、Validation 与 Evidence 报告 | `/sbtd report` |
| `/sbtd strict` | 强化可选检查和确认；硬门禁始终保持 | `/sbtd strict` |
| `/sbtd relaxed` | 降低可选流程强度；不得绕过 BDD、Book Gate、报告真实性和安全硬门禁 | `/sbtd relaxed` |

#### 帮助示例

```text
/sbtd help
/sbtd help on
/sbtd help doctor
/sbtd help onboard
/sbtd help onboard init
```

### 8.3 KPi CLI Commands

```text
kpi
kpi run
kpi plan
kpi review
kpi validate
kpi doctor

kpi onboard check
kpi onboard check-projects
kpi onboard plan
kpi onboard init
kpi onboard reset
kpi onboard init-projects

kpi kit list|install|doctor
kpi skills list|doctor
kpi tools check|install|doctor
kpi rules list|enable|disable|doctor
kpi provider list|login|add|doctor
kpi session list|show|export
kpi report latest|export
kpi completions bash|zsh|fish
```

### 8.4 命令注册、路由与帮助事实源

KPi/OMP Plugin 必须使用统一的确定性 Command Registry。命令解析器、`/sbtd help`、错误提示、文档表格、自动补全和命令测试均由同一元数据生成。

```ts
interface SbtdCommandSpec {
  path: string[];
  aliases?: string[][];
  category: "help" | "runtime" | "environment" | "reporting" | "mode";
  summary: string;
  usage: string;
  examples: string[];
  mutates: boolean;
  requiresConfirmation: boolean;
  availableWhen: Array<"managed" | "needs-onboard" | "degraded" | "blocked">;
}
```

规则：

- 已注册命令进入命令处理器；
- 未注册输入才进入 `run/launch`；
- `/sbtd help` 和 `/sbtd help <nested command>` 不调用模型、不生成 Agent Turn、不修改 Session；
- 未知命令返回最相近候选，并提示 `/sbtd help`；
- 帮助输出必须标注 `mutates` 和 `requiresConfirmation`；
- `install`、`remove`、`list`、`config`、`provider`、`plugin` 等管理词不得静默发送给 LLM；
- 全局 Flags 和子命令顺序必须有确定性；
- P1 Shell Completion 从同一 Registry 生成。

### 8.5 `/sbtd help` 输出契约

默认 `/sbtd help` 按类别输出：

```text
Current state
Runtime
Environment / Onboard
Reporting
Modes
Help
```

每个命令至少显示：

```text
command
summary
usage
examples
mutates: yes | no
confirmation: required | not-required
available states
```

`/sbtd help <command>` 只输出目标命令及其子命令。帮助在 `managed`、`needs-onboard`、`degraded`、`blocked` 和 `enforced/advisory` 下均可使用。

### 8.6 P0 的标准安装与使用流程

P0 没有独立 `kpi` CLI，用户直接通过 OMP 使用插件：

```text
步骤 1：用户先安装并完成 OMP 自身的 Provider/Login/Model 配置

步骤 2：安装插件
  omp plugin install @kpi/omp-sbtd
  # 或 OMP 当前版本支持的等价 Plugin 安装命令

步骤 3：重新加载 Plugin 或启动新的 OMP Session

步骤 4：执行只读检查
  /sbtd doctor

步骤 5：生成 Onboard 计划
  /sbtd onboard plan
  # 交互式收集项目绝对路径、是否写根 Project AGENTS、
  # 是否写必需的 .omp/AGENTS.md Adapter、Trellis username/platform flags、
  # MCP 与可选工具决策

步骤 6：用户审阅并明确确认后执行
  /sbtd onboard init
  # 已有环境升级/修复使用 reset
  # 只初始化项目使用 init-projects

步骤 7：Onboard 修改了 AGENTS、Skills 或 MCP 后，
       优先启动新 Session；若 OMP 提供完整资源 reload，
       可由插件在验证后提示使用 reload

步骤 8：从目标项目根目录启动 OMP
  /sbtd on
```

说明：

- 插件安装和 Onboard 是两个独立动作；
- 插件安装不等于 SBTD 环境已完成；
- `/sbtd onboard plan` 永远只读；
- `init/reset/init-projects` 必须显示完整计划并获得确认；
- P0 不使用 `kpi onboard`，该命令从 P1 才提供。

### 8.7 首次执行 `/sbtd on`

`/sbtd on` 必须先执行只读 Preflight：

1. 验证 Plugin 与内置 Kit；
2. 发现 OMP Global AGENTS、根 Project AGENTS、`.omp/AGENTS.md` Adapter 与更深层 AGENTS；
3. 发现 Required Skills；
4. 检查项目事实和 Trellis；
5. 检查 GitNexus、MCP、Playwright、Maestro 等当前状态；
6. 检查 OMP 当前 Provider/Model Role 是否可用；
7. 计算运行模式。

结果：

- `managed`：完整 Onboard 环境已就绪；
- `needs-onboard`：Plugin 可用，但正常 Onboard 基线明显缺失；
- `degraded`：通用基线存在，但项目规则或部分上下文缺失；
- `blocked`：当前任务所需 Hard Gate 无法满足。

即使进入 `needs-onboard`，插件仍可提供只读 Doctor、Plan 和基础安全拦截，但不得声称完整 SBTD 环境已激活。

### 8.8 `/sbtd on` 与 `/sbtd off` 的产品语义

#### `/sbtd on`：Enforced Mode

`/sbtd on` 不执行安装。它只在当前 Session 内：

- 设置 `sbtd.mode=enforced`；
- 注入可验证的 Runtime Marker；
- 自动执行 SBTD 分类和 Route；
- 自动生成并推进 Book Gate Plan；
- 按场景激活对应 Skills；
- 启用 SBTD 专用 Tool/Stage/Delivery Gate；
- 维护 Stage、Tool Evidence、Validation 和 Report State；
- 在 Compaction、Resume、Branch 后恢复状态。

#### `/sbtd off`：Advisory Mode

`/sbtd off` 不会：

- 删除或卸载 OMP Global、根 Project 或 OMP Project Adapter；
- 让已发现的 Skills 从当前 Session 消失；
- 卸载 Trellis、GitNexus、Playwright、Maestro 或 MCP；
- 撤销 Onboard 对文件和工具的安装；
- 关闭 OMP 自身权限、安全或项目约束。

它只停止：

- 自动 SBTD 分类和 Route；
- 自动 Book Gate Plan；
- Plugin 自动 Skill 调用；
- SBTD 专用 Tool/Stage/Delivery Enforcement；
- SBTD Stage 自动推进和强制报告闭环。

此时仍保留：

- OMP Global Always-on、根 Project Facts 和 OMP Adapter 导入的项目约定；
- 用户手动调用 Skills 的能力；
- OMP 自身的权限与工具审批；
- 当前 Session 已产生的历史和报告。

因此状态必须显示为：

```text
SBTD Plugin Enforcement: off
SBTD Runtime Mode: advisory
Global AGENTS: loaded
Root Project AGENTS: loaded
OMP Project Adapter: loaded
Skills: discovered
Automatic Routing: disabled
Book Gate Enforcement: disabled
```

要获得完全不加载 SBTD AGENTS/Skills 的干净环境，必须使用新的启动 Profile/Agent Directory；这不属于同一 Session 内 `/sbtd off` 的能力。P1 可进一步提供 `kpi run --sbtd=off` 的隔离启动体验。

---

## 9. 总体架构

```text
┌───────────────────────────────────────────────────────┐
│                        KPi CLI                        │
│ run / plan / review / validate / onboard / provider  │
└──────────────────────────┬────────────────────────────┘
                           │
┌──────────────────────────▼────────────────────────────┐
│                       KPi Core                        │
│ Workflow · Rule · Policy · Validation · Context       │
│ Provider · Session · Reporting · Kit/Skill Registry   │
└──────────┬───────────────┬──────────────┬─────────────┘
           │               │              │
┌──────────▼───────┐ ┌─────▼──────┐ ┌────▼─────────────┐
│ Kuno/SBTD Kit    │ │ Tool Bridge │ │ Onboard Adapter  │
│ Skills/Rules     │ │ MCP/CLI     │ │ Python → TS      │
└──────────┬───────┘ └─────┬──────┘ └────┬─────────────┘
           │               │              │
┌──────────▼───────────────▼──────────────▼─────────────┐
│                  Runtime Adapter Layer                 │
│       runtime-omp（P0）       runtime-pi（P3）          │
└──────────┬──────────────────────────────┬──────────────┘
           │                              │
┌──────────▼──────────┐         ┌─────────▼────────────┐
│         OMP          │         │        Pi            │
│ Plugin/Hook/TTSR/TUI │         │ Extension/SDK/TUI    │
└─────────────────────┘         └──────────────────────┘
```

### 9.1 抽象边界

| 抽象 | 责任 | 不负责 |
|---|---|---|
| Runtime Adapter | Session、Context、Tool、Event、Command 与宿主桥接 | SBTD 业务判断 |
| Workflow | 阶段、路由、Gate 顺序和状态转换 | 专项知识全文 |
| Skill | 专项说明、检查清单、模板、脚本 | 全局阶段编排 |
| Rule | 可确定性匹配的提醒、中断、阻断、交付 Gate | 通用 LLM 推理 |
| Policy | 命令、路径、网络、密钥、审批 | 工程方法选择 |
| Validation | 命令选择、报告真实性、Evidence 状态 | 实现代码 |
| Kit | Skills、Rules、Prompts、Workflows、Schemas、Assets 的发行单元 | Runtime 内核 |
| Command Registry | 命令元数据、解析、Help、错误提示、文档与补全事实源 | Agent 业务推理 |
| Provider Gateway | 模型、认证、角色、Fallback、能力矩阵 | 业务工作流 |

### 9.2 P0 的职责分层：项目事实与 OMP 适配分离

| 层 | P0 主要职责 | 不负责 |
|---|---|---|
| `@kpi/omp-sbtd` | Session 开关、SBTD 分类、Route、Book Gates、Rule/Policy、Validation、Report、State 恢复 | 安装 Global Tools、覆盖用户文件 |
| OMP Global `AGENTS.md` | 用户级 Always-on 基线、Mode Contract、通用真实性和安全边界 | 项目命令、详细 Skill 流程、机器状态机 |
| 根 `<project-root>/AGENTS.md` | 跨 Agent 项目事实、命令、路径、约定、项目级安全和验证覆盖 | OMP Runtime Marker、自动 SBTD 状态机 |
| `<project-root>/.omp/AGENTS.md` | OMP/KPi Project Adapter、`@../AGENTS.md` 导入、Mode-aware SBTD Overlay、Plugin State Contract | 通用项目事实的重复副本 |
| Bundled/External Skills | Trellis、BDD、Validation、Book Reviews、Knowledge、Web/Mobile 等详细流程 | 全局阶段编排 |
| Onboard | 安装、双项目文件合并、迁移、备份、回滚、项目初始化、MCP 配置 | 判断当前 Session 是否 callable |
| External Tools | 实际分析、测试、索引、运行和报告 | 决定是否应该启用 SBTD |

核心关系：

```text
OMP Global AGENTS 提供用户级 Always-on 基线
  + 根 AGENTS 提供跨 Agent 项目事实
  + .omp/AGENTS.md 将根事实适配到 OMP 并声明 Mode Contract
  + Plugin 执行状态与门禁
  + Skills 提供详细流程
  + Onboard 管理安装与同步
  + Tools 提供真实执行能力
```

### 9.3 AGENTS 的固定路径与发现契约

#### OMP Global AGENTS

```text
$PI_CODING_AGENT_DIR/AGENTS.md
```

未设置 `PI_CODING_AGENT_DIR` 时：

```text
~/.omp/agent/AGENTS.md
```

规则：

- normal `init/reset` 写入或更新该路径；
- 不再默认写入 `~/.codex/AGENTS.md`；
- 检测到旧 SBTD `~/.codex/AGENTS.md` 时，只作为迁移输入，不自动删除或覆盖；
- Codex 目标平台仍使用 `$CODEX_HOME/AGENTS.md`（默认 `~/.codex/AGENTS.md`）；
- Doctor 报告 `configuredPath`、`resolvedPath`、`discovered`、`effective`、`digest`。

#### 根 Project AGENTS：跨 Agent 项目事实层

```text
<project-root>/AGENTS.md
```

规则：

- normal `init/reset` 和 `init-projects` 均可处理；
- 面向 OMP、Codex 和其他支持独立 `AGENTS.md` 的 Agent；
- 保存项目事实和项目级覆盖，不包含 OMP Runtime Marker；
- Onboard 只自动创建项目根文件，不自动创建更深层独立 AGENTS；
- 更深层 AGENTS 由项目自行维护；
- 若被同层更高优先级 Context 遮蔽，Doctor 必须报告，但 OMP Project Adapter 仍通过 `@../AGENTS.md` 导入根事实。

#### OMP Project Adapter：SBTD 运行时适配层

```text
<project-root>/.omp/AGENTS.md
```

该文件是 P0 Managed Mode 的必需项目适配文件，默认包含：

```markdown
# KPi OMP Project Adapter

@../AGENTS.md

<!-- kpi:managed:start target=omp-project-agents ... -->
...Mode Contract、Runtime Marker Contract、Tool Evidence Contract...
<!-- kpi:managed:end target=omp-project-agents -->
```

规则：

- normal `init/reset` 和 `init-projects` 均创建或更新；
- 通过 `@../AGENTS.md` 导入根项目事实，避免因 OMP Native Provider 优先级而丢失根文件内容；
- 只保存 OMP/KPi 适配逻辑，不复制完整项目事实；
- `.omp/` 目录可同时承载项目级 OMP 配置，但 Onboard 只管理其 AGENTS Managed Block；
- Onboard 默认只创建项目根的 `.omp/AGENTS.md`，不自动为 Monorepo 每个 Workspace 创建适配器；
- 若项目已有更近层级 `<subdir>/.omp/AGENTS.md`，OMP 的 nearest-native 规则可能覆盖根 Adapter；Doctor 必须报告，并要求该文件显式导入或继承根 Adapter；
- Marker 损坏、`@../AGENTS.md` 缺失或指向错误时，Managed Mode 为 `blocked/degraded`，不得静默猜测。

#### 更深层项目规则

- 独立 `<subdir>/AGENTS.md` 继续作为跨 Agent 局部规则；
- `<subdir>/.omp/AGENTS.md` 属于高级项目自维护能力，Onboard 不自动生成；
- 修改对应目录前，Plugin/Context Engine 必须检查实际有效的最近层级 OMP Adapter 和局部事实；
- 内容摘要用于避免 `@` 导入和 Context Bridge 的重复注入。

### 9.4 OMP Global AGENTS 的内容拆分

OMP Global AGENTS 仅包含：

#### A. Always-on 基线

无论 `/sbtd on/off`，始终有效：

- 当前项目事实、代码、配置、测试和更深层规则优先；
- 不假设可选工具、MCP、Provider 或索引可用；
- 优先最小、可验证、可回滚修改；
- 不读取、打印或持久化密钥；
- 破坏性、安装、网络、发布和生产数据操作需要确认；
- 不声称未运行的检查通过；
- 不把 mock/contract/smoke 结果冒充 full-stack；
- 不扩大用户范围。

#### B. SBTD Mode Contract

```text
mode=enforced
  → 遵循 Plugin 提供的 Stage、Route、Book Gate 和 Validation State

mode=advisory 或无 Runtime Marker
  → 不自动启动 SBTD 编排；仍遵守 Always-on 基线，
    用户可手动调用 Skills
```

#### C. 工具职责与输出真实性摘要

只保留 Trellis/GitNexus/Playwright/Maestro/MCP/RTK 的高层不可替代关系，以及 `passed/failed/blocked/skipped/not-needed` 的真实性边界。详细流程属于 Plugin/Skills/Validation。

OMP Global AGENTS 不直接承载完整状态机、Book Gate Predicate、报告算法、External Skill 事务或 Provider 登录实现。

### 9.5 根 Project AGENTS 的内容拆分

根 Project AGENTS 是跨 Agent 项目事实层，建议结构固定为：

```text
1. Managed Metadata
2. Project Facts
3. Commands and Validation
4. BDD Conventions
5. Trellis and Task Artifacts
6. Docs / Context / ADR
7. UI / Component / Platform Facts
8. Cross-repo / Contract / Environment Facts
9. Protected Paths and Project-specific Safety
10. User-owned Project Notes
```

具体内容：

- Package Manager、Workspace、源码和测试路径；
- `lint/test/typecheck/build/e2e` 项目原生命令；
- Feature 路径、Gherkin 语言和 Traceability 约定；
- Trellis 是否采用、Workflow/Spec/Task 路径；
- `docs/CONTEXT.md`、ADR、设计文档和公开边界；
- UI Stack、shadcn、React Bits、Mobile/Web 平台事实；
- Report 目录、CI、Release 和部署约定；
- 依赖的其他仓库、Contract、环境、账号和数据准备事实；
- Generated/Protected/Sensitive 路径；
- 对用户级 Always-on Baseline 的项目级覆盖。

根 Project AGENTS 不包含：

- `/sbtd on/off` 和 Runtime Marker；
- 自动 SBTD 分类与 Book Gate Predicate；
- Plugin Session State；
- 通用 RTK/Caveman 状态机；
- 通用 Web/Mobile 报告算法；
- External Skill 安装和 Provider 配置；
- 可由对应 Skill 延迟加载的详细操作清单。

### 9.6 `.omp/AGENTS.md` 的内容拆分

OMP Project Adapter 只保存：

```text
1. @../AGENTS.md 项目事实导入
2. KPi Plugin/Kit 版本和 Managed Metadata
3. sbtd-runtime Marker Contract
4. enforced/advisory Conditional SBTD Overlay
5. Plugin State 是 Gate/Stage 的机器事实源
6. Tool configured ≠ callable 的 OMP 运行时边界
7. 项目级 OMP 特有覆盖（如有）
8. User-owned OMP Notes
```

示例：

```markdown
# KPi OMP Project Adapter

@../AGENTS.md

<!-- kpi:managed:start target=omp-project-agents ... -->
## Runtime Contract

只有当前 Session 存在有效的
`<sbtd-runtime mode="enforced" ... />` 时，才自动执行：

- SBTD 分类与 Route
- Book Gate Plan
- 自动 Skill 路由
- Tool / Stage / Delivery Gate
- Validation 与 Report 闭环

当 `mode="advisory"` 或缺少有效 Marker 时，不自动启动上述流程；
继续遵守导入的项目事实、Always-on 安全和真实性边界。
<!-- kpi:managed:end target=omp-project-agents -->
```

### 9.7 Mode-aware AGENTS 与 `/sbtd off`

Plugin 在每个主要 Turn 注入：

```xml
<sbtd-runtime
  mode="enforced|advisory"
  route="..."
  stage="..."
  state-version="...">
</sbtd-runtime>
```

- OMP Global AGENTS 和 `.omp/AGENTS.md` 的 Conditional Section 仅在 `mode=enforced` 时自动触发；
- 根 Project AGENTS 不依赖 Runtime Mode，始终提供项目事实；
- `/sbtd off` 切换为 `advisory`，停止自动流程，但不删除当前 Session 已加载 Context；
- Skills 保持可发现，用户仍可手动调用；
- 完全不加载 SBTD Context 需要新 Profile/Agent Directory，P1 可提供 `kpi run --sbtd=off`。

### 9.8 Managed、Needs-onboard、Degraded 与 Blocked

| 模式 | 条件 | 行为 |
|---|---|---|
| `managed` | Plugin、固定 Kit、OMP Global AGENTS、根 Project AGENTS、OMP Project Adapter、Required Skills 和项目上下文完整 | 可进入 `enforced` 或 `advisory` |
| `needs-onboard` | Plugin 与内置 Kit 可用，但 OMP Global AGENTS、OMP Project Adapter、Required Skills 或 normal Onboard 基线缺失 | 允许 Help/Doctor/Plan；引导 Onboard |
| `degraded` | Global/Adapter 基线存在，但根 Project AGENTS、局部项目事实或部分 Context 缺失 | 不猜测项目命令/路径，高风险决策要求确认 |
| `blocked` | 当前 Route 的 Required Gate、Skill、项目事实、Adapter Contract 或安全前提缺失 | 阻断对应阶段并给出恢复方法 |

说明：OMP Project Adapter 因 Onboard 未完成而缺失时为 `needs-onboard`；用户通过高级选项明确跳过并接受能力降级时为 `degraded`。

### 9.9 Kit 单一来源与三目标 AGENTS 同步

KPi Kit 固定保留 `640-skills` 上游快照及转换资产：

```text
kuno-workflow-kit/
├── upstream/
│   ├── upstream.lock.json
│   └── sbtd-workflow-onboard/...
├── transforms/
│   ├── agents-section-map.yaml
│   └── transform-version.json
├── generated/
│   └── agents/omp/
│       ├── AGENTS.global.md
│       ├── AGENTS.project-root.md
│       └── AGENTS.project-omp.md
├── workflows/
├── rules/
├── skills/
├── schemas/
└── sync-reports/
```

`agents-section-map.yaml` 必须为上游 AGENTS 每个语义 Section 指定唯一 Owner：

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

允许 `owner: split`，但每个 Split Target 必须明确列出，且不得形成无人负责或重复冲突。

映射原则：

| 上游内容 | 目标 |
|---|---|
| 项目事实优先、最小变更、安全、真实性 | OMP Global Always-on |
| 项目命令、路径、BDD/Trellis/Docs/UI/CI 事实 | 根 Project AGENTS |
| OMP Runtime Marker、Mode Contract、Tool callable 边界 | OMP Project Adapter |
| SBTD 分类、Book Gate、Delivery Gate | Plugin Workflow/Rule |
| BDD/Trellis/GitNexus/Web/Mobile 详细流程 | 对应 Skills + Plugin Tool Evidence |
| RTK/Caveman 自动状态 | Plugin Policy/Session + Skills |
| 报告状态和 Artifact 真伪 | Validation/Report Schema |
| 工具安装、MCP 配置、模板写入 | Onboard |

规则：

1. 每个上游 Section 必须被映射；
2. 新增、删除、重命名或摘要变化生成 Sync Diff；
3. 未映射 Section 为 `needs-review`，阻断 Kit 发布；
4. 每次生成记录 Source Repo、Commit、Source Digest、Transform Version 和三个 Generated Digest；
5. Conformance Tests 检查生成文本、机器规则和 Skill Trigger 无语义漂移。

### 9.10 三类 Installed AGENTS 的 Managed Block

三个目标分别维护独立 Managed Block：

```markdown
<!-- kpi:managed:start
source=KunoLu/640-skills
commit=<full-sha>
transform=<version>
target=omp-global-agents|project-root-agents|omp-project-agents
digest=<sha256>
-->
...由 Kit 生成的内容...
<!-- kpi:managed:end -->
```

规则：

- Managed Block 外内容归用户或项目所有；
- Global、Root Project、OMP Project Adapter 的 Digest 独立；
- 文件不存在：创建对应文件和 Block；
- 已有相同 Target Block：只替换该 Block；
- 匹配已知旧 SBTD 模板 Digest：备份后迁移；
- 未知自定义内容且无 Marker：Plan 标记 `merge-required`；
- Marker 损坏、重复 Target 或嵌套：`blocked`；
- `.omp/AGENTS.md` 的 Managed Block 必须验证 `@../AGENTS.md`；
- Project `.gitignore` 继续使用逐行幂等策略。

### 9.11 上游更新与用户环境同步

维护者同步流程：

```text
固定新 upstream ref/SHA
  → 比较 AGENTS/Catalog/Skills
  → 运行 Section Mapping
  → 生成三类 OMP AGENTS 模板
  → 更新 Plugin Rules/Skills
  → 生成 sync-report.json/.md
  → 运行 Conformance/Golden Tests
  → 发布新的 Plugin + Kit 版本
```

用户同步流程：

```text
更新 Plugin
  → /sbtd onboard plan
  → 比较三个 installed managed digest 与 bundled kit digest
  → 显示 Global/Root Project/OMP Adapter Diff
  → 用户确认
  → /sbtd onboard reset
  → verify
  → 新 Session / 完整 Reload
```

P0 不在 `/sbtd on` 时联网拉取 `main`，也不在 Plugin 安装时自动重写用户文件。

### 9.12 P0 运行链

```text
omp plugin install @kpi/omp-sbtd
  └─ 只安装 Plugin Runtime + 固定版本的只读 SBTD Kit
       ├─ 不写 AGENTS
       ├─ 不安装 Trellis/GitNexus/Skills
       ├─ 不配置 MCP
       └─ 不修改项目

omp
  ├─ /sbtd doctor
  ├─ /sbtd onboard plan
  └─ /sbtd onboard init（用户确认后）
       ├─ 验证 OMP，不在活跃 OMP Session 内重装 Runtime
       ├─ 检查/安装 Trellis、GitNexus 和 Required Skills
       ├─ 写入 `$PI_CODING_AGENT_DIR/AGENTS.md`（默认 `~/.omp/agent/AGENTS.md`）
       ├─ 写入逐项目根 `AGENTS.md`
       ├─ 写入逐项目 `.omp/AGENTS.md` Adapter
       ├─ 按用户选择配置用户级 OMP MCP
       ├─ 在 Onboard 例外范围内初始化项目 Trellis
       └─ 输出逐项目状态、备份、回滚与阻断

新 OMP Session / 完整资源 Reload
  └─ /sbtd on
       ├─ 加载 Plugin/Core 基础规则
       ├─ 加载 Global、Root Project、OMP Project Adapter 与局部 AGENTS 层级
       ├─ 检测 Skills 与项目事实
       ├─ 检测 Trellis/GitNexus/MCP 的当前 Session 强证据
       └─ 后续每轮执行分类、Gate、验证和报告
```

### 9.13 插件包构成与零副作用安装

`@kpi/omp-sbtd` 包含：

```text
plugin manifest
slash commands
extensions / hooks
custom tools
workflow state
rules / schemas
report renderer
固定版本、只读的 sbtd-workflow-onboard Kit 快照
  ├── catalog.json / schema
  ├── onboard.py
  ├── Global / Root Project / OMP Project Adapter templates
  ├── bundled Skills
  └── external stable metadata/assets
```

该内置快照是 **Bootstrap Source**，用于 Doctor、Plan 和显式 Onboard；它不会在插件安装阶段自动复制到用户全局 Skill 目录。

安装插件时允许的持久变化仅限：

- OMP 自己管理的 Plugin Package/Registry/Lock；
- 插件 Settings Schema 和 Feature Metadata；
- OMP Session 中由插件创建的非敏感状态。

安装插件时禁止：

- 使用 Package `postinstall` 写入任何由 Onboard 解析出的全局 AGENTS 目标；
- 创建或覆盖项目 `AGENTS.md`、`.gitignore`、`.trellis/`、`.gitnexus/`；
- 安装全局 Trellis、GitNexus、RTK、Java、Maestro 或 npm 包；
- 安装 Global/Project Skills；
- 修改 `~/.omp/agent/mcp.json`；
- 登录 Provider、修改 Model Role 或读取认证文件；
- 执行 GitNexus Analyze、Trellis Init、Playwright Install；
- 创建项目报告或 `.kpi/` 目录。

### 9.14 插件与 Onboard 的安装/配置矩阵

| 内容 | 安装插件时 | `/sbtd on` 时 | normal `/sbtd onboard init/reset` | `init-projects` |
|---|---|---|---|---|
| Plugin Runtime/Commands/Hooks | 安装 | 加载 | 不重复安装 | 不涉及 |
| 固定只读 Kit Snapshot | 随插件安装 | 只读校验 | 作为安装源 | 作为项目模板源 |
| OMP Runtime | 前置条件，不安装 | 只读检查 | 只验证；兼容命令可单独安装，但活跃 Session 不重装 | 不检查 |
| OMP Global AGENTS | 不写 | 读取 `$PI_CODING_AGENT_DIR/AGENTS.md`，默认 `~/.omp/agent/AGENTS.md` | 以独立 Managed Block 写入/更新 | 不检查、不修改 |
| 根 Project AGENTS | 不写 | 读取 `<project-root>/AGENTS.md` | 以 `project-root-agents` Block 按用户选择写入 | 同左 |
| OMP Project Adapter | 不写 | 读取 `<project-root>/.omp/AGENTS.md`、校验 `@../AGENTS.md` 和 nearest-native Shadow | 以 `omp-project-agents` Block 创建/更新 | 同左；不处理全局状态 |
| Project `.gitignore` | 不写 | 仅读取 | 逐行幂等补齐 | 逐行幂等补齐 |
| Bundled/External Skills | 不安装 | 发现/延迟加载 | 全局安装、校验、事务回滚 | 不检查、不安装 |
| Trellis CLI | 不安装 | 检测 | normal 模式全局安装/验证 | 不安装，只使用已有 CLI |
| Project `.trellis/` | 不创建 | 检测 | 满足确认条件时初始化并检查 Bootstrap | 满足条件时使用已有 CLI 初始化 |
| GitNexus CLI | 不安装 | 检测 | normal 模式全局安装/验证 | 不安装 |
| GitNexus MCP | 不配置 | 检查是否 callable | 仅用户明确选择时写 OMP 用户级 MCP | 不配置 |
| GitNexus Index/Hook | 不创建 | 检查 Current/Stale/Branch | 默认不自动 Analyze，不新增 Hook | 不涉及 |
| Chrome/Playwright/Maestro MCP | 不配置 | 检查是否 callable | 仅用户明确选择时配置用户级 OMP MCP | 不配置 |
| Playwright CLI | 不安装 | 检查项目适用性 | 适用且用户确认时安装项目 devDependency | 同左 |
| Java/Maestro CLI | 不安装 | 检查 | 命中 Mobile 条件且用户确认时安装/引导 | 不安装 |
| RTK/Caveman | 不安装 | 检查 | 保留当前可选确认策略 | 不安装 |
| React Bits | 不安装 | 检查项目事实 | 仅适用项目和用户确认时配置 | 同左 |
| Provider/Login/Model | 不修改 | 只读报告 OMP 当前状态 | 不修改 | 不涉及 |

---

## 10. SBTD Runtime Workflow

### 10.1 主状态机

```text
intake
  → lessons-gate
  → repo-inspect
  → sbtd-classify
  → tool-evidence
  → route
  → requirement-clarify
  → specification
  → book-gate-plan
  → before-dev
  → implement
  → targeted-validate
  → full-validate
  → runtime/e2e-validate
  → review
  → release-gate
  → report
  → lessons-record
  → completed | blocked
```

### 10.2 状态要求

每个阶段包含：

```ts
interface StageState {
  id: string;
  status: "pending" | "running" | "passed" | "blocked" | "skipped" | "not-needed";
  reason?: string;
  evidence?: EvidenceRef[];
  startedAt?: string;
  completedAt?: string;
}
```

要求：

- 阶段跳过必须有原因；
- Hard Gate 不允许直接从 `pending` 跳到 `skipped`；
- Runtime Resume/Compaction 后恢复当前阶段；
- 用户覆盖 Route 必须记录来源和影响；
- 所有状态可导出为 JSON 与 Markdown。

### 10.3 SBTD 分类器

```ts
interface SBTDClassification {
  sdd: "required" | "recommended" | "not-needed" | "blocked";
  bdd: "required" | "recommended" | "not-needed" | "blocked";
  tdd: "required" | "recommended" | "not-needed" | "blocked";
  ddd: "required" | "recommended" | "not-needed" | "blocked";
  route: WorkflowRouteId;
  reasons: string[];
  userVisibleBehavior: boolean;
  existingProductionCode: boolean;
  existingBehaviorBug: boolean;
  dataRisk: boolean;
  productionPathRisk: boolean;
  crossRepoScope: boolean;
}
```

分类原则：

- 复杂度决定是否进入 Trellis；
- 协作形态决定是否进入 Channel；
- 用户可见行为默认 BDD；
- Bug、高风险逻辑和回归敏感路径评估 TDD；
- 领域语言/边界不清时 DDD；
- 需求需要长期沉淀时 SDD；
- 不能只凭用户用了 “复杂” 一词切换重流程。

---

## 11. SBTD 核心门禁

### 11.1 Book Gate Plan

每个开发任务进入实现前生成：

```text
Book Gate Plan
- Gate
- Required / On-demand
- Objective predicate
- Planned phase
- Gate state
- Reviewer status
- Evidence
```

Gate State 只能是：

```text
planned → running → passed | blocked
not-required
```

### 11.2 五类 Gate

| Gate | 客观强制触发条件 | 通过状态 | 必须阶段 |
|---|---|---|---|
| DDD Boundary Review | 每次完整 `grill-with-docs`；或独立命中领域歧义 | `confirmed` | 需求确认/PRD/design/实现前 |
| DDIA Data Design Review | 持久/共享数据、Schema、Migration、共享/跨请求/跨进程缓存、异步/跨服务、所有权、恢复 | `confirmed` | Design 稳定或实现前 |
| Legacy Change Safety Review | 既有行为 Bug；行为不清、弱测试、隐藏依赖、高回归风险 | `characterized` | 首次行为修改前 |
| Refactoring Review | 修改任何既有生产代码 | `proceed` | 首次实现编辑前 |
| Release Readiness Review | API/Auth/Billing/Notification/Job/Queue/Data Pipeline/Integration/Deployment 等生产路径 | `ready` | 所有测试 Gate 与项目验证后 |

### 11.3 受控 Legacy/Refactoring 回路

```text
Legacy: seam-required
  → Refactoring Review: safety-seam-only
  → 仅实现行为保持的测试 Seam
  → 验证观测等价
  → 回到 Legacy 达到 characterized
  → Refactoring Review normal
  → 实际 Feature/Fix 修改
```

任何 `needs-*`、`seam-required`、`refactor-first` 或 `blocked` 都不能被当成通过。

---

## 12. Trellis 工作流要求

### 12.1 强证据

满足至少一项才判定项目使用 Trellis：

- `.trellis/`；
- `.trellis/workflow.md`；
- `$trellis-*`；
- 项目 AGENTS 明确声明。

### 12.2 运行时规则

- 必须读取 `.trellis/workflow.md`；
- 按需读取 `.trellis/spec`；
- Lessons 先读短入口和 Index，不默认读取全部历史；
- 活跃任务优先读取 `prd.md / design.md / implement.md`；
- 不手动跳过 before-dev/check/finish-work；
- Trellis 升级后先 `trellis update`；
- 不绕过 dirty-data、manifest ownership、safe-name 等安全保护；
- BDD 是 Overlay，不是独立 Trellis Template；
- `native` 为默认；
- `tdd` 仅在明确 tests-first/high-risk 时切换；
- Channel 只在明确协作形态下启用。

### 12.3 初始化边界

普通项目工作流：

- 检测到项目 AGENTS 但无 `.trellis/` 时提示用户；
- 默认不得代用户执行 `trellis init`。

Onboard `init/reset` 例外：

- Trellis CLI 可用；
- 项目无 `.trellis/`；
- 用户已确认 username/platform flags；
- 使用 `--yes --skip-existing`；
- 初始化后检查 `00-bootstrap-guidelines`；
- 某项目 `bootstrap-required` 不得阻止其他项目继续检查。

---

## 13. BDD 行为事实源

### 13.1 强制范围

以下任一用户或外部系统可观察的变化均默认要求持久 BDD：

- UI、API、CLI；
- 导出文件；
- Notification；
- Permission、Error、Status；
- 外部 Integration 的可观察结果；
- 用户可见 Bug Fix。

### 13.2 行为 SOT

- 项目已有 `.feature` 路径/语言/命名时遵循项目；
- 新项目默认 `features/<capability>.feature`；
- Monorepo 写入 owning workspace；
- `.trellis/tasks/**` 是流程产物，不是默认长期行为 SOT；
- PRD 表达意图，Feature 表达行为，Design/Implement 表达技术方案。

### 13.3 语言 Gate

若项目没有 `.feature` 且用户未指定：

- 场景标题、描述、Step Text 使用简体中文；
- Gherkin 结构关键字使用英文；
- 不添加 `# language: zh-CN`；
- 写入前必须显式报告语言决策。

### 13.4 Bug Fix 顺序

```text
正确行为 Scenario
  → 失败回归测试
  → 最小修复
  → 场景、测试、代码一致性验证
```

### 13.5 跨仓 Evidence Gate

| 状态 | 含义 |
|---|---|
| `complete` | 行为、Contract、环境、账号、数据和结果完整 |
| `contract-only` | 有可靠 Contract/Schema/Fixture/真实样例，但链路不可运行 |
| `environment-only` | 环境可运行，但 Contract/Owner/副作用不完整 |
| `missing` | 存在关键事实缺口 |

缺失事实时不得猜测 Scenario 或 Mock。Mock 只能来自 Contract、Schema、真实样例、既有 Fixture、Launch Args 或用户明确确认。

### 13.6 BDD Sync 与 Knowledge Ingest

- `sync/同步`：全工作树行为审计，包含未提交变化，不是 Diff-only；
- 多仓时确认其他仓是否变化，并按需扫描；
- `read/读取` 且无写意图：进入只读 Knowledge Ingest；
- 目标 Branch/Tag 先解析为精确 SHA；
- 不切换开发者活动 Worktree；
- 不向 Feature 增加 ID、Owner Tag 或 Runner；
- 聚合目录是可重建派生视图，源 `.feature` 仍是 SOT。

---

## 14. 工具可用性与职责边界

### 14.1 强证据原则

可用证据包括：

- 当前 Session Tool List；
- Tool Search 返回；
- MCP 可见性与可调用验证；
- CLI `--version/status` 成功；
- 当前项目有效配置/索引；
- 项目文档明确声明并经实际验证。

以下不构成可用证据：

- Marketplace/Catalog 展示；
- 已安装提示但 Session 未加载；
- 其他 Session 的历史状态；
- 本地目录存在但命令损坏；
- 模型凭记忆声称可用。

### 14.2 GitNexus

只有同时满足：

1. GitNexus MCP 可用；
2. 当前项目索引有效。

规则：

- 修改前做 Impact；
- 修改后做 Detect Changes；
- Stale Index 先刷新；
- 结果只作 Advisory；
- 回到源码、Diff、测试与调用链复核；
- Branch 和高级 Mode 明确记录；
- PDG/Taint/Trace 是 Opt-in；
- 不静默新增 Git Hook；
- 不可用时跳过，不应阻塞普通任务，但需在影响风险时说明。

### 14.3 Web/Mobile 工具

| 工具 | 主责 | 不负责 |
|---|---|---|
| Chrome DevTools MCP | 真实浏览器诊断、Console/Network/Storage/Trace/Screenshot | CI Gate |
| Playwright MCP | 探索、Accessibility Snapshot、Locator 辅助 | 项目 Playwright Test |
| Playwright CLI | 可重复 Web E2E、回归、CI | 临时探索 |
| Maestro CLI | Mobile/Hybrid E2E、可选 Web Smoke | Web 主回归 |
| Maestro MCP | 设备/View Hierarchy/Screenshot/Flow 辅助 | 替代 CLI |
| web-ui-autotest-generator | 生成/审计可入库 Playwright 测试资产 | 执行 E2E |
| maestro-mobile-e2e | 从 Feature 派生 Repo-resident Maestro Flow | 替代 BDD/CLI |
| seo-geo | 公开 Web 搜索可见性 | Web Runtime/E2E/Release Gate |

同一浏览器上下文同一时间只能有一个 Controller。

### 14.4 RTK 与 Caveman

- `rtk` 是 Shell 输出压缩层，不是 Test Runner；
- 报告型命令默认原生或 report-safe；
- 报告缺失、mtime/size 未更新、Cache Hit 时原生命令重跑；
- 最终报告 `used / skipped-for-report / fallback-native / not-available / not-needed`；
- `caveman` 只影响对话表达，不改变 Workflow、代码、工具、验证和 Gate；
- KPi 必须在 Compaction/Handoff 中保留自动压缩状态，但安全确认、PRD、Gate、失败、风险和最终报告属于完整输出保护区。

---

## 15. Validation 与 Evidence

### 15.1 最小到完整验证

```text
Focused Validation
  → Affected Scope
  → Planned Full Validation
  → Formal Report Artifact Gate
  → Final Full Rerun
```

### 15.2 正式报告要求

正式验证必须满足：

- Runner 原生报告或明确的 Raw Report；
- 当前运行创建或刷新；
- 路径可追踪；
- 同目录、同 Stem 的中文 Markdown 汇总；
- 不包含密钥、真实账号、PII、生产敏感数据；
- stdout-only 仅为诊断；
- 无法落地报告时标记 `blocked`。

### 15.3 E2E Mode

```text
full-stack
contract-backed
mock-backed
app-mocked
backend-only
smoke-only
blocked
not-needed
```

非 `full-stack` 结果不得升级为完整链路通过。

### 15.4 Web 报告

默认正式路径：

```text
tests/e2e/reports/html/
playwright-report-{feature}-{branch_slug}-{YYYY_mm_dd}-{HH_MM_SS}.html
playwright-report-{feature}-{branch_slug}-{YYYY_mm_dd}-{HH_MM_SS}.md
```

临时目录或默认 `index.html` 不是正式命名报告。

### 15.5 Mobile 报告

Flow：

```text
maestro/flow/**/*.yml
```

正式报告：

```text
.maestro/reports/
maestro-report-{flow}-{branch_slug}-{YYYY_mm_dd}-{HH_MM_SS}.xml|html
maestro-report-{flow}-{branch_slug}-{YYYY_mm_dd}-{HH_MM_SS}.md
```

要求 Java 17+；stdout-only Maestro 运行不能满足正式 Gate。

### 15.6 API 报告

中文 Markdown 必须包含 URI Coverage Matrix：

| Coverage Scope | Method | URI Path | Test/Case | Expected Status/Side Effect | Feature/Contract/Schema |
|---|---|---|---|---|---|

高层服务名、域名或脚本名不能代替 URI Path。

### 15.7 Evidence 状态

```text
Evidence Source:
  developer-local | ci | knowledge-server | not-needed

Source Revision:
  exact | dirty | unknown | not-needed

Environment Alignment:
  verified | unverified | mismatch | not-needed

Evidence Publication:
  local-only | published | blocked | not-configured | not-needed
```

Dirty Worktree 只能作为 Local Evidence，不能冒充 PR Head 的正式证明。

---

## 16. Knowledge Base 与 Lessons

### 16.1 Knowledge Base P1.1

KPi 应保留 `knowledge-base-integration` 的 P1.1 契约：

- Product Registry；
- Workspace Mapping；
- Evidence Policy Decision；
- Target Ref → Exact SHA；
- Revision Set；
- 无 ID Gherkin Catalog；
- Static/Manifest Binding；
- Conflict Candidates；
- Idempotent Ingest/Smoke；
- Detached Isolated Worktree；
- Local/Command Runner Adapter；
- Fresh Artifact Collection；
- Manifest、Checksum、Metrics；
- `Evidence Publication: not-configured`。

Evidence Store、PR Check、Invalidation、Retention、Quarantine 与远端 Gate 属于后续 P2/P3 扩展，不应在 P0 伪装为已实现。

### 16.2 Lessons 分层

Trellis 项目默认：

```text
.trellis/spec/lessons.md
.trellis/lessons/index.md
.trellis/lessons/topics/*.md
.trellis/lessons/archive/YYYY-QN.md
```

非 Trellis 项目可使用：

```text
docs/lessons.md
docs/lessons/index.md
docs/lessons/topics/*.md
docs/lessons/archive/YYYY-QN.md
```

读取策略：

- 默认只读短入口；
- 根据 Tag、Error、Tool、Language、Topic 命中详情；
- 不默认读取 Archive。

必须记录 Durable Lesson 的典型情况：

- Bug Fix、Rollback；
- Tool 判断错误、Mode 切换错误；
- Trellis Stage 错误；
- Task Artifact 缺失或冲突；
- GitNexus Impact 不一致；
- Channel Context Loss、Recursive Dispatch、Worker 异常退出。

---

## 17. Channel 与多 Agent

### 17.1 启用原则

- 复杂度决定是否 Trellis；
- 协作形态决定是否 Channel；
- 大任务优先 Parent/Child Task；
- 仅因文件多或任务复杂不得自动启用 Channel；
- Review/Validation 可主动做 Preflight；
- 未经用户请求或确认不得 Spawn Worker。

### 17.2 所有权

- Review/Validation Worker 默认只读；
- 同一 Checkout 同时只有一个 Writer；
- 同一验证环境同时只有一个 Validation Controller；
- Docker、Migration、Browser E2E、Deploy 等环境敏感验证由主 Session 串行控制；
- Worker 不得自行 Stage/Commit/Archive/Finish/Push/Deploy。

### 17.3 Worker 输出

```text
Verdict: pass | concerns | block
Scope reviewed
Evidence
Findings
Required changes
Optional suggestions
Validation gaps
Files referenced
Confidence
Should write back to
```

完成后必须写回 Task Artifacts 或说明无需长期保留，并清理 Runtime State。

---

## 18. Onboard 子系统

### 18.1 当前权威接口

P0/P1 直接包装 `sbtd-workflow-onboard/scripts/onboard.py`：

```text
check
check-projects
plan
init
reset
init-projects
check-agent-cli
install-agent-cli
ensure-npm
install-rtk
install-caveman
install-java
install-maestro
install-playwright-cli
install-external-skills
promote-external-skills-stable
```

### 18.2 两种模式

#### Normal `init/reset`

- 检查/安装目标 Agent CLI；
- 确保 npm；
- 全局 Trellis、GitNexus；
- Bundled/External Skills；
- 可选 RTK/Caveman/Java/Maestro；
- 用户级/全局 MCP；
- 全局 AGENTS；
- 逐项目 AGENTS、`.gitignore`、Trellis、Playwright、React Bits。

#### Project-only `init-projects`

禁止检查、安装或修改：

- Agent CLI；
- npm/Node/NVM；
- Trellis/GitNexus 全局工具；
- Global Skills；
- Global AGENTS；
- MCP。

只处理：

- 项目 AGENTS；
- `.gitignore`；
- 已有全局 Trellis CLI 下的项目初始化；
- Bootstrap 检测；
- 适用的 Playwright/React Bits 决策。

### 18.3 目标平台兼容

现有 Onboard 支持：

```text
codex
claude
kimi
oh-my-pi | omp
```

KPi P1 默认将自身 Runtime 映射到 `omp`。其他平台参数用于保留 Onboard 兼容和未来 Runtime 委托。

### 18.4 多项目契约

- 仅接受已存在的绝对目录；
- 英语逗号分隔；
- Canonicalize；
- 去重；
- 每个项目独立报告；
- 一个项目 `bootstrap-required` 不阻止其他项目；
- 聚合 Trellis 优先级：

```text
failed
> blocked
> needs-user
> bootstrap-required
> success
> skipped
```

### 18.5 Catalog 与 External Skills

`catalog.json` 为来源事实源，必须校验：

- ID 唯一；
- Path Containment；
- Repository URL；
- Kind/TargetRole；
- Source 类型；
- Bundled Skill Frontmatter Identity；
- Schema。

External Skills：

- `auto`：Upstream Group 失败时才回退 Stable；
- `upstream`：严格当前上游；
- `stable`：严格 Vendored Stable；
- 全部先 Stage，再修改目标；
- Commit 失败尝试完整 Rollback；
- Target-side 失败不得触发 Source Fallback；
- Rollback 不完整时保留备份目录；
- Stable Promotion 必须固定完整 Commit SHA、Digest 和 License。

### 18.6 Bundled Skills

1. `sbtd-workflow-onboard`
2. `trellis-workflow`
3. `trellis-channel`
4. `project-validation`
5. `web-ui-autotest-generator`
6. `gherkin-bdd`
7. `knowledge-base-integration`
8. `maestro-mobile-e2e`
9. `lessons-record`
10. `book-refactoring-pass`
11. `book-legacy-change-safety`
12. `book-ddd-distilled-modeling`
13. `book-ddia-data-design`
14. `book-release-readiness`
15. `seo-geo`

### 18.7 Required External Skills

- `diagnosing-bugs`
- `tdd`
- `grill-me`
- `grill-with-docs`
- `grilling`
- `domain-modeling`
- `codebase-design`
- `handoff`
- `writing-great-skills`
- `to-spec`
- `to-tickets`
- `ui-ux-pro-max`
- `impeccable`
- `shadcn`

### 18.8 P0 Plugin 安装边界

P0 必须把以下两个动作明确分开：

```text
omp plugin install @kpi/omp-sbtd
  = 安装执行层与只读 Bootstrap Source

/sbtd onboard init|reset|init-projects
  = 经 Plan 和用户确认后修改环境
```

Plugin 安装不得触发 Onboard。首次 `/sbtd on` 只能运行只读 Preflight，并根据实际证据返回 `managed`、`needs-onboard`、`degraded` 或 `blocked`。

`/sbtd onboard` 使用插件内固定版本 Kit 的 `onboard.py` 作为 P0 权威实现。为避免供应链与行为漂移：

- Plugin 版本必须固定兼容的 Kit Commit/Tag；
- 不在每次启用时联网拉取 `main`；
- 用户显式执行更新/重置时才评估新版本；
- Plan 必须显示 Plugin Version、Kit Revision、Source Policy 和所有目标路径；
- Onboard 写入后优先使用新 Session 重新发现 AGENTS、Skills 与 MCP；
- Runtime Reload 不完整时不得声称新配置已经 callable。

### 18.9 AGENTS 路径解析和安装策略

#### OMP Global AGENTS

```text
PI_CODING_AGENT_DIR 已设置：
  $PI_CODING_AGENT_DIR/AGENTS.md

未设置：
  ~/.omp/agent/AGENTS.md
```

normal `init/reset` 必须使用 OMP 原生路径。旧 `~/.codex/AGENTS.md` 只作为迁移参考，不自动删除。

#### 根 Project AGENTS

```text
<project-root>/AGENTS.md
```

- normal 与 project-only 均可写入；
- 保存跨 Agent 项目事实；
- 只创建项目根文件；
- 使用 `target=project-root-agents` Managed Block；
- 块外用户/团队内容保留。

#### OMP Project Adapter

```text
<project-root>/.omp/AGENTS.md
```

- normal 与 project-only 均必须生成，除非用户显式接受 `degraded`；
- 使用 `target=omp-project-agents` Managed Block；
- Managed 内容必须包含 `@../AGENTS.md`；
- 不自动生成 Workspace/Package 级 `.omp/AGENTS.md`；
- 若已有最近层级 Adapter，Doctor 报告实际生效路径和继承关系。

#### 参数迁移

P0 新增：

```text
--skip-project-root-agents
--skip-omp-project-adapter   # 高级选项，结果至少 degraded
```

兼容规则：

- 旧 `--skip-project-agents` 在 P0 映射为 `--skip-project-root-agents`，并输出弃用警告；
- 它不再跳过 `.omp/AGENTS.md`，以保证 OMP/KPi Adapter 可用；
- P2 可移除旧别名。

### 18.10 AGENTS 内容所有权和同步契约

上游 `AGENTS.global.md` 与 `AGENTS.project.md` 不直接原样复制，而先经过 Section Mapping：

| 上游内容类型 | KPi 目标 |
|---|---|
| 用户级最小变更、安全和真实性 | `omp-global-agents` |
| 项目命令、路径、BDD/Trellis/Docs/UI/CI 事实 | `project-root-agents` |
| Runtime Marker、Mode Contract、OMP callable 边界 | `omp-project-agents` |
| SBTD 分类、Book Gate、Delivery Gate | `plugin-workflow` / `plugin-rule` |
| BDD/Trellis/GitNexus/Web/Mobile 详细流程 | 对应 Skills + Tool Evidence |
| RTK/Caveman 自动状态 | Plugin Policy/Session + Skills |
| 报告状态和 Artifact 真伪 | Validation/Report Schema |
| 安装、MCP、模板写入 | Onboard |
| 不适用内容 | `ignored-with-reason`，必须人工批准 |

同步产物必须记录 Source File/Heading/Digest、Target Owner、Transform Version、三个 Generated Digest、Added/Changed/Removed/Unmapped Sections 和人工 Review 结论。

### 18.11 Onboard 与 Runtime 的职责分离

```text
Onboard：安装、三类 Managed Block 合并、迁移、备份、回滚、项目初始化
Runtime：读取有效 AGENTS、注入 Mode Marker、检测 Shadowing/Import、执行 Gate
```

Doctor 对三类文件分别报告：

```text
role
configuredPath
exists
discoveredByOMP
loadedByKPiBridge
effective
shadowedBy
imports
sourceCommit
managedDigest
localContentPreserved
```

### 18.12 `/sbtd on/off` 与 AGENTS 生命周期

- AGENTS 在 Session 启动时加载，`/sbtd off` 不删除 Context；
- 根 Project AGENTS 始终提供项目事实；
- Global 和 `.omp/AGENTS.md` 的 Conditional Section 根据 `enforced/advisory` 决定是否自动编排；
- `/sbtd off` 只关闭 Plugin 自动分类、Gate、Skill 路由和交付闭环；
- 完全隔离需新 Profile/Agent Directory；
- Onboard 文件写入和 `/sbtd on/off` 完全解耦。

### 18.13 Trellis 生命周期

#### Onboard 负责

- normal `init/reset` 检查并安装全局 Trellis CLI；
- 对每个项目检查 `.trellis/`；
- 仅在 Onboard 场景且 Username/Platform 已确认时执行 `trellis init --yes --skip-existing`；
- 检查 `00-bootstrap-guidelines`；
- 一个项目 `bootstrap-required` 不终止其他项目；
- `init-projects` 不安装全局 CLI，只使用已有 CLI，否则逐项目 `blocked`。

#### Runtime 负责

- 检测 CLI、`.trellis/`、`workflow.md`、Task Artifacts 与 Bootstrap 状态；
- 普通任务中不得自动执行 `trellis init`；
- Trellis 有强证据时加载 `trellis-workflow` 并遵守 `before-dev/check/finish-work`；
- 升级后按项目规则执行 `trellis update`；
- 小任务或非 Trellis 项目继续 Native Route；
- 仅当项目规则/任务事实要求 Trellis 且缺失时标记 `needs-user/blocked`。

建议状态：

```text
Trellis CLI: installed | missing | broken
Trellis Project: initialized | not-initialized | not-needed
Trellis Workflow: available | missing | stale | blocked
Trellis Bootstrap: ready | bootstrap-required | blocked | not-needed
```

### 18.14 GitNexus 生命周期

#### Onboard 负责

- normal `init/reset` 检查/安装全局 GitNexus CLI；
- 用户选择后配置 OMP 用户级 MCP（不是项目级凭据文件）；
- 检查项目是否有索引信号；
- 不静默新增自动刷新 Git Hook。

#### Runtime 负责

仅当以下条件同时满足时使用：

1. 当前 Session 存在可调用 GitNexus MCP；
2. 当前项目索引存在且对应目标 Branch；
3. 索引不是已知 Stale，或已成功刷新。

修改前执行 Impact，修改后执行 Detect Changes。若刷新失败：

- 结果降级为 Advisory；
- 记录失败命令/原因；
- 回到源码、Diff、测试、构建和真实调用链；
- 不阻断普通任务，除非项目 Hard Rule 明确要求。

建议状态：

```text
GitNexus CLI: installed | missing | broken
GitNexus MCP: callable | configured-only | unavailable | blocked
Project Index: current | stale | missing | branch-mismatch | blocked
GitNexus Use: impact-run | changes-run | advisory | skipped | not-needed
```

### 18.15 External Skills 的 Runtime 处理

Onboard 继续负责来源验证、Upstream/Stable、Staging、Transaction 和 Rollback。Plugin/Core 只负责：

1. 检测 Skill 当前是否可发现；
2. 按 Route/Gate 延迟加载；
3. Required Skill 缺失时阻断对应 Gate；
4. Optional Skill 缺失时使用明确降级；
5. 在 Doctor/Report 中记录 Skill 名称、版本、来源和状态。

插件不得为相邻能力或宽泛推荐静默安装 Skill。

### 18.16 Playwright、Maestro 与 MCP 生命周期

#### Playwright

- Onboard 只在项目已有依赖、配置、脚本或 E2E 目录时判定适用；
- 用户确认后安装到项目 devDependency，不全局安装；
- Runtime 使用项目原生命令，并执行正式 HTML/中文 MD Report Gate。

#### Maestro

- Onboard 检查 Java 17+ 和 Maestro，并保留用户确认；
- Runtime 还必须检查设备、App Artifact、AppId/BundleId、账号、数据和环境；
- CLI 可用而 MCP 不可用时仍可运行既有 Flow，但分开报告状态。

#### MCP

- Onboard 只在用户明确选择时写用户级/全局 MCP 配置；
- Runtime 只以当前 Session 的 callable tools 为准；
- 配置文件存在、Catalog 展示或历史授权不等于当前可调用；
- Auth、OAuth、Cookie、Token 不写入项目、报告或示例。

### 18.17 AGENTS/Plugin 一致性验收

P0/P1 必须维护以下测试矩阵：

| 场景 | 期望 |
|---|---|
| OMP Global + 根 Project + OMP Project Adapter + Plugin | `managed`；可在 `enforced/advisory` 间切换 |
| Plugin + 无 OMP Global AGENTS | `needs-onboard`，不进入完整 Enforced Mode |
| Plugin + 无 OMP Project Adapter | `needs-onboard` 或 `degraded`，不保证 Mode Contract |
| Plugin + 无根 Project AGENTS | `degraded`，不猜测项目事实 |
| 根 AGENTS 被 Native Adapter 遮蔽但 `@../AGENTS.md` 正常 | 项目事实仍有效，Doctor 记录 Shadowing/Import |
| AGENTS 已加载且 `/sbtd off` | Always-on/项目事实继续有效；Conditional SBTD 自动编排停用 |
| AGENTS 与 Kit 机器规则冲突 | Doctor `blocked`，不静默选择 |
| Project-only Onboard | 不修改 Global AGENTS/Tools/Skills/MCP |

---

## 19. Provider 与认证

### 19.1 总体策略

Provider 能力分为三层：

```text
ProviderAdapter
CredentialSource
RuntimeAdapter
```

```ts
type AuthMode =
  | "api-key"
  | "runtime-oauth"
  | "subscription-cli-delegation"
  | "gateway"
  | "local"
  | "none";
```

P0/P1 优先继承 OMP Provider 能力；P2 建立独立 Provider Gateway。

### 19.2 支持矩阵

| Provider/渠道 | P0/P1 实现 | P2/P3 目标 | 安全边界 |
|---|---|---|---|
| OpenAI BYOK | 由 OMP Provider/自定义 Provider 管理 | 原生 OpenAI Adapter | 仅引用 Key，不写项目 |
| Anthropic BYOK | 由 OMP 管理 | 原生 Anthropic Adapter | 同上 |
| OpenAI-compatible | OMP `models.yml` | 通用 Adapter | Base URL Allow/Approval |
| Anthropic-compatible | OMP/自定义配置 | 通用 Adapter | 兼容性探测 |
| DeepSeek BYOK | OpenAI/Anthropic-compatible | 原生 Preset + 动态 Model Catalog | 不硬编码过时 Model ID |
| Codex 订阅 | OMP Runtime 管理或官方 `codex` CLI 委托 | `runtime-codex-cli` 可选 Adapter | KPi 不提取 ChatGPT/Codex Token |
| Claude Code 订阅 | 仅用户控制的本地 Runtime 委托/实验能力 | 经法律与条款审核后决定 | 不嵌入 Claude.ai Login，不代理订阅凭据 |
| Kimi Code 订阅 | OMP Plan 或官方 `kimi` CLI/订阅 API Key | `runtime-kimi-cli`/Kimi Adapter | 使用官方方式，不抓取客户端存储 |
| Local Models | OMP Local/OpenAI-compatible | 原生 Local Adapter | Key 可选 |
| Enterprise Gateway | OMP Custom Provider | 原生 Gateway Adapter | 审计、Allowlist、TLS |

### 19.3 Codex

支持方式：

1. 用户在官方 Codex CLI 中完成 `Sign in with ChatGPT`；
2. KPi 通过外部 Runtime 委托或 OMP 已管理的 Session 使用；
3. 或用户提供 OpenAI API Key；
4. KPi 不读取 Codex 本地认证文件，不导出 OAuth Token，不模拟登录。

### 19.4 Claude Code

KPi 必须区分：

- **Claude API/Console/Bedrock/Vertex/Foundry**：可作为正式 BYOK/企业 Provider；
- **Claude Code Subscription**：只允许用户控制的官方 CLI 委托，不在 KPi 内嵌 Claude.ai OAuth，也不把 Pro/Max 登录凭据转发给第三方服务。

商业发行前必须完成 Anthropic 当前条款和 OAuth 政策审核。未通过时：

- 功能标记为 `experimental` 或禁用；
- 不在产品文案承诺“使用 Claude 订阅额度”；
- 推荐使用 API Key 或受支持的企业云 Provider。

### 19.5 Kimi Code

支持：

- 官方 `kimi` CLI 已登录 Session 的本地委托；
- 官方订阅/会员提供的第三方 API Key；
- OpenAI-compatible/Anthropic-compatible Endpoint；
- KPi 不读取 Kimi 客户端隐藏 Token；
- CLI 包名、登录方式和模型目录通过 Doctor 动态检测，避免绑定旧版实现。

### 19.6 DeepSeek

支持：

- OpenAI-compatible Endpoint；
- Anthropic-compatible Endpoint；
- BYOK；
- Model Catalog 动态刷新；
- 配置以 Model Capability 为准，不将历史 Model ID 永久写死；
- Reasoning、Tool Call、Context Window、JSON/Structured Output 能力经实际探测记录。

### 19.7 Model Roles

```text
default
plan
review
verify
commit
smol
slow
```

要求：

- Role 可映射不同 Provider/Model；
- 支持 Fallback Chain；
- 支持项目路径级覆盖；
- 支持 Capability Gate；
- 失败回退必须记录 Primary、Fallback、原因、Token/Cost（可用时）；
- Provider 不可用不能静默换成能力不等价的模型。

### 19.8 凭据管理

允许：

- 环境变量引用；
- OS Keychain/Secret Service；
- 用户配置的 Credential Helper；
- 企业 Vault/Gateway；
- Runtime 自己管理的 OAuth Session。

禁止：

- 明文写入仓库；
- 写入 AGENTS、Skill、Report、Screenshot；
- 读取浏览器 Cookie；
- 复制 Refresh Token；
- 逆向订阅协议；
- 从其他 CLI 隐藏文件中提取凭据；
- 在日志中输出完整 Key。

---

## 20. Rule Engine 与 Policy Engine

### 20.1 Rule Action

```ts
type RuleAction =
  | "remind"
  | "interrupt"
  | "block-tool"
  | "block-stage"
  | "block-delivery";
```

### 20.2 P0 必备规则

| Rule | 条件 | Action |
|---|---|---|
| `no-trellis-init-outside-onboard` | 普通任务调用 `trellis init` | block-tool |
| `bdd-required-for-visible-behavior` | 可见行为将完成但无 Feature/Trace | block-delivery |
| `rtk-is-not-test-runner` | 报告型测试使用 RTK 且副作用未验证 | block-stage |
| `report-artifact-required` | 正式验证无 Native Report/同 stem 中文 MD | block-delivery |
| `mock-is-not-full-stack` | 非 full-stack 被描述为全链路通过 | interrupt |
| `gitnexus-requires-mcp-and-index` | 缺强证据却使用结论 | block-stage |
| `maestro-requires-java17` | Mobile Gate 无 Java 17+ | block-stage |
| `book-gate-before-edit` | Required Gate 未通过即编辑生产代码 | block-tool |
| `release-gate-before-complete` | 生产路径变化但 Release Gate 未 ready | block-delivery |
| `secret-read-guard` | 读取 `.env/*.key/*.pem` 等 | block-tool |
| `install-requires-approval` | 依赖/全局工具安装未确认 | block-tool |

### 20.3 默认 Policy Profile

```yaml
policy:
  profile: local-guarded

  filesystem:
    read: ["<project-root>"]
    write: ["<project-root>"]
    deny:
      - ".env"
      - ".env.*"
      - ".ssh"
      - "*.pem"
      - "*.key"
      - ".git/objects"

  shell:
    requireApproval:
      - dependency-install
      - global-install
      - network-download
      - destructive-file-op
      - git-reset
      - git-push
      - deploy
      - migration

  network:
    default: approval

  secrets:
    redact: true
```

---

## 21. Session、Context 与报告

### 21.1 SBTD Session State

```ts
interface SBTDSessionState {
  enabled: boolean;
  mode: "strict" | "relaxed";
  route: WorkflowRouteId | "auto";
  stage: string;
  classification?: SBTDClassification;
  activeSkills: string[];
  bookGates: Record<string, GateState>;
  toolEvidence: Record<string, ToolEvidence>;
  validation: ValidationState;
  provider: ProviderSessionState;
  lessons: LessonsState;
  decisions: DecisionRecord[];
}
```

### 21.2 Context 加载顺序

1. KPi Base Prompt；
2. 当前 Runtime 能力；
3. Policy Summary；
4. Workflow Stage；
5. Tool Evidence；
6. 项目 AGENTS 层级；
7. Repo Map；
8. Lessons 短入口；
9. 命中 Topic；
10. Trellis Workflow/Spec/Task；
11. 相关 Feature；
12. Active Skills；
13. 当前 Validation/Evidence Summary。

### 21.3 Compaction/Resume

必须保留：

- SBTD Enabled/Mode；
- Current Stage/Route；
- Classification；
- Book Gate States；
- Tool Evidence；
- Validation/Evidence；
- Pending Approval；
- Caveman 自动状态；
- Lessons 是否已读取；
- 首次提示是否已发送。

### 21.4 最终报告

```text
Goal
Runtime / Provider / Model Role
SBTD Classification
Workflow Route
Book Gate Plan and Results
Files Changed
Commands Run
Trellis
GitNexus
BDD
Web / Mobile / API Validation
Formal Reports
Evidence Source / Revision / Environment / Publication
RTK Status
Skipped / Blocked / Not-needed
Remaining Risks
Suggested Commit / Next Action
```

---

## 22. 功能需求清单

| ID | 需求 | 优先级 |
|---|---|---|
| FR-RUNTIME-001 | OMP 插件可注册 Slash Command、Hook、Context 与 Tool | P0 |
| FR-INSTALL-001 | Plugin 安装只写 OMP Plugin Registry/Lock，不执行 Onboard 或环境修改 | P0 |
| FR-INSTALL-002 | Plugin 内含固定版本、只读 SBTD Kit Bootstrap Source | P0 |
| FR-HELP-001 | `/sbtd help [command]` 由统一 Command Registry 确定性生成，不调用模型或修改状态 | P0 |
| FR-SBTD-001 | `/sbtd on/off/status/doctor/report` | P0 |
| FR-ONBOARD-000 | `/sbtd onboard status/plan/init/reset/init-projects` 与 `/sbtd setup` | P0 |
| FR-SBTD-002 | SBTD 分类器与 Route | P0 |
| FR-GATE-001 | Book Gate Plan 与 5 类 Gate | P0 |
| FR-RULE-001 | P0 必备 Dormant Rules | P0 |
| FR-VALID-001 | 项目原生验证、RTK Gate、报告真实性 | P0 |
| FR-BDD-001 | 用户可见行为 BDD Gate、语言与 Traceability | P0 |
| FR-TOOL-001 | Trellis/GitNexus/Web/Mobile 强证据状态 | P0 |
| FR-AGENTS-001 | OMP Global AGENTS 使用 `$PI_CODING_AGENT_DIR/AGENTS.md`，默认 `~/.omp/agent/AGENTS.md` | P0 |
| FR-AGENTS-002 | 根 Project AGENTS 固定为 `<project-root>/AGENTS.md`，承载跨 Agent 项目事实 | P0 |
| FR-AGENTS-003 | OMP Project Adapter 固定为 `<project-root>/.omp/AGENTS.md`，并导入 `@../AGENTS.md` | P0 |
| FR-AGENTS-004 | Global/Root/OMP Adapter、Plugin、Skills 和 Onboard 职责分离 | P0 |
| FR-AGENTS-005 | Global 与 OMP Adapter 使用 Mode-aware Conditional Sections，支持 `enforced/advisory` | P0 |
| FR-AGENTS-006 | 三类 Managed Block 独立 Digest，并保留块外用户/项目内容 | P0 |
| FR-AGENTS-007 | 检测 OMP Provider Shadowing、nearest-native Adapter 与 Import 有效性 | P0 |
| FR-AGENTS-008 | 缺 Global/Adapter/根项目事实时进入 `needs-onboard/degraded`，不得静默 managed | P0 |
| FR-SYNC-001 | 上游 AGENTS 三目标 Section Mapping、Sync Report 和 Unmapped Section Release Gate | P0 |
| FR-TRELLIS-001 | Onboard 初始化与 Runtime 普通任务的 Trellis 边界分离 | P0 |
| FR-GITNEXUS-001 | CLI/MCP/Index/Branch/Freshness 独立证据模型 | P0 |
| FR-MCP-001 | 配置状态与当前 Session callable 状态分离 | P0 |
| FR-SESSION-001 | Session/Compaction/Resume 状态恢复 | P0 |
| FR-ONBOARD-001 | 包装当前 `onboard.py` 并增加 OMP 双项目文件 Platform Adapter | P0 |
| FR-PROVIDER-001 | 继承 OMP Provider、BYOK 与 Role 状态 | P0 |
| FR-CLI-001 | 独立 `kpi` CLI 与显式命令路由 | P1 |
| FR-CLI-002 | KPi 启动默认 SBTD On | P1 |
| FR-ONBOARD-002 | `kpi onboard` 覆盖全部现有模式 | P1 |
| FR-PROVIDER-002 | `kpi provider doctor` 与凭据引用 | P1 |
| FR-KIT-001 | SBTD Kit Manifest 与安装/Doctor | P1 |
| FR-REPORT-001 | Session/Report JSON 与 Markdown 导出 | P1 |
| FR-CORE-001 | Runtime 无关 Workflow Engine | P2 |
| FR-CORE-002 | Runtime 无关 Rule/Policy/Validation/Context | P2 |
| FR-AGENTS-009 | Runtime-specific AGENTS Renderer、Section Mapping 与 Conformance | P2 |
| FR-PROVIDER-003 | 独立 Provider Gateway 与 BYOK Adapter | P2 |
| FR-ONBOARD-003 | Python → TypeScript 契约等价迁移 | P2 |
| FR-KB-001 | Knowledge P1.1 标准适配 | P2 |
| FR-RUNTIME-002 | `runtime-omp` 标准 Adapter | P2 |
| FR-RUNTIME-003 | `runtime-pi` Adapter | P3 |
| FR-COMPAT-001 | OMP/Pi Workflow Compatibility Suite | P3 |
| FR-CHANNEL-001 | Runtime 无关 Channel/Subagent Bridge | P3 |
| FR-FORK-001 | Controlled Fork Decision Gate | P3 |

## 23. 非功能需求

### 23.1 可靠性

- Hard Gate 不得因模型遗忘而消失；
- Onboard 重复执行幂等；
- Multi-project 失败隔离；
- External Skill 安装事务可回滚；
- Session Resume 后状态一致；
- 正式报告必须是当前运行证据。

### 23.2 可维护性

- Core 不导入 OMP 私有类型；
- Runtime Adapter 通过契约测试；
- Workflow、Rule、Skill、Provider Manifest Schema 化；
- Prompt 与确定性规则分离；
- Python 迁移前建立 Golden Fixture。

### 23.3 性能

- 常驻 Prompt 保持最小；
- Skills/Lessons 按需加载；
- Hidden Tool 按 Route 激活；
- Repo Inspect 可缓存但需以 Git/mtime 失效；
- P0 不追求自研 Native 搜索性能。

### 23.4 可移植性

- macOS、Linux、Windows；
- KPi Core/CLI 使用 Node.js LTS；
- OMP Plugin 可在其要求的 Bun Runtime 运行；
- Python Bridge 显式检查 Python 版本；
- 路径处理使用 Canonical Path，不依赖 POSIX 假设。

### 23.5 隐私

- 本地优先；
- 默认不上传 Session；
- Telemetry 默认关闭或仅本地；
- 报告和 Evidence 必须 Sanitized；
- Provider 凭据不可进入 LLM Context。

---

## 24. 验收标准

### 24.1 P0 验收

- OMP 可安装并加载 `@kpi/omp-sbtd`；
- Plugin 安装前后环境 Diff 为零非 OMP-Plugin 管理写入；
- Plugin 包含可校验的固定 Kit Revision，不在启用时自动拉取 `main`；
- `/sbtd help`、`/sbtd help <command>` 和所有已注册命令说明来自同一 Command Registry；
- Help 不调用模型、不创建 Agent Turn、不修改 Session，并包含用途、Usage、示例、写入和确认标记；
- `/sbtd on/off/status/doctor/report` 可用；
- `/sbtd onboard status/plan/init/reset/init-projects` 与 `/sbtd setup` 可用；
- `/sbtd onboard plan` 不写文件，Apply 类命令显示完整计划并获得确认；
- `/sbtd on` 先做只读 Preflight，未 Onboard 时返回 `needs-onboard/degraded`；
- `/sbtd on` 在 `managed + enforced` 下执行自动分类、工具证据和 Gate；
- `/sbtd off` 切换为 `advisory`，停止自动分类、Book Gate、自动 Skill 路由和 Delivery Gate，但项目事实和 Always-on 安全仍有效；
- OMP Global AGENTS 严格写入 `$PI_CODING_AGENT_DIR/AGENTS.md`，默认 `~/.omp/agent/AGENTS.md`；
- 根 Project AGENTS 严格写入 `<project-root>/AGENTS.md`；
- OMP Project Adapter 严格写入 `<project-root>/.omp/AGENTS.md`，并校验 `@../AGENTS.md`；
- normal 与 project-only Onboard 均能安全处理 Root/OMP 两个项目文件；
- 旧 `--skip-project-agents` 映射为跳过根 Project AGENTS，并给出弃用警告；
- 三类 Managed Block 独立更新，块外用户/项目内容保持不变；
- 已知旧模板可备份迁移，未知无 Marker 文件进入 `merge-required`；
- Doctor 能区分 Global、Root Project、OMP Adapter、Deep AGENTS 的 `exists/discovered/loaded/effective/shadowedBy/imports`；
- Monorepo 出现更近 `.omp/AGENTS.md` 时，Doctor 能识别 nearest-native 覆盖并要求继承根 Adapter；
- 上游 AGENTS 每个 Section 均映射到 `omp-global-agents/project-root-agents/omp-project-agents/plugin/skill/onboard/ignored` 之一；
- 存在 Unmapped Section 时 Kit/Plugin 不得发布；
- Session/Compaction/Resume 后 `enforced/advisory`、Stage 和 Gate State 可恢复；
- 用户可见行为无 BDD 时不能宣称完成；
- 5 类 Book Gate 的 Required 条件和状态可见；
- `rtk`、mock/full-stack、GitNexus、报告文件规则可拦截；
- project-only 模式不修改 Global AGENTS、Global Skills、Global Tools 或 MCP；
- Trellis Onboard 初始化例外与普通 Runtime 禁止自动初始化边界可验证；
- GitNexus 必须同时验证 Session MCP、项目索引、Branch 与 Freshness；
- Provider 凭据不被 KPi 读取或记录；
- 至少覆盖小修改、可见 Feature、Bug、数据变更、生产路径变更五类端到端样例。

### 24.2 P1 验收

- `kpi` 命令可独立安装；
- `kpi run` 默认启用 SBTD；
- 用户无需直接操作 OMP Plugin；
- `kpi onboard` 覆盖现有公开命令与 JSON 输出；
- `kpi doctor` 分别报告 AGENTS loaded、Plugin loaded、Skills discovered、Tools installed/configured/callable/project-ready；
- KPi 启动流程不会静默覆盖 AGENTS；
- 显式命令表和保留词保护通过测试；
- `kpi provider doctor` 可区分 BYOK、Runtime OAuth、CLI Delegation、Unavailable；
- 支持 Session/Report 导出和 Shell Completion；
- 项目配置可覆盖 Runtime、Mode、Kit、Provider Role 和 Policy。

### 24.3 P2 验收

- Workflow/Rule/Policy/Validation/Context/Provider 不依赖 OMP；
- Kit 成为机器规则与 AGENTS 模板的单一来源；
- Mode-aware AGENTS 在 `enforced/advisory` 两种 Runtime Mode 下通过一致性测试；
- OMP 仅通过 `runtime-omp` Adapter 接入；
- TypeScript Onboard 对 Golden Fixture 与 Python 输出达到契约等价；
- Catalog、External Skill Transaction、Legacy Migration、多项目状态测试通过；
- 独立 BYOK Gateway 支持 OpenAI、Anthropic、OpenAI-compatible、Anthropic-compatible、DeepSeek；
- Knowledge P1.1 状态和 Artifact 可进入统一报告；
- Core 可在 Headless 单元测试中完整模拟工作流。

### 24.4 P3 验收

- `kpi run --runtime omp` 与 `--runtime pi` 均可执行同一核心场景；
- 两 Runtime 对 Stage、Gate、Tool Evidence、Validation 与 Report 语义一致；
- Provider 和 Credential 不依赖 Runtime 私有格式；
- Runtime Capability Matrix 可见；
- 缺失能力明确降级为 blocked/skipped/not-needed；
- Fork 决策有 ADR、缺口复现、API 方案评估、维护成本和退出策略；
- 未满足 Fork Gate 时继续使用插件/Adapter。

---

## 25. 成功指标

| 指标 | 目标方向 |
|---|---|
| Hard Gate 漏检率 | 持续下降；P0 关键规则零已知静默绕过 |
| Tool Availability 误判率 | 强证据判断后接近零 |
| 报告陈旧/缺失误报率 | 正式验证中为零 |
| 非 full-stack 冒充率 | 为零 |
| Onboard 幂等性 | 二次执行无重复写入和无意覆盖 |
| Multi-project 隔离 | 单项目失败不丢失其他项目结果 |
| Session 恢复一致性 | Resume 后 Gate/Stage 无重置 |
| Workflow 过度启用率 | 小任务不被无必要重流程阻塞 |
| Provider 连接成功率 | Doctor 可准确给出可用/阻塞原因 |
| OMP 升级兼容性 | Compatibility Suite 可在升级前发现破坏 |
| P2 Runtime 解耦度 | Core 包不导入 OMP/Pi 私有模块 |

---

## 26. 主要风险与缓解

| 风险 | 影响 | 缓解 |
|---|---|---|
| OMP 插件 API 快速变化 | P0/P1 兼容性 | Pin 版本、Adapter、Contract Tests、升级矩阵 |
| SBTD 规则过多导致 Context 膨胀 | 成本和准确率下降 | Dormant Rules、Lazy Skills、状态摘要 |
| 硬门禁过度阻塞小任务 | 用户体验差 | 客观 Predicate、not-required、Route 测试 |
| Python/TS Onboard 行为漂移 | 安装风险 | Golden Fixtures、双跑 Diff、分阶段替换 |
| Provider/Model 频繁变更 | 配置失效 | 动态 Catalog、Capability Probe、Fallback |
| Claude Subscription 合规限制 | 法律/账号风险 | 不嵌入 OAuth；官方 CLI 委托或 API Key；法律审核 |
| OMP 内置 Plan/OAuth 变化 | 订阅不可用 | Runtime Doctor、显式降级、无凭据迁移 |
| Evidence 泄露敏感信息 | 安全风险 | Sanitizer、Schema、Secret Scan、发布前 Gate |
| 多 Agent 写冲突 | 数据损坏 | 单 Writer、独立 Worktree、主 Session 仲裁 |
| 直接 Fork 后长期漂移 | 维护成本失控 | Fork-last、ADR、最小 Patch Set、Upstream Sync Plan |

---

## 27. Fork Decision Gate

只有同时满足以下条件才允许进入 OMP Fork：

1. P0/P1 已证明 SBTD 有真实产品价值；
2. 缺口可稳定复现；
3. Plugin、Extension、Hook、Custom Tool、SDK、RPC、ACP 均无法实现；
4. 缺口属于核心硬门禁，而非便利功能；
5. 已向 OMP 提交或评估上游扩展点；
6. Fork Patch 可保持小而独立；
7. 有持续同步、Native Build、Security Update 和 Release 负责人；
8. 有退出 Fork 或回归上游的路径；
9. 已形成 ADR 并获得产品/工程确认。

否则继续使用 Adapter/Plugin。

---

## 28. 开放决策

1. KPi CLI 默认发行是否捆绑 OMP，还是要求外部安装。
2. P0 Plugin 使用 OMP Marketplace、npm 还是 Git 安装。
3. SBTD `relaxed` 模式允许跳过哪些 Optional Checks。
4. 是否在 P1 引入 `kpi provider login`，或仅做 Runtime Delegation。
5. Claude Code Subscription 委托在商业发行中的法律边界。
6. P2 TypeScript Onboard 是否保留 Python Fallback。
7. `640-skills` 是 Git Submodule、Package、Vendored Snapshot 还是远端 Kit。
8. Evidence P2（Store/PR Check）放入 KPi v0.3 还是独立产品。
9. Pi Runtime 缺少的 Policy/Rule 注入点如何降级。
10. KPi 的长期许可证与 bundled/external Skill 的 License/NOTICE 传播策略。
11. P2 以后是否需要提供完全不安装 Global AGENTS 的 Runtime-native 模式，以及其隔离、兼容和迁移边界。
12. P2 以后是否允许结构化 `.kpi/project.yaml` 辅助生成根 Project AGENTS，但不得削弱根 AGENTS 作为跨 Agent 可读事实层。
13. Monorepo 是否在 P2 提供显式 Workspace Adapter 生成命令，而不是 P0 自动创建深层 `.omp/AGENTS.md`。

---

## 29. 参考来源

### SBTD Workflow

- Repository: https://github.com/KunoLu/640-skills
- Snapshot: `main @ 340f9dd4dc7a92e8b91c31e111de9a8de06cef36`
- `README.md`
- `CHANGELOG.md`
- `sbtd-workflow-onboard/SKILL.md`
- `sbtd-workflow-onboard/REFERENCE.md`
- `sbtd-workflow-onboard/catalog.json`
- `sbtd-workflow-onboard/catalog.schema.json`
- `sbtd-workflow-onboard/scripts/onboard.py`
- `sbtd-workflow-onboard/templates/agents/AGENTS.global.md`
- `sbtd-workflow-onboard/templates/agents/AGENTS.project.md`
- `sbtd-workflow-onboard/templates/skills/**`

### OMP

- Repository: https://github.com/can1357/oh-my-pi
- `README.md`
- `packages/coding-agent/src/cli-commands.ts`
- `packages/coding-agent/src/extensibility/plugins/types.ts`
- `packages/coding-agent/src/extensibility/extensions/types.ts`
- `docs/hooks.md`
- `docs/custom-tools.md`
- `docs/ttsr-injection-lifecycle.md`

### 官方 Coding Agent

- Codex: https://github.com/openai/codex
- Claude Code: https://github.com/anthropics/claude-code

---

## 30. 最终产品决策

> **KPi 采用“OMP-first、KPi-Core-later、Pi/OMP 双 Runtime、Fork-last”的实施路线。P0 以 OMP 插件、OMP Global AGENTS、根 Project AGENTS 与 `.omp/AGENTS.md` Adapter 构成 Managed SBTD 环境，并提供 `/sbtd help`、`enforced/advisory` 模式和安全 Onboard；P1 用独立 `kpi` CLI 建立产品边界；P2 抽离 Runtime 无关核心；P3 支持 OMP 与 Pi 双 Runtime。任何 Fork 必须通过明确的技术与维护 Gate。**
