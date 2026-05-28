# job-search plugin

転職活動を一気通貫で支える 8 skill + 1 orchestrator agent。

## 全体図

```
                         ┌───────────────────────────┐
                         │ interview-prep (agent)    │
                         │  README.md の現フェーズで    │
                         │  必要 skill を順次提案       │
                         └──────────────┬────────────┘
                                        │ 各 skill の SKILL.md を Read
        ┌───────────────┬───────────────┼───────────────┬──────────────────┐
        ▼               ▼               ▼               ▼                  ▼
┌─────────────────┐ ┌──────────────┐ ┌───────────────┐ ┌──────────────────┐ ┌──────────────────────┐
│ company-prep    │ │ interview-qa │ │ reverse-      │ │ interview-       │ │ mock-interview        │
│  企業フォルダ立上 │ │  -generator   │ │  questions-    │ │   retrospective  │ │  AI 面接官 5 ペルソナ  │
│  + ダッシュ追記   │ │  当社特化問答  │ │  curator       │ │  面接ログ作成     │ │  × 4 モード           │
└─────────────────┘ └──────────────┘ └───────────────┘ └──────────────────┘ └──────────────────────┘
        │                                                       │                   │
        ▼                                                       ▼                   ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        feedback-integrator                                       │
│       複数面接（本番 + 模擬）の横断 FB → 5 カテゴリ分類 → コア資産リファクタ           │
│       cv-builder / interview-qa-generator / deep-self-analysis に委譲             │
└─────────────────────────────────────────────────────────────────────────────────┘

                ┌──────────────────┐    ┌────────────────────────┐
                │ offer-evaluator   │    │ agent-interview-prep    │
                │  複数オファー比較   │    │  転職エージェント面談用   │
                └──────────────────┘    └────────────────────────┘
```

## 9 component の起動例

| component | トリガー | 例 |
|---|---|---|
| `company-prep` | 新規応募企業 | `company-prep AcmeCorp --url https://acme.co --axis po` |
| `interview-qa-generator` | 面接前の想定問答カスタム | `interview-qa-generator AcmeCorp` |
| `reverse-questions-curator` | 逆質問キュレーション | `reverse-questions-curator AcmeCorp --round 一次` |
| `interview-retrospective` | 面接実施後の振り返り | `interview-retrospective AcmeCorp --date 2026-06-01 --round 一次` |
| `mock-interview` | 模擬面接 | `mock-interview AcmeCorp --persona テック --strict` |
| `feedback-integrator` | 複数面接横断 FB | `feedback-integrator --from-logs` |
| `offer-evaluator` | 内定比較 | `offer-evaluator AcmeCorp BetaInc` |
| `agent-interview-prep` | 転職エージェント面談 | `agent-interview-prep （例：外資系エージェント） --round 初回` |
| `interview-prep`（agent）| 何でも入り口 | 「AcmeCorpの面接準備」 |

## personal-config の編集

各 skill は `config/personal-config.local.yaml` から個人情報・パスを取得する：

```bash
cd plugins/job-search/config
cp personal-config.example.yaml personal-config.local.yaml
# 編集
vi personal-config.local.yaml
```

主要フィールド：

| キー | 用途 |
|---|---|
| `person.name` | 辞退連絡署名等 |
| `paths.portfolio_root` | 全 skill の `${USER_PORTFOLIO_ROOT}` を解決 |
| `career.*` | 経歴期間・職歴名（将来 Phase 3'-B 完遂時に skill 本文から参照）|
| `side_projects.projects[]` | 副業プロジェクト名 |
| `side_projects.hours_per_week` | 副業時間の実値（公開 SKILL.md には直書きしない） |
| `qualifications.toeic` | 資格スコア |
| `education.gap_period` | 学歴ギャップ説明用 |
| `education.total_years` | 在籍年数など、学歴深掘りの文脈 |
| `global_experience.*` | 海外滞在・オフショア協業などの国際経験 |
| `mobility.*` | 通勤可能エリア・リモート希望・出張可否 |
| `compensation.*` | 希望年収レンジ・最低希望年収 |

詳細：personal-skills repo root の [ADR 0004](../../docs/adr/0004-personal-info-abstraction-strategy.md)（個人情報の段階的汎用化）

## 設計判断のポイント

- **interview-prep agent は orchestrator**：各 skill を直接 invoke せず、`${CLAUDE_PLUGIN_ROOT}/skills/{name}/SKILL.md` を Read してから実行
- **本番ログと模擬ログを `MOCK_` プレフィックスで分離**：mock-interview の生成物は `MOCK_YYYY-MM-DD_*.md`
- **マスタファイル（24 想定問答 / 32 評価チェック / 36 ダッシュボード）は portfolio 側**：本 plugin は portfolio 内のドキュメントを操作する pipeline であり、portfolio がないと動かない
- **`${USER_PORTFOLIO_ROOT}` は personal-config から解決**：絶対パス埋め込みを避け、将来 public 化に対応（Phase 3'-A 完了済み）
- **個人に近い経歴・希望条件は personal-config から解決**：学歴ギャップ、副業件数 / 時間、海外経験、勤務地、希望年収などは公開 SKILL.md に実値を書かず、`personal-config.local.yaml` の値を参照する

## 外部 plugin への依存

本 plugin は他の skill を **読みに行く** ことがある：

| 外部 skill | 参照する場面 |
|---|---|
| `cv-builder` | `interview-qa-generator` の `cv-builder --variant` 提案 |
| `cv-diff` | `feedback-integrator` の整合点検 |
| `sns-helper` | `agent-interview-prep` の外資エージェント連携時 |
| `deep-self-analysis` | `interview-qa-generator` / `interview-retrospective` / `feedback-integrator` / `mock-interview` の C5 検出時 |

これらは `~/.claude/skills/{name}/SKILL.md` を参照（本 plugin 外）。

## skill / agent 一覧

| | name | description（要約） |
|---|---|---|
| skill | company-prep | 企業フォルダ立ち上げ + URL から `企業情報.md` 初期埋め + ダッシュボード行追加 |
| skill | interview-qa-generator | 汎用 Q&A + YAML + 素材 + 行動原理 + 企業情報 から `想定問答_カスタム.md` を企業特化版へ |
| skill | reverse-questions-curator | 企業情報空欄 + 評価チェック未確認 + E 章 + 業界別 から `逆質問.md` 構築 |
| skill | interview-retrospective | 面接後ログ作成 + 24 / 32 / 企業情報 / README / ダッシュボードへ FB |
| skill | feedback-integrator | 横断 FB → 5 カテゴリ分類 → コア資産リファクタを統制 |
| skill | mock-interview | AI 面接官 5 ペルソナ × 4 モード + 5 軸評価 + MOCK_ プレフィックスでログ保存 |
| skill | offer-evaluator | 複数内定比較 + 交渉スクリプト + 辞退連絡テンプレ |
| skill | agent-interview-prep | 転職エージェント別に自己紹介スクリプト + 当日チェックリスト |
| agent | interview-prep | 上記 8 skill のオーケストレータ。フェーズ判定で必要 skill を順次提案 |
