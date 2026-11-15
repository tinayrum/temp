# 開発ガイド

このリポジトリの開発フローの正本。`README.md` / `AGENTS.md` / `CLAUDE.md` はこのページを要約・参照する。
ルールを変えるときは、このページを先に更新する（Docs as Code）。

## 基本方針

- **Issue 駆動**: 開発タスクの起票・相談・進捗・報告はすべて GitHub Issue で行う。
  1 Issue = 1 ブランチ = 1 PR を原則とする。
- **main ＝ リリース / develop ＝ ステージング**: 日々の変更は develop に集め、
  リリース時に develop から main へ反映する。どちらにも直接 push しない。
- **Docs as Code**: `src/` を変えた PR では `docs/` も更新する。
- **CI 必須**: PR は CI（GitHub Actions）が通らないとマージしない。

## ブランチ戦略

| Prefix | 用途 | SemVer影響 | 分岐元 | マージ先（PR の base） | 例 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `main` | メインブランチ（リリース） | なし | — | — | `main` |
| `develop` | ステージング | なし | `main` | `main` | `develop` |
| `feature/` | 新機能追加 | Minor | `develop` | `develop` | `feature/add-login-function` |
| `bugfix/` | バグ修正 | Patch | `develop` | `develop` | `bugfix/fix-crash-on-startup` |
| `hotfix/` | 緊急修正 | Patch | `main` | `main` → `develop` に戻す | `hotfix/fix-security-vulnerability` |
| `release/` | リリース準備 | Patch/Minor | `develop` | `main` → `develop` に戻す | `release/v1.2.0-prep` |
| `docs/` | ドキュメント更新のみ | Patch | `develop` | `develop` | `docs/update-api-docs` |
| `chore/` | その他メンテナンス | Patch | `develop` | `develop` | `chore/update-dependencies` |

`.github/workflows/check_branch.yml` が PR ごとに「ブランチ名の prefix」と「マージ先の組み合わせ」を検証する。

- `main` に向けてよいのは `develop` / `release/*` / `hotfix/*` だけ。
- `develop` に向けてよいのは `feature/*` / `bugfix/*` / `docs/*` / `chore/*` と、戻しの `release/*` / `hotfix/*`。

```mermaid
gitGraph
    commit id: "v1.0.0"
    branch develop
    checkout develop
    branch feature/add-login
    checkout feature/add-login
    commit id: "feat(auth): add login"
    checkout develop
    merge feature/add-login
    branch docs/update-guide
    checkout docs/update-guide
    commit id: "docs: update guide"
    checkout develop
    merge docs/update-guide
    checkout main
    merge develop tag: "v1.1.0"
```

### リリース

1. develop がステージングとして確認済みであることを確かめる。
2. `develop` → `main` の PR（またはバージョン調整が要る場合は `release/*` → `main`）を作る。
3. G3 の承認後、リリース PR は **マージコミット**（`gh pr merge --merge`）でマージする。squash にしない。
    `release-drafter` は、main に入ったコミットに対応する PR のラベルで版数とリリースノートを作る。
    squash にすると main には「リリース PR の 1 コミット」しか入らず、develop で積み上げた各 PR が見えなくなる。
    その結果、版数は常に Patch になり、ノートもリリース PR の 1 件だけになる。
    マージコミットなら develop の各コミット（＝各 PR）が main の履歴に入るため、各 PR のラベルが使われる。
    リリース PR 自体（`develop` ラベル）はノートに出さない（`.github/release-drafter.yml` の `exclude-labels`）。
4. main へのマージで `release-drafter` がリリースノートとタグを自動作成する。タグは手で作らない。
    版数は、前回のリリース以降に main に入った PR のラベルで決まる（`.github/release-drafter.yml` の `version-resolver`）。

    | PR のラベル | 上がる版数 |
    | :--- | :--- |
    | `major` | Major |
    | `feature` / `enhancement` | Minor |
    | 上記以外（`bugfix` / `fix` / `docs` など） | Patch |

    複数の PR が含まれる場合は、いちばん大きい版数の上げ方が採られる。ラベルはブランチ名から自動で付くため、
    ブランチの prefix（`feature/` → Minor など）がそのまま版数に効く。

5. `release/*` / `hotfix/*` を使った場合は、同じブランチから `develop` への PR も作り、差分を戻す。

## コミットメッセージ

- 形式は `type(scope): subject`。例: `feat(api): add login`
- type は `feat` / `fix` / `docs` / `chore` / `refactor` / `test` / `ci`。scope は任意。
- 1 コミット 1 目的。CI（`ci.yml` の commit-lint）が PR 内の全コミットを検査する。
- **PR タイトルも同じ形式にする**（例: `ci: skip commit-lint on release PRs`）。
  日々の PR（develop 向け）は squash でマージし、squash のコミット名は PR タイトルになる（リポジトリ設定で既定値を「PR タイトル」にする）。
  リリース PR（`develop` → `main`）だけはマージコミットでマージする（理由は「リリース」を参照）。
  CI（`ci.yml` の pr-title-lint）が PR タイトルを検査する。
- `develop` → `main`（リリース）の PR では、develop で検査済みのため commit-lint と pr-title-lint を行わない。

## Issue 駆動の進め方

```mermaid
flowchart LR
    I[Issue 起票] --> S[設計業務]
    S --> P[計画を Issue に書く]
    P --> G1{着手承認}
    G1 -- 承認 --> B[ブランチ作成]
    B --> W[実装 + docs 更新]
    W --> R[作業報告を Issue に投稿]
    R --> PR[PR 作成 → CI]
    PR --> G2{マージ承認}
    G2 -- 承認 --> D[develop へマージ]
    D --> G3{リリース承認}
    G3 -- 承認 --> M[main へマージ = リリース]
```

| 場面 | どこに書くか |
| :--- | :--- |
| 新しい作業・不具合・相談 | Issue を起票（テンプレートを使う） |
| 設計成果物（オントロジー図・要件定義・テストシナリオ・設計） | `docs/design/`。Issue からリンクする |
| 着手前の計画（変更ファイル・影響範囲・分割案・ロードマップ） | Issue のコメント |
| 質疑・承認の記録 | Issue のコメント（「質疑記録」形式） |
| 作業報告 | Issue のコメント ＋ PR 本文 |
| 完了 | PR に `Closes #<番号>` を書き、マージで Issue を閉じる |

Issue の外（チャット・口頭）で決めたことも、決まったら Issue に「質疑記録」として残す。

## 設計業務（着手前に行う）

方針を決めて計画を立てる段階を、設計業務として次の順で進める。順番を入れ替えない。

```mermaid
flowchart LR
    O[オントロジー図] --> Q[要件定義]
    Q --> T[テストシナリオ]
    T --> D[設計]
    D --> P[計画を Issue に書く]
```

| 順 | 工程 | 書くこと | 置き場 |
| :--- | :--- | :--- | :--- |
| 1 | オントロジー図 | 作る対象のシステムが扱う概念と、その関係。Mermaid で書く | `docs/design/ontology.md` |
| 2 | 要件定義 | 何を満たせばよいか。目的、対象範囲、対象外、制約 | `docs/design/requirements.md` |
| 3 | テストシナリオ | 完了条件の定義。要件ごとに、合否が判定できる形で書く | `docs/design/test-scenarios.md` |
| 4 | 設計 | どう作るか。構成、データの流れ、選ばなかった案と理由 | `docs/design/design.md` |

- テストシナリオは設計より先に決める。「何ができたら終わりか」を実装の前に合意するため。
- 設計成果物は Issue からリンクし、G1（着手承認）で計画と一緒に承認を受ける。
- 実装中に要件や完了条件が変わったら、設計成果物を先に直し、選択肢方式で承認を取り直す。
- 設計成果物を作るのは、`feature/` と、仕様が変わる `bugfix/` の作業。
  `docs/` と `chore/`、仕様が変わらない修正では省略してよい。省略したら計画にその旨を書く。
- 作業が複数の Issue にまたがる場合は、GitHub のマイルストーンでロードマップを管理する。

## 承認・質疑（選択肢方式）

承認や仕様の確認は、自由記述ではなく **選択肢方式** で問い合わせる。
Claude Code では `AskUserQuestion` ツールを使い、回答者は選択肢を選んで **Submit Answer** を押す。

### 問い合わせのルール

- 1 回の問い合わせは 1〜4 問、各問 2〜4 択。
- 推奨案がある場合は先頭に置き、ラベル末尾に「（推奨）」を付ける。
- 各選択肢の説明に「選んだ場合に何が起きるか」を書く。
- 選択肢で表せない回答（Other の自由記述）が来た場合は、内容を要約して、もう一度選択肢で確認する。
- 回答が来るまで、承認が必要な操作（マージ・リリース・方針変更）は実行しない。
- 回答を受けたら、Issue に次の形式で記録する。

```markdown
### 質疑記録
- 問い: <質問文>
- 選択肢: A（推奨）/ B / C
- 回答: B
- 次の行動: <回答を受けて何をするか>
```

### 承認ゲート

| ゲート | タイミング | 選択肢の例 |
| :--- | :--- | :--- |
| G1 着手承認 | 設計成果物と計画を Issue に書いた後 | 承認して着手（推奨）/ 計画を修正 / 中止 |
| G2 マージ承認 | PR 作成・CI 通過・作業報告の後。テストシナリオの合否を確認する | develop にマージ（推奨）/ 修正してから再確認 / 保留 |
| G3 リリース承認 | develop の確認後 | main にリリース（推奨）/ 延期 / 一部だけ release/ で出す |

小さな仕様の迷い（命名・配置・実装方式）も、推測で決めずに選択肢で確認する。

## 作業報告

作業の区切り（計画完了・実装完了・PR 作成・マージ後）ごとに、次の形式で Issue にコメントする。
PR 作成時は同じ内容を PR 本文の「作業報告」に入れる。

```markdown
## 作業報告（<フェーズ名>）
- 対象 Issue: #<番号> / ブランチ: <branch> / PR: #<番号>
- 変更したファイル:
  - <path>: <何を変えたか>
- 実行したコマンドと結果:
  - `mkdocs build --strict`: 成功 / 失敗（内容）
- 完了条件の結果:
  - <テストシナリオの番号>: 合 / 否 / 未確認（設計成果物を省略した作業では「対象なし」）
- 確認できていないこと:
  - <未実行のテスト、推測で書いた箇所、TODO>
- 次のアクション: <承認待ち（G2）など>
```

報告を投稿したら、続けて承認ゲートの問い合わせ（選択肢方式）を行う。

## レビュー（GitHub Copilot）

PR のレビュアーには GitHub Copilot を設定する。実装は Claude Code、レビューは Copilot、マージは人間が行う。

- 自動で付ける：GitHub の個人設定（Settings → Copilot）で「Automatic Copilot code review」を Enabled にする。
  自分が作成する PR すべてに Copilot のレビューが付く。
- 手で付ける：PR の Reviewers で Copilot を選ぶ。
- Copilot のレビューはコメントだけで、承認にはならない。マージの可否は G2 で人間が決める。
- 指摘には、修正するか、修正しない理由を返信するかのどちらかで対応し、作業報告に結果を書く。
- レビューの観点は `.github/copilot-instructions.md` に書く。

## CI

| ワークフロー | タイミング | 内容 |
| :--- | :--- | :--- |
| `ci.yml` | PR（main / develop 向け）、develop への push | `mkdocs build --strict`、コミットメッセージ・PR タイトル検査（develop → main の PR は対象外）、Docs as Code 検査 |
| `check_branch.yml` | PR | ブランチ名 prefix とマージ先の組み合わせ検査 |
| `pr-labeler.yml` | PR | ブランチ名からラベル付与（`docs` も `docs/` ブランチだけに付く。リリースノートで同じ PR が 2 区分に出ないようにするため） |
| `release-drafter.yml` | main への push | リリースノート・タグの自動作成（`GITHUB_TOKEN` で動く） |

`ci.yml` / `check_branch.yml` / `release-drafter.yml` は、[tinayrum/.github](https://github.com/tinayrum/.github) の再利用ワークフロー
（`reusable-ci.yml` / `reusable-check-branch.yml` / `reusable-release-drafter.yml`）を `@main` で呼び出すだけにしている。
CI の中身を変えるときは tinayrum/.github 側を変更する。変更は呼び出している全リポジトリに反映される。
docs ビルドの有無や Python のバージョンなどは、呼び出し側の `with:` で切り替えられる（入力の一覧は各ワークフローの先頭コメント）。

Docs as Code 検査は、`src/` に変更があるのに `docs/` に変更がない PR を失敗させる。
ドキュメント更新が本当に不要な場合だけ、PR に `skip-docs` ラベルを付けて理由を PR 本文に書く。
