# 00-global.md — 全体方針

## 前提：このファイルを読んだら確認すること

以下が埋まっているか確認する。埋まっていなければ作業を止めてユーザーに通知する。

- [ ] `docs/ai-context/project-summary.md` が記入済みか
- [ ] `docs/ai-context/glossary.md` が記入済みか
- [ ] `docs/product/use-cases.md` の承認状況はどうか

---

## 開発ワークフロー

必ずこの順序で進める:

```
Explore → Plan → Implement → Test
```

1. **Explore**: `docs/ai-context/` と関連ドキュメントを読み、現状を理解する
2. **Plan**: 変更の影響範囲・リスクを整理し、実装方針をユーザーに提示して合意を得る
3. **Implement**: 合意した計画に従って実装する（スコープ外に触れない）
4. **Test**: Feature Testを追加・実行して動作を確認する

---

## AI開発パイプライン

```
[Gate 0] docs/ai-context/ 記入完了
    ↓
[Gate 1] docs/product/requirements.md — レビュアー承認済み
    ↓  ※ AIによる use-cases.md 叩き台生成可
use-cases.md 作成（AIたたき台 → 人間修正）
    ↓  ※ AIによるモック生成可（/generate-mock）
モック作成・ビジネス側レビュー（docs/product/mockups/）
    ↓  モックフィードバックを use-cases.md に反映
[Gate 2] docs/product/use-cases.md   — レビュアー最終承認済み ★
    ↓  ※ AIによる acceptance-criteria.md / data-model.md 叩き台生成可
[Gate 3] docs/architecture/data-model.md — レビュアー承認済み
    ↓
AI コード生成（Claude Code）
    ↓
AI テスト生成
    ↓
人間レビュー・マージ
```

---

## 品質ゲート詳細

| Gate | 条件 | 解禁されること |
|---|---|---|
| Gate 0 | `docs/ai-context/` の必須ファイルが記入済み | AI支援の開始 |
| Gate 1 | `docs/product/requirements.md` レビュアー承認済み | use-cases.md 叩き台生成・モック生成 |
| Gate 2 ★ | `docs/product/use-cases.md` レビュアー最終承認済み | コード生成・acceptance-criteria / data-model 叩き台生成 |
| Gate 3 | `docs/architecture/data-model.md` レビュアー承認済み | DB実装・マイグレーション作成 |

---

## 絶対禁止

- Gate 2 通過前のコード生成
- コード先行（ドキュメント未確認での実装）
- 大規模変更の一括コミット
- テストなし実装（バグ修正には再発防止テストを必ず追加）
- スコープ外ファイルの編集
- secrets・本番資格情報をプロンプトに含める
- `docs/original-docs/` の編集・削除・ファイル新規作成（参照のみ可）

---

## 変更時の確認リスト

- [ ] 関連ADRを確認したか
- [ ] use-cases.md に変更の影響はないか
- [ ] CR（Change Request）の場合、traceability-matrix.md を更新したか
- [ ] マイグレーション計画はあるか（DB変更の場合）
- [ ] テストを追加したか
- [ ] ドキュメントを更新したか
