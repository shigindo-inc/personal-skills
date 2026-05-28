---
project: personal-skills
status: draft
version: 0.0.1
updated: 2026-05-28
audience: [human, agent]
---

# ADR 0002 — Marketplace + plugin 構造の採用

- **Status**: Accepted
- **Date**: 2026-05-28
- **Deciders**: personal-skills maintainers

## 文脈

転職活動向けの 8 skill + 1 agent を、当初 `~/.claude/skills/` と `~/.claude/agents/` に直置きで運用していたが、以下の不都合が顕在化した：

- **他プロジェクトでもノイズとして表示される**：他リポジトリで作業中に `company-prep` 等の skill が skill list に並ぶ
- **グループ化されない**：関連性のある 9 component が他 skill と混在
- **enable / disable の単位がない**：全 skill 一括で有効、プロジェクト別の切替が不可能
- **バージョン管理されない**：SKILL.md の進化を追えない

## 決定

**Claude Code の marketplace + plugin 機構を採用する。**

- ローカル marketplace（`extraKnownMarketplaces.source = local`）として 1 つ作成
- その中に第一 plugin として `job-search` を配置
- skill / agent はすべて plugin 配下に集約
- enable / disable は `~/.claude/settings.json` の `enabledPlugins["{plugin}@{marketplace}"]` で制御
- プロジェクト側 `.claude/settings.json` の同フィールドで上書き可能

## 帰結

**ポジティブ**:

- プロジェクト毎の enable / disable が `settings.json` 1 行で切替可能
- skill list で `job-search:*` 名前空間に集約、視覚的にグループ化
- plugin 単位で version 管理可能（`plugin.json` の `version` フィールド）
- 将来 `productivity` / `writing-aid` 等の他 plugin を同 marketplace に追加可能

**ネガティブ**:

- 初期セットアップが `marketplace.json` + `plugin.json` + settings 編集の 3 段階で複雑
- SKILL.md 内の他 skill 参照を namespace 付き表記に書き換える必要あり
- plugin 化に伴うトラブルシュート（marketplace 認識不良など）の新たな知識が必要

## 代替案

- **`~/.claude/skills/` 直置きのまま**：上記の不都合が解消されない
- **portfolio repo 内の `.claude/skills/`**：portfolio に強く紐づくが、複数 portfolio 共有や OSS 化の道が断たれる
- **Symlink 運用**：linker 経由で重複稼働になる、保守性が悪い

参照：[ADR 0003](./0003-source-dir-vs-claude-cache-separation.md)（ソース配置と cache の分離）、[ADR 0007](./0007-repo-naming-and-physical-location.md)（命名と物理配置）。
