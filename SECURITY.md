# Security Policy / 安全说明

<details open>
<summary><strong>中文</strong></summary>

本仓库描述的是一种私有 Cloud-to-Local 执行桥模式。最重要的安全边界不是云端页面本身，而是云端页面被允许让本地机器做什么。

## 预期安全模型

- 单用户私有使用。
- 云端访问由身份提供方或访问网关保护，例如 Cloudflare Access、Auth0、Clerk、Supabase Auth、GitHub OAuth 或私有自建登录。
- 本地桥接器主动连接云端中转层，不暴露公网监听端口。
- 每个任务都有唯一 ID、时间戳、签名和审计记录。
- 本地执行仅限明确允许的工作区。
- Codex 默认启用沙箱。
- 模型和推理强度由本地桥接器策略决定，不由云端页面任意指定。
- 高风险动作需要人工审批。

## 必要防护

- 不要把本地认证文件或密钥上传到任何云端服务器，包括 `~/.codex/auth.json`、API key、SSH key、浏览器 cookie、云服务凭据、`.env` 文件或 token 缓存。
- 不允许网页向本地桥接器透传任意 shell 命令。
- 不要把本地桥接器直接暴露到公网。
- 默认不要授予完整用户目录访问权限。
- 不要多人共享个人 Codex 登录态。
- 不要在任务日志或产物中保存密钥。

## 实现前检查清单

构建概念验证前，先确认：

- 云端页面有真实的身份层保护。
- 本地桥接器主动连出到中转层。
- 本地桥接器有独立的共享密钥、token 或签名密钥。
- 每个任务都有唯一 ID、时间戳和重放保护。
- 本地桥接器只接受明确允许的工作区。
- 本地桥接器强制执行超时和输出大小限制。
- 本地桥接器限制可用模型、推理强度和并发数量。
- 日志上传前已经脱敏。
- 高风险命令需要人工审批。
- 系统可以取消正在运行的任务。
- 系统为每个任务记录审计轨迹。

## 高风险动作

以下动作应要求人工审批：

- 删除、移动或覆盖大量文件。
- 执行部署、发布、重置或回滚命令。
- 修改 `.env`、SSH、支付、云服务或生产配置。
- 安装依赖，或执行来自网络的脚本。
- 访问配置工作区之外的路径。
- 触发邮件、支付、消息发送或生产系统副作用。

## 威胁模型

| 风险 | 典型路径 | 影响 | 防护 |
| --- | --- | --- | --- |
| 未授权访问 | 云端登录绕过或会话泄露 | 远程触发本地执行 | 身份提供方 / 访问网关、JWT 校验、额外桥接器 token |
| 重放攻击 | 旧任务包被重复提交 | 危险动作重复执行 | 唯一任务 ID、时间戳、签名、nonce |
| Prompt 注入 | 任务诱导 Codex 读取敏感文件 | 密钥或隐私泄露 | 工作区允许列表、敏感路径拒绝列表、日志脱敏 |
| 任意命令执行 | 网页透传原始 shell 命令 | 本地机器被接管 | 命令策略、审批流、沙箱 |
| 权限过大 | Codex 可访问整个用户目录 | 文件泄露或误改 | 独立 OS 用户、容器、固定工作目录 |
| 日志泄露 | 密钥被上传到日志 | 云端数据库泄露敏感信息 | 脱敏、存储权限、保留周期 |
| 资源耗尽 | 长任务或无限输出 | 本地机器变慢或不可用 | 超时、输出上限、并发限制 |
| 额度失控 | 云端任务随意指定强模型或最高推理强度 | 订阅额度被快速消耗 | 本地模型白名单、任务分级、并发限制、每日上限 |
| 账号边界混乱 | 多人共享同一个个人登录态 | 合规与账号风险 | 单用户模型；团队场景使用 API key 或企业授权 |

## 推荐默认值

早期私有原型建议：

- 只使用一个本地工作区。
- 只运行一个本地桥接器实例。
- 使用 `workspace-write` 沙箱。
- 在本地配置模型白名单和任务分级，例如只允许特定任务使用 `gpt-5.5` 与 `xhigh`。
- 先使用签名轮询，再增加 WebSocket 复杂度。
- 初期只保存日志和最终输出。
- 避免自动删除、部署、发布或生产动作。
- 桥接器配置保留在本地，不让云端 API 成为唯一策略来源。

## 报告问题

本仓库目前是概念文档。如果你发现架构说明中的安全问题，请提交 issue，并指出具体章节和建议修改。

</details>

<details>
<summary><strong>English</strong></summary>

This repository describes a private cloud-to-local execution bridge pattern. The most important security boundary is not the cloud page itself, but what the cloud page is allowed to make the local machine do.

## Intended Security Model

- Single-user private use.
- Cloud access protected by an identity provider or access gateway, such as Cloudflare Access, Auth0, Clerk, Supabase Auth, GitHub OAuth, or a private self-hosted login.
- Local bridge connects outbound to the cloud relay; it should not expose a public listening port.
- Every task has a unique ID, timestamp, signature, and audit trail.
- Local execution is limited to allowlisted workspaces.
- Codex runs with sandboxing enabled by default.
- Model and reasoning level are selected by local bridge policy, not freely specified by the cloud page.
- High-risk actions require explicit human approval.

## Required Guardrails

- Do not upload local auth files or secrets to any cloud server, including `~/.codex/auth.json`, API keys, SSH keys, browser cookies, cloud credentials, `.env` files, or token caches.
- Do not let the web page pass arbitrary shell commands to the local bridge.
- Do not expose the local bridge directly to the public internet.
- Do not run with full user-directory access by default.
- Do not share a personal Codex login session with other users.
- Do not store secrets in task logs or artifacts.

## Pre-Implementation Checklist

Before building a proof of concept, confirm these items:

- The cloud page is protected by a real identity layer.
- The local bridge connects outbound to the relay.
- The local bridge has its own shared secret, token, or signature key.
- Each task has a unique ID, timestamp, and replay protection.
- The local bridge only accepts workspaces from an explicit allowlist.
- The local bridge enforces a timeout and output-size limit.
- The local bridge restricts allowed models, reasoning levels, and concurrency.
- Logs are redacted before being uploaded.
- High-risk commands require manual approval.
- The system can cancel a running task.
- The system records an audit trail for every task.

## High-Risk Actions

The following actions should require manual approval:

- Deleting, moving, or overwriting many files.
- Running deployment, release, publish, reset, or rollback commands.
- Modifying `.env`, SSH, payment, cloud, or production configuration.
- Installing dependencies or executing scripts from the internet.
- Accessing paths outside the configured workspace.
- Triggering email, payment, messaging, or production-side effects.

## Threat Model

| Risk | Typical path | Impact | Mitigation |
| --- | --- | --- | --- |
| Unauthorized access | Cloud auth bypass or leaked session | Remote local execution | IdP / access gateway, JWT checks, extra bridge token |
| Replay attack | Re-submitting old task payload | Repeated dangerous execution | Unique task IDs, timestamps, signatures, nonce |
| Prompt injection | Task asks Codex to read sensitive files | Secret or privacy leakage | Workspace allowlist, sensitive path denylist, redaction |
| Arbitrary command execution | Web page passes raw shell commands | Local machine compromise | Command policy, approval flow, sandbox |
| Overbroad permissions | Codex can access the whole user directory | File leakage or unintended edits | Dedicated OS user, container, fixed working directory |
| Log leakage | Secrets uploaded in logs | Cloud database exposure | Redaction, storage permissions, retention policy |
| Resource exhaustion | Long-running tasks or unlimited output | Local machine slowdown | Timeouts, output caps, concurrency limits |
| Quota runaway | Cloud tasks freely request stronger models or the highest reasoning level | Subscription quota is consumed quickly | Local model allowlist, task tiers, concurrency limits, daily caps |
| Account boundary confusion | Multiple users share one personal login | Compliance and account risk | Single-user model; use API keys or enterprise auth for teams |

## Recommended Defaults

For an early private prototype:

- Use one local workspace.
- Use one local bridge instance.
- Use `workspace-write` sandbox.
- Keep a local model allowlist and task tiers, for example allowing `gpt-5.5` with `xhigh` only for specific task types.
- Use signed polling before adding WebSocket complexity.
- Store only logs and final output at first.
- Avoid automatic file deletion, deployment, publishing, or production actions.
- Keep the bridge config local rather than making the cloud API the only source of policy.

## Reporting

This is currently a concept repository. If you notice a security issue in the architecture description, open an issue with the specific section and suggested correction.

</details>
