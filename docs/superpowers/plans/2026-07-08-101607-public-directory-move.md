# public ディレクトリ移動 実装計画書

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** NGワードチェッカーの公開ファイル `index.html` を新設した `public/` ディレクトリへ移動し、README/CLAUDE.md のパス記載を整合させる。

**Architecture:** 既存プロジェクトの構成変更（リファクタリング相当・機能不変）。`git mv` で `index.html` を `public/` へ移動し、ドキュメントのパス表記・デプロイ記載を修正する。`index.html` は外部参照が Tailwind CDN のみでローカル相対パス参照を持たないため、ファイル内修正は不要。GitHub Pages の公開設定・GitHub Actions ワークフロー作成は本計画の対象外（後日ユーザーが実施）。

**Tech Stack:** HTML + Tailwind CSS（CDN）+ 純 JavaScript（変更なし）。ホスティング: GitHub Pages（本計画では構成整備のみ）。

## Global Constraints

- **コミットメッセージは日本語**・接頭辞付き（`追加:` `修正:` `整理:` `計画:` `文書:`）。プロジェクトルール（preferences.md）準拠。
- 機種依存文字・中国語表現は使わない。自然な日本語・正書法を守る。
- **機能を変更しない**: 本計画はディレクトリ移動とドキュメント更新のみ。`index.html`・`test.html` のロジック（JS）には一切手を入れない。
- **`test.html` はルートに残留**させる（`public/` へは移動しない）。
- **GitHub Actions ワークフロー（`.github/workflows/*.yml`）は作成しない**（YAGNI・公開設定は後日ユーザー実施）。
- **一括置換後は必ず `grep -rn` で全リポジトリスキャン**し、取りこぼし0を確認する（workflow-rules.md準拠）。
- **完了報告前には必ず検証を実施**し、結果（証拠）を提示する。

---

### Task 1: index.html を public/ へ移動

**Files:**
- Move: `index.html` → `public/index.html`

**Interfaces:**
- Consumes: なし（最初のタスク）
- Produces: `public/index.html` の存在。後続タスク（Task 2/3）はこの移動を前提にドキュメントのパス表記を修正する。

- [ ] **Step 1: 移動前に index.html 内のローカル相対パス参照がないことを再確認**

Run:
```bash
grep -nE '(src="|href=")([^h]|h[^t]|ht[^t]|htt[^p])' index.html
```
Expected: 何も出力されないこと（`http`/`https` で始まらないローカル相対パス参照 = ゼロ）。Tailwind CDN（`https://cdn.tailwindcss.com`）だけが引っかからず存在することは別途確認済み。

念のため CDN 参照が残っていることも確認:
```bash
grep -n 'cdn.tailwindcss.com' index.html
```
Expected: `<script src="https://cdn.tailwindcss.com"></script>` の1行が出力されること。

- [ ] **Step 2: git mv で public/ へ移動**

Run:
```bash
git mv index.html public/index.html
```
Expected: エラーなく完了（`public/` ディレクトリは自動作成される）。

- [ ] **Step 3: 移動結果を確認**

Run:
```bash
ls -la public/index.html && echo "---ルート確認---" && ls -la index.html 2>/dev/null || echo "ルートの index.html は削除済み（OK）"
```
Expected:
- `public/index.html` が存在する（14KB程度・`index.html` と同じサイズ）
- 「ルートの index.html は削除済み（OK）」が出力される

- [ ] **Step 4: git status で移動が staged されていることを確認**

Run:
```bash
git status --short
```
Expected: `R  index.html -> public/index.html`（リネームとして認識されていること）。

- [ ] **Step 5: コミット**

```bash
git commit -m "整理: index.html を public/ ディレクトリへ移動"
```

---

### Task 2: CLAUDE.md のパス表記・デプロイ記載を修正

**Files:**
- Modify: `CLAUDE.md`（「ファイル構成」セクションの index.html パス / 「デプロイ」セクションの Source 記載）

**Interfaces:**
- Consumes: Task 1 で `public/index.html` が存在すること。
- Produces: CLAUDE.md の記載が実態（`public/index.html`）と整合した状態。

- [ ] **Step 1: 技術スタック セクションの構成記載を修正**

`CLAUDE.md` の10行目付近:
```
- 構成: `index.html` 1ファイル完結
```
を以下へ変更:
```
- 構成: `public/index.html` 1ファイル完結（公開ディレクトリ `public/` 配下）
```

- [ ] **Step 2: ファイル構成セクションの index.html パスを修正**

`CLAUDE.md` の14行目付近:
```
- `index.html` - アプリ本体（HTML + Tailwind + 全JSロジック）
```
を以下へ変更:
```
- `public/index.html` - アプリ本体（HTML + Tailwind + 全JSロジック・公開対象）
- `public/` - GitHub Pages の公開ディレクトリ（本ファイル配下を公開）
```

- [ ] **Step 3: デプロイセクションを修正**

`CLAUDE.md` の47行目付近:
```
## デプロイ

GitHub Pages（Source: main ブランチ / ルート）。`git push origin main` のみ。
```
を以下へ変更:
```
## デプロイ

GitHub Pages で `public/` ディレクトリを公開対象とする。公開設定は GitHub Web UI で実施（`public/` を指定可能な公開方法を選択）。コード変更後は `git push origin main` のみで公開ファイルが更新される。

※ `public/` を公開ディレクトリとして指定するには、現状は GitHub Actions（カスタムワークフロー）を使う必要がある（branch source の場合は `/`（root）か `/docs` の2択のみ）。ワークフロー作成は別タスク（必要時に実施）。
```

- [ ] **Step 4: 変更履歴セクションへ追記**

`CLAUDE.md` の変更履歴（末尾）:
```
- 2026-07-08: 初版作成（v1.0.0）
```
の下へ1行追記:
```
- 2026-07-08: index.html を `public/` ディレクトリへ移動（GitHub Pages 公開ディレクトリ整備）
```

- [ ] **Step 5: CLAUDE.md 内に古いパス表記（`index.html` 単体参照）が残っていないか確認**

Run:
```bash
grep -nE '`?index\.html`?' CLAUDE.md
```
Expected: 出力される `index.html` はすべて `public/index.html` の形（先頭に `public/` が付いている）であること。裸の `index.html` が残っていれば Step 1-4 の修正漏れなので直す。

- [ ] **Step 6: コミット**

```bash
git add CLAUDE.md
git commit -m "文書: CLAUDE.md のパス表記とデプロイ記載を public/ 構成へ更新"
```

---

### Task 3: README.md に GitHub Pages 公開手順を追記

**Files:**
- Modify: `README.md`（GitHub Pages 公開手順の追記・ローカルパス表記の整合確認）

**Interfaces:**
- Consumes: Task 1 で `public/index.html` が存在すること。
- Produces: README.md に非エンジニア向け公開手順が記載された状態。

**注意:** README.md の現状は「公開されたURLにブラウザでアクセスします」という抽象的な記述のみで、`index.html` というローカルパス表記は**含まれていない**。したがって修正ではなく「GitHub Pages 公開手順」の追記が主目的。

- [ ] **Step 1: README.md に index.html というローカルパス表記がないことを確認**

Run:
```bash
grep -n 'index.html' README.md
```
Expected: 何も出力されないこと（出力された場合は Task 2 と同様に `public/index.html` へ修正）。

- [ ] **Step 2: README.md 末尾に GitHub Pages 公開手順セクションを追記**

`README.md` の末尾（「動作環境」セクションの後）へ以下を追記:
```markdown

## 公開について（GitHub Pages）

このツールは GitHub Pages で公開しています。公開ファイルは `public/index.html` です。

公開設定（初回のみ・リポジトリ管理者が実施）:
1. GitHub のリポジトリページで「Settings」→「Pages」を開く
2. 公開方法で `public/` ディレクトリを公開対象に指定する
3. 設定後、数分で `https://<アカウント名>.github.io/<リポジトリ名>/` でアクセス可能になる

※ 公開ファイル（`public/` 配下）を更新したら、`main` ブランチへ反映（push）するだけで公開サイトも自動更新されます。
```

- [ ] **Step 3: 追記内容を確認**

Run:
```bash
tail -20 README.md
```
Expected: Step 2 で追記した「公開について（GitHub Pages）」セクションが表示されること。

- [ ] **Step 4: コミット**

```bash
git add README.md
git commit -m "文書: README に GitHub Pages 公開手順を追記"
```

---

### Task 4: リグレッション検証（移動による影響がないことの確認）

**Files:**
- 検証対象: `public/index.html`, `test.html`

**Interfaces:**
- Consumes: Task 1-3 の完了（移動＋ドキュメント更新済み）。
- Produces: 移動が機能に影響していないことの証拠。

**目的:** `index.html` の移動がロジック・テストに影響していないことを確認する。`test.html` は `index.html` と独立して動作する（設計書の調査結果通り）が、念のため実行確認する。

- [ ] **Step 1: public/index.html のファイルサイズ・内容が移動前と同一であることを確認**

Run:
```bash
wc -c public/index.html && git log --oneline -1 -- public/index.html
```
Expected:
- ファイルサイズが 14622 バイト（移動前と同一）であること
- ログに移動コミットが表示されること

- [ ] **Step 2: test.html で31件全合格を確認（移動の影響がないことの検証）**

`test.html` は `index.html` への依存を持たない（設計書の調査結果）。本タスクでは `test.html` 自体を変更していないため、移動前と同じく全合格するはずである。

検証方法は環境に応じていずれかを選ぶ:

**方法A（ブラウザ・確実）**: `test.html` をブラウザで開き、画面のサマリーを確認。
- Expected: 「全 31 件 / 合格 31 / 失敗 0」と表示されること。

**方法B（Node・CI向き）**: 前回セッションで確立した手順（DOM スタブ付きで test.html のテスト関数を抽出して実行）を用いる。手順は `~/ai-knowledge/.last-summary` の記録を参照。
- Expected: 31件全合格。

失敗があった場合は、それが移動によるものか（本来起きないはず）・環境差かを切り分けて報告する。

- [ ] **Step 3: 全リポジトリで index.html への参照をスキャンし、取りこぼし0を確認**

Run:
```bash
grep -rn 'index\.html' --include='*.md' --include='*.html' --include='*.js' --include='*.json' . | grep -v 'public/index.html' | grep -v '^\./docs/superpowers/'
```
Expected:
- 設計書・計画書（`docs/superpowers/`）は除く（それらは歴史的記録として `index.html` を含む）
- 残りの出力は、意図的に `public/index.html` を除いたものだけ。裸の `index.html` 参照（`public/` 無し）があれば修正漏れなので確認する。

ただし `test.html` 内・その他の構成ファイル内に `index.html` への参照がある場合は、それが意図的か確認する（test.html は `index.html` に依存しない設計なので、参照がある場合は本来存在しないはず）。

- [ ] **Step 4: git status がクリーンであることを確認**

Run:
```bash
git status --short
```
Expected: 何も出力されないこと（全コミット済み）。

- [ ] **Step 5: 検証結果を報告**

実装者は以下を報告する:
- `public/index.html` のファイルサイズ（移動前と同一か）
- test.html のテスト結果（全31件合格か）
- `grep` スキャンの結果（取りこぼしなしか）
- `git status` がクリーンか

※ このタスクは検証のみのため、コミットは発生しない（変更がない場合）。何らかの修正が発生した場合のみコミットする。

---

## 完了後の運用（計画対象外・参考）

以下は本計画の対象外だが、完了後にユーザーが実施する事項:

- **GitHub Pages 公開設定**: GitHub Web UI で `public/` を公開ディレクトリとして指定。必要に応じて GitHub Actions ワークフローを作成（別セッション）。
- **main ブランチへ push**: 作業の初めと終わりに `git push origin main`（workflow-rules.md準拠）。
- **tag 作成**: main マージ後に次の minor（v1.1.0）を自動作成（tag運用ルール準拠）。
