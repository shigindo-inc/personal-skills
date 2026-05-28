---
name: interview-retrospective
description: 個人の転職活動で、面接実施後に `20_面接対策/companies/{企業名}/面接ログ/YYYY-MM-DD_{ラウンド}.md` を `_TEMPLATE.md` から作成し、対話で「聞かれた質問・うまく答えられた点・答えられなかった点・印象（グリーン / レッドフラグ）・逆質問結果・総合手応え」を引き出して埋め、さらに抽出内容を `24_面接想定問答_逆質問.md`（新出質問の汎用化）、`想定問答_カスタム.md`（補強）、`企業情報.md`（カルチャー観察メモ）、`README.md`（フェーズ進行・評価スコア再計算）、`36_応募ダッシュボード.md`（進捗更新）へフィードバックするスキル。面接を学習ループ化する。「○○社の面接振り返り」「面接ログ書く」「interview-retrospective ○○ 一次」「面接後対応」などのリクエストでトリガー。portfolio リポジトリ（${USER_PORTFOLIO_ROOT}/）配下の専用パイプラインのみ対象。自己分析深掘りは `deep-self-analysis` へ、想定問答の本格再生成は `interview-qa-generator` へ委譲。
---

# Interview Retrospective

面接後の振り返り対話 → ログ作成 → 関連ファイル群への学習フィードバックを一気通貫で行うスキル。

## 設定ファイル

> `${USER_PORTFOLIO_ROOT}` は `${CLAUDE_PLUGIN_ROOT}/config/personal-config.local.yaml` の `paths.portfolio_root` から取得。
> パイロット skill (offer-evaluator) で先行実装、他 skill は将来展開（ADR-0004 参照）。

## 構成と場所（絶対パス固定）

```
PORTFOLIO_ROOT   = ${USER_PORTFOLIO_ROOT}
COMPANIES_DIR    = ${PORTFOLIO_ROOT}/20_面接対策/companies
LOG_TEMPLATE     = ${COMPANIES_DIR}/_TEMPLATE/面接ログ/_TEMPLATE.md
MASTER_QA        = ${PORTFOLIO_ROOT}/20_面接対策/24_面接想定問答_逆質問.md
CHECKLIST        = ${PORTFOLIO_ROOT}/30_応募活動/32_企業評価チェックリスト.md
DASHBOARD        = ${PORTFOLIO_ROOT}/30_応募活動/36_応募ダッシュボード.md
BEHAVIOR_DOC     = ${PORTFOLIO_ROOT}/40_素材/47_行動原理・性格分析.md
```

生成先：`${COMPANIES_DIR}/{企業名}/面接ログ/YYYY-MM-DD_{ラウンド}.md`

## 起動時フロー

1. **企業フォルダ存在確認**：無ければ `company-prep` 起動を提案して中止
2. **面接日 + ラウンドの確定**：
   - `--date YYYY-MM-DD --round 一次|二次|最終|カジュアル` 引数 or 対話で確認
   - 同名ログがすでに存在 → 上書き / 別名 / 追記 を確認
3. **対話モード or バッチモード判定**：
   - ユーザーがメモを貼ってくれば「バッチモード」（一括解析）
   - 「これから振り返り始める」なら「対話モード」（1 問ずつ深掘り、[references/extraction-prompts.md](references/extraction-prompts.md) 参照）

## 基本ワークフロー

### Step 1：ログファイル作成

1. `${LOG_TEMPLATE}` を `${COMPANIES_DIR}/{企業名}/面接ログ/YYYY-MM-DD_{ラウンド}.md` にコピー
2. `## 面接情報` の表（企業名・日時・ラウンド・形式・面接官・時間）をユーザー確認後に埋める

### Step 2：対話 or バッチでログを埋める

[references/extraction-prompts.md](references/extraction-prompts.md) の質問テンプレを使い、以下を順に埋める：

1. **聞かれた主な質問**（番号付きリスト、5〜15 件目安）
2. **うまく答えられたこと / 準備が活きた点**
3. **うまく答えられなかった / 準備不足だった点**
4. **企業への印象**：
   - グリーンフラグ（`${CHECKLIST}` の項目を提示して該当を選ばせる）
   - レッドフラグ（同上）
5. **逆質問の結果**：聞いた質問・回答要旨・評価メモを表に
6. **総合手応え**（◎/○/△/✕）と根拠 1〜2 文
7. **次アクション**

対話モードの原則：
- 1 問ずつ聞く（複数質問を一度に投げない）
- 「面接官が言った具体的な言葉」を引き出す（要約だけでは情報損失）
- 多肢選択を活用（グリーン / レッドフラグは特に）
- 沈黙を許容（ユーザーが思い出すまで待つ）

### Step 3：フィードバック振り分け（[references/feedback-routing.md](references/feedback-routing.md)）

抽出した情報を以下に振り分け、**ユーザーに承認を取ってから** 各ファイルを更新：

| 抽出内容 | 振り分け先 | 編集内容 |
|---------|---------|---------|
| 新出質問（マスタ 24 にない） | `${MASTER_QA}` | 該当章（A / B / C / E）への追加候補を提示、承認後に追記 |
| 答えられなかった質問 | `${COMPANIES_DIR}/{企業名}/想定問答_カスタム.md` | 該当 Q への補強 / 要カスタムマーカー追加 |
| 深掘りすべき内面論点 | `${BEHAVIOR_DOC}` | 「追加で深掘りしたい質問」章への追加候補 → 承認後に `deep-self-analysis` 起動を提案 |
| グリーン / レッドフラグ | `${COMPANIES_DIR}/{企業名}/企業情報.md` | `## カルチャー観察メモ` への追記 |
| 評価スコア再計算 | `${COMPANIES_DIR}/{企業名}/README.md` | `## 評価スコア` を `${CHECKLIST}` に従って再算出 |
| 逆質問への回答ログ | `${COMPANIES_DIR}/{企業名}/逆質問.md` | `## メモ` への追記 |

### Step 4：フェーズ進行 + ダッシュボード更新

1. `${COMPANIES_DIR}/{企業名}/README.md` の現フェーズを次段階へ進行：
   - `一次面接予定` → `一次面接済`
   - `一次面接済` → 結果次第で `二次面接予定` or `不合格`
   - `最終面接済` → `内定` or `不合格`
2. `## 最終更新` を当日 YYYY/MM/DD に
3. `${DASHBOARD}` セクション 1 の該当行を更新：現フェーズ・次アクション・期日・評価スコア
4. ログファイル末尾の `## ログから派生して更新したファイル` チェックリストにチェックを入れる

### Step 5：次アクション提示

- 答えられなかった質問が多ければ → `interview-qa-generator <企業名> --section deep-attack` で補強
- 自己分析論点が抽出されたら → `deep-self-analysis` 起動
- 次ラウンドが決まったら → `reverse-questions-curator <企業名> --round {次ラウンド}` で次の逆質問準備
- 最終面接後で内定が見えてきたら → 他企業との比較タイミング → `offer-evaluator`（次点 skill）

## 操作カタログ

| 操作 | トリガー語 | 動作 |
|------|----------|------|
| 対話で 1 から振り返り | 「○○社の面接振り返り」 | Step 1〜5 全工程、対話モード |
| メモ貼付からの一括解析 | 「○○社の面接ログ：以下のメモから」 | バッチモード、Step 1〜5 |
| 既存ログへの追記 | 「○○社の {YYYY-MM-DD} ログに追記」 | Step 2 の続き、Step 3〜5 はオプション |
| フィードバックのみ実行 | 「○○社のログから 24 / 32 にフィードバック」 | Step 3〜4 のみ（ログは既存前提） |

詳細：
- 振り返り対話の質問テンプレ → [references/extraction-prompts.md](references/extraction-prompts.md)
- 抽出内容 → 各ファイルへの振り分け規則 → [references/feedback-routing.md](references/feedback-routing.md)

## ファイル編集ルール

| 対象 | 操作 |
|------|------|
| `${COMPANIES_DIR}/{企業名}/面接ログ/{日付}_{ラウンド}.md` | 新規作成 + 追記 |
| `${COMPANIES_DIR}/{企業名}/企業情報.md`（カルチャー観察メモ章） | 追記 |
| `${COMPANIES_DIR}/{企業名}/README.md`（現フェーズ・評価スコア・最終更新） | 該当箇所のみ更新 |
| `${COMPANIES_DIR}/{企業名}/想定問答_カスタム.md` | 要カスタムマーカー追加、回答補強 |
| `${COMPANIES_DIR}/{企業名}/逆質問.md`（`## メモ` 章） | 追記 |
| `${MASTER_QA}` | **ユーザー承認後のみ** 追記。深掘り想定パターンの追加 |
| `${BEHAVIOR_DOC}`（「追加で深掘りしたい質問」章） | **ユーザー承認後のみ** 追記 |
| `${DASHBOARD}` セクション 1 | 該当行のみ更新 |
| `${LOG_TEMPLATE}`, `${CHECKLIST}` | 編集不可 |

## 引数（スラッシュコマンド呼出時）

`/interview-retrospective` の引数：

| 引数 | 動作 |
|------|------|
| `<企業名>` | 対話モードで全工程実行 |
| `<企業名> --date YYYY-MM-DD --round 一次\|二次\|最終\|カジュアル` | 日付・ラウンドを明示 |
| `<企業名> --batch` | バッチモード（ユーザー貼付メモから一括解析） |
| `<企業名> --feedback-only --log <ファイル名>` | 既存ログからフィードバックのみ実行 |
| `<企業名> --no-master-update` | `${MASTER_QA}` への追記を提案しない |

## 責務分離（やらないこと）

| やらないこと | 委譲先 |
|------------|--------|
| 自己分析論点の対話深掘り | `deep-self-analysis`（フラグ立て + 起動提案のみ） |
| 想定問答の本格再生成（応募軸トーン全体再適用など） | `interview-qa-generator` |
| 逆質問の次ラウンド向け再キュレーション | `reverse-questions-curator` |
| 企業情報の追加初期収集 | `company-prep --info-only` |
| 内定確定後の複数オファー比較 | `offer-evaluator` |
| git commit / push | `commit-helper` |
| 自動 hook での発火 | **しない**（明示要求 or `interview-prep` agent からの誘発のみ） |

## 禁止事項 / 終了時チェックリスト

- ❌ `${MASTER_QA}` / `${BEHAVIOR_DOC}` への追記をユーザー承認なしで実行しない
- ❌ 面接官の発言を「要約」だけにせず、固有名詞 / 数値 / フレーズは原文記録（記憶の鮮明なうちに）
- ❌ 手応え（総合判断）を勝手に算出しない（ユーザーの主観を聞く）
- ❌ レッドフラグ検出だけで「応募取り下げ」を勧めない（評価は総合）
- ❌ 自動 hook で起動しない（毎面接後の判断は人間が下す）
- ✅ 終了時：ログファイルの `## 面接情報` 表が全項目埋まっているか
- ✅ 終了時：聞かれた質問 5 件以上、グリーン / レッドフラグそれぞれ 1 件以上記入されているか
- ✅ 終了時：`${DASHBOARD}` の該当行が更新されているか
- ✅ 終了時：`README.md` 現フェーズが進行しているか
- ✅ 終了時：ログ末尾の派生チェックリストに該当チェックが入っているか
- ✅ 終了時：自己分析論点が抽出された場合、`deep-self-analysis` 起動が次アクションに記載されているか
