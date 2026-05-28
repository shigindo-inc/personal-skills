---
project: personal-skills
status: draft
version: 0.0.1
updated: 2026-05-29
audience: [human, agent]
---

# Contributing

## Workflow

1. Create a topic branch.
2. Open a pull request into `main`.
3. Wait for required checks and CODEOWNERS review.
4. Merge only after approval and resolved conversations.

Direct pushes to `main` are not part of the normal workflow, even for
maintainers. Use a reviewed pull request for every public change.

## Public Content Rules

- Keep real personal data in `plugins/*/config/*.local.yaml` only.
- Commit only `*.example.yaml` templates with placeholders or clearly marked
  samples.
- Do not add secrets, tokens, credentials, private keys, real local paths, or
  real job-search profile details.
- Prefer `{{ config.path }}` placeholders for user-specific values.

## Agent Contributions

AI-assisted changes are welcome when a human maintainer reviews the final diff.
Agents must follow `AGENTS.md`, update `docs/tasks/current.md` at start/end of
work, and never add AI attribution trailers to commits.

## Validation

Before requesting review, run or wait for the validation workflow. It checks
JSON/YAML syntax, ignored local config leakage, and privacy-oriented denylist
patterns.
