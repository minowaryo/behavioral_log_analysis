# 10-laravel.md — Laravel固有ルール

## アーキテクチャ方針

### Controller
- 薄く保つ（Fat Controller禁止）
- バリデーションは `FormRequest` に委譲
- ビジネスロジックは `Service` / `Action` に委譲
- 直接 `DB::` を呼ばない

### Service / Action
- 1クラス1責務を守る
- `Action` クラスは `execute()` メソッドに処理を集約
- トランザクションはServiceレイヤーで管理

### Model
- `$fillable` を明示する（`$guarded = []` 禁止）
- スコープはModelに定義する
- リレーションは積極的に定義する
- ビジネスロジックをModelに書かない

### Authorization
- **Policy / Gate を必ず使う**（手動チェック禁止）
- `authorize()` をControllerで明示的に呼ぶ
- ロールチェックはMiddlewareまたはPolicyに集約

### FormRequest
- バリデーションルールはFormRequestに書く
- `authorize()` も適切に実装する

## 命名規則

| 対象 | 規則 | 例 |
|---|---|---|
| Controller | PascalCase + Controller | `UserController` |
| Service | PascalCase + Service | `UserRegistrationService` |
| Action | PascalCase + Action | `RegisterUserAction` |
| FormRequest | PascalCase + Request | `StoreUserRequest` |
| Policy | PascalCase + Policy | `UserPolicy` |
| Event | PascalCase（過去形） | `UserRegistered` |
| Job | PascalCase | `SendWelcomeEmail` |

## 禁止事項

- `DB::statement()` での生SQL（必要な場合はADRを書く）
- `$guarded = []`
- Controllerでのビジネスロジック
- N+1クエリ（`with()` で積極的にEager Loading）
