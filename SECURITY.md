---
project: personal-skills
status: draft
version: 0.0.1
updated: 2026-05-29
audience: [human, agent]
---

# Security Policy

## Supported Versions

This repository is pre-1.0. Security fixes target the latest `main` branch and
the latest protected release tag.

## Reporting a Vulnerability

Do not open a public issue for suspected secret leakage, prompt injection,
malicious plugin behavior, or supply-chain concerns.

Report privately through GitHub Security Advisories when available. If advisories
are unavailable, contact the repository maintainers through a private channel and
include:

- affected file or plugin name
- reproduction steps
- expected impact
- whether any secret, personal data, or local path appears in public content

## Security Expectations

- Do not commit `.env`, `*.local.yaml`, credentials, tokens, private keys, local
  paths containing user identifiers, or real personal profile data.
- Public examples must use placeholders or clearly marked samples.
- Changes to marketplace metadata, plugin manifests, workflow files, or security
  docs require maintainer review before merge.
- Release tags are immutable once published.

## Agent Safety

AI agents must not:

- push directly to protected branches
- merge pull requests without explicit human approval
- weaken validation, CODEOWNERS, or repository protection settings
- add new remote code execution behavior without documenting and reviewing it
