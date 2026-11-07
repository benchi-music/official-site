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

### 配信前の作品（段階的な情報公開）

配信前の作品は `upcoming: true` を付けて、告知記事に合わせて情報を書き足していく。

- 表示: トップに「配信予定」欄（「最新作」欄とは別）、一覧は「配信アルバム」の先頭に「配信予定」バッジ付き、詳細ページは掲載日を隠し「配信予定日」と表示
- `release_date` は確定前なら `"2026年12月中旬"` のような自由文でよい
- `cover.image` が未設定の間は COMING SOON パネルを表示。`tracks` は未設定なら収録曲欄ごと非表示
- `date` は告知記事と同じ日時（`2026-11-08T00:00:00+09:00` のように JST を明記。日付だけだと UTC 扱いで、毎日 0:10 の自動ビルドに乗らない）
- 配信日: `upcoming` を削除し、`date`・`release_date` を配信日にして `streaming_url` を追加する

### ブログ記事のファイル名規則

記事は並行して作成されるため、作成順と公開順が一致するとは限らない。表示順は frontmatter の `date` が制御するため、ファイル名を連番にする必要はない。

- **新規記事**: 内容を表す英語スラッグを使う（例: `melody-comes-down.md`）。連番は付けない。
- **既存の `post-XX.md` 系ファイル**: 過去記事は URL 変更によるリンク切れを避けるため、リネームしない。
- タイトル内の通し番号（例: `"59. メロディは「降ってくる」のか"`）を付ける場合は、執筆時点ではなく公開確定時に採番すること。

### 未公開情報の扱い（リポジトリはパブリック）

GitHub リポジトリは公開されているため、サイトに表示されないファイル（`draft: true` の記事、未来日付の記事、コメント内のメモ）もリポジトリ上では誰でも読める。一度プッシュした内容は削除しても履歴に残る。

- 下書き記事はコミットせず、ローカルにだけ置く。公開日の分だけをコミットする（`git add .` / `git add -A` は使わず、ファイルを指定して add する）
- 配信前作品（`upcoming: true`）のディスコグラフィーにも、その時点で公表済みの情報だけを書く
- 先出しを避けたい場合は、公開日時（JST 0:00）以降にプッシュする

## GitHub Pages デプロイ

`.github/workflows/hugo.yml` で公式 Hugo GitHub Actions ワークフローを使用。`main` へのプッシュをトリガーにビルド・デプロイ。

`config.toml` の必須設定：
```toml
baseURL = "https://<username>.github.io/<repo>/"
```

リポジトリの Settings > Pages で、ソースを **GitHub Actions** に設定すること（`gh-pages` ブランチではなく）。
