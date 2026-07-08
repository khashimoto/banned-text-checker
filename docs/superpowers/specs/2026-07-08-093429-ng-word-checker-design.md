# NGワードチェッカー 設計書

- 作成日: 2026-07-08
- ステータス: 承認済み（設計フェーズ完了）

## 1. 概要

### 目的

予め設定した単語（デフォルト単語）と、ユーザー自身が追加する単語（localStorage 保存）を統合し、ユーザーが入力したテキストをチェックする。設定単語に合致した場合、ヒット一覧と本文ハイライトでアラートを表示する。

社内10名未満・非エンジニアを想定した、GitHub Pages で動作する軽量なチェックツール。

### 要件サマリ

| 項目 | 内容 |
|------|------|
| 目的 | 設定単語（デフォルト + ユーザー追加）で入力テキストをチェック → ヒット時にアラート |
| 入力方式 | textarea に貼り付けて「チェック」ボタンで実行 |
| 判定方式 | 部分一致 + 正規化（大小区別・全角半角・ひらがなカタカナを吸収） |
| デフォルト単語ソース | JavaScript 配列にハードコード（`DEFAULT_WORDS`） |
| ユーザー追加単語 | localStorage に保存（ブラウザ毎・サーバー非共有） |
| 単語管理UI | メイン画面に常時表示（追加欄 + リスト + 削除ボタン） |
| アラート表示 | ヒット一覧表示 + 別エリアの本文ハイライト |
| チェック範囲 | textarea 全文 |
| エクスポート | 不要（画面確認のみ） |

### 技術スタック

- HTML + Tailwind CSS（CDN 読込）+ 純 JavaScript（フレームワークなし）
- ホスティング: GitHub Pages
- 構成: 単一 HTML ファイル（`index.html` 完結）

## 2. 全体アーキテクチャ

### ファイル構成

```
index.html （1ファイル完結）
├── <head>: Tailwind CSS (CDN読込)
├── <body> UI レイアウト
│   ├── ヘッダー（タイトル・簡単な説明）
│   ├── 左ペイン: 入力エリア
│   │   ├── textarea（チェック対象テキスト入力）
│   │   └── 「チェック」ボタン / 「クリア」ボタン
│   ├── 右ペイン: 結果エリア
│   │   ├── ヒット一覧（件数 + 単語名 + 出現回数）
│   │   └── ハイライト表示エリア（本文をマークアップして描画）
│   └── 下部ペイン: 単語管理
│       ├── デフォルト単語リスト（読み取り専用・折りたたみ表示）
│       └── ユーザー追加単語（入力欄 + 追加ボタン + 削除×ボタン付きリスト）
└── <script>: アプリロジック（純JS・フレームワークなし）
```

### モジュール（関数）構成

単一ファイル内だが、JSは責務ごとに関数を分離。各関数は単一目的・独立してテスト可能。

| 関数 | 役割 | 入出力 |
|------|------|--------|
| `normalize(str)` | 文字列の正規化（小文字化・全角→半角・ひらがな→カタカナ） | `str → str` |
| `getAllWords()` | デフォルト単語 + localStorage のユーザー単語を統合（重複排除） | `→ string[]` |
| `checkText(text, words)` | 正規化済みテキストに対し各単語で部分一致検索 | `text, words → Match[]` |
| `renderHighlight(text, matches)` | 本文をヒット箇所ハイライト付きで描画 | `text, matches → HTML文字列` |
| `renderResultList(matches)` | ヒット一覧を描画 | `matches → DOM更新` |
| `loadUserWords()` | localStorage からユーザー単語を読込 | `→ string[]` |
| `saveUserWords(words)` | localStorage へユーザー単語を保存 | `string[] → void` |
| `renderWordManager()` | 単語管理UIの描画 | `→ DOM更新` |
| `escapeHtml(str)` | XSS対策のHTMLエスケープ | `str → str` |

## 3. データフロー・判定ロジック

### チェック実行フロー

```
[チェック]ボタン押下
  → text = textarea の値
  → words = getAllWords()  // デフォルト配列 ∪ localStorage配列（重複排除）
  → normalizedText = normalize(text)
  → matches = checkText(text, words)
       ※ 各 word について:
          normalizedWord = normalize(word)
          if normalizedWord === '' → skip
          count = normalizedText 中の normalizedWord の出現回数
          if count > 0 → matches.push({word, normalizedWord, count})
  → renderResultList(matches)    // 右ペイン上: 一覧
  → renderHighlight(text, words) // 右ペイン下: 本文ハイライト
```

### 正規化 `normalize(str)` の仕様

1. 小文字化（`toLowerCase`）
2. 全角英数字 → 半角（`String.prototype.normalize("NFKC")` を利用）
3. ひらがな → カタカナに統一（Unicodeコードポイント差分 `0x60` 加算でカタカナ化）

> ※ 実装上の注意: `NFKC` 正規化は全角半角・濁点結合を吸収するが、ひらがな→カタカナ変換はしないため別途処理が必要。この2段構えで「きょう／キョウ」のような表記ゆれを吸収する。漢字の異体字・同義語はスコープ外。

### ハイライト描画 `renderHighlight` の仕様

- 元の `text`（非正規化）を描画しつつ、ヒット箇所を `<mark>` タグで囲む
- **XSS 対策**: ユーザー入力テキストを HTML に挿入するため、必ずエスケープ（`< > & "` を実体参照化）してから `<mark>` を挿入する
- 処理順序: 先に全体をエスケープ → 正規化テキストのインデックスに基づき対応範囲を `<mark>` で囲む
- 文字数が変わりうる NFKC 正規化（例: `ﾞ` 結合）に配慮し、正規化前後の対応を追跡する

### データ型

```js
Match = {
  word: string,           // 設定単語の元の表記（一覧表示用）
  normalizedWord: string, // 正規化後の単語
  count: number           // 出現回数
}
```

### エッジケース

| ケース | 扱い |
|--------|------|
| textarea 空 | 「テキストを入力してください」表示・チェックしない |
| 単語リスト空 | 「チェック単語がありません」表示 |
| 重複単語（デフォルトとユーザーで同一） | 正規化後の単語で重複排除 |
| 空文字・空白のみの単語 | 追加時に弾く（trim後空は拒否） |

## 4. localStorage・単語管理・エラー処理

### localStorage のデータ構造

```js
キー: "ngwordchecker.userWords"
値: JSON文字列の配列  例: '["社外秘","2026年度予定"]'
```

### 単語管理UI の振る舞い

| 操作 | 振る舞い |
|------|----------|
| ユーザー単語 追加 | 入力欄 → 追加ボタン（Enter でも可）→ trim → 空なら拒否 → 既存と重複（正規化後で比較）なら「既に追加済み」表示 → localStorage に追記 → リスト再描画 |
| ユーザー単語 削除 | リスト各項目の × ボタン → localStorage から除去 → リスト再描画 |
| デフォルト単語 | 読み取り専用で折りたたみ表示（削除不可）。ユーザーは自身の追加単語のみ編集可 |
| 初回訪問 | localStorage 未設定なら空配列として扱う（エラーにしない） |

### エラー処理

| ケース | 対処 |
|--------|------|
| localStorage 利用不可（プライベートモード・容量超過・無効化） | try-catch で捕捉 → 画面上部にバナー表示「保存機能が使えません。単語はこのセッション中のみ有効です」→ メモリ上で動作継続（チェック機能は使える） |
| localStorage の値が破損（不正JSON） | `JSON.parse` 失敗時は空配列として扱い、破損値をクリア |
| ブラウザ非対応チェック | `<script>` 先頭で `window.localStorage` と `JSON` の存在確認 |

### アクセシビリティ・非エンジニア配慮

- 全ボタンに分かりやすい日本語ラベル（「チェック」「クリア」「追加」「削除」）
- 結果0件の時は「チェック単語に合致する箇所はありませんでした」と安心メッセージ
- 初回表示時に使い方のヒントを薄く表示
- レスポンシブ: モバイルは縦並び、PCは横並び（Tailwind の `md:` ブレイクポイント）

## 5. テスト戦略

### 層1: ロジック層のユニットテスト

純粋関数（`normalize`, `getAllWords`, `checkText`, `loadUserWords`）は入出力が明確。Node環境を前提としない運用のため、軽量方式を採用。

- テスト用 HTML（`test.html`、`index.html` とは別・GitHub Pages には配置しないか noindex 運用）を用意
- ブラウザで開いて `console.assert` で検証
- Node 不要・本番に影響しない

### テストケース（正規化ロジック）

```
normalize("ＡＢＣ")     === "abc"        // 全角→半角・小文字化
normalize("キーワード")  === "キーワード"   // カタカナはそのまま
normalize("きーワード")  === "キーワード"   // ひらがな→カタカナ
normalize("Test")      === "test"
```

### テストケース（マッチング）

```
checkText("機密資料を送付", ["機密"])   → 1件ヒット（部分一致）
checkText("キーワード", ["きーわーど"])  → 1件ヒット（正規化一致）
checkText("安全な文章", ["機密"])       → 0件
checkText("", ["機密"])                → 0件（空テキスト）
```

### 層2: 手動 E2E 検証（実機確認）

検証観点のチェックリスト:

- [ ] GitHub Pages 上でデプロイ後、textarea → チェック → ハイライト表示が動く
- [ ] ユーザー単語の追加・削除が localStorage に永続化される（リロード後も残る）
- [ ] プライベートモードで開いてもチェック機能は動く（バナー表示）
- [ ] XSS: `<script>alert(1)</script>` を含むテキストを入れても実行されない
- [ ] モバイル幅で縦並びになる

### 検証コマンド

- テストHTMLをブラウザで開き `console.assert` 結果を確認
- 手動チェックリストを実施
- 完了報告前には必ずこれらを実行し結果を提示する

## 6. デプロイ・運用

### GitHub Pages デプロイ

```
リポジトリ構成:
  index.html      （アプリ本体）
  test.html       （テスト用・公開しない/noindex）
  README.md       （使い方・非エンジニア向け）
  CLAUDE.md       （AI向け仕様メモ）
  docs/superpowers/specs/   （設計書）
  docs/superpowers/plans/   （計画書）
```

- GitHub リポジトリ Settings → Pages → Source: `main` ブランチ / ルート(`/(root)`)
- `index.html` をルートに配置 → `https://<user>.github.io/<repo>/` で公開
- デプロイは `git push origin main` のみ（CI 不要・ビルド不要）

### デフォルト単語の運用フロー

- デフォルト単語は `index.html` 内の JS 配列 `DEFAULT_WORDS` として管理
- 単語追加・変更時は配列を編集 → commit → push で反映（数分で全員に反映）
- 各ユーザーの追加単語は各端末の localStorage（サーバー非共有・プライバシー配慮）

### ドキュメント整備

- README.md: 非エンジニア向けの「URLにアクセス → テキスト貼り付け → チェック」手順
- CLAUDE.md: AI 向けの仕様メモ（正規化ロジック・モジュール構成・テスト方法）を常に最新化

### Git/tag 運用

- feature ブランチ → main マージ後、tag 自動作成（`v1.0.0` から開始、minor を +1）
- マージ都度 tag を打ち全履歴を残す運用
