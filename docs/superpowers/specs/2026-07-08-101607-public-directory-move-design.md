# public ディレクトリ移動 設計書

- 作成日: 2026-07-08
- ステータス: 承認済み（実装待ち）
- 対象リポジトリ: banned-text-checker (k_hashimoto)

## 1. 目的

NGワードチェッカーの公開ファイル（`index.html`）を、GitHub Pages で「公開ディレクトリ」として指定しやすい構成にするため、新設した `public/` ディレクトリへ移動する。

GitHub Pages は公開ディレクトリを指定して公開できる（本セッションでは公開設定自体は行わず、構成の整備のみ実施する）。

## 2. 背景・現状

### 現状のディレクトリ構成（ルート）

```
/var/www/html/
├── index.html        ← ツール本体（公開対象）・ルートにある
├── test.html         ← ユニットテスト・ルートにある
├── README.md
├── CLAUDE.md
└── docs/             ← 設計書・計画書
```

### 相対パス参照の調査結果

- `index.html` の外部参照は **Tailwind CDN のみ**（`https://cdn.tailwindcss.com`）。ローカルファイルへの相対パス参照は**一切なし**。
- `test.html` も `index.html` への参照なし（独立して動作）。

→ `index.html` を `public/` へ移動しても、**ファイル内のパス修正は発生しない**（自己完結）。

## 3. 要件（ユーザー決定事項）

| 項目 | 決定内容 |
|------|----------|
| 移動対象ファイル | `index.html` のみ |
| `test.html` の取り扱い | **ルートに残留**（移動しない） |
| `public/` ディレクトリ | 新規作成 |
| GitHub Pages 公開設定 | **このセッションでは実施しない**（後でユーザーが GitHub Web UI で設定） |
| GitHub Actions ワークフロー | **作成しない**（YAGNI・公開設定時に必要になれば別セッションで対応） |
| コードロジックの変更 | なし（移動のみ） |

## 4. 移動後のディレクトリ構成

```
/var/www/html/
├── public/
│   └── index.html        ← ルートから移動（ツール本体・公開対象）
├── test.html             ← ルートに残留（テスト専用・非公開想定）
├── docs/                 ← 設計書・計画書（そのまま）
├── README.md             ← 更新（パス記載の修正）
└── CLAUDE.md             ← 更新（ファイル構成・デプロイ記載の修正）
```

## 5. 実装ステップ

### ステップ1: index.html の移動

```bash
git mv index.html public/index.html
```

- `public/` ディレクトリは `git mv` で自動作成される。
- `index.html` 内のパス修正は不要（外部参照は CDN のみ）。

### ステップ2: README.md の更新

- 「index.html を開く」系の手順記載があれば `public/index.html` へ修正。
- GitHub Pages の公開手順に「`public/` ディレクトリを公開対象とする」旨を追記。

### ステップ3: CLAUDE.md の更新

- 「ファイル構成」セクション: `index.html` のパスを `public/index.html` へ変更。
- 「デプロイ」セクション: 現在「GitHub Pages（Source: main / root）」となっている記載を、`public/` を公開する方針へ修正。

### ステップ4: 検証

- `git mv` 後、`index.html` が `public/` に存在しルートから消えていることを確認。
- `grep -rn "index.html"` でドキュメント内のパス参照の取りこぼしがないか全件スキャン。
- ドキュメント（README/CLAUDE.md）と実装の整合性を突合。

## 6. Git 運用

- feature ブランチ → main へ直接マージ（develop 不使用）。
- コミットメッセージは日本語（接頭辞付き）。
- main マージ後に tag を自動作成（次の minor: v1.1.0）。

## 7. 実施しないこと（YAGNI）

- `.github/workflows/*.yml` の作成（公開設定は後でユーザー実施のため）。
- `test.html` の移動（ルートに残留）。
- コードロジックの変更。
- GitHub Pages の公開設定（Web UI 操作）。

## 8. 完了前の検証項目

- [ ] `public/index.html` が存在する
- [ ] ルートの `index.html` が消失している
- [ ] `test.html` がルートに残留している
- [ ] `grep -rn "index.html"` の結果と、README/CLAUDE.md の記載が整合している
- [ ] test.html をブラウザ（または Node 実行）で開き、31件のテストが全合格すること（移動による影響がないことの確認）

## 変更履歴

- 2026-07-08: 初版作成
