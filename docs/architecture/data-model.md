# data-model.md — データモデル設計

> DB変更前に必ず参照すること。
> マイグレーション方針は `.claude/rules/20-mysql.md` を参照。
> Gate 3 承認後にマイグレーション作成を開始する。

## ER図（概要）

```
[users]
  │
  ├──< [daily_logs]
  │       ├──< [voice_diaries]
  │       └──< [youtube_signals] >── [youtube_video_catalog]
  │
  ├──< [youtube_tokens]
  └──< [analysis_reports]
```

---

## テーブル定義

### users

| カラム | 型 | Nullable | デフォルト | 説明 |
|---|---|---|---|---|
| id | bigint | NO | auto | 主キー |
| name | varchar(255) | NO | - | 表示名 |
| email | varchar(255) | NO | - | メールアドレス（unique） |
| email_verified_at | timestamp | YES | null | メール確認日時 |
| password | varchar(255) | NO | - | ハッシュ化パスワード |
| remember_token | varchar(100) | YES | null | Laravel Sanctum セッション用 |
| created_at | timestamp | NO | now() | 作成日時 |
| updated_at | timestamp | NO | now() | 更新日時 |

**Index**: `email` (unique)

---

### daily_logs

1日分のログ集約エンティティ。VoiceDiary と YouTubeSignal の親レコード。

| カラム | 型 | Nullable | デフォルト | 説明 |
|---|---|---|---|---|
| id | bigint | NO | auto | 主キー |
| user_id | bigint | NO | - | FK → users.id |
| log_date | date | NO | - | 記録日（1ユーザー1日1レコード） |
| created_at | timestamp | NO | now() | 作成日時 |
| updated_at | timestamp | NO | now() | 更新日時 |

**Index**: `(user_id, log_date)` (unique)
**FK**: `user_id` → `users(id)` ON DELETE CASCADE

---

### voice_diaries

音声日記エントリ。1日に複数保存可能。

| カラム | 型 | Nullable | デフォルト | 説明 |
|---|---|---|---|---|
| id | bigint | NO | auto | 主キー |
| user_id | bigint | NO | - | FK → users.id |
| daily_log_id | bigint | NO | - | FK → daily_logs.id |
| audio_path | varchar(500) | NO | - | ストレージ上の音声ファイルパス（例: voice/{user_id}/{YYYY}/{MM}/{DD}/{uuid}.mp4） |
| transcribed_text | text | NO | - | Whisper API による文字起こし結果 |
| recorded_at | timestamp | NO | - | 録音日時 |
| duration_seconds | int | YES | null | 録音時間（秒） |
| created_at | timestamp | NO | now() | 作成日時 |
| updated_at | timestamp | NO | now() | 更新日時 |

**Index**: `user_id`, `daily_log_id`
**FK**: `user_id` → `users(id)` ON DELETE CASCADE, `daily_log_id` → `daily_logs(id)` ON DELETE CASCADE

---

### youtube_tokens

Google OAuth2 トークン保管。1ユーザー1レコード（upsert）。

| カラム | 型 | Nullable | デフォルト | 説明 |
|---|---|---|---|---|
| id | bigint | NO | auto | 主キー |
| user_id | bigint | NO | - | FK → users.id (unique) |
| access_token | text | NO | - | Google アクセストークン（暗号化保存） |
| refresh_token | text | NO | - | Google リフレッシュトークン（暗号化保存） |
| token_expires_at | timestamp | NO | - | アクセストークン有効期限 |
| scopes | varchar(500) | NO | - | 付与スコープ（例: youtube.readonly） |
| created_at | timestamp | NO | now() | 作成日時 |
| updated_at | timestamp | NO | now() | 更新日時 |

**Index**: `user_id` (unique)
**FK**: `user_id` → `users(id)` ON DELETE CASCADE
**備考**: 連携解除時はこのレコードのみ削除する。YouTubeSignal は保持する

---

### youtube_signals

YouTube 行動シグナルエンティティ。高評価・プレイリスト追加・購読等のイベントを `source_type` で区別。

| カラム | 型 | Nullable | デフォルト | 説明 |
|---|---|---|---|---|
| id | bigint | NO | auto | 主キー |
| user_id | bigint | NO | - | FK → users.id |
| daily_log_id | bigint | YES | null | FK → daily_logs.id（occurred_at の日付で紐づけ） |
| source_type | varchar(50) | NO | - | シグナル種別: `activity_like` / `liked_video` / `playlist_item` / `subscription` |
| video_id | varchar(20) | YES | null | YouTube 動画ID（subscription 以外は基本あり） |
| channel_id | varchar(30) | YES | null | YouTube チャンネルID |
| playlist_id | varchar(50) | YES | null | プレイリストID（playlist_item のみ） |
| occurred_at | timestamp | NO | - | シグナル発生日時（API から取得） |
| raw_payload | json | YES | null | API レスポンスの生データ（デバッグ用） |
| created_at | timestamp | NO | now() | 作成日時 |
| updated_at | timestamp | NO | now() | 更新日時 |

**Index**: `user_id`, `daily_log_id`, `(user_id, source_type, video_id, occurred_at)` (unique — 重複除外用)
**FK**: `user_id` → `users(id)` ON DELETE CASCADE, `daily_log_id` → `daily_logs(id)` ON DELETE SET NULL
**備考**: `video_id` は `youtube_video_catalog.video_id` への論理参照（外部キー制約は張らない — カタログ未取得でも保存できる設計）

---

### youtube_video_catalog

`videos.list` で取得した動画メタデータのマスタ。シグナルに意味を付与するために使う。

| カラム | 型 | Nullable | デフォルト | 説明 |
|---|---|---|---|---|
| video_id | varchar(20) | NO | - | YouTube 動画ID（主キー） |
| title | varchar(500) | NO | - | 動画タイトル |
| channel_title | varchar(255) | YES | null | チャンネル名 |
| category_id | varchar(10) | YES | null | YouTube カテゴリID |
| tags | json | YES | null | タグ一覧（配列） |
| topic_categories | json | YES | null | Wikipedia カテゴリURL 一覧（配列） |
| duration_seconds | int | YES | null | 動画時間（秒） |
| published_at | timestamp | YES | null | 動画公開日時 |
| fetched_at | timestamp | NO | now() | メタデータ取得日時 |
| created_at | timestamp | NO | now() | 作成日時 |
| updated_at | timestamp | NO | now() | 更新日時 |

**PK**: `video_id`
**備考**: 複数ユーザーで共有するマスタテーブル（user_id なし）。`fetched_at` が古い場合は再取得を検討する

---

### analysis_reports

AI 分析結果。分析タイプ・期間・Claude の出力テキストを保持。

| カラム | 型 | Nullable | デフォルト | 説明 |
|---|---|---|---|---|
| id | bigint | NO | auto | 主キー |
| user_id | bigint | NO | - | FK → users.id |
| analysis_type | varchar(50) | NO | - | 分析種別: `retrospective` / `learning_filter` / `mental_trigger` / `ai_coaching` / `business_idea` |
| period_type | varchar(20) | NO | - | 実行種別: `weekly` / `monthly` |
| period_start | date | NO | - | 分析対象期間の開始日 |
| period_end | date | NO | - | 分析対象期間の終了日 |
| content | longtext | NO | - | Claude API の出力テキスト（business_idea の場合は JSON 配列形式） |
| model_used | varchar(100) | NO | - | 使用モデル（例: claude-sonnet-4-6） |
| created_at | timestamp | NO | now() | 作成日時 |
| updated_at | timestamp | NO | now() | 更新日時 |

**Index**: `user_id`, `(user_id, analysis_type, period_start)` (重複防止用の参照に使用)
**FK**: `user_id` → `users(id)` ON DELETE CASCADE

---

## 設計方針

- 主キーは `bigint` (auto increment)
- タイムスタンプは `timestamp`、文字コードは `utf8mb4`、照合順序は `utf8mb4_unicode_ci`
- 論理削除が必要なテーブルは `deleted_at` を追加（現 MVP では users のみ検討）
- 金額カラムは `decimal(15, 2)`（float 禁止）
- 外部キーには必ずインデックスを作成
- `youtube_tokens` の `access_token` / `refresh_token` は Laravel の `encrypt()` で暗号化して保存する
- `youtube_video_catalog` は user_id なしの共有マスタ。シグナルとの結合は `video_id` で行う

## 変更履歴

| 日付 | 変更内容 | ADR |
|---|---|---|
| 2026-04-18 | 初版作成。確定エンティティ（daily_logs / voice_diaries / youtube_tokens / youtube_signals / youtube_video_catalog / analysis_reports）を定義 | ADR-0007, ADR-0008 |
