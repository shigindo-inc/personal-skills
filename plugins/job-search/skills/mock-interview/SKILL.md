---
name: mock-interview
description: 個人の転職活動で、AI が面接官役を演じて模擬面接を実施するスキル。5 つの面接官ペルソナ（人事 / 現場エンジニア / テックインタビュアー / マネージャ / 役員）から選択し、4 つのセッションモード（full / pinpoint / weakness / cold）で進行。1 問ずつ提示 → 回答 → 必要に応じて深掘り（学歴ギャップ / 副業比率 / 数値根拠は必ず突く）→ 終了後に 5 軸評価ルーブリック（説得力 / 具体性 / 深掘り耐性 / 応募軸整合 / マナー）で評価し、`{企業}/面接ログ/MOCK_YYYY-MM-DD_{ラウンド}_{ペルソナ}.md` に保存（本番ログと区別する MOCK_ プレフィックス）。改善点が明確なら `feedback-integrator` / `interview-qa-generator` / `deep-self-analysis` 起動を提案。「模擬面接」「mock-interview」「面接練習」「○○社の面接シミュレート」「技術面接の練習」「面接ロールプレイ」などのリクエストでトリガー。portfolio リポジトリ（${USER_PORTFOLIO_ROOT}/）配下の専用パイプラインのみ対象。本番面接の振り返りは `interview-retrospective` へ、コア資産リファクタは `feedback-integrator` へ委譲。
---

# Mock Interview

AI が面接官役を演じる模擬面接スキル。ペルソナ × モードで強度を調整、終了後に評価 + 改善誘導まで一気通貫。

## 設定ファイル

> `${USER_PORTFOLIO_ROOT}` は `${CLAUDE_PLUGIN_ROOT}/config/personal-config.local.yaml` の `paths.portfolio_root` から取得。
> 学歴ギャップ・副業時間・希望条件などの個人に近い値は `education.*`, `side_projects.*`, `mobility.*`, `compensation.*` から取得し、公開用 SKILL.md には実値を書かない。

## 構成と場所（絶対パス固定）

```
PORTFOLIO_ROOT   = ${USER_PORTFOLIO_ROOT}
COMPANIES_DIR    = ${PORTFOLIO_ROOT}/20_面接対策/companies
MASTER_QA        = ${PORTFOLIO_ROOT}/20_面接対策/24_面接想定問答_逆質問.md
SELF_PR          = ${PORTFOLIO_ROOT}/20_面接対策/21_自己PR_アピール材料.md
CUSTOM_PO        = ${PORTFOLIO_ROOT}/20_面接対策/23_応募先別カスタマイズ指針_PO軸.md
CUSTOM_ENG       = ${PORTFOLIO_ROOT}/20_面接対策/23_応募先別カスタマイズ指針_実装エンジニア軸.md
YAML_PO          = ${PORTFOLIO_ROOT}/50_職務経歴書関連/generator/draft_PO軸.yaml
YAML_ENG         = ${PORTFOLIO_ROOT}/50_職務経歴書関連/generator/draft_実装エンジニア軸.yaml
BEHAVIOR_DOC     = ${PORTFOLIO_ROOT}/40_素材/47_行動原理・性格分析.md
SOZAI_DIR        = ${PORTFOLIO_ROOT}/40_素材
```

保存先：`${COMPANIES_DIR}/{企業名}/面接ログ/MOCK_YYYY-MM-DD_{ラウンド}_{ペルソナ}.md`
- `--company` 未指定時は `${COMPANIES_DIR}/_MOCK/面接ログ/MOCK_YYYY-MM-DD_{ペルソナ}.md`（_MOCK フォルダが無ければ作成）

## 起動時フロー

1. **パラメータ確認**：
   - `--company`：対象企業（任意。指定時は企業特化質問プールが充実）
   - `--axis po|engineer`：応募軸（必須、README から取得 or 確認）
   - `--round カジュアル|一次|二次|最終`：ラウンド（既定：カジュアル）
   - `--persona 人事|現場|テック|マネージャ|役員`：ペルソナ（既定：ラウンドから推定）
   - `--mode full|pinpoint|weakness|cold`：モード（既定：full）
   - `--questions N`：質問数（既定：full=8 / pinpoint=3 / weakness=5 / cold=6）
   - `--strict`：詰まってもヒント無し（本番強度）
2. **質問プール構築**：[references/session-modes.md](references/session-modes.md) のモード別ロジックで構築
3. **面接官紹介**：ペルソナの肩書きと面接時間目安を提示してから開始

## 基本ワークフロー

### Step 1：オープニング

```
それでは、{ペルソナ} 役の面接を始めます。
時間目安：約 {N} 分（{N} 問）
モード：{mode}（{strict 時の補足}）

途中で「タイム」と言えば一時停止、「ヒント」と言えば軽い助言を入れます（--strict 時を除く）。
「終了」と言えばその時点で評価フェーズに入ります。

では、まず自己紹介をお願いします。{ペルソナの追加注文}
```

### Step 2：質問進行（1 問ずつ）

各ラウンドで以下を実施：

1. **質問提示**：質問プールから選択 + ペルソナのトーンに合わせて文面調整（[references/interviewer-personas.md](references/interviewer-personas.md)）
2. **ユーザー回答待ち**：複数文の回答を 1 メッセージで受け取る
3. **深掘り判定**：
   - 回答に学歴ギャップ / 副業比率 / 数値（X 割削減 / N% 等）/ 職種転換 が含まれる → [references/attack-rules.md](references/attack-rules.md) に従って **必ず深掘り**
   - 回答が抽象的・具体例不足 → 「具体的には？」「直近の例を 1 つ」
   - 回答が応募軸とズレる → ペルソナがマネージャ / 役員なら必ず突く
4. **詰まり対応**：
   - 30 秒以上の沈黙 / 「分からない」回答 → `--strict` 無しなら 1 ヒント、ありなら次へ
5. **次の質問へ**：質問プールが尽きるか、`--questions` 数に達するか、ユーザーが「終了」で評価へ

### Step 3：評価フェーズ

[references/evaluation-rubric.md](references/evaluation-rubric.md) の 5 軸 × ◎○△✕ で評価：

| 軸 | 観点 |
|---|---|
| 説得力 | 主張に裏付け（具体例・数値・経験）があるか |
| 具体性 | 抽象論で逃げず、固有名詞・期間・規模を出せたか |
| 深掘り耐性 | 突っ込まれた時に崩れず、追加情報を出せたか |
| 応募軸整合 | 強調軸が応募軸（PO / 実装）と一致しているか、トーンダウンすべき箇所を回避できたか |
| マナー | 24 D 章の NG ルール（数値創作・個人事業詳述・家族事情）を踏んでいないか |

質問ごとに：
- ✅ 良かった点（1〜2 行）
- 🔧 改善余地（1〜2 行）
- 📌 参考：マスタ 24 / 素材のどこを再確認すべきか

総合：
- 5 軸平均 + 一文評価
- 「本番なら通過しそうか」の主観（あくまで模擬の参考）

### Step 4：結果保存

`${COMPANIES_DIR}/{企業名 or _MOCK}/面接ログ/MOCK_YYYY-MM-DD_{ラウンド}_{ペルソナ}.md` を作成：

```markdown
# 模擬面接ログ：{企業名 or 汎用}（YYYY-MM-DD / {ラウンド} / {ペルソナ}）

> 模擬面接の記録（本番ログと区別するため MOCK_ プレフィックス）。

## セッション情報

| 項目 | 内容 |
|------|------|
| 企業 | {名 or 汎用} |
| 応募軸 | {PO / 実装} |
| ラウンド | {} |
| 面接官ペルソナ | {} |
| モード | {} |
| 質問数 | {} |
| Strict | {Yes/No} |

## 質疑応答ログ

### Q1. {質問本文}

回答：
> {ユーザー回答}

深掘り（あれば）：
> Q: {追加質問}
> A: {ユーザー回答}

評価：
- ✅ {良かった点}
- 🔧 {改善余地}
- 📌 参照：{24 章 / 素材}

（Q2〜以下同様）

## 総合評価

| 軸 | 評価 | コメント |
|---|---|---|
| 説得力 | ○ | ... |
| 具体性 | △ | ... |
| 深掘り耐性 | ○ | ... |
| 応募軸整合 | ◎ | ... |
| マナー | ◎ | ... |

**総合**：{一文評価} / 本番想定：{通過しそう / 五分 / 厳しい}

## 改善優先度（上位 3 件）

1. {論点}（参照：24 X-Y / 素材 4Z）
2. ...

## 次アクション

- [ ] `interview-qa-generator <企業名> --section deep-attack`（個別 Q 補強）
- [ ] `feedback-integrator`（複数模擬で同パターン検出時、コア資産リファクタ）
- [ ] `deep-self-analysis`（自己理解の浅さ検出時）
```

### Step 5：改善誘導

評価結果に応じて：
- **個別 Q 改善で済む** → `interview-qa-generator <企業名> --section deep-attack` を提案
- **複数の弱点が見えた / 過去 MOCK ログと同パターン** → `feedback-integrator` を提案
- **自己理解の浅さが出た**（C5 兆候、24 A-6 / B-9 で詰まる等） → `deep-self-analysis` を提案
- **企業情報が足りなくて当社特化質問が薄かった** → `company-prep --info-only <企業名>` を提案

## 操作カタログ

| 操作 | トリガー語 | 動作 |
|------|----------|------|
| 企業特化フル模擬 | 「○○社の一次面接を模擬で」 | `--company {名} --mode full` |
| 汎用コールド練習 | 「ランダムに面接練習」 | `--mode cold` |
| 弱点集中練習 | 「最近答えられない論点で模擬」 | `--mode weakness`、`feedback-integrator` 出力 / 過去ログ Grep |
| 特定 Q 深掘り | 「B-2 学歴ギャップの深掘り練習」 | `--mode pinpoint --question B-2` |
| 技術習熟度の検証 | 「技術面接ガッツリ」 | `--persona テック --strict` |
| 本番強度練習 | 「本番想定で厳しめに」 | `--strict --persona {役員 or マネージャ}` |

詳細：
- ペルソナ別の質問傾向 / 口調 → [references/interviewer-personas.md](references/interviewer-personas.md)
- モード別の質問プール構築 → [references/session-modes.md](references/session-modes.md)
- 5 軸評価ルーブリック → [references/evaluation-rubric.md](references/evaluation-rubric.md)
- 必ず突く論点・深掘りパターン → [references/attack-rules.md](references/attack-rules.md)

## ファイル編集ルール

| 対象 | 操作 |
|------|------|
| `${COMPANIES_DIR}/{企業 or _MOCK}/面接ログ/MOCK_*.md` | 新規作成（MOCK_ プレフィックス必須） |
| `${COMPANIES_DIR}/*/README.md` の現フェーズ | **更新しない**（模擬は本番フェーズに影響させない） |
| `${MASTER_QA}` / `${BEHAVIOR_DOC}` / 素材 / YAML | 編集不可（読み込みのみ。改善は他 skill 経由） |
| `36_応募ダッシュボード.md` | 編集不可（模擬は記録しない） |

## 引数（スラッシュコマンド呼出時）

`/mock-interview` の引数：

| 引数 | 動作 |
|------|------|
| なし | 対話でパラメータを聞き取り |
| `<企業名>` | `--company <企業名>`（短縮形） |
| `--company <企業名>` | 企業特化質問を有効化 |
| `--axis po\|engineer` | 応募軸 |
| `--round カジュアル\|一次\|二次\|最終` | ラウンド |
| `--persona 人事\|現場\|テック\|マネージャ\|役員` | 面接官ペルソナ |
| `--mode full\|pinpoint\|weakness\|cold` | セッションモード |
| `--question <章番号>` | pinpoint モード時の対象 Q |
| `--questions <N>` | 質問数を明示 |
| `--strict` | ヒント無し本番強度 |
| `--no-save` | ログ保存しない（軽い試し打ち用） |

## 責務分離（やらないこと）

| やらないこと | 委譲先 |
|------------|--------|
| 本番面接の振り返り・ログ作成 | `interview-retrospective`（MOCK_ プレフィックス無しのファイル） |
| 改善点のコア資産反映 | `feedback-integrator` |
| 自己理解の対話深掘り | `deep-self-analysis` |
| 想定問答の事前カスタム | `interview-qa-generator` |
| 逆質問の練習 | （本 skill では「逆質問する」役を AI がやらない。逆質問キュレーションは `reverse-questions-curator`） |
| 内定後の比較 | `offer-evaluator` |
| 企業情報の追加収集 | `company-prep --info-only` |
| 本番フェーズの進行 | しない（模擬は README フェーズに影響させない） |
| ダッシュボード記録 | しない |

## 禁止事項 / 終了時チェックリスト

- ❌ MOCK_ プレフィックス無しでログを保存しない（本番ログと混同するため）
- ❌ `${COMPANIES_DIR}/{企業名}/README.md` の現フェーズを更新しない（模擬は本番に影響させない）
- ❌ `36_応募ダッシュボード.md` を編集しない
- ❌ 24 D 章の NG ルール（数値創作・個人事業詳述・家族事情）を **AI 面接官が真似て** ユーザーを誘導しない
- ❌ ユーザー回答を「正解 / 不正解」の二値で裁定しない（評価ルーブリックは多軸）
- ❌ 評価で過度な励まし / 過度な厳しさに偏らない（具体的根拠を必ず添える）
- ❌ ペルソナを途中で勝手に切り替えない（一貫性維持）
- ✅ 終了時：ログファイルが `MOCK_` プレフィックスで保存されているか
- ✅ 終了時：5 軸評価が ◎○△✕ で記入され、各軸にコメント 1 行以上あるか
- ✅ 終了時：改善優先度上位 3 件と次アクションが明示されているか
- ✅ 終了時：必要に応じて `feedback-integrator` / `interview-qa-generator` / `deep-self-analysis` 起動が提案されているか
