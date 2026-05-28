---
name: company-prep
description: 個人の転職活動で新規応募候補が出た際に、`20_面接対策/companies/{企業名}/` を `_TEMPLATE/` から立ち上げ、URL 提供時は WebFetch で `企業情報.md` / `README.md` を初期埋めし、`32_企業評価チェックリスト.md` の必須条件を仮判定、`36_応募ダッシュボード.md` に新規行を追加するスキル。「○○社の企業フォルダ作って」「新しい応募先のセットアップ」「company-prep ○○」「企業分析」「企業フォルダ初期化」などのリクエストでトリガー。portfolio リポジトリ（${USER_PORTFOLIO_ROOT}/20_面接対策/companies/）配下の専用パイプラインのみを対象とする。志望動機の本文確定は手動、想定問答は `interview-qa-generator` へ、逆質問は `reverse-questions-curator` へ、職務経歴書バリアントは `cv-builder` へ委譲。
---

# Company Prep

新規応募候補が出た時点で実行する、企業別面接対策フォルダの立ち上げスキル。`_TEMPLATE/` の複製・URL からの初期情報抽出・ダッシュボード追記までを 1 手で行う。

## 設定ファイル

> `${USER_PORTFOLIO_ROOT}` は `${CLAUDE_PLUGIN_ROOT}/config/personal-config.local.yaml` の `paths.portfolio_root` から取得。
> パイロット skill (offer-evaluator) で先行実装、他 skill は将来展開（ADR-0004 参照）。

## 構成と場所（絶対パス固定）

```
PORTFOLIO_ROOT  = ${USER_PORTFOLIO_ROOT}
COMPANIES_DIR   = ${PORTFOLIO_ROOT}/20_面接対策/companies
TEMPLATE_DIR    = ${COMPANIES_DIR}/_TEMPLATE
DASHBOARD       = ${PORTFOLIO_ROOT}/30_応募活動/36_応募ダッシュボード.md
CHECKLIST       = ${PORTFOLIO_ROOT}/30_応募活動/32_企業評価チェックリスト.md
CUSTOM_PO       = ${PORTFOLIO_ROOT}/20_面接対策/23_応募先別カスタマイズ指針_PO軸.md
CUSTOM_ENG      = ${PORTFOLIO_ROOT}/20_面接対策/23_応募先別カスタマイズ指針_実装エンジニア軸.md
MOTIVATION_BANK = ${PORTFOLIO_ROOT}/30_応募活動/34_志望動機サンプル集.md
```

新規生成先：`${COMPANIES_DIR}/{企業名}/`（README.md / 企業情報.md / 想定問答_カスタム.md / 逆質問.md / 面接ログ/_TEMPLATE.md）

## 起動時フロー

1. **企業名を確定**：ユーザー指定の社名から、`COMPANIES_DIR` 配下に既存か `ls` で確認
2. **既存時の挙動**：既存フォルダがあれば「上書き / 不足項目埋め / 中止」を選択させる。デフォルトは「不足項目埋め」
3. **URL の有無確認**：公式 URL / JD URL / 求人票 URL を 1 つでも持っていれば WebFetch 適用、無ければ全項目を `要確認` プレースホルダで残す
4. **応募軸**：ユーザー指定があれば採用、無ければ「未確定」のまま README に書き、後段の skill で確定を促す

## 基本ワークフロー

1. **フォルダ複製**：`cp -R "${TEMPLATE_DIR}/" "${COMPANIES_DIR}/{企業名}/"`（既存時は `cp -n` で上書き防止 → 不足ファイルのみ補完）
2. **`企業情報.md` 初期埋め**（URL あり時）：
   - WebFetch で公式 / JD ページを取得 → [references/webfetch-extraction.md](references/webfetch-extraction.md) の抽出ルールで以下を埋める：事業内容 / 技術スタック表 / 組織（規模・拠点）/ 採用情報（JD 要旨・選考フロー・想定年収）
   - 抽出できなかった項目は `要確認` で残す（`—` のまま放置しない）
3. **`README.md` 初期埋め**：
   - 基本情報表（企業名 / URL / 業界 / 規模 / 拠点 / エージェント / 応募軸 / 年収 / 副業 / リモート）を `企業情報.md` から転記、未確定欄は `要確認`
   - 現フェーズを `情報収集中` で確定、`最終更新` を当日 YYYY/MM/DD で記入
4. **必須条件仮判定**：`${CHECKLIST}` の必須条件 1（多重下請け SES）/ 3（勤務地）/ 5（レガシー技術のみか）を WebFetch 結果から仮判定し、`README.md` の評価スコア欄に `必須条件クリア：仮 X / 5（面談で確定）` の形で下書き
5. **志望動機骨子の提示**（応募軸確定時）：
   - 業界（自社 SaaS / AI スタートアップ / コンサル / スタートアップ / 外資）と職種を `${CUSTOM_PO}` or `${CUSTOM_ENG}` から特定
   - 該当セクションの強調軸とテンプレ骨子を、README の `## 志望動機要旨` に **下書きとして** 挿入（最終確定は手動）
6. **ダッシュボード追記**：`${DASHBOARD}` セクション 1 の企業別進捗テーブルに新規行を追加。詳細は [references/dashboard-update.md](references/dashboard-update.md)
7. **次アクション提示**：README の `## 次アクション` に以下を箇条書きで書く：
   - `reverse-questions-curator` で逆質問初版
   - `interview-qa-generator` で想定問答カスタム（応募軸確定後）
   - `cv-builder --variant {企業名}` で職務経歴書バリアント（応募確定後）

## 操作カタログ

| 操作 | トリガー語 | 動作 |
|------|----------|------|
| 新規フォルダ立ち上げ | 「○○社の企業フォルダ作って」 | 全工程実行 |
| URL からの企業情報埋め | 「○○社の企業情報埋めて URL: ...」 | フォルダ既存前提で `企業情報.md` のみ更新 |
| 必須条件仮判定 | 「○○社の必須条件チェック」 | `README.md` の評価スコア欄のみ更新 |
| 志望動機骨子提示 | 「○○社の志望動機骨子」 | `README.md` の志望動機要旨欄のみ更新（応募軸必須） |
| ダッシュボード追記 | 「○○社をダッシュボードに追加」 | `${DASHBOARD}` セクション 1 への行追加のみ |

詳細：
- 全プレースホルダの自動埋め可否マトリクス → [references/template-fields.md](references/template-fields.md)
- WebFetch 抽出ルール → [references/webfetch-extraction.md](references/webfetch-extraction.md)
- ダッシュボード編集規約 → [references/dashboard-update.md](references/dashboard-update.md)

## ファイル編集ルール

| 対象 | 操作 |
|------|------|
| `${COMPANIES_DIR}/{企業名}/*.md`（新規生成分） | 編集可（追記・上書き） |
| `${TEMPLATE_DIR}/` 配下 | 編集不可（雛形は不変） |
| `${DASHBOARD}` | 追記限定（既存行は触らない、新規企業の行のみ追加） |
| `${CHECKLIST}`, `${CUSTOM_PO}`, `${CUSTOM_ENG}`, `${MOTIVATION_BANK}` | 編集不可（読み込み専用） |

未確定欄は **`—` のままにしない**。必ず `要確認` か仮値（`仮 X / 5`）で埋める。後段スキルが「埋められる箇所」を見分けるため。

## 引数（スラッシュコマンド呼出時）

`/company-prep` の引数：

| 引数 | 動作 |
|------|------|
| `<企業名>` | フォルダ立ち上げ（URL 無し、全項目 `要確認`） |
| `<企業名> --url <URL>` | URL 提供（複数指定可：`--url A --url B`） |
| `<企業名> --axis po\|engineer` | 応募軸を確定して志望動機骨子を生成 |
| `<企業名> --agent <エージェント名>` | エージェント経由応募として記入（`30_応募活動/agents/{agent}.md` 参照） |
| `--info-only <企業名>` | フォルダ既存前提で `企業情報.md` のみ更新 |
| `--dashboard-only <企業名>` | `${DASHBOARD}` への追記のみ |

引数の組合せ可（例：`AcmeCorp --url https://acme.co --axis po`）。

## 責務分離（やらないこと）

| やらないこと | 委譲先 |
|------------|--------|
| 想定問答の企業特化カスタム | `interview-qa-generator` |
| 逆質問のキュレーション | `reverse-questions-curator` |
| 面接後の振り返り・ログ作成 | `interview-retrospective` |
| 職務経歴書の応募先バリアント生成 | `cv-builder --variant` |
| 自己分析の深掘り | `deep-self-analysis` |
| 志望動機本文の確定（骨子提示までで止める） | 手動 |
| git commit / push | `commit-helper` |
| 内定後の比較・交渉 | `offer-evaluator` |
| エージェント面談の準備 | `agent-interview-prep` |

## 禁止事項 / 終了時チェックリスト

- ❌ `_TEMPLATE/` 配下を編集しない
- ❌ `${DASHBOARD}` の既存行や `35_応募管理テンプレ.md`（フォーマット定義）を編集しない
- ❌ WebFetch 失敗時に憶測でフィールドを埋めない（`要確認` で残す）
- ❌ 志望動機本文を一文字でも自動確定しない（骨子・テンプレ骨子のみ）
- ❌ 検証不能な成果数値・社内固有情報を `企業情報.md` に書かない
- ✅ 終了時：`{企業名}/` 配下 5 ファイル（README / 企業情報 / 想定問答_カスタム / 逆質問 / 面接ログ/_TEMPLATE.md）が揃っているか
- ✅ 終了時：`${DASHBOARD}` セクション 1 に新規行が 1 行追加されたか
- ✅ 終了時：`README.md` の現フェーズと最終更新が記入されているか
- ✅ 終了時：次アクションが箇条書きされているか
