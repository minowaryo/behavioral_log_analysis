# /review — コードレビューコマンド

以下の観点でコードレビューを実施してください。

## レビュー前に読むファイル

- `docs/product/use-cases.md` — 実装が要件と一致しているか確認するため
- `docs/architecture/data-model.md` — DBスキーマ・マイグレーションの整合性確認のため
- `docs/product/mockups/` — UI実装がモックと一致しているか確認するため（存在する場合）

## レビュー対象

直近の変更ファイル（または指定されたファイル）

## チェックリスト

### 機能・設計
- [ ] `docs/product/use-cases.md` の要件と実装が一致しているか
- [ ] Fat Controller になっていないか
- [ ] Policy / Gate を通しているか
- [ ] N+1クエリがないか

### セキュリティ（`.claude/rules/40-security.md` 参照）
- [ ] バリデーションが適切か
- [ ] secrets・PII がコードに含まれていないか
- [ ] ログに個人情報が出ていないか

### テスト（`.claude/rules/30-testing.md` 参照）
- [ ] Feature Testが追加されているか
- [ ] 正常系・異常系・認可のテストがあるか

### ドキュメント（`.claude/rules/60-docs.md` 参照）
- [ ] 設計変更が docs に反映されているか
- [ ] ADRが必要な判断をしていないか

## アウトプット形式

1. **総評**（1〜2文）
2. **問題点**（重大度: HIGH / MEDIUM / LOW）
3. **推奨する修正**
4. **追加すべきテスト**
