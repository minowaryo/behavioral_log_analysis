# overview.md — システム全体像

## システム概要

[システムの目的と全体像を1〜2段落で説明]

## アーキテクチャ図

```
[ブラウザ / モバイル]
        ↓ HTTPS
[Load Balancer]
        ↓
[Laravel Application]
   ├─ Routes (api.php / web.php)
   ├─ Middleware (Auth / Throttle)
   ├─ Controllers
   ├─ Services / Actions
   └─ Models (Eloquent)
        ↓
[MySQL]    [Redis (Cache / Queue)]    [S3 (Storage)]
        ↓
[Queue Worker (Horizon)]
```

## 主要コンポーネント

| コンポーネント | 役割 | 技術 |
|---|---|---|
| Web Application | APIサーバー・フロントエンド | Laravel [version] |
| Database | データ永続化 | MySQL [version] |
| Cache | セッション・APIキャッシュ | Redis |
| Queue | 非同期ジョブ | Laravel Horizon + Redis |
| Storage | ファイル保管 | AWS S3 |
| CDN | 静的アセット配信 | [例: CloudFront] |

## 外部連携

| 外部サービス | 連携目的 | 方向 |
|---|---|---|
| [サービス名] | [目的] | 送信/受信/双方向 |

## デプロイ構成

| 環境 | 用途 | URL |
|---|---|---|
| local | ローカル開発 | http://localhost |
| staging | テスト・QA | [staging URL] |
| production | 本番 | [production URL] |

## 関連ドキュメント

- 詳細設計: `docs/architecture/data-model.md`
- 認証認可: `docs/architecture/authz-authn.md`
- ADR: `docs/adr/`
