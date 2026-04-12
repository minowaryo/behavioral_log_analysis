# coding-standards.md — コーディング規約

> 詳細なLaravelルールは `.claude/rules/10-laravel.md` を参照。

## 基本方針

- PSR-12 準拠
- Laravel Pint でフォーマット（CI必須）
- PHPStan Level 6 以上をクリア

## PHP

```php
<?php

declare(strict_types=1);

namespace App\Services;

use App\Models\User;

final class RegisterUserService
{
    public function execute(array $data): User
    {
        // 実装
    }
}
```

### 命名規則

| 対象 | 規則 | 例 |
|---|---|---|
| クラス名 | PascalCase | `UserService` |
| メソッド名 | camelCase | `findById()` |
| 変数名 | camelCase | `$userId` |
| 定数 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| DBカラム | snake_case | `created_at` |

### 型宣言

- `declare(strict_types=1)` を全ファイルに付ける
- 引数・戻り値の型宣言を必ず書く
- nullable は `?Type` で表現（`mixed` 乱用禁止）

## コメント

- 自明なコードにコメントを書かない
- **なぜ** そう書いたかをコメントする（**何を** ではなく）
- PHPDoc は public API にのみ書く

```php
// OK: なぜを説明している
// MySQLのROW LOCK を使うのは、同一ユーザーの並行予約を防ぐため
$booking->lockForUpdate()->find($id);

// NG: 何をするか（コードを読めばわかる）
// ユーザーを取得する
$user = User::find($id);
```

## Git

- コミットメッセージは日本語または英語で意図を書く
- 1コミット1変更（混在させない）
- PRは小さく保つ（レビューしやすいサイズ）

### コミットメッセージ形式

```
[type]: [変更の概要]

[必要なら詳細説明]
```

type: `feat` / `fix` / `refactor` / `test` / `docs` / `chore`

例:
```
feat: ユーザー一覧APIにページネーションを追加

無限スクロール対応のため cursor-based pagination を実装。
offset-based から変更した理由は ADR-0005 を参照。
```
