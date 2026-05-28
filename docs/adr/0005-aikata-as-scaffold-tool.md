---
project: personal-skills
status: draft
version: 0.0.1
updated: 2026-05-28
audience: [human, agent]
---

# ADR 0005 — aikata を scaffold ツールとして採用

- **Status**: Accepted
- **Date**: 2026-05-28
- **Deciders**: personal-skills maintainers

## 文脈

複数の AI ツール（Claude Code / Cursor / Codex / Gemini CLI）が共通の `AGENTS.md` 規約を採用しつつある。本リポジトリは「Claude Code 専用 plugin」ではなく「AGENTS.md 規格に準拠した skill 集合」として将来展開可能にしたい。

その際、CLAUDE.md / `.cursor/rules/` / GEMINI.md 等の AI ツール固有ファイルを **手作業で管理すると drift する**。

## 決定

**[aikata](https://github.com/shigindo-inc/aikata)（Go バイナリ、`~/.local/bin/aikata` 設置済み）を本リポジトリの scaffold + 整合管理ツールとして採用する。**

- **`AGENTS.md` を真の source（SoT）として運用**
- `aikata generate` で各 AI ツール用ファイル（CLAUDE.md 等）を自動生成
- 生成物は `.gitignore` 対象（`AGENTS.md` から再生成可能なため）
- `aikata doctor` で SoT と生成物の整合点検

### 採用構成

- preset：`standard`（`AGENTS.md` / `SPEC.md` / `ARCHITECTURE.md` / `GLOSSARY.md` / `docs/adr/` / `docs/tasks/` を生成）
- lang：`ja`
- ai-tools：`claude`（必要時 `cursor` / `codex` を追加）
- 初期コマンド：`aikata init personal-skills --preset standard --lang ja --ai-tools claude --no-interactive`

## 帰結

**ポジティブ**:

- AGENTS.md / CLAUDE.md の drift が `aikata doctor` で検出可能
- AI ツール追加（Cursor / Gemini）は `aikata generate` で対応可能、手作業の追加コードゼロ
- ADR / glossary / troubleshooting / tasks の置き場所が aikata 標準で固定され、迷わない

**ネガティブ**:

- aikata 自体への依存（バージョン互換性、shigindo-inc のメンテ継続）
- `.gitignore` で CLAUDE.md が除外されるため、IDE で初回開いた時に CLAUDE.md が存在しない可能性 → `aikata generate` を README に明記して回避

## 代替案

- **手動で CLAUDE.md を維持**：drift リスク高、AGENTS.md と二重メンテ
- **CLAUDE.md を SoT、AGENTS.md を生成物**：複数 AI ツール対応が難しくなる
- **テンプレートエンジン自作**：オーバーエンジニアリング

## 運用ルール

- `AGENTS.md` を編集したら必ず `aikata generate` を実行
- ADR を追加したら `docs/adr/NNNN-title.md` 形式（aikata の `0001-record-architecture-decisions.md` に準拠）
- `aikata doctor` を PR / 公開前に必ず実行

参照：aikata の install / コマンド詳細は upstream repository の README を確認する。
