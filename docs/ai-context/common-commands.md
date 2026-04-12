# common-commands.md — よく使うコマンド集

## テスト

```bash
# 全テスト実行
php artisan test

# 特定ファイル
php artisan test tests/Feature/UserTest.php

# カバレッジ付き
php artisan test --coverage

# 並列実行（高速）
php artisan test --parallel
```

## コードスタイル

```bash
# フォーマット（自動修正）
./vendor/bin/pint

# チェックのみ（修正なし）
./vendor/bin/pint --test

# 静的解析
./vendor/bin/phpstan analyse
```

## データベース

```bash
# マイグレーション実行
php artisan migrate

# ロールバック
php artisan migrate:rollback

# DB リセット（開発環境のみ）
php artisan migrate:fresh --seed

# マイグレーション状態確認
php artisan migrate:status
```

## アプリケーション

```bash
# 開発サーバー起動
php artisan serve

# キャッシュクリア
php artisan cache:clear
php artisan config:clear
php artisan route:clear
php artisan view:clear

# キュー起動（開発）
php artisan queue:work

# スケジューラ起動（開発）
php artisan schedule:work
```

## コード生成（Artisan）

```bash
# コントローラ
php artisan make:controller UserController --resource

# モデル + マイグレーション + Factory + Seeder
php artisan make:model User -mfs

# FormRequest
php artisan make:request StoreUserRequest

# Policy
php artisan make:policy UserPolicy --model=User

# Action / Service（カスタム）
php artisan make:class Actions/RegisterUserAction
```

## Composer

```bash
# 依存関係インストール
composer install

# パッケージ追加
composer require [package]

# 脆弱性チェック
composer audit

# オートロード再生成
composer dump-autoload
```
