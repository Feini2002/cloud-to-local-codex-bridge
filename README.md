# Cloud-to-Local Codex Bridge

![Status: Concept](https://img.shields.io/badge/status-concept-blue)
![Docs: Architecture](https://img.shields.io/badge/docs-architecture-informational)
![Security: Threat Model](https://img.shields.io/badge/security-threat%20model-informational)

云端发任务，本地 Codex 执行，结果回写云端。  
Route tasks from a private cloud UI to a local Codex CLI runner, then send logs and results back to the cloud.

## 初衷 / Motivation

这个文档最初来自一个很现实的想法：在个人、私有、安全且合规的边界内，能否把一些原本需要 API key 调用、会产生额外 token/API 花费的自动化分析任务，改由本地 Codex CLI 执行，从而减少重复成本。

更长远的愿望是：对于只服务个人、只访问自己有权限的数据、只在受控本地环境里执行的任务，能否用一个 Codex 订阅承载更多原本依赖 API 的个人自动化工作流。但这个仓库不把这种愿望包装成绕过计费、账号共享或公开代理方案；它关注的是边界清晰的私有自托管架构，以及哪些场景适合用本地 Codex，哪些场景仍应使用官方 API、企业认证或 Codex access token。

This document started from a practical question: within personal, private, secure, and compliant boundaries, can some automation tasks that would normally require API-key calls and extra token/API spend be routed through a local Codex CLI runner instead?

The longer-term hope is to understand whether a single Codex subscription can support more personal automation workflows that only use data the user is authorized to access and only execute inside a controlled local environment. This repository does not present that as a way to bypass billing, share accounts, or build a public proxy. It focuses on a private self-hosted architecture with clear boundaries, and on deciding which workloads fit local Codex execution versus official APIs, enterprise authentication, or Codex access tokens.

展开对应语言即可在当前页面阅读说明。

<details open>
<summary><strong>中文说明</strong></summary>

## 这是什么

**Cloud-to-Local Codex Bridge** 是一份概念与架构说明，用来描述如何把私有云端页面里的任务，安全地交给本地 Codex CLI 执行。

它描述的是一种私有、自托管的执行模式：

```text
云端页面
  -> 云端控制面
  -> 任务 / 中转层
  -> 本地桥接器
  -> 本地 Codex 运行环境
  -> 结果存储
  -> 云端页面
```

云端侧是控制面：负责身份、任务创建、状态、日志和结果展示。本地侧是执行面：本地桥接器接收任务、校验策略、在允许列表工作区里启动 Codex，并把结果回传云端。

核心原则：**云端可以请求执行任务，但本地桥接器必须决定什么任务允许执行。**

## 这不是什么

- 不是公开 API 代理。
- 不是账号共享方案。
- 不是绕过使用限制、计费系统、速率限制或安全机制的方法。
- 不是 OpenAI 官方项目，也不是 OpenAI 政策解释。

它描述的是单用户、私有、自托管的执行器模式：用户远程控制自己的本地机器。

## 30 秒理解

常见云端 AI 应用大致是：

```text
浏览器 -> 云端 API -> 模型服务 -> 浏览器
```

这个模式是：

```text
浏览器 -> 云端控制面 -> 本地桥接器 -> 本地 Codex 进程 -> 云端结果页
```

关键点：

- 浏览器不运行 Codex。
- 云端控制面不直接进入本地机器。
- 本地桥接器主动连出、校验任务、在受控工作区中运行 Codex，并回传日志和结果。

## 模型与推理强度

这套方案真正有吸引力的地方，不只是“能从云端触发本地执行”，还包括可以在本地 Codex CLI 里选择合适的模型和推理强度。很多人付费使用 Codex，看中的正是更强模型在复杂代码、长上下文分析、架构判断和自动修复上的能力。

桥接器可以把任务分成不同档位：

- 普通摘要、格式整理、轻量文档：使用较快、较省的模型或较低推理强度。
- 代码审查、复杂排错、跨文件改造：使用更强模型和更高推理强度。
- 关键架构分析、长期任务规划、疑难修复：可以显式指定 `gpt-5.5` 这类强模型，并使用 `xhigh` 推理强度。

一次性任务可以类似这样调用：

```bash
codex exec --json --model gpt-5.5 -c model_reasoning_effort="xhigh" --sandbox workspace-write "分析这个仓库的架构风险，并给出修复建议"
```

也可以在本地 `~/.codex/config.toml` 里设置默认模型，再让云端任务只传入“任务类型”，由本地桥接器决定最终使用哪个模型和推理强度。更推荐这种方式，因为模型选择属于本地执行策略，不应该完全交给云端页面随意控制。

模型名称、可用档位和额度规则会随 Codex 产品变化。真正落地时，应以当前 `codex --help`、`codex exec --help`、`codex debug models` 和官方文档为准。

## 架构概览

从上往下读这张图：任务从浏览器开始，经过云端控制面，被本地桥接器领取，在本地工作区中运行，然后回到云端结果页。

```mermaid
flowchart TD
  Browser["浏览器 / 云端页面"] --> Access["身份提供方 / 访问网关"]
  Access --> API["云端控制面"]
  API --> Queue["任务 / 中转层"]
  API --> Store["结果存储"]
  Queue <--> Bridge["本地桥接器"]
  Bridge --> Policy["工作区策略"]
  Policy --> Sandbox["沙箱"]
  Sandbox --> Codex["Codex 进程"]
  Codex --> Collector["事件与产物收集器"]
  Collector --> Bridge
  Bridge --> Queue
  Queue --> API
  API --> Store
  Store --> Browser
```

## 范围与边界

适合：

- 个人私有系统。
- 单用户控制自己的本地开发工作区。
- 云端页面启动本地 Codex 任务，同时不暴露本地公网端口。
- 部署在 Cloudflare、Vercel、Supabase、AWS、GCP、Azure、VPS 或自建后端上。
- 执行受工作区策略、沙箱、审批、超时、输出限制和审计日志约束。

不适合：

- 公开多用户访问。
- 多人共享一个个人 Codex 或 ChatGPT 登录态。
- 把个人订阅包装成公开 API。
- 从网页向本地机器透传任意 shell 命令。
- 让 Codex 默认访问整个用户目录、SSH key、浏览器 cookie、云服务凭据、`.env` 文件或 token 缓存。

## 平台无关映射

Cloudflare 是一种实现选项，不是要求。同一组职责也可以映射到其他平台。

| 职责 | 通用组件 | 可选实现 |
| --- | --- | --- |
| 身份校验 | 身份提供方 / 访问网关 | Cloudflare Access, Auth0, Clerk, Supabase Auth, GitHub OAuth |
| 云端页面 | 前端托管 | Cloudflare Pages, Vercel, Netlify, 静态站点托管 |
| 控制面 | API / 后端 | Cloudflare Worker, Next.js API routes, FastAPI, Express, Lambda, Cloud Run |
| 任务协调 | 队列 / 状态存储 | Durable Objects, Redis, Postgres, SQS, Pub/Sub, RabbitMQ |
| 中转 / 传输 | 轮询、WebSocket、SSE、隧道、网状私有网络 | WebSocket 服务, Tailscale, SSH 反向隧道, VPS 中转服务 |
| 结果存储 | 数据库 / 对象存储 | Postgres, SQLite, Supabase, D1, S3, R2, MinIO |

## 最小可行流程

第一版不需要完整的交互式运行环境。最小版本可以用签名轮询和 `codex exec`：

```text
1. 云端页面创建任务
2. 云端 API 将任务保存为 pending 状态
3. 本地桥接器轮询待处理任务
4. 本地桥接器校验签名、过期时间、nonce、工作区和风险策略
5. 本地桥接器在允许列表工作区中运行 `codex exec`
6. 本地桥接器脱敏 / 截断日志，并上传事件
7. 云端页面展示 completed 或 failed 状态
```

即使是私有概念验证（PoC），也应该包含：

- `task_id`, `nonce`, `expires_at`
- 工作区允许列表
- 默认开启沙箱
- 输出大小限制
- 超时和取消机制
- 日志脱敏
- 重放保护
- 可审计的任务状态

## 路线图

**阶段 1：签名轮询 + `codex exec`**  
跑通最小安全闭环：创建任务，本地桥接器领取任务，本地 Codex 执行，结果返回云端。

**阶段 2：实时事件 + 可取消执行**  
加入实时日志、有序事件、超时处理、取消机制和幂等结果更新。

**阶段 3：审批 + 产物 + 审计**  
加入高风险操作审批，保存差异补丁（diff）、报告等产物，做日志脱敏，并保留完整审计轨迹。

**阶段 4：会话型运行环境**  
探索 `codex app-server`、多工作区策略、更丰富的交互和长会话。

## 安全边界

云端侧不应该被本地桥接器盲信。云端可以提交任务请求，但本地执行必须由本地策略治理。

必要防护：

- 不上传本地认证文件，例如 `~/.codex/auth.json`、API key、SSH key、浏览器 cookie、云服务凭据、`.env` 文件或 token 缓存。
- 不把本地桥接器直接暴露到公网。
- 不允许网页透传任意 shell 命令。
- 默认不使用完整用户目录访问权限。
- 不多人共享个人 Codex 登录态。
- 本地策略必须留在本地：工作区允许列表、风险规则、审批决策都必须由桥接器强制执行。
- 只保存必要任务数据，敏感日志上传前先脱敏。

完整威胁模型见 [SECURITY.md](SECURITY.md)。

## 仓库状态

状态：**概念 / 架构说明**

可运行代码：**暂未提供**

主要内容：

- 架构概览
- 安全边界
- 平台映射
- MVP 路线图
- 实现说明

## 仓库结构

```text
README.md              # 同页中英文说明
docs/architecture.md   # 完整架构说明，中文长文，含英文摘要
SECURITY.md            # 威胁模型和安全检查清单
DISCLAIMER.md          # 使用边界和免责声明
CONTRIBUTING.md        # 贡献说明
.gitignore
```

## 阅读方式

- 快速理解：读当前 README。
- 完整长文：读 [docs/architecture.md](docs/architecture.md)。
- 做概念验证（PoC）之前：读 [SECURITY.md](SECURITY.md)。
- 面向团队、公开用户或商业服务前：读 [DISCLAIMER.md](DISCLAIMER.md)。
- 提 issue 或改文档前：读 [CONTRIBUTING.md](CONTRIBUTING.md)。

</details>

<details>
<summary><strong>English README</strong></summary>

## What It Is

**Cloud-to-Local Codex Bridge** is a concept and architecture note for safely routing tasks from a private cloud UI to a local Codex CLI runner.

It describes a private, self-hosted execution pattern:

```text
Web UI
  -> Cloud Control Plane
  -> Task / Relay Layer
  -> Local Bridge
  -> Local Codex Runtime
  -> Result Store
  -> Web UI
```

The cloud side is the control plane: identity, task creation, state, logs, and result display. The local side is the execution plane: a local bridge receives tasks, validates policy, starts Codex in an allowlisted workspace, and sends results back.

The core idea is simple: **the cloud can request work, but the local bridge decides what is allowed to execute.**

## What It Is Not

- It is not a public API proxy.
- It is not intended for account sharing.
- It is not a way to bypass usage limits, billing systems, rate limits, or safety mechanisms.
- It is not an official OpenAI project or policy interpretation.

It is a private self-hosted runner pattern for a single user controlling their own local machine.

## 30-Second Model

Most cloud AI apps look like this:

```text
browser -> cloud API -> model service -> browser
```

This pattern looks like this:

```text
browser -> cloud control plane -> local bridge -> local Codex process -> cloud result view
```

Key points:

- The browser does not run Codex.
- The cloud control plane does not directly enter the local machine.
- The local bridge connects outward, validates the task, runs Codex in a controlled workspace, and returns logs/results.

## Model And Reasoning Level

The value of this pattern is not only that a cloud page can trigger local execution. It also lets the local Codex CLI choose the right model and reasoning level for the task. For many paid Codex users, the point is access to stronger models for complex coding, long-context analysis, architecture work, and automated repair.

The bridge can route tasks into different execution tiers:

- Lightweight summaries, formatting, and small documentation tasks can use a faster model or lower reasoning level.
- Code review, debugging, and multi-file changes can use a stronger model and higher reasoning level.
- Architecture analysis, long-horizon planning, and difficult fixes can explicitly use a strong model such as `gpt-5.5` with `xhigh` reasoning.

A one-shot task can look like this:

```bash
codex exec --json --model gpt-5.5 -c model_reasoning_effort="xhigh" --sandbox workspace-write "Analyze this repository for architecture risks and propose fixes"
```

Another option is to set default models in the local `~/.codex/config.toml`, then let the cloud task send only a task type. The local bridge decides which model and reasoning level to use. This is usually safer, because model selection is part of local execution policy and should not be fully controlled by the cloud page.

Model names, available reasoning levels, and quota behavior can change as Codex evolves. Use the current `codex --help`, `codex exec --help`, `codex debug models`, and official documentation when implementing this.

## Architecture Overview

Read the diagram from top to bottom. A task starts in the browser, moves through the cloud control plane, is picked up by the local bridge, runs in a local workspace, then returns to the cloud result view.

```mermaid
flowchart TD
  Browser["Browser / Web UI"] --> Access["Identity Provider / Access Gateway"]
  Access --> API["Cloud Control Plane"]
  API --> Queue["Task / Relay Layer"]
  API --> Store["Result Store"]
  Queue <--> Bridge["Local Bridge"]
  Bridge --> Policy["Workspace Policy"]
  Policy --> Sandbox["Sandbox"]
  Sandbox --> Codex["Codex Process"]
  Codex --> Collector["Event & Artifact Collector"]
  Collector --> Bridge
  Bridge --> Queue
  Queue --> API
  API --> Store
  Store --> Browser
```

## Scope And Boundaries

This architecture is suitable for:

- Private personal systems.
- A single user controlling their own local development workspace.
- A cloud UI that starts local Codex tasks without exposing a local public port.
- Platform-agnostic deployments on Cloudflare, Vercel, Supabase, AWS, GCP, Azure, a VPS, or a custom backend.
- Systems where execution is constrained by workspace policy, sandboxing, approval, timeout, output limits, and audit logs.

This architecture is not suitable for:

- Public multi-user access.
- Sharing one personal Codex or ChatGPT login across users.
- Reselling or wrapping a personal subscription as a public API.
- Passing arbitrary shell commands from a web page to a local machine.
- Letting Codex access the entire user directory, SSH keys, browser cookies, cloud credentials, `.env` files, or token caches by default.

## Platform-Agnostic Mapping

Cloudflare is one possible implementation, not a requirement. The same responsibilities can be mapped to many platforms.

| Responsibility | Generic Component | Example Implementations |
| --- | --- | --- |
| Identity | IdP / access gateway | Cloudflare Access, Auth0, Clerk, Supabase Auth, GitHub OAuth |
| Web UI | Frontend hosting | Cloudflare Pages, Vercel, Netlify, static hosting |
| Control plane | API / backend | Cloudflare Worker, Next.js API routes, FastAPI, Express, Lambda, Cloud Run |
| Task coordination | Queue / state store | Durable Objects, Redis, Postgres, SQS, Pub/Sub, RabbitMQ |
| Relay / transport | Polling, WebSocket, SSE, tunnel, mesh network | WebSocket server, Tailscale, SSH reverse tunnel, VPS relay |
| Result storage | Database / object storage | Postgres, SQLite, Supabase, D1, S3, R2, MinIO |

## Minimal Viable Flow

The first prototype does not need a full interactive runtime. A minimal version can use signed polling and `codex exec`:

```text
1. Cloud UI creates a task
2. Cloud API stores the task as pending
3. Local bridge polls for pending work
4. Local bridge verifies signature, expiry, nonce, workspace, and risk policy
5. Local bridge runs codex exec in an allowlisted workspace
6. Local bridge redacts/truncates logs and uploads events
7. Cloud UI displays completed or failed status
```

Even a private proof of concept should include:

- `task_id`, `nonce`, and `expires_at`
- workspace allowlist
- sandbox enabled by default
- output-size limit
- timeout and cancellation
- log redaction
- replay protection
- auditable task state

## Roadmap

**Phase 1: signed polling + `codex exec`**  
Build the smallest safe loop: create task, local bridge claims it, Codex runs locally, result returns to cloud.

**Phase 2: real-time events + cancellable execution**  
Add live logs, ordered events, timeout handling, cancellation, and idempotent result updates.

**Phase 3: approvals + artifacts + audit**  
Add human approval for risky operations, persist artifacts such as diffs and reports, redact logs, and keep a full audit trail.

**Phase 4: session runtime**  
Explore `codex app-server`, multi-workspace policies, richer interaction, and long-lived sessions.

## Security Boundaries

The cloud side should not be blindly trusted by the local bridge. The cloud can submit a task request, but local execution must remain locally governed.

Required guardrails:

- Do not upload local auth files such as `~/.codex/auth.json`, API keys, SSH keys, browser cookies, cloud credentials, `.env` files, or token caches.
- Do not expose the local bridge directly to the public internet.
- Do not allow arbitrary shell command passthrough from the web page.
- Do not run with full user-directory access by default.
- Do not share a personal Codex login session with other users.
- Keep local policy local: workspace allowlists, risk rules, and approval decisions must be enforced by the bridge.
- Store only the minimum task data needed, and redact sensitive logs before upload.

See [SECURITY.md](SECURITY.md) for the threat model and implementation checklist.

## Repository Status

Status: **concept / architecture note**

Runnable code: **not yet**

Primary output:

- architecture overview
- security boundaries
- platform mapping
- MVP roadmap
- implementation notes

## Repository Structure

```text
README.md              # Same-page Chinese and English README
docs/architecture.md   # Full architecture note, Chinese body with English summary
SECURITY.md            # Threat model and security checklist
DISCLAIMER.md          # Scope and usage disclaimer
CONTRIBUTING.md        # Contribution guidelines
.gitignore
```

## How To Read

- Quick overview: read this README.
- Full architecture note: read [docs/architecture.md](docs/architecture.md).
- Before building a PoC: read [SECURITY.md](SECURITY.md).
- Before adapting this for teams, public users, or commercial services: read [DISCLAIMER.md](DISCLAIMER.md).
- Before opening issues or proposing changes: read [CONTRIBUTING.md](CONTRIBUTING.md).

</details>

## References

Codex documentation:

- [Codex Authentication](https://developers.openai.com/codex/auth)
- [Codex Non-interactive Mode](https://developers.openai.com/codex/noninteractive)
- [Codex App Server](https://developers.openai.com/codex/app-server)

Policy and account-boundary references:

- [OpenAI Account Sharing Policy](https://help.openai.com/en/articles/10471989)
- [OpenAI Terms of Use](https://openai.com/policies/terms-of-use/)

Specific CLI flags and product behavior may change over time. Treat this repository as an architecture note, and use current official documentation for implementation details.
