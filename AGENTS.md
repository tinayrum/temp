# AGENTS.md

このリポジトリで作業する AI エージェント（Claude Code / GitHub Copilot）向けの共通ルール。
人間が作業する場合も同じルールに従う。ツール固有の補足は `CLAUDE.md` と
`.github/copilot-instructions.md` にあるが、判断の基準は常にこのファイルが優先する。

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

- `main` への直接 push は禁止。変更は必ず PR 経由。
- ブランチ名の prefix は `feature/` `bugfix/` `hotfix/` `release/` `docs/` `chore/` のいずれか。
  `develop` のみ例外。`.github/workflows/check_branch.yml` が PR 時に検証し、違反すると CI が落ちる。
- ラベルはブランチ名と変更ファイルから自動付与される（`.github/labeler.yml`）。手動で付けない。

## コミット

- `type(scope): subject` の形式。type は feat / fix / docs / chore / refactor / test / ci。
- 1コミット1目的。無関係なフォーマット変更や改行コードの一括変換を混ぜない。

## Docs as Code（このリポジトリの最重要ルール）

- `src/` を変更したら、同じ PR で `docs/` も更新する。
- 新規ページは `docs/` に追加し、`mkdocs.yml` の `nav` にも登録する。
  nav への登録漏れは `mkdocs build --strict` では検出されないので、目視で確認する。
- 図は Mermaid で書く（`pymdownx.superfences` 設定済み）。画像の貼り付けで済ませない。
- 数式は `pymdownx.arithmatex` + MathJax が有効。

## PR

- `.github/pull_request_template.md` を埋める。チェックボックスは実際に確認した項目だけ入れる。
- 関連 Issue を `Closes #<番号>` で紐づける。
- Issue テンプレートは bug_report / feature_request / docs_update / chore / hotfix / release を用意している。

## エージェントの作業手順

1. 着手前に、変更するファイルと影響範囲を列挙して提示する。
2. 承認を得てから編集する。
3. `mkdocs build --strict` を通してから PR を作る。
4. 根拠が不確かな箇所は推測で埋めず、`TODO:` を残して報告する。
5. 報告は「変更したファイル / 実行したコマンドと結果 / 確認できていないこと」の3点を含める。

## やってはいけないこと

- `main` への直接 push、force push。
- 顧客名・認証情報・API キー・個人情報をコミットに含める。
- `requirements.txt` のバージョン指定を、依頼されていないのに変更する。
- `.github/workflows/` 配下の変更。CI の挙動が変わるため、必ず人間の確認を取ってから。
- ビルド生成物（`site/`）のコミット。
- 改行コードの一括変換。このリポジトリは Windows と Linux の両方で編集されるため、
  差分がファイル全体に広がり、レビュー不能になる。

## このリポジトリ固有（Organization: tinayrum / 業務用テンプレート）

- `main` への push で `release-drafter` がリリースを自動公開する。タグを手動で作らない。
- `notify_portal.yml` は Portal リポジトリへ更新を通知する。App 単体運用のリポジトリでは削除する。
- 必要な secrets / vars: `ORG_ADMIN_TOKEN`, `PORTAL_REPO_NAME`。値をコードやドキュメントに書かない。
- 業務案件で使うテンプレートのため、特定顧客に固有の名称・仕様をここに残さない。
