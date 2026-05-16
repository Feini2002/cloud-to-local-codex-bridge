# Cloud-to-Local Codex Bridge

![Status: Concept](https://img.shields.io/badge/status-concept-blue)
![Docs: Architecture](https://img.shields.io/badge/docs-architecture-informational)
![Security: Threat Model](https://img.shields.io/badge/security-threat%20model-informational)

云端发任务，本地 Codex 执行，结果回写云端。  
Route tasks from a private cloud UI to a local Codex CLI runner, then send logs and results back to the cloud.

## Language

- [中文说明](docs/zh/README.md)
- [English README](docs/en/README.md)

## Quick Scope

This repository is a **concept / architecture note**, not a runnable software package yet.

It describes a private, self-hosted execution pattern:

```text
Web UI -> Cloud Control Plane -> Local Bridge -> Local Codex Runtime -> Result Store
```

The key boundary:

```text
The cloud can request work.
The local bridge decides what is allowed to execute.
```

## Repository Map

```text
README.md              # Language entry and repository overview
docs/README.md         # Documentation index
docs/zh/README.md      # 中文 README
docs/en/README.md      # English README
docs/architecture.md   # Full architecture note, currently in Chinese
SECURITY.md            # Threat model and security checklist
DISCLAIMER.md          # Scope and usage disclaimer
CONTRIBUTING.md        # Contribution guidelines
.gitignore
```

## Important Boundary

This project is not a public API proxy, not an account-sharing guide, and not a way to bypass usage limits, billing systems, rate limits, or safety mechanisms.

It is a private self-hosted runner pattern for a single user controlling their own local machine.

## Documentation

Start with the language version you prefer:

- [中文说明](docs/zh/README.md)
- [English README](docs/en/README.md)

Then continue with:

- [Documentation Index](docs/README.md)
- [Security Policy](SECURITY.md)
- [Disclaimer](DISCLAIMER.md)
- [Contributing](CONTRIBUTING.md)
