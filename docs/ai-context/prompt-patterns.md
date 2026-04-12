# prompt-patterns.md — よく使うプロンプト定型文

> Claude Code / Codex に効果的に指示するための定型プロンプト集です。
> そのままコピーして使えます。

## 新機能実装

```
[機能名] を実装してください。

まず以下を確認してください:
1. docs/ai-context/project-summary.md（全体概要）
2. docs/product/use-cases.md（関連ユースケース）
3. docs/architecture/data-model.md（DB設計）

実装前に計画を提示してください（Explore → Plan → Implement の順で）。
```

## バグ修正

```
[バグの説明] を修正してください。

修正前に:
1. 原因を特定して説明してください
2. 再発防止テストを先に提示してください
3. 修正方針を確認してから実装してください
```

## コードレビュー

```
以下のコードをレビューしてください:
[コードまたはファイルパス]

確認観点:
- docs/development/review-checklist.md に従ったチェック
- セキュリティ観点（.claude/rules/40-security.md 参照）
- テストカバレッジ
- 設計パターンへの準拠
```

## マイグレーション作成

```
[テーブル名/変更内容] のマイグレーションを作成してください。

注意:
- docs/architecture/data-model.md を確認してください
- .claude/rules/20-mysql.md のMigration方針に従ってください
- 後方互換を意識してください
- 危険な操作（DROP等）の場合は警告してください
```

## ADR作成

```
[意思決定の内容] についてADRを作成してください。

テンプレート: docs/adr/ の既存ADRを参照してください。
必須項目: Context / Decision / Rationale / Consequences
```

## テスト作成

```
[機能/クラス名] のFeature Testを作成してください。

対象:
- 正常系
- 認証・認可（未認証/権限なし）
- バリデーションエラー

テスト命名は日本語で。
.claude/rules/30-testing.md の方針に従ってください。
```

## ドキュメント更新確認

```
この変更によって更新が必要なドキュメントを洗い出してください。
.claude/rules/60-docs.md の対応表を参考にしてください。
```
