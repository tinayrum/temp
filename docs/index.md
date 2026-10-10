# <Application Name>

<!-- このページはテンプレートです。プロジェクトの内容に書き換えてください。 -->

## 概要

<何をするアプリケーションか、1〜3行で>

## セットアップ

```bash
python -m venv .venv
. .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
mkdocs serve
```

## 構成

```mermaid
flowchart LR
    A[入力] --> B[処理]
    B --> C[出力]
```

## ドキュメントの追加方法

`docs/` に Markdown を追加し、`mkdocs.yml` の `nav` に登録する。
登録しないとサイトに表示されない。

ブランチ戦略・Issue 駆動・承認と作業報告のルールは [開発ガイド](development.md) を参照。
