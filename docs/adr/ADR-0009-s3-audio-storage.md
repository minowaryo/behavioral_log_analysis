# ADR-0009: AWS S3 を音声ファイルのストレージとして採用する（ローカルは storage/app/）

## Status
Accepted

## Date
2026-04-12

## Context

音声日記（F-001）で録音した音声ファイルをどこに保存するかを決定する必要があった。
MVP 段階ではローカル環境での動作を優先しつつ、将来の AWS 移行を見越した設計が求められる。

選択肢:
1. **AWS S3 + ローカルフォールバック（Laravel Filesystem 抽象化）**: 環境に応じて切り替え
2. **DB に BLOB として保存**: シンプルだが DB 肥大化のリスク
3. **ローカルストレージのみ**: MVP には十分だが将来移行が困難

## Decision

**Laravel Filesystem（`Storage` ファサード）で抽象化し、ローカルは `storage/app/voice/`、本番は AWS S3** を使用する。

## Rationale

- Laravel の `Storage` ファサードはドライバーを設定ファイル（`config/filesystems.php`）で切り替えられるため、コードを変更せずにローカル→S3 移行が可能
- 音声ファイルをDBに保存するとマイグレーションコストが高く、RDBMSのパフォーマンスが低下する
- S3 は耐久性 99.999999999%（11 nines）でデータ損失リスクが極めて低い

**ファイル命名規則**:
```
voice/{user_id}/{YYYY}/{MM}/{DD}/{uuid}.mp4
```

**ローカル環境（.env）**:
```
FILESYSTEM_DISK=local
```

**本番環境（.env）**:
```
FILESYSTEM_DISK=s3
AWS_BUCKET=behavioral-log-analysis
AWS_DEFAULT_REGION=ap-northeast-1
```

## Consequences

- MVP（ローカル開発）では `storage/app/voice/` に保存される
- 本番移行時は `.env` の `FILESYSTEM_DISK` を `s3` に変更するだけでよい
- AWS S3 バケットのアクセスポリシー設定が必要（非公開バケット + 署名付き URL）
- 音声ファイルの署名付き URL は有効期限（例: 1時間）を設けてセキュリティを確保する
- Whisper API に送信後、音声ファイルをサーバーから削除するかどうかはユーザー設定で選択可能にする
