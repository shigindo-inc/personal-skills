---
project: personal-skills
status: draft
version: 0.0.1
updated: 2026-05-28
audience: [human, agent]
---

# ADR 0004 — 個人情報の段階的汎用化方針

- **Status**: Accepted
- **Date**: 2026-05-28
- **Deciders**: personal-skills maintainers

## 文脈

skill / agent ファイルには個人固有の以下が散在していた（初期スキャンで多数検出）：

- **絶対パス**：ローカル portfolio / workspace への実パス
- **人名**：フルネーム / イニシャル / 名 / 姓 等の各表記揺れ
- **企業名・職歴具体**：職歴名 / 取引先名 / 参画先名 / 期間 等
- **数値（実績・年収）**：資格スコア、実績件数、削減率、希望年収 等
- **副業プロジェクト名**：個別プロダクト名、チャネル名、案件数 等
- **エージェント名**：利用中の転職エージェント名 等

将来 public 公開する場合、これらは段階的に抽象化する必要がある。一方、すべてを一気にプレースホルダ化すると skill 本文が抽象的になりすぎ、使い勝手が落ちる。

## 決定

**3 段階の汎用化方針を採用する：**

### Phase 3'-A：絶対パスの汎用化（即時実施）

- ローカル portfolio / workspace への実パス → `${USER_PORTFOLIO_ROOT}` プレースホルダ
- `${USER_PORTFOLIO_ROOT}` の解決元：`${CLAUDE_PLUGIN_ROOT}/config/personal-config.local.yaml` の `paths.portfolio_root`
- **完了済み**：全 8 SKILL.md + 1 agent を sed 一括置換

### Phase 3'-B：人名・副業名等の設定ファイル化（パイロット → 段階展開）

- `personal-config.example.yaml`（公開テンプレ）+ `personal-config.local.yaml`（gitignore、実値）に集約
- skill 起動時に Claude が `local.yaml` を Read して `{{ person.name }}` 等を展開
- **初期実装**：`offer-evaluator/SKILL.md` の辞退連絡署名から開始
- **公開前展開**：学歴ギャップ・副業時間・海外経験・通勤可能エリア・希望年収などの個人に近い具体値を `personal-config.example.yaml` の placeholder へ集約

### Phase 3'-C：例として残す箇所への「例：」明記（公開直前に対応）

- 企業名（例：職歴名、例：取引先名等）、数値（例：資格スコア等）、エージェント名（例：エージェント種別等）は「例：」と明記して残す
- 完全削除より「具体例として再利用可能」な状態を保つ
- **本 ADR では実装しない**、[ADR 0006](./0006-publication-readiness-checklist.md) の公開前チェックリストで対応

## 帰結

**ポジティブ**:

- 即効性のある汎用化（パス）と将来作業（人名等）を分離し、現段階で過剰投資しない
- パイロット skill で運用知見を蓄積してから他 skill に展開できる
- 公開戦略（ADR 0006）と整合（即公開ではなく段階公開）

**ネガティブ**:

- `personal-config.example.yaml` のキーが増えるため、利用者は初期設定時に自分の実値を埋める必要がある
- 新 skill 追加時に、個人に近い具体値を本文へ直書きしないレビューが必要

## 代替案

- **一気に全検出箇所を汎用化**：作業量過大、抽象化されすぎて読みにくくなる
- **公開しない前提で何もしない**：将来の翻意で再着手する場合のコストが大きい
- **テンプレートエンジン（Mustache 等）を導入**：オーバーエンジニアリング、Claude が Read で解決できる範囲を超える

## Phase 3'-B 全展開時の todo

- 全 SKILL.md / references の人名「個人」を `{{ person.name }}` にするか、一般名詞として残すかを見直す
- 新規具体値は `career.*`, `education.*`, `side_projects.*`, `qualifications.*`, `global_experience.*`, `mobility.*`, `compensation.*` のいずれかへ追加する
- skill 起動時の config Read を全 skill 共通フロー化

参照：[ADR 0005](./0005-aikata-as-scaffold-tool.md)、[ADR 0006](./0006-publication-readiness-checklist.md)。
