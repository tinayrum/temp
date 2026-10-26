# CLAUDE.md

最初に `@AGENTS.md` を読むこと。開発フローの正本は `docs/development.md`。
本ファイルは Claude Code 固有の補足だけを書く。

## 基本姿勢：Claude Code 単体でプロジェクト完了まで進める

人間の役割は「Issue を起票する」「選択肢で承認・回答する」の 2 つだけに絞る。
それ以外（計画・ブランチ作成・実装・docs 更新・CI 確認・PR 作成・報告・承認後のマージ）は Claude Code が行う。
ただし承認ゲートを飛ばして先に進めない。

## 進め方（フェーズと承認ゲート）

| # | フェーズ | Claude Code がやること | 終わりの合図 |
| :--- | :--- | :--- | :--- |
| 1 | 把握 | `gh issue view <番号>` で Issue を読む。Issue がなければ起票案を作り G0 で確認して `gh issue create` | Issue 番号が決まる |
| 2 | 計画 | 変更ファイル・影響範囲・分割案・完了条件を Issue にコメント | **G1 着手承認** |
| 3 | 実装 | `develop` から規約どおりのブランチを切り、実装と `docs/` 更新、`mkdocs build --strict` | ローカル検証が終わる |
| 4 | 報告・PR | 作業報告を Issue に投稿し、`develop` 向け PR を作成（`Closes #<番号>`）。CI 結果を確認 | **G2 マージ承認** |
| 5 | 統合 | 承認後に `gh pr merge` で develop へマージし、Issue に結果を記録 | Issue が閉じる |
| 6 | リリース | 依頼された場合のみ、`develop` → `main` の PR を作成して報告 | **G3 リリース承認** |

- 1 PR で完結する粒度に分ける。大きい依頼は計画フェーズで分割案を出し、G1 で合意する。
- CI が落ちたら原因を直して push し直す。直せない場合は報告して選択肢で方針を聞く。

## 承認・質疑は AskUserQuestion（選択肢方式）で行う

承認・確認・仕様の質問は、本文で自由に問いかけず、必ず `AskUserQuestion` ツールで出す。
回答者は選択肢を選んで **Submit Answer** を押す。

- 1 回 1〜4 問、各問 2〜4 択。`header` は 12 文字以内（例: `着手承認`、`マージ`、`命名`）。
- 推奨案を先頭に置き、`label` の末尾に「（推奨）」を付ける。`description` に選んだ結果を書く。
- 「Other」で自由記述が返ってきたら、要約して再度選択肢で確認してから動く。
- 回答待ちの間、承認が必要な操作は実行しない。回答を受けたら Issue に「質疑記録」を残す
  （形式は `docs/development.md` の「承認・質疑」）。
- 次のものは必ずゲートを通す: 着手、`develop` / `main` へのマージ、方針・仕様の変更、
  依存関係の追加、`.github/workflows/` の変更、ファイルの削除。

承認ゲートの問い合わせ例（G2）:

```text
question: "PR #12（Closes #8）を develop にマージしてよいですか？ 作業報告は Issue #8 に投稿済みです。"
header:   "マージ"
options:
  - label: "develop にマージ（推奨）"
    description: "CI 通過済み。gh pr merge --squash で develop に取り込み、Issue #8 を閉じます"
  - label: "修正してから再確認"
    description: "指摘をコメントしてもらい、修正後にもう一度この確認を出します"
  - label: "保留"
    description: "PR は開いたまま待機します"
```

## 作業報告の提出

- 区切りごとに `gh issue comment <番号> --body-file <file>` で Issue に投稿する（形式は `docs/development.md`）。
- PR 作成時は同じ内容を PR 本文の「作業報告」欄に入れる。
- 報告には必ず「変更したファイル / 実行したコマンドと結果 / 確認できていないこと」を入れる。
  `mkdocs build --strict` を通していない場合はその旨を明記する。
- 投稿したら、続けて該当する承認ゲートを AskUserQuestion で出す。チャットの最終返答は要点だけにし、詳細は Issue を参照させる。

## 実行環境

- ローカル PC（Windows）と GCP 開発 VM（Ubuntu）の両方で動かす。
  OS 固有のパスやコマンドを手順に書かない。書く必要がある場合は両方を併記する。
- 仮想環境は `.venv` を使う。グローバルに pip install しない。

## 権限

- `.claude/settings.json` の設定に従う。deny 対象の操作は実行せず、提案として提示する。
- `.github/workflows/` の編集は deny。必要なら差分を提示し、人間に適用してもらう。
  `ci.yml` / `check_branch.yml` / `release-drafter.yml` の中身は tinayrum/.github の再利用ワークフローにあるため、CI の挙動を変える提案はそちらへの Issue として出す。
- push は作業ブランチだけ（`git push -u origin <prefix>/<name>`）。`main` / `develop` へは push しない。

## コミット

- メッセージは `type(scope): subject`。PR タイトルも同じ形式で付ける（`gh pr create --base develop --title "type(scope): subject"`）。
- マージは承認後に行う。日々の PR（develop 向け）は `gh pr merge --squash`。squash のコミット名は PR タイトルになる。
- リリース PR（`develop` → `main`）だけは `gh pr merge --merge`（マージコミット）で行う。
  squash にすると develop の各 PR が main から見えなくなり、release-drafter の版数が Patch に、ノートが 1 件だけになるため
  （詳細は `docs/development.md` の「リリース」）。
- 既存ファイルは差分編集する。全文を書き直さない（改行コードの差分が全行に広がるため）。
- 変更していないファイルをステージしない。`git add -A` ではなくパスを指定する。
