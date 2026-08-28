# CLAUDE.md

このファイルは、Claude Code (claude.ai/code) がこのリポジトリで作業する際のガイダンスを提供します。

## 言語

**Claude Code とのやりとりはすべて日本語で行うこと。** コード内のコメント・変数名・コミットメッセージは英語でよいが、会話・説明・提案はすべて日本語で返答する。

## プロジェクト概要

音楽グループの公式サイト。Hugo で構築し、GitHub Pages でホスティング。

**ページ構成：**
- トップページ — グループ紹介・SNSリンク
- Discography — アルバム・楽曲一覧
- Blog — 制作秘話などの記事

## コマンド

```bash
# Hugo インストール（macOS）
brew install hugo

# Hugo サイトの初期化（プロジェクトルートで一度だけ実行）
hugo new site . --force

# テーマのサブモジュール取得
git submodule update --init --recursive

# ドラフト含めてローカル開発サーバー起動（baseURL を localhost に上書き）
hugo server -D --baseURL http://localhost:1313/

# 本番ビルド（./public に出力）
hugo

# ブログ記事の新規作成
hugo new blog/記事スラッグ.md

# ディスコグラフィーエントリの新規作成
hugo new discography/アルバムスラッグ.md
```

## ディレクトリ構成

```
.
├── config.toml (or hugo.toml)  # baseURL・テーマ・メニュー等の設定
├── content/
│   ├── _index.md               # トップページコンテンツ
│   ├── blog/                   # ブログ記事（Markdown）
│   └── discography/            # アルバム・楽曲エントリ
├── layouts/
│   ├── _default/               # ベーステンプレート
│   ├── index.html              # トップページ用テンプレート
│   ├── blog/                   # ブログ一覧・個別テンプレート
│   └── discography/            # ディスコグラフィー一覧・個別テンプレート
├── static/                     # そのまま配信されるアセット（画像・音声など）
├── assets/                     # Hugo Pipes で処理するアセット（SCSS・JS）
├── themes/                     # テーマ（git submodule）
└── .github/workflows/          # GitHub Actions デプロイ設定
```

## コンテンツのフロントマター規約

ブログ記事（`content/blog/*.md`）:
```yaml
---
title: "記事タイトル"
date: 2024-01-01
draft: false
tags: ["制作秘話"]
---
```

ディスコグラフィー（`content/discography/*.md`）:
```yaml
---
title: "アルバム名"
date: 2024-01-01
release_date: "2024-01-15"   # 実際のリリース日（手入力）
release_type: "album"        # album | single | ep（Hugo予約語の type は使わない）
streaming_url: "https://..." # 配信プラットフォームへのリンク
cover:
  image: "images/covers/album-name.jpg"  # 先頭に / を付けない（PaperModのcover.htmlがabsURLで解決するため、先頭/だとbaseURLのサブパス(/official-site/)が抜け落ちる）
tracks:
  - title: "曲名"
    duration: "3:45"
---
```

シングルは1曲＝1ファイル（`single-曲名.md`）として管理し、曲ごとに `release_date` と `streaming_url` を持たせる。

## GitHub Pages デプロイ

`.github/workflows/hugo.yml` で公式 Hugo GitHub Actions ワークフローを使用。`main` へのプッシュをトリガーにビルド・デプロイ。

`config.toml` の必須設定：
```toml
baseURL = "https://<username>.github.io/<repo>/"
```

リポジトリの Settings > Pages で、ソースを **GitHub Actions** に設定すること（`gh-pages` ブランチではなく）。
