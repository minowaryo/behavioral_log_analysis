# secrets-handling.md — シークレット・機密情報の取り扱い

> **AI時代に特に重要なルールです。**
> secrets漏洩はサービス停止・データ流出・費用爆発につながります。

## 絶対禁止（AIを使う場合も含む）

| 禁止行為 | 理由 |
|---|---|
| APIキー・パスワードをコードにハードコード | Git履歴に残り、永続的に漏洩する |
| `.env` 実値をコミット | リポジトリ経由で全員に見える |
| 本番資格情報をAIプロンプトに貼り付ける | AIの学習データや会話ログに残るリスク |
| secrets をチャット・Slackに貼る | ログ・通知経由で漏洩するリスク |
| ログにsecrets・PIIを出力する | ログ基盤経由で漏洩するリスク |

## 正しい管理方法

### 開発環境
```
# .env（Gitignore済み・実値を書く）
DB_PASSWORD=your_actual_password

# .env.example（Gitにコミット・キー名のみ）
DB_PASSWORD=
```

### 本番環境
- AWS Secrets Manager / GCP Secret Manager / HashiCorp Vault を使う
- CI/CDの環境変数機能を使う（GitHub Actions Secrets等）
- `.env` ファイルを本番サーバーに直接置かない

### AIへの提供時
```
# OK: ダミー値でコンテキストを渡す
DB_HOST=localhost
DB_PASSWORD=dummy_password_for_context

# NG: 実値を貼る
DB_PASSWORD=actual_prod_password_here
```

## 機密情報の種類

| 種類 | 例 | 管理場所 |
|---|---|---|
| DB接続情報 | host, user, password | .env / Secret Manager |
| APIキー | OpenAI, Stripe, AWS | .env / Secret Manager |
| JWT秘密鍵 | APP_KEY | .env / Secret Manager |
| OAuth credentials | client_id, client_secret | .env / Secret Manager |
| PII | 氏名、メール、電話番号 | DB（暗号化） |

## PII（個人識別情報）の取り扱い

- ログにPIIを出力しない（ユーザーIDのみ記録）
- デバッグ時にPIIを含むデータをダンプしない
- AIへの相談時はPIIをマスクする（例: `user@example.com` → `[email]`）

## インシデント発生時

secrets が漏洩した場合の対応:
1. 即座に対象のキー・パスワードをローテーション（無効化）
2. アクセスログを確認して不正利用を検知
3. 影響範囲を特定して関係者に報告
4. 原因と再発防止策をADRに記録

## チェックリスト（PR作成時）

- [ ] `.env` の実値がコードに含まれていないか
- [ ] `git diff` でsecrets混入がないか
- [ ] ログ出力にPIIが含まれていないか
- [ ] `composer audit` で既知脆弱性がないか
