---
name: interview-qa-generator
description: 個人の転職活動で、企業別フォルダ `20_面接対策/companies/{企業名}/想定問答_カスタム.md` を、汎用 Q&A マスタ（`24_面接想定問答_逆質問.md`）+ 職務経歴書 YAML（`50_職務経歴書関連/generator/draft_*.yaml`）+ 職歴素材（`40_素材/`）+ 行動原理（`47_行動原理・性格分析.md`）+ 企業情報を突き合わせて「企業特化版」に更新するスキル。応募軸（PO / 実装エンジニア）に応じてカスタムの軸・強調 / トーンダウン、当社特化質問（Q20 〜）、要カスタムマーカー、自己分析誘導フラグまでを生成する。「○○社の想定問答カスタムして」「想定問答生成」「interview-qa-generator ○○」「面接想定問答」などのリクエストでトリガー。portfolio リポジトリ（${USER_PORTFOLIO_ROOT}/）配下の専用パイプラインのみ対象。逆質問は `reverse-questions-curator` へ、自己分析深掘りは `deep-self-analysis` へ、職務経歴書本体更新は `cv-builder` へ委譲。
---

# Interview QA Generator

汎用 Q&A マスタ + 経歴正本 + 素材 + 企業情報 を統合して、企業別 `想定問答_カスタム.md` を更新する中核スキル。

## 設定ファイル

> `${USER_PORTFOLIO_ROOT}` は `${CLAUDE_PLUGIN_ROOT}/config/personal-config.local.yaml` の `paths.portfolio_root` から取得。
> パイロット skill (offer-evaluator) で先行実装、他 skill は将来展開（ADR-0004 参照）。

## 構成と場所（絶対パス固定）

```
PORTFOLIO_ROOT   = ${USER_PORTFOLIO_ROOT}
COMPANIES_DIR    = ${PORTFOLIO_ROOT}/20_面接対策/companies
MASTER_QA        = ${PORTFOLIO_ROOT}/20_面接対策/24_面接想定問答_逆質問.md
CUSTOM_PO        = ${PORTFOLIO_ROOT}/20_面接対策/23_応募先別カスタマイズ指針_PO軸.md
CUSTOM_ENG       = ${PORTFOLIO_ROOT}/20_面接対策/23_応募先別カスタマイズ指針_実装エンジニア軸.md
SELF_PR          = ${PORTFOLIO_ROOT}/20_面接対策/21_自己PR_アピール材料.md
GENERATOR_DIR    = ${PORTFOLIO_ROOT}/50_職務経歴書関連/generator
YAML_PO          = ${GENERATOR_DIR}/draft_PO軸.yaml
YAML_ENG         = ${GENERATOR_DIR}/draft_実装エンジニア軸.yaml
SOZAI_DIR        = ${PORTFOLIO_ROOT}/40_素材
BEHAVIOR_DOC     = ${PORTFOLIO_ROOT}/40_素材/47_行動原理・性格分析.md
SELF_ANALYSIS    = ${PORTFOLIO_ROOT}/40_素材/46_自己分析.md
MOTIVATION_BANK  = ${PORTFOLIO_ROOT}/30_応募活動/34_志望動機サンプル集.md
```

更新対象：`${COMPANIES_DIR}/{企業名}/想定問答_カスタム.md`

## 起動時フロー

1. **企業フォルダ存在確認**：無ければ `company-prep` 起動を提案して中止
2. **応募軸の確定**：
   - `${COMPANIES_DIR}/{企業名}/README.md` の応募軸欄を読む
   - 未確定なら必ず確認（軸によって強調 / トーンダウンが全く違うため）
3. **企業情報の完備度確認**：
   - `${COMPANIES_DIR}/{企業名}/企業情報.md` の事業内容・技術スタック・カルチャー観察メモが空欄だらけなら、まず `company-prep --info-only` での更新を促す
   - 最低限：事業内容 + 技術スタック + 業界カテゴリが特定できる状態が必要
4. **既存 `想定問答_カスタム.md` を読む**：既にユーザーが手動カスタムした箇所は保持

## 基本ワークフロー

### Step 1：入力の読み込み

並列で以下を取得：
- `${COMPANIES_DIR}/{企業名}/README.md`, `企業情報.md`
- `${MASTER_QA}`（汎用 Q&A マスタ）
- 応募軸に応じて `${CUSTOM_PO}` or `${CUSTOM_ENG}`
- 応募軸に応じて `${YAML_PO}` or `${YAML_ENG}`
- `${SOZAI_DIR}/4X_職歴_*.md`（突かれやすいエピソードの抽出元）
- `${BEHAVIOR_DOC}`（特に「対話で確認」「未回答質問」「追加で深掘りしたい質問」章）
- `${MOTIVATION_BANK}`（志望動機 Q2 のカスタム参照）

### Step 2：抽出

[references/mapping-rules.md](references/mapping-rules.md) と [references/weakness-attack-patterns.md](references/weakness-attack-patterns.md) と [references/axis-tone.md](references/axis-tone.md) を参照しながら、以下を出力する内部リストに整理：

A. **カスタムの軸**
   - 強調ポイント（3〜5 件）：企業の事業フェーズ × 技術スタック × グリーンフラグから導出
   - トーンダウンポイント（1〜3 件）：求められていない領域・避けるべき主張
   - 応募軸 × 業界の組み合わせは axis-tone.md のマトリクスを使う

B. **当社特化質問候補（Q20 枠以降）**：3〜5 件
   - 事業フェーズ・技術スタック・グリーン / レッドフラグから「面接官が必ず聞きたい論点」を推定
   - 各質問にドラフト回答（30〜60 秒で話せる長さ）を 1 つ
   - 質問例：
     - 「弊社の AI プロダクトで、過去のオフショア窓口経験はどう活かせる？」
     - 「シリーズ A の規模感で、副業の比重をどう調整する？」
     - 「自社開発と受託の混在環境で、どの軸足を置くか？」

C. **逆襲想定（弱み深掘り）**：weakness-attack-patterns.md で検出された箇所のリスト
   - 学歴ギャップ（{{ education.gap_period }}）
   - 大学在籍 {{ education.total_years }} 年
   - ネットワーク → フルスタックへの職種転換
   - 副業比率（実装率と週時間）
   - 数値の検証可能性（「例：8 割削減」「例：60% リファクタ」等）
   - 個人事業の商流・販売手法（24 の D 章 NG ルール）

D. **応募軸トーン適用後の優先順序**
   - 自己紹介（30 秒 / 2 分版）の冒頭で何を打ち出すか
   - 経歴質問（B 章）の語り順序
   - 技術質問（C 章）と意思決定質問（A 章）のどちらに比重を置くか

E. **自己分析誘導フラグ**
   - `${BEHAVIOR_DOC}` の「未回答質問」「追加で深掘りしたい質問」と、企業特化質問が交差する論点があれば抽出
   - 抽出された論点については **`deep-self-analysis` 起動を提案**

### Step 3：`想定問答_カスタム.md` の更新

**追記主体ルール**：既存 Q1〜Q19 の本文を全文書き換えしない。以下のみ編集：

| セクション | 編集内容 |
|---------|---------|
| `## カスタムの軸` | A の結果を 3 行で記入 |
| `## 自己紹介（30 秒版 / 2 分版）` | D の優先順序を反映した冒頭 1 文と「重点 3 ポイント」を記入（本文ドラフトは雛形のまま `—` で残す or 軽い骨子のみ） |
| `## キャリア質問` Q2 志望動機 | `${MOTIVATION_BANK}` から該当業界テンプレを引用 + 企業特化フック 1〜2 行 |
| 各 Q1〜Q19 で要カスタム箇所 | 該当 Q の見出し直下に `> [このQ要カスタム：理由 = ...]` マーカーのみ追加 |
| `## 当社特化質問` Q20〜 | B のリストを Q & ドラフト回答付きで記入 |
| **新規セクション** `## 想定される深掘り・逆襲質問` | C のリストを箇条書きで追加（既存セクション末尾 or `## 当社特化質問` の前） |
| **新規セクション** `## 自己分析が必要な論点` | E のリストを箇条書きで追加（あれば。`deep-self-analysis` 起動推奨を明記） |

### Step 4：次アクション提示

- 自己分析誘導フラグがあれば `deep-self-analysis` 起動を提案
- 応募軸に応じて職務経歴書バリアントが未生成なら `cv-builder --variant {企業名}` 起動を提案
- 逆質問が未整備なら `reverse-questions-curator` 起動を提案
- 面接実施後は `interview-retrospective` で振り返り

## 操作カタログ

| 操作 | トリガー語 | 動作 |
|------|----------|------|
| 全工程カスタマイズ | 「○○社の想定問答カスタムして」 | Step 1〜4 全実行 |
| 当社特化質問のみ生成 | 「○○社の特化質問だけ作って」 | Step 2-B + Step 3 の Q20〜のみ |
| 弱み深掘り対策のみ | 「○○社の逆襲質問対策」 | Step 2-C + Step 3 の `## 想定される深掘り・逆襲質問` のみ |
| カスタムの軸のみ確定 | 「○○社のカスタムの軸決めて」 | Step 2-A + Step 3 の `## カスタムの軸` のみ |
| 自己分析論点の抽出 | 「○○社で深掘りすべき自己分析論点」 | Step 2-E + 提案のみ（ファイル編集なし） |

詳細：
- 汎用 Q&A 章 → 企業特化への写像規則 → [references/mapping-rules.md](references/mapping-rules.md)
- 突かれやすい箇所の検出パターン → [references/weakness-attack-patterns.md](references/weakness-attack-patterns.md)
- 応募軸別の強調・トーンダウンマトリクス → [references/axis-tone.md](references/axis-tone.md)

## ファイル編集ルール

| 対象 | 操作 |
|------|------|
| `${COMPANIES_DIR}/{企業名}/想定問答_カスタム.md` | 追記 + マーカー挿入 + 当社特化質問 / 深掘り / 自己分析論点セクションの新規追加 |
| 既存 Q1〜Q19 の本文 | 編集不可（マーカー追加のみ） |
| `${MASTER_QA}` | 編集不可（マスタは別経路で更新） |
| `${CUSTOM_*}`, `${YAML_*}`, `${SOZAI_DIR}/*`, `${BEHAVIOR_DOC}`, `${MOTIVATION_BANK}` | 編集不可（読み込みのみ） |

## 引数（スラッシュコマンド呼出時）

`/interview-qa-generator` の引数：

| 引数 | 動作 |
|------|------|
| `<企業名>` | 全工程実行（応募軸は README から取得 or 確認） |
| `<企業名> --axis po\|engineer` | 応募軸を明示 |
| `<企業名> --section custom-axis\|special-q\|deep-attack\|self-analysis` | 特定セクションのみ更新 |
| `<企業名> --self-analysis-only` | 自己分析論点の抽出のみ（ファイル編集なし） |
| `<企業名> --no-edit` | ドラフトを表示のみ、ファイル編集なし |

## 責務分離（やらないこと）

| やらないこと | 委譲先 |
|------------|--------|
| 逆質問のキュレーション | `reverse-questions-curator` |
| 自己分析論点の対話深掘り | `deep-self-analysis`（フラグ立て + 起動提案のみ） |
| 職務経歴書 YAML / DOCX の更新 | `cv-builder` |
| `${MASTER_QA}` の更新（新 Q 追加 / 全社共通の改訂） | 手動 / `interview-retrospective` 経由 |
| 企業情報の初期収集 | `company-prep` |
| 面接後の振り返り・新出質問の取り込み | `interview-retrospective` |
| 検証不能な成果数値の創作 | **絶対禁止**（24 の D 章ルール準拠） |

## 禁止事項 / 終了時チェックリスト

- ❌ 既存 Q1〜Q19 の本文を全文書き換えしない（マーカーのみ）
- ❌ 検証不能な数値（合格率・削減率・売上影響等）をドラフト回答に入れない
- ❌ 個人事業の商流・販売手法を自分から詳述する回答を作らない（24 D-2 ルール）
- ❌ 家族 / 生活事情を年収根拠に置く回答を作らない（24 D-3 ルール）
- ❌ 応募軸が未確定のまま当社特化質問を生成しない
- ❌ 企業情報が空欄だらけの状態で生成を強行しない（→ `company-prep` へ戻す）
- ✅ 終了時：`## カスタムの軸` に強調 / トーンダウンが各 3 件以上記入されているか
- ✅ 終了時：`## 当社特化質問` に Q20〜Q24 相当が 3〜5 件追加されているか
- ✅ 終了時：`## 想定される深掘り・逆襲質問` セクションが新規追加されているか
- ✅ 終了時：自己分析誘導フラグがあれば `## 自己分析が必要な論点` が追加され、`deep-self-analysis` 起動が次アクションに記載されているか
- ✅ 終了時：要カスタムマーカー（`> [このQ要カスタム：...]`）が該当 Q に挿入されているか
