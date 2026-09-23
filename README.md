# [App Name] Application Repository

## 🚀 テンプレートから作成した直後にやること

- [ ] `README.md` の `[App Name]` とこのセクション以下を、プロジェクトの内容に書き換える
- [ ] `mkdocs.yml` の `site_name` / `site_description` を設定し、`repo_name` / `repo_url` のコメントを外して埋める
- [ ] `docs/index.md` をプロジェクトの概要に書き換える
- [ ] `AGENTS.md` の「セットアップとビルド」に、実装言語のビルド・テストコマンドを追記する
- [ ] `.github/CODEOWNERS` にレビュー担当を設定する
- [ ] App 単体運用の場合は `.github/workflows/notify_portal.yml` を削除する
- [ ] リポジトリ設定で `main` ブランチ保護（PR 必須・force push 禁止）を有効にする
- [ ] Secrets / Variables に `ORG_ADMIN_TOKEN` と `PORTAL_REPO_NAME` を登録する（Portal 連携する場合）

## 🤖 AI エージェントを使う場合

`AGENTS.md` に Claude Code / GitHub Copilot 共通のルールを置いている。
ツール固有の補足は `CLAUDE.md` と `.github/copilot-instructions.md`、
Claude Code の権限設定は `.claude/settings.json` にある。
プロジェクト固有のビルド・テスト手順は `AGENTS.md` に追記して育てること。

## 🛠 開発ルール (Docs as Code)

### ブランチ命名規則

| Prefix | 用途 | SemVer影響 | 例 |
| :--- | :--- | :--- | :--- |
| `main` | メインブランチ | なし | `main` |
| `develop` | ステージングブランチ | なし | `develop` |
| `feature/` | 新機能追加 | Minor | `feature/add-login-function` |
| `bugfix/` | バグ修正 | Patch | `bugfix/fix-crash-on-startup` |
| `hotfix/` | 緊急修正 | Patch | `hotfix/fix-security-vulnerability` |
| `release/` | リリース準備 | Patch/Minor | `release/v1.2.0-prep` |
| `docs/` | ドキュメント更新のみ | Patch | `docs/update-api-docs` |
| `chore/` | その他メンテナンス | Patch | `chore/update-dependencies` |

### コミットメッセージ規約

- `type(scope): subject` 例: `feat(api): add login`
- type例: feat, fix, docs, chore, refactor, test, ci
- scopeは任意、subjectは簡潔に

### 運用のポイント

- **Docs as Code**: コード修正時はdocs/も必ず更新
- **main直Push禁止**: PR経由でマージ
- **CI/CD必須**: GitHub Actions等で自動テスト・デプロイ
- **README.md整備**: QuickStart・開発手順・依存関係を明記
- **テンプレート活用**: PRテンプレート・Issueテンプレートを用意
