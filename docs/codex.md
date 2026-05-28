---
project: personal-skills
status: draft
version: 0.0.1
updated: 2026-05-29
audience: [human, agent]
---

# Codex で使う

Codex は Claude Code の marketplace 設定をそのまま読むのではなく、
Agent Skills として `SKILL.md` をインストールして使う。

## 推奨方針

初回は重複管理を避けるため、Claude Code plugin 構造を source of truth とし、
Codex 側では GitHub directory URL から必要な skill を取り込む。

## インストール例

Codex で `$skill-installer` を使える場合：

```text
$skill-installer install https://github.com/shigindo-inc/personal-skills/tree/v0.1.0/plugins/job-search/skills/company-prep
$skill-installer install https://github.com/shigindo-inc/personal-skills/tree/v0.1.0/plugins/job-search/skills/interview-qa-generator
$skill-installer install https://github.com/shigindo-inc/personal-skills/tree/v0.1.0/plugins/job-search/skills/reverse-questions-curator
$skill-installer install https://github.com/shigindo-inc/personal-skills/tree/v0.1.0/plugins/job-search/skills/interview-retrospective
$skill-installer install https://github.com/shigindo-inc/personal-skills/tree/v0.1.0/plugins/job-search/skills/feedback-integrator
$skill-installer install https://github.com/shigindo-inc/personal-skills/tree/v0.1.0/plugins/job-search/skills/mock-interview
$skill-installer install https://github.com/shigindo-inc/personal-skills/tree/v0.1.0/plugins/job-search/skills/offer-evaluator
$skill-installer install https://github.com/shigindo-inc/personal-skills/tree/v0.1.0/plugins/job-search/skills/agent-interview-prep
```

インストール後、Codex セッションを再起動して skill discovery を更新する。

## repo-scoped に置く場合

特定プロジェクトだけで使うなら、各 skill directory を対象 repo の
`.agents/skills/` 配下へコピーする。

```text
target-repo/
└── .agents/
    └── skills/
        └── company-prep/
            └── SKILL.md
```

## 注意点

- Codex での tool 名や権限モデルは Claude Code と同一ではないため、各 skill の
  手順を実行前に読み替える。
- `personal-config.local.yaml` は各利用環境で作成し、公開 repo にコミットしない。
- release tag を指定してインストールし、任意の `main` 更新を自動で取り込まない。
