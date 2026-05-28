---
project: personal-skills
status: draft
version: 0.0.1
updated: 2026-05-28
audience: [human, agent]
---

# ADR 0003 — ソース配置と Claude Code cache の分離

- **Status**: Accepted
- **Date**: 2026-05-28
- **Deciders**: personal-skills maintainers

## 文脈

`~/.claude/plugins/` 配下は Claude Code が外部 marketplace（GitHub / local source）を取り込んで `marketplaces/` / `cache/` に同期する **自動管理領域** である。

ここに自前ソースを直接置くと：

- Claude Code の自動同期で上書き / 削除されるリスク
- どこが「ソース」でどこが「cache」か責務不明瞭
- ユーザー操作（`git init` 等）と Claude Code 操作が衝突

## 決定

**ソース実体は任意のユーザー管理ディレクトリ（例：`~/path/to/personal-skills/`）に置く。`~/.claude/` 配下には設定の参照のみを置く。**

- ソース：`~/path/to/personal-skills/`（自分で git 管理、編集する）
- Claude Code 設定：`~/.claude/settings.json` の `extraKnownMarketplaces["personal-skills"].source = { source: "local", path: <絶対パス> }` で source 位置を指す
- Claude Code 自動管理：`~/.claude/plugins/marketplaces/personal-skills/` 等は **手で触らない**

## 帰結

**ポジティブ**:

- ソースと cache の責務が完全分離、混乱が起きない
- git 管理が clean（自動同期される領域に `.git/` を作らないで済む）
- ユーザー管理の OSS / plugin 作業ディレクトリに置くことで保守ローテに乗る
- 将来 GitHub 公開時、リポジトリ root をそのまま push できる

**ネガティブ**:

- `~/.claude/settings.json` への `extraKnownMarketplaces` 登録が必要（追加の設定手順）
- 別マシン移行時、`extraKnownMarketplaces` を手で再設定するか、settings.json 自体を持ち運ぶ必要あり

## 代替案

- **`~/.claude/marketplaces/personal-skills/` に直接置く**：自動管理領域との混在、危険
- **portfolio repo 配下に置く**：portfolio に強く紐づき、複数 portfolio 共有や独立 OSS 化が困難
- **GitHub private repo + Claude Code が clone**：マシン跨ぎでよいが、ローカル試作段階では過剰

参照：[ADR 0002](./0002-marketplace-plugin-structure.md)、[ADR 0007](./0007-repo-naming-and-physical-location.md)。
