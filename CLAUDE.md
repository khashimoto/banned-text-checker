# NGワードチェッカー - AI向け仕様メモ

## 概要

設定単語（デフォルト + ユーザー追加）で入力テキストを部分一致＋正規化でチェックする、GitHub Pages のツール。

## 技術スタック

- HTML + Tailwind CSS（CDN）+ 純 JavaScript（フレームワークなし・ビルド不要）
- 構成: `index.html`（本体）+ `default-words.js`（デフォルト単語定義）の2ファイル（ルート配置）

## ファイル構成

- `index.html` - アプリ本体（HTML + Tailwind + 全JSロジック・公開対象）
- `default-words.js` - デフォルト単語定義（`window.DEFAULT_WORDS`・460件。正規化後重複排除で実効452件）
- `test.html` - ユニットテスト（ブラウザで開いて `console.assert` + 画面集計・noindex）
- `README.md` - 非エンジニア向け手順
- `docs/superpowers/specs/` - 設計書
- `docs/superpowers/plans/` - 計画書

## モジュール（関数）構成

| 関数 | 役割 |
|------|------|
| `normalize(str)` | 小文字化 + NFKC正規化（全角→半角）+ ひらがな→カタカナ |
| `checkText(text, words)` | 部分一致・非重複スキップで出現回数をカウント → Match[] |
| `getAllWords()` | デフォルト + ユーザー単語を統合（正規化後ベースで重複排除）。`DEFAULT_WORDS`（`default-words.js`）を参照 |
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

GitHub Pages（Source: main ブランチ / ルート）。`git push origin main` のみで公開ファイルが更新される。

## 変更履歴

- 2026-07-08: 初版作成（v1.0.0）
- 2026-07-08: デフォルト単語を外部化（`default-words.js`）。禁止ワード460件を設定（実効452件）。旧デフォルト3件（機密/社外秘/個人情報）は削除
- 2026-07-08: `public/` 構成を廃止しルート配置へ戻す（index.html / default-words.js をルートへ）
