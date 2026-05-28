---
name: feedback-integrator
description: 個人の転職活動で、面接後に感じた課題・弱み・違和感、あるいは複数面接を跨いだ気づきを受け取り、それを 5 カテゴリに分類して影響を受けるコア資産（`24_面接想定問答_逆質問.md` / `23_応募先別カスタマイズ指針_*.md` / `21_自己PR_アピール材料.md` / `34_志望動機サンプル集.md` / `40_素材/*.md` / `47_行動原理・性格分析.md` / `50_職務経歴書関連/generator/draft_*.yaml`）を特定し、必要に応じて他 skill（`cv-builder` / `interview-qa-generator` / `deep-self-analysis` / `cv-diff` 等）に委譲しつつコア資産そのもののリファクタを進めるスキル。「面接で X が刺さらなかった」「最近 Y を突かれる」「自己 PR を見直したい」「24 の B-4 を書き直したい」「フィードバック反映」「コア資産のリファクタ」「同じ弱みを 3 社で突かれた」などのリクエストでトリガー。`interview-retrospective` の単一ログ振り返りより一段上の、複数ログ横断・コア資産横断のリファクタ層を担う。portfolio リポジトリ（${USER_PORTFOLIO_ROOT}/）配下の専用パイプラインのみ対象。深掘り対話は `deep-self-analysis` へ、職務経歴書再生成は `cv-builder` へ、差分点検は `cv-diff` へ委譲。
---

# Feedback Integrator

FB（面接後の課題感・複数ログ横断の気づき）を受けて、コア資産のリファクタを統制するオーケストレータ skill。

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
SELF_INTRO       = ${PORTFOLIO_ROOT}/20_面接対策/25_エージェント面談用_自己紹介.md
MOTIVATION_BANK  = ${PORTFOLIO_ROOT}/30_応募活動/34_志望動機サンプル集.md
CHECKLIST        = ${PORTFOLIO_ROOT}/30_応募活動/32_企業評価チェックリスト.md
SOZAI_DIR        = ${PORTFOLIO_ROOT}/40_素材
BEHAVIOR_DOC     = ${PORTFOLIO_ROOT}/40_素材/47_行動原理・性格分析.md
SELF_ANALYSIS    = ${PORTFOLIO_ROOT}/40_素材/46_自己分析.md
YAML_PO          = ${PORTFOLIO_ROOT}/50_職務経歴書関連/generator/draft_PO軸.yaml
YAML_ENG         = ${PORTFOLIO_ROOT}/50_職務経歴書関連/generator/draft_実装エンジニア軸.yaml
SOURCE_MD_DIR    = ${PORTFOLIO_ROOT}/50_職務経歴書関連/source_md
LOGS_GLOB        = ${COMPANIES_DIR}/*/面接ログ/*.md
```

更新対象：上記のいずれか（FB 分類で決定）

## 起動時フロー

1. **入力の確認**：
   - ユーザー発話の FB テキスト（「○○が刺さらなかった」等）
   - `--from-logs` 引数 → `${LOGS_GLOB}` を Grep して直近の「うまく答えられなかった点」「レッドフラグ」を集計
   - `--company {企業名}` 引数 → 特定企業のログのみ参照
2. **入力の不足確認**：FB が曖昧（例：「うまくいかなかった」のみ）なら、`deep-self-analysis` 型の 1 問深掘りで具体化（質問 1 つに絞る）

## 基本ワークフロー

### Step 1：FB の分類

[references/classification.md](references/classification.md) の 5 カテゴリで FB を分類：

| カテゴリ | 例 |
|---------|---|
| C1 回答内容（個別 Q の答え方） | 「B-4 生成 AI の語りが浅い」「弱みの答えが弱すぎ」 |
| C2 経歴の語り方（時系列・転換理由） | 「学歴ギャップの説明が苦しい」「{{ career.transition_topics[0] }} の理由が曖昧」 |
| C3 強み・弱み・自己 PR | 「副業の説明がブレる」「強み 3 つの並びがバラバラ」 |
| C4 志望動機・カスタマイズ指針 | 「PO 軸の志望動機が刺さらない」「業界別の押しが弱い」 |
| C5 自己理解の浅さ | 「同じ質問を 3 社で答えられない」「動機を言語化できない」 |

FB が複数カテゴリに跨る場合は主カテゴリ + 副カテゴリで記録。

### Step 2：影響マトリクスの提示

[references/impact-matrix.md](references/impact-matrix.md) を参照して、分類に応じた **影響ファイル一覧** をユーザーに提示：

```
FB 分類：C1（回答内容、B-4 生成 AI）

影響ファイル：
1. ${MASTER_QA} の B-4 章（直接編集）
2. ${SOZAI_DIR}/41_職歴_（例：現職）.md（生成 AI 関連エピソードの補強）
3. ${YAML_*} の experience[現職].achievements（数値・成果の整合）
4. ${SELF_PR} の AI 実装関連項目（強調軸の同期）
5. 各社 ${COMPANIES_DIR}/*/想定問答_カスタム.md の Q6（汎用変更が波及する範囲）

承認するもの番号で（複数可、`all` で全部、`none` で全部不採用、`details N` で N 番の詳細）：
```

ユーザー承認後、影響ファイルごとに編集アクションを決定。

### Step 3：編集計画と委譲

[references/refactor-patterns.md](references/refactor-patterns.md) のパターンに従って、影響ファイルごとに編集アクションを決定：

| アクション | 対応 |
|----------|------|
| **直接編集**（軽微な追記・章順序入替） | 本 skill が Edit |
| **`cv-builder` 経由**（YAML 編集 → DOCX 再生成が必要） | `cv-builder` SKILL.md を Read し、YAML 編集 + ビルドを案内 |
| **`interview-qa-generator` 経由**（汎用変更を当該企業の `想定問答_カスタム.md` に波及） | 該当企業フォルダを列挙して個別起動を提案 |
| **`deep-self-analysis` 経由**（C5 自己理解の浅さ） | `${BEHAVIOR_DOC}` の「追加で深掘り」に追加 → 起動推奨 |
| **`cv-diff` 経由**（編集後の整合点検） | 全編集完了後に起動を案内 |

### Step 4：実行（段階的、承認を挟む）

1. 影響ファイル 1 件ずつ「これから {ファイル} の {セクション} を {変更内容} で編集します。よいですか？」と承認を取る
2. **複数ファイルの一括編集を勝手にやらない**（暴走防止）
3. C5（自己理解）が分類された場合は **編集より先に `deep-self-analysis` 起動を強く推奨**。深掘り知見が得られてから C1〜C4 のリファクタを進める順序が望ましい

### Step 5：整合点検 + 後続誘導

1. 編集完了したら `cv-diff` で素材 ↔ YAML の drift チェックを案内
2. YAML を変更していたら `cv-builder --build all` で DOCX 再生成
3. 当該企業群の `想定問答_カスタム.md` 再生成が必要なら `interview-qa-generator <企業名>` のリストを提案
4. リファクタ要旨を `${BEHAVIOR_DOC}` の「対話で確認」章 or 新セクション「リファクタログ」に記録（変更履歴の永続化）

## 操作カタログ

| 操作 | トリガー語 | 動作 |
|------|----------|------|
| FB 受け取り → 全工程 | 「面接で X が刺さらなかった、見直したい」 | Step 1〜5 |
| 複数ログ集計から自動検出 | 「最近よく突かれる弱み洗い出して」 | `--from-logs` モード、Step 1〜5 |
| 特定企業のログのみ参照 | 「○○社のログから FB 統合」 | `--company {名}`、Step 1〜5 |
| 分類だけ確認 | 「この FB はどのカテゴリ？」 | Step 1 のみ |
| 影響マトリクス確認のみ | 「B-4 を直したら何に波及する？」 | Step 1〜2 のみ |
| リファクタログ追記のみ | 「先週のリファクタ記録残して」 | Step 5 の永続化のみ |

詳細：
- 5 カテゴリ分類規則 → [references/classification.md](references/classification.md)
- 分類 → 影響ファイル対応 → [references/impact-matrix.md](references/impact-matrix.md)
- リファクタの典型パターン → [references/refactor-patterns.md](references/refactor-patterns.md)

## ファイル編集ルール

| 対象 | 操作 |
|------|------|
| `${MASTER_QA}` の章本文 | 直接編集（章の再構成・写像見直し・優先度入替） |
| `${CUSTOM_PO}` / `${CUSTOM_ENG}` の業界別・職種別セクション | 直接編集 |
| `${SELF_PR}`, `${SELF_INTRO}`, `${MOTIVATION_BANK}` | 直接編集 |
| `${SOZAI_DIR}/4X_職歴_*.md` | 直接編集（事実情報は両軸 YAML との整合保持） |
| `${BEHAVIOR_DOC}` の「追加で深掘り」「対話で確認」「リファクタログ」 | 追記主体（深層仮説の改訂は `deep-self-analysis` 経由を強く推奨） |
| `${YAML_*}` | 直接編集はせず、必ず `cv-builder` 経由（DOCX 再生成までセット） |
| `${SOURCE_MD_DIR}/*` | 編集後 `cv-diff` で YAML との整合点検 |
| `${COMPANIES_DIR}/*/想定問答_カスタム.md` | 直接編集はせず、`interview-qa-generator` 経由（応募軸トーンを再適用するため） |
| `${COMPANIES_DIR}/*/面接ログ/*` | 編集不可（読み込みのみ） |
| `${CHECKLIST}` | 編集不可（評価軸の改訂は別判断） |

## 引数（スラッシュコマンド呼出時）

`/feedback-integrator` の引数：

| 引数 | 動作 |
|------|------|
| なし | 対話で FB を聞き取り、Step 1〜5 |
| `"<FB テキスト>"` | FB を直接渡して Step 1〜5 |
| `--from-logs [--since N]` | 直近 N 件（既定 5）の面接ログから「答えられなかった点」「レッドフラグ」を集計して FB 化 |
| `--company <企業名>` | 特定企業のログのみ参照 |
| `--classify-only` | Step 1 のみ |
| `--matrix-only` | Step 1〜2 のみ（編集しない） |
| `--dry-run` | 全工程シミュレートして編集計画だけ提示 |
| `--no-delegate` | 他 skill 委譲を提案せず、本 skill で完結可能な範囲のみ編集 |

## 責務分離（やらないこと）

| やらないこと | 委譲先 |
|------------|--------|
| 単一面接の振り返り・ログ作成 | `interview-retrospective` |
| 自己理解の対話深掘り | `deep-self-analysis`（C5 検出時は強く推奨） |
| 職務経歴書 YAML 編集 + DOCX 再生成 | `cv-builder` |
| 素材 ↔ YAML drift 点検 | `cv-diff` |
| 当該企業の `想定問答_カスタム.md` 再生成 | `interview-qa-generator` |
| 企業フォルダの新規立ち上げ | `company-prep` |
| 内定比較 | `offer-evaluator` |
| `${CHECKLIST}` 自体の改訂 | 範囲外（評価軸変更は別判断） |
| 自動 hook での発火 | しない（FB は人間が言語化する行為） |
| 一括編集の暴走 | しない（ファイル 1 件ずつ承認） |

## 禁止事項 / 終了時チェックリスト

- ❌ ユーザー承認なしに複数ファイルを連続編集しない
- ❌ FB が曖昧（「うまくいかなかった」のみ等）な状態でリファクタを強行しない（→ 深掘り 1 問）
- ❌ C5（自己理解の浅さ）検出時に C1〜C4 のリファクタを先行させない（深掘りが先）
- ❌ YAML を本 skill 直接編集しない（必ず `cv-builder` 経由）
- ❌ `想定問答_カスタム.md` を本 skill 直接編集しない（必ず `interview-qa-generator` 経由）
- ❌ 検証不能な数値・成果を素材 / YAML に新規投入しない（24 D-1 ルール）
- ❌ `${MASTER_QA}` の既存章番号体系（A / B / C / D / E）を本 skill 単独で改訂しない（破壊的変更）
- ✅ 終了時：FB が 5 カテゴリのいずれかに分類されているか
- ✅ 終了時：影響マトリクスがユーザーに提示され、承認結果が記録されているか
- ✅ 終了時：編集したファイル一覧が要旨と共に `${BEHAVIOR_DOC}` の「リファクタログ」（新セクション可）に追記されているか
- ✅ 終了時：YAML 変更があれば `cv-builder --build all` が次アクションに記載されているか
- ✅ 終了時：C5 検出時は `deep-self-analysis` 起動が次アクションに記載されているか
