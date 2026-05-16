# Contributing

This repository is a concept and architecture note for a private cloud-to-local Codex execution bridge.

Contributions are welcome when they make the idea clearer, safer, or more practical.

## Good Contributions

- Clarify architecture wording.
- Improve diagrams.
- Add platform mapping examples.
- Add threat-model improvements.
- Add proof-of-concept sketches.
- Add safer local bridge implementation notes.
- Improve explanations for beginners.

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

For teams, public users, or commercial services, the safer direction is official API keys, enterprise authentication, or other officially supported automation flows.
