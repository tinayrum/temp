# AGENTS.md

このリポジトリで作業する AI エージェント（Claude Code / GitHub Copilot）向けの共通ルール。
人間が作業する場合も同じルールに従う。ツール固有の補足は `CLAUDE.md` と
`.github/copilot-instructions.md` にあるが、判断の基準は常にこのファイルが優先する。
開発フローの詳細（ブランチ戦略・承認ゲート・報告形式）の正本は `docs/development.md`。

## リポジトリの構成

```
src/            実装
docs/           MkDocs のソース（Docs as Code の対象）
mkdocs.yml      サイト設定。nav に登録しないとページは公開されない
requirements.txt  ドキュメントビルドの依存関係
.github/        Issue / PR テンプレート、ワークフロー
```

## セットアップとビルド

```bash
python -m venv .venv
. .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
mkdocs serve                  # http://127.0.0.1:8000 で確認
mkdocs build --strict         # PR を出す前に必ず通す
```

実装コードを追加したら、テストの実行コマンドをこのセクションに追記すること。

## ブランチ

- `main` ＝ リリース、`develop` ＝ ステージング。どちらへも直接 push は禁止。変更は必ず PR 経由。
- ブランチ名の prefix は `feature/` `bugfix/` `hotfix/` `release/` `docs/` `chore/` のいずれか。
  `develop` のみ例外。
- 通常の作業ブランチは `develop` から切り、`develop` に PR する。
  `main` に PR してよいのは `develop` / `release/*` / `hotfix/*` だけ。
  `hotfix/*` は `main` から切り、`main` へのマージ後に `develop` へも戻す。
- `.github/workflows/check_branch.yml` が PR 時に prefix とマージ先を検証し、違反すると CI が落ちる。
- ラベルはブランチ名と変更ファイルから自動付与される（`.github/labeler.yml`）。手動で付けない。

## コミット

- `type(scope): subject` の形式。type は feat / fix / docs / chore / refactor / test / ci。
- 1コミット1目的。無関係なフォーマット変更や改行コードの一括変換を混ぜない。
- PR タイトルも同じ形式にする。日々の PR は squash でマージし、PR タイトルがそのままコミット名になる。
- リリース PR（`develop` → `main`）だけはマージコミットでマージする。squash にすると develop の各 PR が
  main から見えなくなり、`release-drafter` の版数とリリースノートが正しく作られない（`docs/development.md` の「リリース」）。
- `.github/workflows/ci.yml` が PR 内の全コミットのメッセージと PR タイトルを検査する（develop → main の PR は対象外）。

## Issue 駆動

- 開発タスクの管理とやり取り（相談・質疑・承認・作業報告）は GitHub Issue で行う。
- 1 Issue = 1 ブランチ = 1 PR。Issue がない作業は、先に Issue を起票してから始める。
- チャットなど Issue の外で決まったことも、Issue に「質疑記録」として残す。

## Docs as Code（このリポジトリの最重要ルール）

- `src/` を変更したら、同じ PR で `docs/` も更新する。
- 新規ページは `docs/` に追加し、`mkdocs.yml` の `nav` にも登録する。
  nav への登録漏れは `mkdocs build --strict` では検出されないので、目視で確認する。
- 図は Mermaid で書く（`pymdownx.superfences` 設定済み）。画像の貼り付けで済ませない。
- 数式は `pymdownx.arithmatex` + MathJax が有効。

## PR

- `.github/pull_request_template.md` を埋める。チェックボックスは実際に確認した項目だけ入れる。
- 関連 Issue を `Closes #<番号>` で紐づける。
- Issue テンプレートは task（Claude Code 実行用）/ bug_report / feature_request / docs_update / chore / hotfix / release を用意している。
- PR 本文の「作業報告」「承認記録」を埋める。

## エージェントの作業手順

1. 対象 Issue を読み、着手前に変更するファイル・影響範囲・完了条件を Issue にコメントする。
2. 承認（G1）を **選択肢方式** で取ってから編集する。
3. `develop` から作業ブランチを切り、実装と `docs/` 更新を行い、`mkdocs build --strict` を通す。
4. 作業報告を Issue に投稿し、`develop` 向けの PR を作る。
5. CI 通過を確認し、マージの承認（G2）を選択肢方式で取ってからマージする。
6. 根拠が不確かな箇所は推測で埋めず、`TODO:` を残して報告する。
7. 報告は「変更したファイル / 実行したコマンドと結果 / 確認できていないこと」の3点を含める。

## 承認・質疑（選択肢方式）

- 承認・確認・仕様の質問は、自由記述ではなく選択肢（2〜4 択、推奨案を先頭）で問い合わせる。
  Claude Code では `AskUserQuestion` ツールを使う（回答は Submit Answer で返る）。
- 回答が来るまで、マージ・リリース・方針変更など承認が必要な操作は実行しない。
- 回答は Issue に「質疑記録」として残す。形式は `docs/development.md` を参照。

## やってはいけないこと

- `main` / `develop` への直接 push、force push。
- 承認ゲート（着手・マージ・リリース）を通さずに次の段階へ進むこと。
- 顧客名・認証情報・API キー・個人情報をコミットに含める。
- `requirements.txt` のバージョン指定を、依頼されていないのに変更する。
- `.github/workflows/` 配下の変更。CI の挙動が変わるため、必ず人間の確認を取ってから。
- ビルド生成物（`site/`）のコミット。
- 改行コードの一括変換。このリポジトリは Windows と Linux の両方で編集されるため、
  差分がファイル全体に広がり、レビュー不能になる。

## このリポジトリ固有（Organization: tinayrum / 業務用テンプレート）

- `main` への push で `release-drafter` がリリースを自動公開する。タグを手動で作らない。
- `ci.yml` / `check_branch.yml` / `release-drafter.yml` は tinayrum/.github の再利用ワークフローを呼ぶだけ。
  CI の中身を変えるときは tinayrum/.github 側を変更する（全リポジトリに反映される）。
- ワークフローは `GITHUB_TOKEN` だけで動く。追加の Secret は不要。認証情報をコードやドキュメントに書かない。
- 業務案件で使うテンプレートのため、特定顧客に固有の名称・仕様をここに残さない。
