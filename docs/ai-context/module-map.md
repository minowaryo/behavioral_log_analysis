# module-map.md — モジュール・ディレクトリ担当一覧

## Backend (Laravel)

| パス | 役割 | 注意 |
|---|---|---|
| `app/Http/Controllers/` | HTTPリクエスト受付・レスポンス返却 | Fat Controller禁止 |
| `app/Http/Requests/` | バリデーションルール | FormRequestを必ず使う |
| `app/Http/Middleware/` | 認証・認可・ロギング | グローバル適用は慎重に |
| `app/Services/VoiceDiaryService.php` | 音声ファイルアップロード・Whisper API 呼び出し・文字起こし結果保存 | Whisper API キーは .env で管理 |
| `app/Services/YouTubeService.php` | Google OAuth2 フロー・YouTube Data API v3 で視聴履歴取得 | OAuth トークンは YouTubeToken モデルで管理 |
| `app/Services/AnalysisService.php` | Claude API 呼び出し・6観点分析レポート生成・保存 | Claude API キーは .env で管理 |
| `app/Services/` (その他) | その他ビジネスロジック | 1クラス1責務 |
| `app/Actions/` | 単一操作のアクション | `execute()` メソッドに集約 |
| `app/Models/` | Eloquentモデル・リレーション | ビジネスロジックを書かない |
| `app/Policies/` | 認可ルール | 必ずGate経由で呼ぶ |
| `app/Events/` | ドメインイベント | 過去形の名前 |
| `app/Listeners/` | イベントハンドラ | 重い処理はQueueに |
| `app/Jobs/ScheduledAnalysisJob.php` | 週次・2ヶ月ごとの自動分析を実行するジョブ | Horizon経由で実行 |

## Frontend (Vue.js + Inertia.js + Tailwind CSS)

| パス | 役割 | 注意 |
|---|---|---|
| `resources/js/Pages/Diary/` | 音声日記UI（録音・一覧・詳細） | モバイルファーストで設計 |
| `resources/js/Pages/Analysis/` | 分析結果表示UI（6観点のレポート閲覧） | - |
| `resources/js/Pages/Dashboard/` | ダッシュボード（ログ概要・最近の分析） | - |
| `resources/js/Pages/Settings/` | YouTube OAuth 連携設定・アカウント管理 | - |
| `resources/js/Components/` | 汎用・共通コンポーネント（録音ボタン等） | 単一責務・再利用可能に保つ |
| `resources/js/Composables/` | ロジック分離用 Composable（`use~` 命名） | 録音・API呼び出し等のロジックを集約 |
| `resources/js/stores/` | Pinia Store（`useXxxStore` 命名） | ローカル状態は入れない |
| `resources/js/app.js` | エントリポイント | Inertia の初期化処理 |

## Database

| パス | 役割 |
|---|---|
| `database/migrations/` | スキーマ変更履歴（実行済みファイルは変更禁止） |
| `database/factories/` | テスト用データ生成 |
| `database/seeders/` | 初期データ投入 |

## Tests

| パス | 役割 |
|---|---|
| `tests/Feature/` | 統合テスト（最優先）|
| `tests/Unit/` | 単体テスト（ビジネスロジック） |

## Docs

| パス | 役割 | 更新者 |
|---|---|---|
| `docs/ai-context/` | AI向け要約（短く正確に） | 開発者 |
| `docs/product/` | 要件・ユースケース | ビジネス側 |
| `docs/architecture/` | システム設計 | 開発者 |
| `docs/adr/` | 意思決定記録 | 開発者 |
| `docs/development/` | 開発プロセス | 開発者 |
| `docs/security/` | セキュリティポリシー | セキュリティ担当 |

## 触ってはいけない領域

`docs/ai-context/do-not-touch.md` を参照。
