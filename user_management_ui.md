# ユーザー管理画面仕様（User Management UI）

- Status: ACTIVE
- Version: v1.0
- Last Updated: 2025-12-16
- Owner: milmuch / ChatGPT
- Target: Ruby on Rails 7.x
- Depends On:
  - Login UI v1.0
  - Authentication & Ownership v1.0（users/roles/has_secure_password）

---

## 1. 目的
- admin が営業アカウントを作成・編集・無効化できるようにする
- 担当者別の提案・進捗・目標管理の前提となる
- 運用上「退職者を消さずに無効化」できるようにする

---

## 2. 権限
- admin：閲覧/作成/編集/無効化 すべて可能
- sales：アクセス不可（403 or リダイレクト）

---

## 3. 画面一覧
- UM-01：ユーザー一覧（管理者）
- UM-02：ユーザー新規作成
- UM-03：ユーザー編集（名前/権限/有効無効）
- UM-04：パスワード再設定（adminによるリセット）

---

## 4. UM-01 ユーザー一覧

### 4.1 URL
- GET `/admin/users`

### 4.2 一覧表示項目（必須）
- 名前（name）
- メール（email）
- 権限（role：admin / sales）
- 状態（active：有効/無効）
- 最終更新日（updated_at）
- 操作：[編集]

### 4.3 フィルタ（MVP）
- キーワード検索（name/email 部分一致）
- 状態（有効のみ / 無効のみ / 全て）
- 権限（admin / sales / 全て）

### 4.4 操作
- 右上：[新規作成]

---

## 5. UM-02 ユーザー新規作成

### 5.1 URL
- GET `/admin/users/new`
- POST `/admin/users`

### 5.2 入力項目（必須）
- name（表示名）
- email（ログインID）
- role（初期は sales 推奨）
- password（初期パスワード）
- password_confirmation

### 5.3 バリデーション
- email：必須、形式チェック、ユニーク
- password：必須、8文字以上（MVP）
- role：admin/sales のみ
- active：作成時は true 固定

### 5.4 成功時
- ユーザー一覧へ戻す
- フラッシュ：`ユーザーを作成しました`

### 5.5 運用ルール（重要）
- 初期パスワードは admin が発行し、別経路で本人に渡す
- 初回ログイン後の変更は v2（MVPでは不要）

---

## 6. UM-03 ユーザー編集

### 6.1 URL
- GET `/admin/users/:id/edit`
- PATCH `/admin/users/:id`

### 6.2 編集可能項目
- name
- role（sales ⇄ admin）
- active（有効/無効）

### 6.3 禁止ルール（事故防止）
- 最後の admin を sales に変更できない（adminが0人になるのを防ぐ）
- 自分自身を無効化できない（任意）

### 6.4 無効化の意味
- active=false のユーザーはログイン不可
- ただし proposals 等の履歴は残る（監査・集計のため）

---

## 7. UM-04 パスワード再設定（adminリセット）

### 7.1 URL
- GET `/admin/users/:id/password`
- PATCH `/admin/users/:id/password`

### 7.2 入力項目
- new_password
- new_password_confirmation

### 7.3 仕様（MVP）
- admin が本人確認した上でリセット
- リセット後は本人へ別経路で伝達

※ 「本人が自分で忘れた」リセット機能（メール送信）は v2

---

## 8. ナビゲーション
- admin のみヘッダーに `管理` メニューを表示
  - `管理 > ユーザー`
  - `管理 > 辞書`
  - `管理 > マージン（全体）`
  - `管理 > 追跡アラート`
  - `管理 > 営業ダッシュボード`

---

## 9. 受入基準（Acceptance Criteria）
- admin がユーザーを作成できる
- sales は /admin/users にアクセスできない
- admin がユーザーを無効化でき、無効ユーザーはログインできない
- 最後の admin を消せない（0人防止）
- パスワードを admin がリセットできる

---

## 10. 非スコープ（v2）
- パスワードリセットメール
- 初回ログインでの強制パスワード変更
- 2FA / SSO

---