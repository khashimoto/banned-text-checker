# NGワードチェッカー - AI向け仕様メモ

## 概要

設定単語（デフォルト + ユーザー追加）で入力テキストを部分一致＋正規化でチェックする、GitHub Pages 単一HTMLのツール。

## 技術スタック

- HTML + Tailwind CSS（CDN）+ 純 JavaScript（フレームワークなし・ビルド不要）
- 構成: `public/index.html` 1ファイル完結（公開ディレクトリ `public/` 配下）

## ファイル構成

- `public/index.html` - アプリ本体（HTML + Tailwind + 全JSロジック・公開対象）
- `public/` - GitHub Pages の公開ディレクトリ（本ファイル配下を公開）
- `test.html` - ユニットテスト（ブラウザで開いて `console.assert` + 画面集計・noindex）
- `README.md` - 非エンジニア向け手順
- `docs/superpowers/specs/` - 設計書
- `docs/superpowers/plans/` - 計画書

## モジュール（関数）構成

| 関数 | 役割 |
|------|------|
| `normalize(str)` | 小文字化 + NFKC正規化（全角→半角）+ ひらがな→カタカナ |
| `checkText(text, words)` | 部分一致・非重複スキップで出現回数をカウント → Match[] |
| `getAllWords()` | デフォルト + ユーザー単語を統合（正規化後ベースで重複排除） |
| `loadUserWords()` / `saveUserWords(words)` | localStorage の読み書き（キー: `ngwordchecker.userWords`） |
| `renderHighlight(text, words)` | 元テキスト走査方式で本文ハイライト描画（XSSエスケープ付き） |
| `renderResultList(matches)` | ヒット一覧描画 |
| `renderWordManager()` | デフォルト/ユーザー単語リスト描画 |
| `escapeHtml(str)` | XSS対策（`< > & " '` を実体参照化） |

## 正規化ロジックの要点

1. `toLowerCase()` で小文字化
2. `normalize('NFKC')` で全角→半角・濁点結合を正規化
3. ひらがな(U+3041-3096) に +0x60 してカタカナ化

## テスト方法

`test.html` をブラウザで開く。サマリーに「全 N 件 / 合格 N / 失敗 0」と表示されれば全合格。
各テストケース前に `localStorage.clear()` する前提。
現在31テストケース（escapeHtml 7 / normalize 8 / checkText 7 / localStorage 5 / getAllWords 4）。

## デプロイ

GitHub Pages で `public/` ディレクトリを公開対象とする。公開設定は GitHub Web UI で実施（`public/` を指定可能な公開方法を選択）。コード変更後は `git push origin main` のみで公開ファイルが更新される。

※ `public/` を公開ディレクトリとして指定するには、現状は GitHub Actions（カスタムワークフロー）を使う必要がある（branch source の場合は `/`（root）か `/docs` の2択のみ）。ワークフロー作成は別タスク（必要時に実施）。

## 変更履歴

- 2026-07-08: 初版作成（v1.0.0）
- 2026-07-08: index.html を `public/` ディレクトリへ移動（GitHub Pages 公開ディレクトリ整備）
