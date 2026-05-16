# Contributing / 贡献说明

<details open>
<summary><strong>中文</strong></summary>

本仓库是一份关于私有 Cloud-to-Local Codex 执行桥的概念与架构说明。

欢迎贡献能让这个想法更清晰、更安全、更实用的内容。

## 欢迎的贡献

- 优化架构表述。
- 改进图示。
- 补充平台映射示例。
- 改进威胁模型。
- 补充概念验证草案。
- 补充更安全的本地桥接器实现说明。
- 让说明对通用读者更易理解。

## 请避免

- 把本项目改成公开 API 代理教程。
- 鼓励账号共享。
- 鼓励绕过使用限制、速率限制、计费系统或安全机制。
- 建议把本地桥接器暴露给不可信用户。
- 添加缺少策略校验、却能执行任意 shell 命令的生产代码。

## Issue 建议格式

提交 issue 时，建议包含：

- 你评论的是哪个章节。
- 哪些地方不清楚或有风险。
- 你建议的表述或设计修改。
- 你的反馈侧重于清晰性、实现、安全、架构还是项目边界。

## 项目方向

本项目应保持聚焦：单用户、私有、自托管的本地执行器模式。

```text
云端控制面 -> 安全中转层 -> 本地桥接器 -> 本地 Codex CLI -> 云端结果页
```

团队、公开用户或商业服务场景，更稳妥的方向是官方 API key、企业认证、Codex access token，或其他官方支持的自动化方式。

</details>

<details>
<summary><strong>English</strong></summary>

This repository is a concept and architecture note for a private cloud-to-local Codex execution bridge.

Contributions are welcome when they make the idea clearer, safer, or more practical.

## Good Contributions

- Clarify architecture wording.
- Improve diagrams.
- Add platform mapping examples.
- Add threat-model improvements.
- Add proof-of-concept sketches.
- Add safer local bridge implementation notes.
- Improve explanations for a general audience.

## Please Avoid

- Turning this into a public API proxy guide.
- Encouraging account sharing.
- Encouraging bypassing usage limits, rate limits, billing systems, or safety mechanisms.
- Suggesting designs that expose the local bridge to untrusted users.
- Adding production code that runs arbitrary shell commands without policy checks.

## Suggested Issue Format

When opening an issue, include:

- The section you are commenting on.
- What is unclear or risky.
- Your suggested wording or design change.
- Whether your feedback is about clarity, implementation, security, architecture, or project scope.

## Project Direction

The project should stay focused on a single-user, private, self-hosted runner pattern:

```text
cloud control plane -> secure relay -> local bridge -> local Codex CLI -> cloud result view
```

For teams, public users, or commercial services, the safer direction is official API keys, enterprise authentication, Codex access tokens, or other officially supported automation flows.

</details>
