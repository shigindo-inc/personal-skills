---
project: personal-skills
status: draft
version: 0.0.1
updated: 2026-05-28
audience: [human, agent]
---

# ADR 0007 — リポジトリ命名と物理配置

- **Status**: Accepted
- **Date**: 2026-05-28
- **Deciders**: personal-skills maintainers

## 文脈

本リポジトリの物理配置先と命名は、当初 `~/.claude/marketplaces/personal-job-search/` で計画されていた。しかし：

- `~/.claude/` 配下は Claude Code 自動管理領域（[ADR 0003](./0003-source-dir-vs-claude-cache-separation.md)）
- 「Claude Code 専用 marketplace」名（`personal-job-search`）にすると：
  - 将来 Cursor / Gemini 等の AI ツール対応時に違和感
  - 第二 plugin（例：productivity / writing-aid）を追加するときに名前が job-search に縛られる

## 決定

**物理配置はユーザー管理の OSS / plugin 作業ディレクトリ、命名は `personal-skills` を採用する。**

### 物理配置の選定理由

| 候補 | 選定 | 理由 |
|---|---|---|
| ホーム直下や作業ディレクトリ直下 | ✕ | カテゴリ無し、雑多になる |
| 開発用ディレクトリ直下 | ✕ | private / work / OSS などの区別がない |
| **ユーザー管理の OSS / plugin 作業ディレクトリ** | **○** | 同種の AI ツール OSS と同居でき、将来 OSS 化の含意が明確 |

### 命名の選定理由

| 候補 | 選定 | 理由 |
|---|---|---|
| `personal-job-search` | ✕ | 単一目的の名前、将来拡張で違和感 |
| `claude-personal-pack` | ✕ | Claude Code に強く紐づく、他 AI ツール展開時に違和感 |
| `personal-ai-toolkit` | ✕ | 「toolkit」のニュアンスが過剰、ツール集合という含意 |
| **`personal-skills`** | **○** | シンプル、Claude Code 非依存、AGENTS.md 規格（複数 AI ツール対応）志向 |

### 第一 plugin 名

`job-search` を採用。将来 `productivity` / `writing-aid` / `code-review` 等を同 marketplace 配下に追加可能な拡張構造。

## 帰結

**ポジティブ**:

- 同種の AI ツール OSS と同居することで「AI ツール関連 OSS」カテゴリが視覚化
- 名前が Claude Code 非依存、AGENTS.md 規格対応の将来性
- 第二 plugin 追加時に marketplace 名を変更しなくて済む
- public 化時、リポジトリ root 名がそのまま公開名になる（再命名のコストなし）

**ネガティブ**:

- 「personal-skills」だけだと内容が分かりにくい（marketplace 名としては抽象的）→ `marketplace.json` の `metadata.description` で補完

## 代替案

詳細は上記表のとおり。検討した命名すべて記録済み。

## 関連

- [ADR 0002](./0002-marketplace-plugin-structure.md)
- [ADR 0003](./0003-source-dir-vs-claude-cache-separation.md)
- [ADR 0006](./0006-publication-readiness-checklist.md)
