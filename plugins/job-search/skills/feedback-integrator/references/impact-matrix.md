# Impact Matrix — 分類 → 影響ファイル対応

`classification.md` の 5 カテゴリ × FB の具体性に応じた **影響ファイル一覧** とその編集アクション。

## マトリクス本体

### C1：回答内容（個別 Q の答え方）

| 影響ファイル | 編集アクション | 委譲先 |
|-----------|--------------|--------|
| `${MASTER_QA}` の該当章（A / B / C） | 直接編集（章本文の書き換え or 深掘り想定の追加） | 直接 |
| `${SOZAI_DIR}/4X_職歴_*.md` の該当エピソード | 直接編集（具体例・数値の追加） | 直接 |
| `${YAML_*}` の experience[].achievements | YAML 編集 → DOCX 再生成 | `cv-builder` |
| `${SELF_PR}` の関連項目 | 直接編集 | 直接 |
| 各社 `${COMPANIES_DIR}/*/想定問答_カスタム.md` の該当 Q | 要カスタムマーカー追加 + 補強 | `interview-qa-generator` （応募軸トーン再適用） |
| `${BEHAVIOR_DOC}` の「リファクタログ」 | 追記（変更履歴） | 直接 |

### C2：経歴の語り方（時系列・転換理由）

| 影響ファイル | 編集アクション | 委譲先 |
|-----------|--------------|--------|
| `${MASTER_QA}` の B 章（特に B-1, B-2, B-5, B-7） | 直接編集（語り順序・時系列表現の改訂） | 直接 |
| `${SOZAI_DIR}/41_職歴_（例：現職）.md` etc | 直接編集（事実情報の整合保持） | 直接 |
| `${YAML_*}` の experience[].period / role / tasks | YAML 編集 → DOCX 再生成 | `cv-builder` |
| `${SOURCE_MD_DIR}/職務経歴書_*.md` | YAML 変更後に整合点検 | `cv-diff` |
| `${BEHAVIOR_DOC}` の「リファクタログ」 | 追記 | 直接 |

### C3：強み・弱み・自己 PR

| 影響ファイル | 編集アクション | 委譲先 |
|-----------|--------------|--------|
| `${SELF_PR}` | 直接編集（強み 3 つの並び・弱みの厚み・副業の見せ方） | 直接 |
| `${MASTER_QA}` の A-4 / A-5 / B-7 | 直接編集 | 直接 |
| `${YAML_*}` の self_pr 章 + extracurricular | YAML 編集 → DOCX 再生成 | `cv-builder` |
| `${SOZAI_DIR}/42_職歴_副業_Flutter.md`, `45_個人開発.md` | 直接編集（副業 / 個人開発の見せ方統一） | 直接 |
| `${CLAUDE_PLUGIN_ROOT}/skills/interview-qa-generator/references/axis-tone.md` | 強調軸 / トーンダウンの並びを更新（メタ的） | 直接（skill メンテ） |
| `${SELF_INTRO}` | 直接編集（エージェント面談用の自己 PR トーン同期） | 直接 |
| `${BEHAVIOR_DOC}` の「リファクタログ」 | 追記 | 直接 |

### C4：志望動機・カスタマイズ指針

| 影響ファイル | 編集アクション | 委譲先 |
|-----------|--------------|--------|
| `${CUSTOM_PO}` or `${CUSTOM_ENG}` の業界別 / 職種別セクション | 直接編集（強調軸 + テンプレ骨子の改訂） | 直接 |
| `${MOTIVATION_BANK}` の業界別サンプル | 直接編集 | 直接 |
| 各社 `${COMPANIES_DIR}/*/想定問答_カスタム.md` Q2 | 要カスタムマーカー追加 + テンプレ参照更新 | `interview-qa-generator` |
| 各社 `${COMPANIES_DIR}/*/README.md` の志望動機要旨 | 当該業界該当企業のみ再生成提案 | `company-prep`（志望動機骨子の再提示）or 手動 |
| `${CLAUDE_PLUGIN_ROOT}/skills/interview-qa-generator/references/mapping-rules.md`「業界別の追加論点」 | 当社特化質問生成テンプレを更新（メタ的） | 直接（skill メンテ） |
| `${BEHAVIOR_DOC}` の「リファクタログ」 | 追記 | 直接 |

### C5：自己理解の浅さ

| 影響ファイル | 編集アクション | 委譲先 |
|-----------|--------------|--------|
| `${BEHAVIOR_DOC}` の「追加で深掘りしたい質問」 | 追記（FB を質問形式に変換） | 直接 |
| `${BEHAVIOR_DOC}` の「対話で確認」 | **深掘り完了後** に新規セクション追加 | `deep-self-analysis`（先に起動推奨） |
| `${SELF_ANALYSIS}` | 補助情報の追記 | 直接（軽微） |
| `${MASTER_QA}` の A-1 / A-6 / B-9（PO 軸時） | **深掘り完了後** に回答の改訂 | 直接（C1 連鎖） |
| `${CUSTOM_*}`（応募軸の選択軸自体に関わる場合） | **深掘り完了後** に応募軸の方針改訂 | 直接（C4 連鎖） |
| `${BEHAVIOR_DOC}` の「リファクタログ」 | 追記 | 直接 |

**重要**：C5 は **編集より先に `deep-self-analysis` 起動**。深掘り知見を得てから C1〜C4 のリファクタを連鎖的に進める。

## 影響範囲の表示形式

ユーザー提示時のフォーマット：

```
【FB 分類】C{N}（{カテゴリ名}）

【影響ファイル】

[1] ${MASTER_QA} の B-4 章
    アクション：直接編集（Azure / OpenAI / Anthropic の使い分けセクションを追加）
    委譲先：なし

[2] ${SOZAI_DIR}/41_職歴_（例：現職）.md
    アクション：直接編集（生成 AI 実装エピソードの具体化）
    委譲先：なし
    注：事実情報は YAML との整合を保つ

[3] ${YAML_*} の experience[現職].achievements
    アクション：YAML 編集 + DOCX 再生成
    委譲先：cv-builder
    注：[2] の編集後、両軸 YAML に反映

[4] 各社 ${COMPANIES_DIR}/*/想定問答_カスタム.md の Q6
    アクション：影響を受ける企業フォルダを列挙 + 個別再カスタム
    委譲先：interview-qa-generator
    対象企業：AcmeCorp / （他、企業情報.md で AI 主題のもの）

[5] ${BEHAVIOR_DOC} の「リファクタログ」
    アクション：変更履歴を追記
    委譲先：なし

【推奨順序】
1. [1] [2] を直接編集（並列）
2. [3] を cv-builder 経由
3. [4] を interview-qa-generator 経由（企業ごと）
4. cv-diff で全体 drift 確認
5. [5] を追記

承認するもの番号で（複数可、`all` で全部、`none` で全部不採用、`details N` で N 番の詳細を見る）：
```

## 影響ファイルの自動列挙ロジック

「各社 `想定問答_カスタム.md` のうち、本 FB が波及するもの」を判定：

| FB の論点 | 波及判定の Grep / Read |
|----------|---------------------|
| C1 B-4 生成 AI | `${COMPANIES_DIR}/*/企業情報.md` で「AI / ML」「生成 AI」「LLM」に何らか記述がある企業 |
| C1 C-2 テスト戦略 | `${COMPANIES_DIR}/*/企業情報.md` で「テスト文化」「CI/CD」関連のあるもの |
| C2 学歴ギャップ | 全社（A 章質問のため） |
| C3 副業 | `${COMPANIES_DIR}/*/README.md` で「副業可：要確認 / 条件付」のもの優先 |
| C4 PO 軸志望動機 | `${COMPANIES_DIR}/*/README.md` で「応募軸：PO」のもの |
| C4 自社 SaaS 向けカスタマイズ | `${COMPANIES_DIR}/*/企業情報.md` の業界判定で「自社 SaaS」のもの |

自動列挙された企業は「対象企業 N 件：A / B / C / ...」とユーザーに提示。
