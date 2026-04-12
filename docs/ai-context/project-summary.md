# project-summary.md — プロジェクト全体要約

> AIが最初に読むファイルです。3〜5分で全体像を把握できるように保ちます。
> 詳細は各 `docs/` ファイルを参照してください。

## プロジェクト概要

| 項目 | 内容 |
|---|---|
| プロジェクト名 | behavioral_log_analysis |
| 目的 | 個人の行動ログ（音声日記・YouTube視聴履歴）を毎日蓄積し、Claude API が6つの観点で定期自動分析するパーソナルツール |
| 対象ユーザー | 個人（将来的にマルチユーザー対応を見越した設計） |
| フェーズ | MVP（要件定義フェーズ） |

## 技術スタック

| 層 | 技術 |
|---|---|
| Backend | Laravel 11 |
| DB (ローカル) | MySQL 8.0 |
| DB (AWS) | Amazon RDS / Aurora MySQL |
| Frontend | Vue.js 3 + Inertia.js + Tailwind CSS（モバイルファースト） |
| Auth | Laravel Sanctum |
| Queue | Laravel Horizon + Redis |
| Storage (ローカル) | `storage/app/`（音声ファイル） |
| Storage (AWS) | AWS S3（音声ファイル） |
| AI 分析 | Claude API (Anthropic) |
| 音声変換 | OpenAI Whisper API |
| YouTube 連携 | YouTube Data API v3 (Google OAuth2) |
| スケジューラ | Laravel Task Scheduler |

## 主要ドメイン

| ドメイン | 説明 | 主なモデル |
|---|---|---|
| users | ユーザー管理・認証 | User |
| logs | 日次ログ管理（音声日記・YouTube履歴の集約） | DailyLog, VoiceDiary, WatchHistory |
| analysis | AI 定期分析・レポート管理 | AnalysisReport |
| integrations | 外部サービス連携（YouTube OAuth） | YouTubeToken |

## ディレクトリ構成（概要）

```
app/
  Http/Controllers/   - コントローラ（薄く保つ）
  Services/           - ビジネスロジック（Whisper変換・YouTube取得・Claude分析）
  Actions/            - 単一責務アクションクラス
  Models/             - Eloquent モデル
  Policies/           - 認可ポリシー
  Jobs/               - 非同期ジョブ（ScheduledAnalysisJob 等）
resources/js/
  Pages/Diary/        - 音声日記UI（録音・一覧・詳細）
  Pages/Analysis/     - 分析結果表示UI
  Pages/Dashboard/    - ダッシュボード（ログ概要）
  Pages/Settings/     - YouTube OAuth 連携設定
docs/                 - 設計ドキュメント全体
.claude/              - Claude Code 用ルール・コマンド
```

詳細は `docs/ai-context/module-map.md` を参照。

## 6つの分析観点

| # | 観点 | 説明 | 実行頻度 |
|---|---|---|---|
| 1 | 自動回顧録 | 蓄積ログから振り返りレポートを自動生成 | 週次・2ヶ月ごと |
| 2 | 学習フィルター | 学習傾向・興味分野を抽出・可視化 | 週次・2ヶ月ごと |
| 3 | メンタルトリガー検知 | 気分・感情パターンの検知とアラート | 週次 |
| 4 | AI コーチング | ログをもとにアドバイス・目標提示 | 2ヶ月ごと |
| 5 | タイムカプセル | 過去の自分から未来の自分へのメッセージ | 手動作成・定期開封 |
| 6 | ビジネスアイデア提案 | 視聴・日記・学習傾向を掛け合わせた新規アイデア提案 | 2ヶ月ごと |

## デザイン方針

- **モバイルファースト**：スマートフォンブラウザでの利用を主とする
- Tailwind CSS でレスポンシブ対応（sm / md / lg ブレークポイント）
- ネイティブアプリは作らない（PWA は将来検討）

## 現在のフォーカス

要件定義フェーズ（Gate 0〜Gate 1）。
ドキュメント整備が完了次第、Gate 2（use-cases.md 承認）を経てコード生成へ進む。

## 読む順序（AIへの案内）

1. このファイル（概要把握）
2. `docs/ai-context/module-map.md`（構造把握）
3. タスクに応じた詳細ドキュメント（CLAUDE.md の対応表を参照）
