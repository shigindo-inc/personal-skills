---
project: personal-skills
status: draft
version: 0.0.1
updated: 2026-05-28
audience: [human, agent]
---

# 現在の作業 — personal-skills

> 本ファイルはエージェントの**短期作業メモリ**。作業の進行に合わせて
> 自由に書き換えてよい。不変ルールの
> 更新は [AGENTS.md](../../AGENTS.md) で行う。

---

## 状況

public 化済み。Claude Code は GitHub marketplace + release tag 参照へ切り替え済み。Codex も marketplace 登録は成功し、repo 側の Codex plugin manifest を追加中。

## 次のアクション

- [x] 公開対象ファイルと ignore 済みローカルファイルを分けて棚卸しする
- [x] 名前・メール・電話番号・ローカルパス・秘密情報らしき文字列を横断検索する
- [x] 残存リスクと修正候補を整理する
- [x] `personal-config.example.yaml` に経歴・副業・希望条件の placeholder を追加する
- [x] skill / references 内の具体値を config 参照または明示サンプルへ寄せる
- [x] README / ADR 0006 を更新し、再スキャンする
- [x] `docs/` 配下を再帰的に確認し、公開向けでない具体例があれば一般化する
- [x] private repo のまま `origin/main` へ push する
- [x] 可能な範囲で GitHub merge / security settings を安全側へ寄せる
- [x] SECURITY / CONTRIBUTING / CODEOWNERS / validation workflow を追加する
- [x] Claude Code marketplace を GitHub source + release tag 前提に更新する
- [x] Codex 向け外部 install 手順を追加する
- [x] workflow 通過後、public 化して branch / tag protection を適用する
- [ ] Codex 用 `.agents/plugins/marketplace.json` / `.codex-plugin/plugin.json` を PR 化し、release tag を切る

## メモ

直接的な実名・個人メール・実ローカルパスは公開候補には見つからなかった。`plugins/job-search/config/personal-config.local.yaml` には実値があるが `.gitignore` 対象。公開候補に出ていた `.serena/` は `.gitignore` に追加済み。学歴ギャップ・副業時間・海外経験・通勤可能エリアなど、個人に近い経歴/希望条件の具体例は `personal-config.example.yaml` の placeholder または「例：」付きサンプルへ寄せた。検証では direct identifier / email / 機密情報 scan は問題なし。`personal-config.example.yaml` は Ruby YAML parser で読み込み成功。
`docs/` 配下についても、実ローカルパス・具体 org 例・過去スキャン件数・具体的な経歴/副業サンプルを一般化した。再スキャンでは direct identifier / email / 機密情報 / 準個人情報パターンはいずれもヒットなし。
private repo では GitHub plan 制約により branch protection / rulesets API が 403 になった。public 化前に外部第三者は push できないため、公開運用ファイルを入れて push し、public 化直後に ruleset を適用する順序に調整する。
ローカル検証では public candidate の JSON/YAML、privacy scan、marketplace sanity check が通過。
Codex は `codex plugin marketplace add shigindo-inc/personal-skills --ref v0.1.0` で marketplace 登録自体は成功。ただし Codex plugin として確実に認識させるため、Codex 用 manifest を追加して `v0.1.1` tag で案内する。
