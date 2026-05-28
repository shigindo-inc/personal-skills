---
project: personal-skills
status: draft
version: 0.0.1
updated: 2026-05-28
audience: [human, agent]
---

# personal-skills

> 個人用 AI skills marketplace（Claude Code 規格 + AGENTS.md 規格対応）。
> 第一 plugin として `job-search`（転職活動向け 8 skill + 1 orchestrator agent）を収録。

このリポジトリは [aikata](https://github.com/shigindo-inc/aikata) で
スキャフォールド（`standard --lang ja --ai-tools claude`）したのち、
[Claude Code の plugin / marketplace 仕様](https://docs.claude.com/ja/docs/claude-code/plugins)
を重ねた構成です。

## 何が入っているか

```
personal-skills/                              # この repo
├── AGENTS.md                                 # 真の source（aikata 管理）
├── SPEC.md / ARCHITECTURE.md / GLOSSARY.md
├── CLAUDE.md                                 # aikata generate で再生成（.gitignore）
├── docs/
│   ├── adr/                                  # ADR 0001〜0007
│   ├── tasks/ / troubleshooting.md / prompts.md
│   └── aikata-feedback.md                    # aikata 側へのフィードバック
├── .claude-plugin/marketplace.json           # Claude Code marketplace 定義
├── .agents/plugins/marketplace.json          # Codex marketplace 定義
└── plugins/
    └── job-search/                           # 第一 plugin
        ├── .claude-plugin/plugin.json
        ├── .codex-plugin/plugin.json
        ├── config/
        │   ├── personal-config.example.yaml  # 公開テンプレ
        │   └── personal-config.local.yaml    # 実値（.gitignore）
        ├── skills/                           # 8 skill
        │   ├── company-prep/
        │   ├── interview-qa-generator/
        │   ├── reverse-questions-curator/
        │   ├── interview-retrospective/
        │   ├── feedback-integrator/
        │   ├── mock-interview/
        │   ├── offer-evaluator/
        │   └── agent-interview-prep/
        └── agents/
            └── interview-prep.md             # orchestrator
```

詳細：[plugins/job-search/README.md](./plugins/job-search/README.md)

## 最初に読むもの

| 目的 | ドキュメント |
|---|---|
| 何を／なぜ | [SPEC.md](./SPEC.md) |
| どう作るか（技術） | [ARCHITECTURE.md](./ARCHITECTURE.md) |
| 用語 | [GLOSSARY.md](./GLOSSARY.md) |
| エージェント運用ルール | [AGENTS.md](./AGENTS.md) |

### 設計・決定

- [`docs/adr/`](./docs/adr/) — Architecture Decision Records（7 件）
  - 0001：ADR を採用する判断（aikata 自動生成）
  - 0002：Marketplace + plugin 構造の採用
  - 0003：ソース配置と Claude Code cache の分離
  - 0004：個人情報の段階的汎用化方針
  - 0005：aikata を scaffold ツールとして採用
  - 0006：公開戦略と公開前チェックリスト
  - 0007：リポジトリ命名と物理配置

### 運用ノート（頻繁に更新）

- [`docs/tasks/current.md`](./docs/tasks/current.md) — 現在進行中の作業
- [`docs/troubleshooting.md`](./docs/troubleshooting.md) — 既知の落とし穴
- [`docs/prompts.md`](./docs/prompts.md) — 再利用可能なプロンプト集
- [`docs/aikata-feedback.md`](./docs/aikata-feedback.md) — aikata 側への改善提案（後日 aikata repo に共有予定）

## クイックスタート（GitHub marketplace として使う）

### 前提

- Claude Code がインストール済み
- GitHub から `shigindo-inc/personal-skills` を取得できる
- 実値を入れる `personal-config.local.yaml` は利用者のローカル環境で作成する

### 1. personal-config を作成

```bash
git clone https://github.com/shigindo-inc/personal-skills.git
cd plugins/job-search/config
cp personal-config.example.yaml personal-config.local.yaml
# personal-config.local.yaml を編集し、自分の値を入れる
# （.gitignore 対象、git にコミットされない）
```

### 2. Claude Code に marketplace を登録

`~/.claude/settings.json` に以下を追記：

```json
{
  "extraKnownMarketplaces": {
    "personal-skills": {
      "source": {
        "source": "github",
        "repo": "shigindo-inc/personal-skills",
        "ref": "v0.1.0"
      }
    }
  },
  "enabledPlugins": {
    "job-search@personal-skills": false
  }
}
```

`enabledPlugins` を `false` にしておくと、グローバルでは無効。
個別プロジェクトの `.claude/settings.json`（or `settings.local.json`）で `true` に上書きすると、そのプロジェクトでだけ有効化される。

### 2.5. marketplace を制限する（任意）

組織管理端末などで、未承認 marketplace の追加を避けたい場合は
Claude Code の managed settings で `strictKnownMarketplaces` を使う。
以下は `personal-skills` と公式 marketplace だけを許可する例：

```json
{
  "strictKnownMarketplaces": [
    {
      "source": "github",
      "repo": "shigindo-inc/personal-skills",
      "ref": "v0.1.0"
    },
    {
      "source": "github",
      "repo": "anthropics/claude-plugins-official"
    }
  ]
}
```

### 3. プロジェクトで有効化

例：転職活動 portfolio リポジトリで `job-search` を有効化：

```jsonc
// portfolio/.claude/settings.local.json
{
  "enabledPlugins": {
    "job-search@personal-skills": true
  }
}
```

### 4. 動作確認

```bash
# Claude Code を再起動 / セッション開始
# skill list に "job-search:*" が 8 件 + agent 1 件出ているか
```

## Codex で使う

Codex は `codex plugin marketplace add` で同じ GitHub repo を marketplace として追加できる。
詳細は [`docs/codex.md`](./docs/codex.md) を参照。

```bash
codex plugin marketplace add shigindo-inc/personal-skills --ref v0.1.1
```

## aikata 連携

このリポジトリは **AGENTS.md を真の source（SoT）** として運用する：

- `AGENTS.md` を編集したら `aikata generate` で `CLAUDE.md` を再生成
- PR や公開前に `aikata doctor` で整合点検
- 詳細は [ADR 0005](./docs/adr/0005-aikata-as-scaffold-tool.md)

> ⚠️ 現在、`aikata doctor` は `plugins/**/SKILL.md` 等の Claude Code 規格 frontmatter を aikata 規格として誤検出し、error 62 件を返す。これは既知の問題で、[`docs/aikata-feedback.md`](./docs/aikata-feedback.md) に aikata 側への提案を整理済み（後日共有）。当面は SKILL.md 系の error は無視して運用する。

## 設定

- aikata 設定：[`.aikata/aikata.yaml`](./.aikata/aikata.yaml)
- personal-config（個人情報・経歴・希望条件）：`plugins/job-search/config/personal-config.local.yaml`（.gitignore 対象）
- 環境変数：[`.env.example`](./.env.example) を参考にする

## 公開運用とセキュリティ

- 直接 `main` へ push しない。変更は pull request 経由で取り込む
- `.github/CODEOWNERS` 対象の marketplace / plugin / workflow / security 変更は maintainer review 必須
- release tag（例：`v0.1.0`）は移動・削除しない
- 秘密情報や実値は `.local.yaml` / `.env` にのみ置き、公開ファイルへ書かない
- 詳細は [SECURITY.md](./SECURITY.md) と [CONTRIBUTING.md](./CONTRIBUTING.md)

## ライセンス

MIT License — [LICENSE](./LICENSE) を参照。

このリポジトリは個人向けに作成された skill 集合を OSS として再利用可能にしたもの。`config/personal-config.example.yaml` をコピーして自分用 `.local.yaml` を作成すれば、自分の転職活動でも利用可能（実値の `.local.yaml` は `.gitignore` 対象なので、コミット時に個人情報・経歴・希望条件が漏れることはない）。

## 拡張

将来、同 marketplace 配下に他 plugin を追加する場合：

1. `plugins/<new-plugin>/.claude-plugin/plugin.json` を作成
2. `.claude-plugin/marketplace.json` の `plugins[]` に追記
3. `~/.claude/settings.json` の `enabledPlugins` に登録

詳細は [ADR 0002](./docs/adr/0002-marketplace-plugin-structure.md) と [ADR 0007](./docs/adr/0007-repo-naming-and-physical-location.md)。
