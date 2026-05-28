---
project: personal-skills
status: draft
version: 0.0.1
updated: 2026-05-28
audience: [human, agent]
---

# aikata へのフィードバック（LLM 向け要約）

> 本ドキュメントは aikata（https://github.com/shigindo-inc/aikata）の
> 改善余地を、aikata 側の LLM / メンテナーが読みやすい形でまとめたもの。
> personal-skills repo の運用で観察された具体ケースを起点に、
> 仕様提案を含む。

## 観察事象

### コンテキスト

- 利用バージョン：aikata v0.6.1
- 利用構成：`aikata init <name> --preset standard --lang ja --ai-tools claude --no-interactive`
- 対象 repo：Claude Code の marketplace + plugin（`.claude-plugin/marketplace.json` + `plugins/<name>/.claude-plugin/plugin.json` + `plugins/<name>/skills/<name>/SKILL.md` 等を含む）

### 事象

`aikata doctor` が `plugins/job-search/skills/*/SKILL.md` および `plugins/job-search/skills/*/references/*.md` に対して、aikata 規格の frontmatter（`project`、`status`、`version`、`updated`、`audience`）が無いと **error** を返す。

```
plugins/job-search/skills/mock-interview/SKILL.md: [error] frontmatter missing required key "project"
plugins/job-search/skills/mock-interview/SKILL.md: [error] frontmatter missing required key "status"
（他、計 62 件）
```

しかし `SKILL.md` は Claude Code（Anthropic 公式）の plugin 仕様で定義された別規格のドキュメントで、frontmatter には `name` と `description` を必須とし、aikata 規格のキー（project / status / version / updated / audience）は **想定しない**。

両規格は同じ md ファイルでは共存しづらい（過剰な frontmatter を入れると Claude Code 側の skill list 表示で description が長くなりすぎる、または invalid と扱われる可能性）。

## 影響

- aikata doctor を CI gate にしている場合、Claude Code の plugin 構造を含む repo は doctor が pass しない
- 利用者は以下のいずれかの逃げ道を選ばざるを得ない：
  1. SKILL.md に aikata 規格の frontmatter を追加（Claude Code 側の挙動次第で副作用リスク）
  2. aikata doctor の error を意図的に無視（CI gate を緩める）
  3. SKILL.md の frontmatter を aikata 規格に合わせ、Claude Code 側で description キーは別途確保（=規格折衷の独自対応、可搬性悪化）

## 提案

aikata 側で吸収していただけるとありがたい仕様：

### 提案 A：`.aikata/aikata.yaml` に lint exclude を追加

```yaml
version: 1
project:
  name: personal-skills
  lang: ja
ai_tools:
  - claude
features:
  monorepo: false
docs:
  generate_gitignore: true
  task_file_location: docs/tasks/current.md
lint:
  exclude:
    - "plugins/**"            # Claude Code plugin 配下を doctor 対象外に
    - "**/SKILL.md"            # 特定ファイル名で除外
    - "**/.claude-plugin/**"
  frontmatter_required_paths:  # 検証対象を逆に絞る指定方式も併用可
    - "docs/**"
    - "AGENTS.md"
    - "ARCHITECTURE.md"
    - "SPEC.md"
    - "GLOSSARY.md"
    - "README.md"
```

直感的でメンテしやすい。

### 提案 B：標準で「Claude Code plugin 構造」を認識

- `.claude-plugin/marketplace.json` / `.claude-plugin/plugin.json` が存在する場合、`plugins/**/skills/**/SKILL.md` と `plugins/**/agents/**/*.md` は **Claude Code 規格として frontmatter 検証スキーマを切替**
- aikata 規格と Claude Code 規格を併存させる流儀を組み込み的にサポート

将来 cursor / codex / gemini-cli 等の plugin 構造が増えても、各ツール固有 frontmatter 検証として拡張可能。

### 提案 C：`doctor` に severity 切替

- 現状：必須キー欠如 = `error`
- 提案：必須キー欠如のうち、aikata が「自分の管理対象でない」と判定したファイル群は `info` または `warn` に降格
- `--strict` 時のみ error 化、平時は warn

## ユースケース別の期待挙動

| repo の構成 | 期待挙動 |
|---|---|
| aikata 単独（plugin なし） | 現状通り、全 md を aikata 規格で検証 |
| aikata + Claude Code plugin（本ケース） | aikata 規格は docs/ / AGENTS.md 等のみ。plugin/skills/ は除外 |
| aikata + 他 AI tool plugin（将来） | 各 tool の規格に従い検証スキーマ切替 |
| 非 aikata repo に aikata install | 検証対象外、`aikata init` を促す |

## 関連リンク

- Claude Code plugin spec: https://docs.claude.com/ja/docs/claude-code/plugins
- 本 repo の plugin 構造： `plugins/job-search/`（参照可、AGENTS.md と並列で skill / agent 群が同居）
- 影響ファイル一覧（62 件全部）：`aikata doctor 2>&1` 出力参照

## 暫定対応（personal-skills 側）

aikata 側の修正を待つ間、personal-skills では：

- `aikata doctor` の error は **既知の誤検出として無視** する運用
- 本ドキュメント自体を `audience: [human, agent]` で共有
- aikata 側で対応されたらこのフィードバック文書は `Status: Superseded` で残し、`.aikata/aikata.yaml` に `lint.exclude` 等を追記

## 共有先

- 後日、ユーザー個人が aikata の repo / issue tracker / メンテナーに共有予定
- メンテナーが LLM ベースで本 md を読んで仕様調整をする想定（aikata 自身が AI コーディング時代向けツールのため）
