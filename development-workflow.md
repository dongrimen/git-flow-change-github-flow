# 開発フロー手順書

## 📌 このドキュメントについて

本ドキュメントは、チームの開発ブランチ戦略を定めた手順書です。  
従来の GitFlow から、`release` ブランチを挟む GitHub Flow ベースのフローに移行しています。

> **新旧フローの主な違い**
> - `develop` ブランチが廃止され、`release` ブランチに一本化
> - `feature/` ブランチの派生元が `develop` → `main` に変更
> - すべてのマージは必ずプルリクエスト（PR）経由で行う

---

## 🌿 ブランチ構成

| ブランチ | 役割 | ライフサイクル |
|---------|------|--------------|
| `main` | 本番リリース済みのコード。変更が加わると自動デプロイが実行される | 🔒 常設・直接push禁止 |
| `release` | リリース準備・QA用。複数の修正をここに集約してから `main` へ反映する | 🔒 常設・直接push禁止 |
| `feature/*` | 機能開発・バグ修正用の作業ブランチ | 🔁 都度作成・マージ後削除は任意 |
| `merge/*` | 複数の作業ブランチを集約するための一時ブランチ | 🔁 都度作成・マージ後削除は任意 |
| `hotfix/*` | 本番緊急修正用ブランチ | 🔁 都度作成・マージ後削除 |

> ⚠️ `main` と `release` への直接 `git push` は禁止です。必ずPRを経由してください。

---

## 🟢 パターン 1：通常の開発フロー

機能開発・バグ修正など、日常的な開発作業はこのパターンで行います。

```
main → feature/* → release → main
```

### 手順

#### 1. 作業ブランチを作成する

必ず `main` の最新状態から作業ブランチを作成します。

```bash
git checkout main
git pull origin main
git checkout -b feature/your-feature-name
```

> **ブランチ名のつけ方**  
> `feature/` プレフィックスをつけ、内容がわかる名前にしてください。  
> 例：`feature/add-login-page`、`feature/fix-user-validation`

#### 2. 作業・コミットを行う

```bash
# ファイルを編集後
git add .
git commit -m "feat: ログイン画面を追加"
```

> **コミットメッセージの目安**  
> 「何をしたか」が一目でわかるメッセージを書いてください。  
> 例：`fix: パスワードバリデーションのバグを修正`、`feat: ユーザー一覧APIを追加`

#### 3. リモートにプッシュする

```bash
git push origin feature/your-feature-name
```

#### 4. `release` ブランチへのPRを作成する

GitHub上でPRを作成します。

- **base（マージ先）**：`release`
- **compare（マージ元）**：`feature/your-feature-name`

→ PR作成時の注意点は [PR作成ガイド](#-pr作成ガイド) を参照してください。

#### 5. レビュー・マージ

以下がすべて満たされたタイミングでマージします。

- レビュワーの承認を得ている
- テストが完了している
- リリースすることが確定している

マージ後、作業ブランチの削除は任意です。

#### 6. `release` → `main` のPRはリリース担当者が作成する

リリースのタイミングで担当者が `release → main` のPRを作成してマージします。  
個々の開発者がこのPRを作成する必要はありません。

---

## 🟣 パターン 2：複数ブランチの一括マージ

複数の機能を同一リリースにまとめてデプロイしたい場合や、依存関係のある機能を一緒にリリースしたい場合に使います。

```
main → feature/* → merge/* → release → main
```

### 手順

#### 1. 各作業ブランチを作成・開発する

パターン1の手順1〜2と同様に、それぞれの作業ブランチで開発を進めます。

#### 2. マージブランチを作成してリモートにプッシュする

```bash
git checkout main
git pull origin main
git checkout -b merge/feature-a-and-b
git push origin merge/feature-a-and-b
```

> GitHub上でPRのマージ先として指定するため、先にリモートへプッシュしておく必要があります。

#### 3. 各作業ブランチからマージブランチへのPRを作成する

履歴を残すため、ローカルでの `git merge` は行わず、GitHub上でPRを作成してマージします。

各作業ブランチに対して以下のPRを作成してください。

- **base（マージ先）**：`merge/feature-a-and-b`
- **compare（マージ元）**：各作業ブランチ（`feature/feature-a`、`feature/feature-b` など）

#### 4. `release` へのPRを作成する

GitHub上でPRを作成します。

- **base（マージ先）**：`release`
- **compare（マージ元）**：`merge/feature-a-and-b`

#### 5. 以降はパターン1の手順5〜6と同様

---

## 🔴 パターン 3：緊急修正（Hotfix）

本番環境で緊急のバグが発生した場合に使います。通常のリリースサイクルを待たず、`main` へ直接マージします。

```
main → hotfix/* → ② main → ③ release
```

> ⚠️ `main` へのマージを先に行い、その後 `release` にも適用します。`release` には次回リリース予定の修正がすでに含まれているため、動作確認の起点は `main` になります。

### 手順

#### 1. `main` から hotfix ブランチを作成する

```bash
git checkout main
git pull origin main
git checkout -b hotfix/fix-critical-bug
```

#### 2. 修正・コミットを行う

```bash
git add .
git commit -m "hotfix: 決済処理のクリティカルなバグを修正"
```

#### 3. リモートにプッシュする

```bash
git push origin hotfix/fix-critical-bug
```

#### 4. `main` へのPRを作成してマージする（優先）

- **base（マージ先）**：`main`
- **compare（マージ元）**：`hotfix/fix-critical-bug`

レビューとテストを経てマージし、本番への緊急リリースを実行します。

#### 5. `release` へのPRも作成してマージする

- **base（マージ先）**：`release`
- **compare（マージ元）**：`hotfix/fix-critical-bug`

次回リリース時に同じバグがリグレッションしないよう、`release` にも適用します。  
コンフリクトが発生した場合は [コンフリクト解消手順](#-コンフリクト解消手順) を参照してください。

#### 6. hotfix ブランチを削除する

両方のPRがマージされたら、hotfix ブランチを削除してください。

---

## ⚠️ コンフリクト解消手順

PRにコンフリクトが発生した場合は、以下のいずれかの方法で解消します。

### 方法 A：ローカルで解消する（推奨）

変更箇所が多い・複雑な競合の場合はこちらを推奨します。IDEの差分ツールを使いながら丁寧に解消できます。

```bash
# 作業ブランチに移動
git checkout feature/your-feature-name

# マージ先ブランチ（releaseなど）を取り込む
git fetch origin
git merge origin/release

# コンフリクトを解消する
# エディタやIDEでコンフリクト箇所（<<<<<<< HEAD など）を修正

# 解消後にコミット
git add .
git commit -m "fix: コンフリクトを解消"

# プッシュ（PRが自動的に更新される）
git push origin feature/your-feature-name
```

### 方法 B：GitHub上で解消する

変更箇所が少なく単純な競合の場合は、GitHubのPR画面上の「Resolve conflicts」ボタンから解消できます。

> ⚠️ 複雑な競合をブラウザ上で解消するとミスが起きやすいため、迷ったら方法Aを選んでください。

---


