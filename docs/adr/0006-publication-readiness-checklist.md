---
project: personal-skills
status: draft
version: 0.0.1
updated: 2026-05-28
audience: [human, agent]
---

# ADR 0006 — 公開戦略と公開前チェックリスト

- **Status**: Accepted
- **Date**: 2026-05-28
- **Deciders**: personal-skills maintainers

## 文脈

本リポジトリは現在 **個人の転職活動向け skill 集合**だが、将来：

1. 他 portfolio リポジトリ（用途別 portfolio 等）でも再利用したい
2. GitHub private repo として持ち運び性を確保したい
3. さらに先、汎用 OSS として public 公開したい

の段階的展開を視野に入れている。

公開のタイミングでまだ個人情報が残っていると即座にプライバシー漏洩、汎用性も低い。

## 決定

**3 段階の公開戦略を採用する：**

### Stage 1：当面 private（現状）

- LICENSE：`UNLICENSED`（`plugins/job-search/.claude-plugin/plugin.json`）
- リポジトリ root：private GitHub repo（将来作成）
- commit / push：本セッションでは行わず、人間が後で実施
- `personal-config.local.yaml` は `.gitignore` で除外

### Stage 2：private GitHub repo へ push（近い将来）

- private repo を作成（例：`<org-or-user>/personal-skills`）
- 初回 commit + push（commit 単位の分割は ADR 0001 の Conventional Commits 規則に従う）
- マシン跨ぎが clone 一発で可能になる
- README にセットアップ手順（example yaml コピー → local 編集 → settings.json 編集）を明記

### Stage 3：public 化（将来、未確定）

公開前に **本 ADR の「公開前チェックリスト」を完遂する**：

#### Phase 3'-B 完遂

- [ ] 全 SKILL.md / references の人名「個人」を `{{ person.name }}` に
- [x] 副業プロジェクト名（例：副業プロジェクト A / B）を `{{ side_projects.projects[N].name }}` に
- [x] 数値（例：資格スコア・副業時間・希望年収）を `{{ qualifications.* }}` / `{{ side_projects.* }}` / `{{ compensation.* }}` に
- [x] 学歴・経歴期間を `{{ career.* }}` / `{{ education.* }}` に
- [x] 海外経験・通勤可能エリアを `{{ global_experience.* }}` / `{{ mobility.* }}` に

#### Phase 3'-C：例として残す箇所への「例：」明記

- [ ] 企業名（例：職歴名 / 取引先名 / 参画先名）に「例：」を付与
- [ ] エージェント名（例：外資系エージェント） / （例：国内大手エージェント）に「例：」を付与
- [x] 副業案件数（5 件 / 平日 3 時間 等）を `{{ side_projects.* }}` に寄せる

#### 法的・倫理的

- [ ] LICENSE を `UNLICENSED` → `MIT` or `Apache-2.0` に切替
- [ ] 商標・他社情報の利用範囲確認（特に企業名・エージェント名の実名利用）
- [ ] portfolio repo との dependency（`24_面接想定問答.md` 等のドキュメント参照）を切る or サンプル化
- [ ] サンプル portfolio 構造を `docs/sample-portfolio/` に提供

#### 検証

- [ ] 個人情報スキャン再実施（grep で人名 / 住所 / 連絡先 / 企業名）
- [ ] `aikata doctor` がエラー無し
- [ ] サンプル portfolio で全 skill が動作することを別ユーザー（or 別マシン）で検証
- [x] README に「これは個人ユース skill だが、`config/personal-config.example.yaml` をコピーして自分用にカスタムすれば再利用可能」明記

#### 公開後の汚染防止

- [ ] `main` は pull request 必須、直接 push 禁止
- [ ] force push / branch deletion 禁止
- [ ] release tag（`v*`）の更新・削除禁止
- [ ] CODEOWNERS review 必須
- [ ] validation workflow 必須
- [ ] Claude Code marketplace は GitHub source + protected release tag で案内
- [ ] Codex 向け install 手順を公開 docs に記載

## 帰結

**ポジティブ**:

- 公開のたびに何を確認すべきかが明文化、漏れがない
- Stage 1〜3 で投資量を段階分解、いきなりフル公開対応しない
- LICENSE 切替タイミングが明確

**ネガティブ**:

- Stage 3 への到達には大きな作業量
- チェックリストのメンテナンス（新 skill 追加時に項目追加が必要）

## 代替案

- **最初から public 想定で全汎用化**：作業量過大、当面の運用に支障
- **ずっと private**：他人との skill 共有や OSS 文化への貢献機会を失う

## 関連

- [ADR 0004](./0004-personal-info-abstraction-strategy.md)（個人情報の段階的汎用化）
- [ADR 0005](./0005-aikata-as-scaffold-tool.md)（aikata 整合点検）
