# ADR-0007: YouTube Data API v3 で「行動シグナル」を収集する（完全視聴履歴は対象外）

## Status
Accepted（2026-04-12 公式仕様確認により内容を更新）

## Date
2026-04-12

## Context

YouTube 視聴行動（F-002）をログの入力源として活用するために、
YouTube Data API v3 の公式仕様を確認し、MVP で実現可能な取得範囲を確定する必要があった。

### 公式仕様確認の結果（2026-04-12）

YouTube Data API v3 の公式ドキュメントを調査した結果、以下が判明した。

**取得可能（公式サポート）**

| エンドポイント | スコープ | 取得できるもの |
|---|---|---|
| `activities.list(mine=true)` | `youtube.readonly` | 直近の YouTube アクティビティ（高評価・プレイリスト追加等） |
| `videos.list(myRating=like)` | `youtube.readonly` | ユーザーが高評価した動画一覧 |
| `playlists.list(mine=true)` | `youtube.readonly` | 自分が作成したプレイリスト |
| `playlistItems.list` | `youtube.readonly` | プレイリスト内の動画一覧 |
| `subscriptions.list(mine=true)` | `youtube.readonly` | 購読中チャンネル一覧 |
| `videos.list(id=...)` | `youtube.readonly` | 動画メタデータ（タイトル・カテゴリ・タグ・duration・topicCategories 等） |
| `videos.getRating` | `youtube.readonly` | 指定動画への評価（like / dislike / none） |

**取得不可（公式に安定取得できない）**

| 対象 | 理由 |
|---|---|
| 完全な視聴履歴（watch history） | `channels.list` の `relatedPlaylists.watchHistory` は現行 API では公式に公開されていない |
| `eventType=watch` による視聴ログ | `activities.list` に `watch` タイプは documented されていない |
| watchLater プレイリスト | private system playlist として公式に読み取れない |

**クォータ消費（参考）**

| 操作 | コスト（ユニット） |
|---|---|
| `activities.list` | 1/リクエスト |
| `videos.list` | 1〜3/リクエスト |
| `subscriptions.list` | 1/リクエスト |
| `playlists.list` | 1/リクエスト |
| 1回の同期（全エンドポイント合計） | 約 50〜150 ユニット |
| 無料クォータ | 10,000 ユニット/日 |

→ 日次同期でも余裕があり、クォータは問題にならない。

### 選択肢の比較

| 案 | 概要 | 結論 |
|---|---|---|
| A: API-only（行動シグナル） | 公式エンドポイントで取れるシグナルのみ収集。完全視聴履歴は諦める | **採用** |
| B: Takeout インポート + API 補完 | 手動エクスポートで視聴履歴を取得し、API でメタデータ補完 | 将来対応（MVP対象外） |
| C: 非公式 API・スクレイピング | 技術的には可能だが規約違反・アカウントリスクあり | 除外 |

## Decision

**案A: YouTube Data API v3（行動シグナル収集）を採用する。**

収集対象を「視聴履歴の完全再現」ではなく、以下の行動シグナルに限定する:

1. `activities.list(mine=true)` — 直近アクティビティ（高評価・プレイリスト追加等）
2. `videos.list(myRating=like)` — 高評価動画
3. `playlists.list(mine=true)` + `playlistItems.list` — 自分のプレイリスト
4. `subscriptions.list(mine=true)` — 購読チャンネル
5. `videos.list(id=...)` — 動画メタデータ補完（タイトル・カテゴリ・タグ・topicCategories）

## Rationale

- 視聴履歴の「完全自動取得」は公式 API で保証されないため、要件として約束できない
- 一方、行動シグナル（高評価・プレイリスト・購読）は明示的な興味表明であり、分析精度が高い
- `videos.list` の `topicCategories`（Wikipedia カテゴリ）と `tags` を活用すれば、テーマ・ジャンルの傾向分析は十分に行える
- クォータ消費が少なく（日次同期で約 50〜150 ユニット）、無料枠で運用できる

## Consequences

- 要件・UC で「視聴履歴」という表現を「YouTube 行動シグナル」に修正する
- DB テーブルは `youtube_signals` として以下のカラムを持つ:
  - `id`, `user_id`, `source_type`（activity_like / liked_video / playlist_item / subscription）
  - `video_id`, `channel_id`, `playlist_id`, `occurred_at`, `raw_payload`
- 動画マスタは `youtube_video_catalog` として別管理（`videos.list` で補完）:
  - `video_id`, `title`, `channel_title`, `category_id`, `tags`, `duration`, `topic_categories`, `published_at`
- Google Cloud Console でプロジェクト作成・OAuth2 クライアント設定が必要
- クライアント ID / シークレットは `.env` で管理（ドキュメントには記載しない）
- ユーザーが YouTube との連携を解除した場合は YouTubeToken（アクセストークン・リフレッシュトークン）のみ即座に削除する。取得済みの YouTubeSignal データは分析継続のために保持する
- **将来対応**: Google Takeout インポートによる視聴履歴の取り込みは別 ADR で検討する
