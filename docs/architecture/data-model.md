# data-model.md — データモデル設計

> DB変更前に必ず参照すること。
> マイグレーション方針は `.claude/rules/20-mysql.md` を参照。

## ER図（概要）

```
[users] ──< [user_roles] >── [roles]
   |
   └──< [posts]
           |
           └──< [comments]
```

## テーブル定義

### users

| カラム | 型 | Nullable | デフォルト | 説明 |
|---|---|---|---|---|
| id | bigint | NO | auto | 主キー |
| name | varchar(255) | NO | - | 表示名 |
| email | varchar(255) | NO | - | メールアドレス（unique） |
| email_verified_at | timestamp | YES | null | メール確認日時 |
| password | varchar(255) | NO | - | ハッシュ化パスワード |
| created_at | timestamp | NO | now() | 作成日時 |
| updated_at | timestamp | NO | now() | 更新日時 |
| deleted_at | timestamp | YES | null | 論理削除日時 |

**Index**: `email` (unique)

---

### [table_name]

| カラム | 型 | Nullable | デフォルト | 説明 |
|---|---|---|---|---|
| id | bigint | NO | auto | 主キー |
| [column] | [type] | NO/YES | [default] | [説明] |
| created_at | timestamp | NO | now() | 作成日時 |
| updated_at | timestamp | NO | now() | 更新日時 |

**Index**: [インデックス定義]
**FK**: `[column]` → `[referenced_table](id)`

---

## 設計方針

- 主キーは `bigint` (auto increment)
- タイムスタンプは `timestamp`（文字コードは utf8mb4 を使用）
- 論理削除が必要なテーブルは `deleted_at` を追加
- 金額は `decimal(15, 2)`（float禁止）
- 外部キーには必ずインデックスを作成

## 変更履歴

| 日付 | 変更内容 | ADR |
|---|---|---|
| YYYY-MM-DD | 初版作成 | - |
