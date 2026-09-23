# GitHub Copilot Instructions

このリポジトリのルールはリポジトリ直下の `AGENTS.md` に集約されている。
提案・チャット応答・エージェント実行の前に `AGENTS.md` の内容に従うこと。

特に守るもの:

- ブランチ prefix は `feature/` `bugfix/` `hotfix/` `release/` `docs/` `chore/`。`main` へ直接コミットしない。
- コミットメッセージは `type(scope): subject`。
- Docs as Code: `src/` を変更したら `docs/` も同じ PR で更新し、新規ページは `mkdocs.yml` の `nav` に登録する。
- 図は Mermaid で書く。
- `.github/workflows/` の変更は提案に留め、実行しない。
- 認証情報・顧客名・個人情報を提案に含めない。

回答は日本語で行う。存在を確認できないファイルパス・API・設定項目を作らない。
不明な箇所は `TODO:` コメントとして残す。
