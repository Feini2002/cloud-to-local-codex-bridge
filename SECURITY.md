# Security Policy

This repository describes a private cloud-to-local execution bridge pattern. The most important security boundary is not the cloud page itself, but what the cloud page is allowed to make the local machine do.

## Intended Security Model

- Single-user private use.
- Cloud access protected by an identity provider or access gateway, such as Cloudflare Access, Auth0, Clerk, Supabase Auth, GitHub OAuth, or a private self-hosted login.
- Local bridge connects outbound to the cloud relay; it should not expose a public listening port.
- Every task has a unique ID, timestamp, signature, and audit trail.
- Local execution is limited to allowlisted workspaces.
- Codex runs with sandboxing enabled by default.
- High-risk actions require explicit human approval.

## Required Guardrails

- Do not upload `~/.codex/auth.json` to any cloud server.
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
- The local bridge only accepts allowlisted workspaces.
- The local bridge enforces a timeout and output-size limit.
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
| Account boundary confusion | Multiple users share one personal login | Compliance and account risk | Single-user model; use API keys or enterprise auth for teams |

## Recommended Defaults

For an early private prototype:

- Use one local workspace.
- Use one local bridge instance.
- Use `workspace-write` sandbox.
- Use signed polling before adding WebSocket complexity.
- Store only logs and final output at first.
- Avoid automatic file deletion, deployment, publishing, or production actions.
- Keep the bridge config local rather than making the cloud API the only source of policy.

## Reporting

This is currently a concept repository. If you notice a security issue in the architecture description, open an issue with the specific section and suggested correction.
