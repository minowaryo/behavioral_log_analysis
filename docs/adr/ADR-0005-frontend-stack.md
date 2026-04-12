# ADR-0005: フロントエンドスタックの選定（Vue 3 + Inertia.js + Pinia）

## Status
Accepted

## Date
2026-04-12

## Context

Laravel モノリスにフロントエンドを追加するにあたり、技術スタックの選定が必要になった。
検討した選択肢は以下の通り:

1. **Vue 3 + Inertia.js**（サーバー駆動・API レス）
2. **Vue 3 + SPA**（API 分離・Laravel は REST API として機能）
3. **React + Inertia.js**
4. **Blade（Laravel 標準テンプレート）**

## Decision

**Vue 3 + Inertia.js + Pinia** を採用する。

- コンポーネントスタイル: Composition API + `<script setup>`
- 状態管理: Pinia
- TypeScript: 任意（プロジェクト単位で判断）
- Vue バージョン: 3.3 以上

## Rationale

### Inertia.js を選んだ理由
- Laravel の Controller・Route・Middleware をそのまま活用でき、REST API を別途設計・維持するコストが不要
- 認証・認可（Sanctum / Policy）を Laravel 側に集約できる
- チーム規模・保守コストの観点から SPA（API 分離）より運用負荷が低い

### Vue 3 を選んだ理由
- Composition API + `<script setup>` により、ロジックの再利用性と TypeScript 親和性が高い
- Laravel コミュニティでの採用実績が豊富（Laravel Breeze の Vue オプション等）
- React と比較してテンプレート構文が Laravel Blade に近く、学習コストが低い

### Pinia を選んだ理由
- Vue 3 公式推奨の状態管理ライブラリ
- Vuex と比較して型安全でボイラープレートが少ない

## Consequences

- フロントエンドは `resources/js/` 以下に配置する（`Pages/`, `Components/`, `Composables/`, `stores/`）
- Vue Router は使用しない（ルーティングは Inertia + Laravel Router が担う）
- フロントエンドのビルドは Vite（Laravel Vite Plugin）を使用する
- 詳細な実装ルールは `.claude/rules/15-vue.md` を参照
