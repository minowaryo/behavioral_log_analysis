# 30-testing.md — テスト方針

## テスト優先順位

1. **Feature Test（最優先）**: HTTPリクエスト〜レスポンスの統合テスト
2. **Unit Test**: 複雑なビジネスロジック・計算ロジック
3. **E2E Test**: クリティカルなユーザーフロー

## テスト作成前に読むファイル

- `docs/product/use-cases.md` — テストケース名・網羅範囲の導出元（UCが正常系/異常系/権限の基準）
- `docs/architecture/data-model.md` — DBアサーション・Factoryの型・制約の確認
- `docs/product/mockups/` — E2Eテストの画面構造・操作フローの確認（存在する場合）

## 基本ルール

- 変更には必ずFeature Testを追加する
- バグ修正時は再発防止テストを先に書く（TDD）
- テストはDBをモックしない（実DBを使う）
- Factoryを活用してテストデータを生成する
- テストケース名は `use-cases.md` のUCタイトル・フローを基に命名する

## 命名規則

```php
// Feature Test: 何をテストするか明確に
test('管理者はユーザー一覧を取得できる', function () { ... });
test('一般ユーザーはユーザー一覧にアクセスできない', function () { ... });
test('未認証ユーザーはログインページにリダイレクトされる', function () { ... });
```

## テスト構造（AAA パターン）

```php
test('example', function () {
    // Arrange: テストデータ・前提条件を準備
    $user = User::factory()->create();

    // Act: テスト対象の処理を実行
    $response = $this->actingAs($user)->get('/dashboard');

    // Assert: 期待する結果を検証
    $response->assertOk();
});
```

## 必ずテストすること

- [ ] 正常系（ハッピーパス）
- [ ] 認証・認可（未認証/権限なし）
- [ ] バリデーションエラー
- [ ] 境界値・エッジケース
- [ ] 削除・更新の副作用

## コマンド

```bash
# 全テスト実行
php artisan test

# 特定ファイルのみ
php artisan test tests/Feature/UserTest.php

# カバレッジ確認
php artisan test --coverage
```
