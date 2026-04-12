# glossary.md — 用語集

> プロジェクト固有の用語・略語を定義します。
> AIと人間の共通言語にするために正確に記載してください。

## ドメイン用語

| 用語 | 説明 | 別名 |
|---|---|---|
| DailyLog | 1日分のログ集約エンティティ。VoiceDiary と WatchHistory をまとめて管理する | 日次ログ |
| VoiceDiary | 音声日記エントリ。音声ファイル・文字起こしテキスト・日付を保持する | 音声日記 |
| WatchHistory | YouTube 視聴履歴エントリ。動画ID・タイトル・チャンネル・視聴日時を保持する | 視聴履歴 |
| AnalysisReport | 6観点での AI 分析結果。分析タイプ・期間・Claude の出力テキストを保持する | 分析レポート |
| YouTubeToken | Google OAuth2 トークン（アクセストークン・リフレッシュトークン）を保持するエンティティ | - |
| Retrospective | 分析観点①：自動回顧録。蓄積ログから振り返りレポートを生成する | 自動回顧録 |
| LearningFilter | 分析観点②：自分専用学習フィルター。学習傾向・興味分野を抽出・可視化する | 学習フィルター |
| MentalTrigger | 分析観点③：メンタルトリガー検知。気分・感情パターンを検知してアラートを出す | メンタルトリガー検知 |
| AICoaching | 分析観点④：AI コーチング。ログをもとにアドバイス・目標を提示する | AIコーチング |
| TimeCapsule | 分析観点⑤：未来へのタイムカプセル。過去の自分から未来の自分へのメッセージ | タイムカプセル |
| BusinessIdeaGenerator | 分析観点⑥：新規ビジネスアイデア提案。視聴・日記・学習傾向を掛け合わせてアイデアを提案する | ビジネスアイデア提案 |

## 技術用語（プロジェクト固有の使い方）

| 用語 | このプロジェクトでの意味 |
|---|---|
| Action | 単一責務のビジネス操作クラス（`app/Actions/`） |
| Service | 複数Actionを束ねる上位サービスクラス（`app/Services/`） |
| ADR | Architecture Decision Record（意思決定記録） |
| Gate | Laravel認可の仕組み（Policy経由で呼ぶ） |
| RCID | Requirement Change ID（要件変更ID・トレーサビリティ用） |
| Inertia.js | Laravel とフロントフレームワークをサーバー駆動でつなぐアダプター。SPA ライクなUXを API 設計なしで実現する |
| Pinia | Vue 3 公式推奨の状態管理ライブラリ。Vuex の後継 |
| Composable | `use~` 命名の関数。Composition API のロジックを再利用可能な形で切り出したもの（`resources/js/Composables/`） |
| Page コンポーネント | Inertia のルートに対応する `Pages/` 配下の `.vue` ファイル。Controller の return で指定される |
| Whisper API | OpenAI が提供する音声文字起こし API。音声ファイルを送信してテキストを返す |
| YouTube Data API | Google が提供する YouTube の動画・履歴データ取得 API（v3）。OAuth2 認証が必要 |
| ScheduledAnalysisJob | 週次・2ヶ月ごとに自動で分析を実行する Laravel Job |

## 略語

| 略語 | 正式名称 |
|---|---|
| BA | Business Analyst |
| ADR | Architecture Decision Record |
| PII | Personally Identifiable Information（個人識別情報） |
| ERD | Entity Relationship Diagram |
| MVP | Minimum Viable Product |
| UC | Use Case |

## 分析タイプ定義

| タイプ値 | 対応観点 | 実行頻度 |
|---|---|---|
| `retrospective` | 自動回顧録 | 週次・2ヶ月ごと |
| `learning_filter` | 学習フィルター | 週次・2ヶ月ごと |
| `mental_trigger` | メンタルトリガー検知 | 週次 |
| `ai_coaching` | AI コーチング | 2ヶ月ごと |
| `time_capsule` | タイムカプセル | 手動 |
| `business_idea` | ビジネスアイデア提案 | 2ヶ月ごと |
