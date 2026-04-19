# use-cases.md — ユースケース定義

> **重要度: ★★★★★（最重要）**
> 対象: 人間レビュー最終ポイント（Gate 2）
> ここで仕様品質が決定する。AIのコード生成品質はこの資料の質に依存する。

---

## レビュー基準

各ユースケースに以下が定義されているか確認する:

- [ ] Actor が明確か
- [ ] 入力が定義されているか
- [ ] 出力が定義されているか
- [ ] エラーが定義されているか
- [ ] 業務ルールが明記されているか
- [ ] 状態変更が明確か
- [ ] 権限考慮があるか

---

### UC-001: 音声日記を録音・保存する

**Actor**: ユーザー
**前提条件**: ログイン済み。ブラウザが MediaRecorder API に対応している
**関連要件**: F-001

#### 基本フロー
1. ユーザーが Diary 画面を開く
2. 録音ボタンをタップすると録音開始（MediaRecorder API）
3. 再度タップすると録音停止
4. システムが音声ファイルをサーバーにアップロード
5. システムが Whisper API に音声を送り文字起こしを行う
6. 文字起こし結果が画面に表示される
7. ユーザーが内容を確認・必要に応じて編集して保存ボタンをタップ
8. VoiceDiary レコードと DailyLog（当日分）が保存される

#### 入力
| 項目 | 型 | 必須 | バリデーション |
|---|---|---|---|
| 音声ファイル | binary (audio/mp4 or audio/webm) | 必須 | 最大 25MB（Whisper API 制限）、録音時間 1秒以上 |
| 日付 | date | 必須 | デフォルト: 今日（変更可） |
| テキスト（確認・編集後） | string | 必須 | 最大 10,000文字 |

#### 出力
| 項目 | 型 | 説明 |
|---|---|---|
| voice_diary_id | integer | 保存された VoiceDiary の ID |
| transcribed_text | string | 文字起こし結果テキスト |
| recorded_at | datetime | 録音日時 |

#### 業務ルール
- 同一日に複数の VoiceDiary を保存できる
- 音声ファイルはサーバーのストレージに保存し、テキストは DB に保存する
- iOS Safari では webm が非対応のため mp4 (AAC) を使用する
- DailyLog（当日分）が存在しない場合は自動的に作成する

#### エラーケース
| ケース | HTTPステータス | メッセージ |
|---|---|---|
| 音声ファイルが 25MB 超 | 422 | 音声ファイルが大きすぎます（最大25MB） |
| Whisper API エラー | 503 | 文字起こしサービスが一時的に利用できません |
| 未認証 | 401 | ログインが必要です |

#### 権限
- ログイン済みユーザー: 自分のレコードのみ作成可

---

### UC-002: YouTube 行動シグナルを同期する

**Actor**: ユーザー、システム
**前提条件**: ログイン済み
**関連要件**: F-002

> **注**: YouTube Data API v3 では「完全な視聴履歴」の安定取得は公式に提供されていない。
> MVP では取得可能な行動シグナル（高評価・プレイリスト・購読・アクティビティ）を収集し、
> 動画メタデータを付与して分析に活用する。

#### 基本フロー
1. ユーザーが Settings 画面で「YouTube 連携」ボタンをタップ
2. Google の OAuth2 認可画面でユーザーが `youtube.readonly` スコープを許可する
3. システムがアクセストークン・リフレッシュトークンを YouTubeToken に保存する
4. システムが以下の順で YouTube Data API v3 を呼び出す（バックグラウンドジョブ）:
   - `activities.list(mine=true)` — 直近のアクティビティ（高評価、プレイリスト追加等）
   - `videos.list(myRating=like)` — 高評価動画の一覧
   - `playlists.list(mine=true)` + `playlistItems.list` — 自分のプレイリスト内容
   - `subscriptions.list(mine=true)` — 購読チャンネル一覧
   - `videos.list(id=...)` — 上記で得た動画IDのメタデータ補完（タイトル・カテゴリ・タグ等）
5. 各シグナルを YouTubeSignal テーブルに保存（重複除外）し、DailyLog に紐づける
6. ユーザーに同期完了件数を通知する

#### 取得シグナルの種類

| source_type | 取得元エンドポイント | 意味 |
|---|---|---|
| `activity_like` | `activities.list` | アクティビティ上の高評価イベント |
| `liked_video` | `videos.list(myRating=like)` | 明示的な高評価動画 |
| `playlist_item` | `playlistItems.list` | 自分のプレイリストに追加した動画 |
| `subscription` | `subscriptions.list` | 購読中チャンネル（初回取得のみ） |

#### 動画メタデータ（補完項目）

| フィールド | 取得元 | 分析への活用 |
|---|---|---|
| title | `videos.list` | キーワード・テーマ抽出 |
| channelTitle | `videos.list` | チャンネル軸の傾向把握 |
| categoryId | `videos.list` | ジャンル分類 |
| tags | `videos.list` | トピッククラスタリング |
| topicCategories | `videos.list` | Wikipedia カテゴリベースのテーマ推定 |
| duration | `videos.list` | 短尺 / 長尺の習慣パターン |

#### 入力
| 項目 | 型 | 必須 | 説明 |
|---|---|---|---|
| OAuth2 authorization code | string | 必須 | Google から返却されるコード |

#### 出力
| 項目 | 型 | 説明 |
|---|---|---|
| synced_count | integer | 新規保存されたシグナル件数 |
| last_synced_at | datetime | 同期完了日時 |

#### 業務ルール
- 同じ `source_type` + `video_id` + `occurred_at` の組み合わせは重複保存しない
- アクセストークンが期限切れの場合はリフレッシュトークンで自動更新する
- 完全な視聴履歴の再現は行わない（API 制約のため）
- 同期はバックグラウンドジョブ（Laravel Queue）で実行する
- API クォータ消費量の目安: 1回の同期で約 50〜150 ユニット（上限 10,000/日）
- YouTube 連携を解除した場合は YouTubeToken（アクセストークン・リフレッシュトークン）のみ削除する。取得済みの YouTubeSignal データは分析継続のため保持する

#### エラーケース
| ケース | HTTPステータス | メッセージ |
|---|---|---|
| OAuth 未連携 | 403 | YouTube 連携が必要です |
| トークン無効（再認証必要） | 401 | YouTube の再連携が必要です |
| API クォータ超過 | 429 | YouTube API の利用制限に達しました。しばらく後に再試行してください |

#### 権限
- ログイン済みユーザー: 自分のアカウントのみ連携可

---

### UC-003: 日次ログを確認する

**Actor**: ユーザー
**前提条件**: ログイン済み
**関連要件**: F-003

#### 基本フロー
1. ユーザーが Dashboard または Diary 画面を開く
2. 日付ごとにまとめられたログ一覧が表示される
3. ユーザーが日付を選択すると詳細画面が開く
4. 詳細画面に音声日記テキスト・YouTube 行動シグナルが一覧表示される

#### 入力
| 項目 | 型 | 必須 | バリデーション |
|---|---|---|---|
| 日付 | date | 任意 | デフォルト: 今日 |

#### 出力
| 項目 | 型 | 説明 |
|---|---|---|
| daily_log | DailyLog | 指定日のログ集約 |
| voice_diaries | VoiceDiary[] | 当日の音声日記一覧 |
| youtube_signals | YouTubeSignal[] | 当日の YouTube 行動シグナル一覧 |

#### 業務ルール
- ログが存在しない日は「記録なし」と表示する
- 最新30日分をデフォルト表示（それ以前はページネーション）

#### エラーケース
| ケース | HTTPステータス | メッセージ |
|---|---|---|
| 未認証 | 401 | ログインが必要です |
| 他ユーザーのログへのアクセス | 403 | アクセス権限がありません |

---

### UC-004: 週次の自動分析レポートを受け取る

**Actor**: システム（スケジューラ）
**前提条件**: 対象ユーザーの1週間分の DailyLog が存在する
**関連要件**: F-004, F-005, F-006, F-009

#### 基本フロー
1. Laravel スケジューラが毎週土曜日 AM 7:00 に ScheduledAnalysisJob を実行
2. 対象ユーザーの過去7日分のログ（VoiceDiary + YouTubeSignal）を集約
3. Claude API に以下3観点の分析を依頼:
   - 自動回顧録（retrospective）
   - 学習フィルター（learning_filter）
   - メンタルトリガー検知（mental_trigger）
4. 分析結果を AnalysisReport に保存
5. ユーザーが次回 Analysis 画面を開いたときにレポートが表示される

#### 入力
| 項目 | 型 | 説明 |
|---|---|---|
| （なし） | - | スケジューラが自動実行。ユーザー操作は不要 |

#### 出力
| 項目 | 型 | 説明 |
|---|---|---|
| analysis_reports | AnalysisReport[] | 3観点（retrospective / learning_filter / mental_trigger）の分析結果。各レコードに analysis_type・period_start・period_end・content を保持 |

#### 状態変更
- AnalysisReport が最大3件（観点ごと）保存される
- 同一 `(user_id, analysis_type, period_type, period_start)` のレコードが既にある場合は `content` / `model_used` / `updated_at` を上書きする（upsert）。リトライ時の重複を防ぐ

#### 業務ルール
- 分析対象期間: 実行日の前7日間
- ログが3日分未満の場合は分析をスキップする（データ不足）
- Claude API への送信内容に個人識別情報（氏名・メールアドレス）を含めない

#### エラーケース
| ケース | 対処 |
|---|---|
| Claude API タイムアウト | リトライ（最大3回）後にエラーログを記録 |
| ログが3日未満 | スキップし、次回実行まで待機 |

---

### UC-005: 月次総合 AI 分析を受け取る

**Actor**: システム（スケジューラ）
**前提条件**: 対象ユーザーの1ヶ月分の DailyLog が存在する
**関連要件**: F-004〜F-007, F-009, F-011

#### 基本フロー
1. Laravel スケジューラが毎月1日 AM 7:00 に ScheduledAnalysisJob を実行
2. 対象ユーザーの過去30日分のログを集約
3. Claude API に以下5観点の分析を依頼:
   - 自動回顧録（retrospective）
   - 学習フィルター（learning_filter）
   - メンタルトリガー検知（mental_trigger）
   - AI コーチング（ai_coaching）
   - ビジネスアイデア提案（business_idea）
4. 各観点の分析結果を AnalysisReport に保存
5. Analysis 画面に「月次レポート」として表示される

#### 入力
| 項目 | 型 | 説明 |
|---|---|---|
| （なし） | - | スケジューラが自動実行。ユーザー操作は不要 |

#### 出力
| 項目 | 型 | 説明 |
|---|---|---|
| analysis_reports | AnalysisReport[] | 5観点（retrospective / learning_filter / mental_trigger / ai_coaching / business_idea）の分析結果 |

#### 状態変更
- AnalysisReport が最大5件（観点ごと）保存される
- 同一 `(user_id, analysis_type, period_type, period_start)` のレコードが既にある場合は `content` / `model_used` / `updated_at` を上書きする（upsert）。リトライ時の重複を防ぐ
- Analysis 画面の「月次レポート」タブに最新レポートが表示される

#### 業務ルール
- YouTube 行動シグナルから推定した興味カテゴリ・日記の関心事・学習傾向の3軸を掛け合わせてビジネスアイデアを生成する
- 分析ログが10日未満の場合はスキップ

---

### UC-006: ログイン・認証を行う

**Actor**: ユーザー
**前提条件**: アカウントが存在する
**関連要件**: F-010

#### 基本フロー
1. ユーザーがログイン画面でメールアドレス・パスワードを入力
2. Laravel Sanctum がセッション認証を処理
3. 認証成功後、Dashboard にリダイレクト

#### 業務ルール
- セッションは HttpOnly + Secure Cookie で管理
- CSRF トークンを必ず検証する
- 将来のマルチユーザー対応を見越し、全クエリに `user_id` スコープを適用する

#### エラーケース
| ケース | HTTPステータス | メッセージ |
|---|---|---|
| メールアドレス・パスワード不一致 | 422 | メールアドレスまたはパスワードが正しくありません |
| アカウントなし | 422 | 同上（セキュリティのため詳細は非表示） |

---

### UC-007: ビジネスアイデア提案レポートを受け取る

**Actor**: システム（スケジューラ）
**前提条件**: 1ヶ月分の蓄積ログが存在する（UC-005 の一部として実行）
**関連要件**: F-011, F-009

#### 基本フロー
1. UC-005（月次分析）の中で business_idea 観点の分析が実行される
2. Claude API が以下を掛け合わせて3〜5件のアイデアを提案:
   - YouTube 行動シグナルから推定した関心カテゴリ TOP5（高評価・プレイリスト・購読を元に集計）
   - 日記に頻出するキーワード・関心事
   - 学習フィルターで抽出されたスキル・興味領域
3. 各アイデアは「タイトル」「概要」「実現ヒント」の形式で出力
4. AnalysisReport（type: business_idea）に保存
5. Analysis 画面の「アイデア」タブに表示

#### 入力
| 項目 | 型 | 説明 |
|---|---|---|
| （なし） | - | UC-005（月次分析）の中で自動実行される |

#### 出力
| 項目 | 型 | 説明 |
|---|---|---|
| analysis_report | AnalysisReport | type: business_idea。content は JSON 配列形式で `[{title, summary, hint}]` の3〜5件を含む |

#### 状態変更
- AnalysisReport（type: business_idea）が1件新規保存される
- Analysis 画面の「アイデア」タブに新しいアイデア一覧が表示される

#### 業務ルール
- アイデアは実在のビジネス・プロジェクトではなく「可能性の提示」であることを UI で明示する
- 既存のアイデア（前回分析）との差分も表示して成長を可視化する

---

### UC-008: PLAUD 音声メモを Zapier 経由で自動取り込みする

**Actor**: システム（外部 Webhook）
**前提条件**: PLAUD アプリと Zapier が連携済みで、Zapier Zap が本システムの Webhook エンドポイントに向いている
**関連要件**: F-012

> **注**: PLAUD はハードウェア音声レコーダー。録音 → PLAUD 側で文字起こし → Zapier トリガー発火 → 本システムの Webhook エンドポイントにテキストが POST される。音声ファイルはサーバーに保存しない。

#### 基本フロー
1. PLAUD での録音・文字起こしが完了すると Zapier トリガーが発火する
2. Zapier が `POST /api/webhooks/zapier/plaud` に以下を送信する:
   - 文字起こしテキスト（`transcript`）
   - 録音日時（`recorded_at`）
   - タイトル・メモ名（`title`、任意）
3. システムが Webhook シークレットでリクエストを検証する
4. 文字起こしテキストから VoiceDiary レコードを作成する（`source: plaud`）
5. `recorded_at` の日付に対応する DailyLog が存在しなければ自動作成して紐づける

#### 入力（Webhook ペイロード）
| フィールド | 型 | 必須 | 説明 |
|---|---|---|---|
| transcript | string | 必須 | PLAUD が生成した文字起こしテキスト |
| recorded_at | string (ISO8601) | 必須 | 録音日時 |
| title | string | 任意 | PLAUD 上のメモ名 |

#### 出力
| 項目 | 型 | 説明 |
|---|---|---|
| voice_diary_id | integer | 作成された VoiceDiary の ID |

#### 状態変更
- VoiceDiary が `source: plaud` で作成される（`audio_path` は null）
- 対応する DailyLog が存在しなければ自動作成される

#### 業務ルール
- Webhook シークレット（`ZAPIER_WEBHOOK_SECRET`）を `.env` で管理し、リクエストヘッダーで検証する
- `recorded_at` が取得できない場合は受信日時を使用する
- テキストが空の場合はスキップしてエラーを返す（400）
- ブラウザ録音（UC-001）と併用可能。どちらも VoiceDiary として同一テーブルに保存される

#### エラーケース
| ケース | HTTPステータス | 対処 |
|---|---|---|
| シークレット不一致 | 401 | リクエストを拒否、ログに記録 |
| transcript が空 | 400 | スキップ、Zapier 側にエラー返却 |
| recorded_at のパース失敗 | - | 受信日時で代替して処理継続 |

---

## 承認記録

| 日付 | レビュアー | 結果 | コメント |
|---|---|---|---|
| 2026-04-12 | - | 叩き台作成 | Gate 2 承認前。レビューを経て Gate 2 を通過すること |
