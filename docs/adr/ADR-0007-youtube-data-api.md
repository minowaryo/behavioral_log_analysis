# ADR-0007: YouTube Data API v3 + Google OAuth2 で視聴履歴を取得する

## Status
Accepted

## Date
2026-04-12

## Context

YouTube 視聴履歴（F-002）をログの入力源として取得するために、
どの方法でデータを取得するかを決定する必要があった。

選択肢:
1. **YouTube Data API v3 + Google OAuth2**: 公式 API でリアルタイムに近い形で取得
2. **Google Takeout エクスポート（手動）**: JSON ファイルをダウンロードして手動インポート
3. **スクレイピング**: 非公式・利用規約違反のため除外

## Decision

**YouTube Data API v3 + Google OAuth2** を採用する。

## Rationale

- Google Takeout は手動操作が必要で、「毎日自動でログを蓄積する」というコンセプトに合わない
- YouTube Data API v3 は公式 API で利用規約に準拠している
- OAuth2 認証により、ユーザーが明示的に許可した範囲でのみデータを取得できる（プライバシー面で安全）
- アクセストークンは自動更新（リフレッシュトークン）でき、定期同期が可能

**取得するデータ**:
- `activities.list` エンドポイント（`eventType=watch`）で視聴履歴を取得
- 動画ID・タイトル・チャンネルID・チャンネル名・視聴日時

**注意**: YouTube Data API v3 は視聴履歴の直接取得が `youtube.readonly` スコープで制限される場合がある。
代替として `activities.list` + `videos.list` の組み合わせを使用する。

## Consequences

- Google Cloud Console でプロジェクト作成・OAuth2 クライアント設定が必要
- クライアント ID / シークレットは `.env` で管理（ドキュメントには記載しない）
- YouTube Data API v3 の無料クォータ: 10,000 ユニット/日（視聴履歴取得は1リクエスト約1〜5ユニット）
- アクセストークン・リフレッシュトークンを `youtube_tokens` テーブルで管理する
- ユーザーが YouTube との連携を解除した場合はトークンを即座に削除する
