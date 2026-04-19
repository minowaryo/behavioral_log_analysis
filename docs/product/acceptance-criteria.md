# acceptance-criteria.md — 受け入れ基準

> 各機能のリリース判定基準を定義します。
> QAテストの基準としても使用します。
> 命名: `AC-{UC番号}-{連番}` 形式

---

## AC-001: 音声日記を録音・保存する（UC-001 / F-001）

### 正常系

```
Given: ログイン済みユーザーが Diary 画面を開いている
When:  録音ボタンをタップして録音を開始し、再度タップして停止する
Then:  音声ファイルがサーバーにアップロードされる
  And: Whisper API により文字起こしが実行される
  And: 文字起こし結果が画面に表示される
```

```
Given: 文字起こし結果が表示されている
When:  ユーザーが内容を確認（必要に応じて編集）し、保存ボタンをタップする
Then:  VoiceDiary レコードが DB に保存される
  And: 当日の DailyLog が存在しない場合は自動的に作成される
  And: DailyLog に VoiceDiary が紐づく
```

### 異常系

```
Given: ログイン済みユーザーが 25MB を超える音声ファイルをアップロードしようとする
When:  ファイルを送信する
Then:  HTTP 422 が返却される
  And: 「音声ファイルが大きすぎます（最大25MB）」のエラーメッセージが表示される
```

```
Given: Whisper API が一時的に応答しない状態
When:  音声ファイルのアップロードが完了する
Then:  HTTP 503 が返却される
  And: 「文字起こしサービスが一時的に利用できません」のメッセージが表示される
```

```
Given: 未認証のユーザーが Diary 画面にアクセスする
When:  ページをリクエストする
Then:  HTTP 401 が返却され、ログイン画面にリダイレクトされる
```

---

## AC-002: YouTube 行動シグナルを同期する（UC-002 / F-002）

### 正常系

```
Given: ログイン済みユーザーが Settings 画面で「YouTube 連携」をタップする
When:  Google OAuth2 認可画面で youtube.readonly スコープを許可する
Then:  アクセストークン・リフレッシュトークンが YouTubeToken に保存される
  And: バックグラウンドジョブが起動し、YouTube Data API v3 から行動シグナルを取得する
  And: 同期完了後、synced_count（新規保存件数）と last_synced_at がユーザーに通知される
```

```
Given: 重複するシグナル（同一 source_type + video_id + occurred_at）が既に存在する
When:  同期ジョブが実行される
Then:  重複レコードは保存されない（新規件数は増えない）
```

### 連携解除

```
Given: YouTube 連携済みのユーザーが連携解除を行う
When:  設定画面で「YouTube 連携を解除する」を実行する
Then:  YouTubeToken（アクセストークン・リフレッシュトークン）が削除される
  And: 取得済みの YouTubeSignal データは保持される（削除されない）
```

### 異常系

```
Given: YouTube 未連携のユーザーが手動同期をリクエストする
When:  同期 API を呼び出す
Then:  HTTP 403 が返却される
  And: 「YouTube 連携が必要です」のメッセージが表示される
```

```
Given: アクセストークンが期限切れになっている
When:  同期ジョブが実行される
Then:  リフレッシュトークンで自動更新が行われる
  And: 更新成功後、シグナル取得が継続される
```

---

## AC-003: 日次ログを確認する（UC-003 / F-003）

### 正常系

```
Given: ログイン済みユーザーが Dashboard を開く
When:  日付を選択する（デフォルト: 今日）
Then:  当日の DailyLog が取得される
  And: 音声日記テキスト一覧と YouTube 行動シグナル一覧が表示される
```

```
Given: 対象日にログが存在しない
When:  その日付を選択する
Then:  「記録なし」と表示される（エラーではない）
```

### 異常系

```
Given: 認証済みユーザーが他のユーザーの DailyLog ID を直接 URL で指定する
When:  ページをリクエストする
Then:  HTTP 403 が返却される
```

---

## AC-004: 週次の自動分析レポートを受け取る（UC-004 / F-004, F-005, F-006, F-009）

### 正常系

```
Given: 毎週土曜日 AM 7:00 になる
  And: 対象ユーザーの過去7日分の DailyLog が3日以上存在する
When:  ScheduledAnalysisJob が実行される
Then:  retrospective / learning_filter / mental_trigger の3観点で Claude API が呼び出される
  And: 各観点の AnalysisReport が DB に保存される
  And: 次回 Analysis 画面を開いたときに週次レポートが表示される
```

### スキップ条件

```
Given: 過去7日分のうち DailyLog が2日以下しかない
When:  ScheduledAnalysisJob が実行される
Then:  分析はスキップされる
  And: AnalysisReport は保存されない
  And: スキップログが記録される
```

### 異常系

```
Given: Claude API がタイムアウトする
When:  AnalysisReport の生成中にタイムアウトが発生する
Then:  最大3回リトライが行われる
  And: 3回失敗後もエラーの場合、エラーログに記録される
  And: 次回スケジュール実行まで待機する（ユーザーには通知しない）
```

---

## AC-005: 月次総合 AI 分析を受け取る（UC-005 / F-004〜F-007, F-009, F-011）

### 正常系

```
Given: 毎月1日 AM 7:00 になる
  And: 対象ユーザーの過去30日分の DailyLog が10日以上存在する
When:  ScheduledAnalysisJob が実行される
Then:  5観点（retrospective / learning_filter / mental_trigger / ai_coaching / business_idea）で Claude API が呼び出される
  And: 各観点の AnalysisReport が DB に保存される
  And: Analysis 画面に「月次レポート」として表示される
```

### スキップ条件

```
Given: 過去30日分のうち DailyLog が9日以下しかない
When:  ScheduledAnalysisJob が実行される
Then:  月次分析はスキップされる
  And: AnalysisReport は保存されない
```

---

## AC-006: ログイン・認証を行う（UC-006 / F-010）

### 正常系

```
Given: アカウントが存在する
When:  正しいメールアドレスとパスワードでログインする
Then:  Dashboard 画面にリダイレクトされる
  And: セッションが HttpOnly + Secure Cookie で管理される
```

### 異常系

```
Given: 誤ったパスワードを入力する
When:  ログインフォームを送信する
Then:  HTTP 422 が返却される
  And: 「メールアドレスまたはパスワードが正しくありません」と表示される
  And: 詳細（どちらが間違いか）は表示されない
```

```
Given: 未認証ユーザーが保護されたページに直接アクセスする
When:  URL を直接リクエストする
Then:  ログインページにリダイレクトされる
```

---

## AC-007: ビジネスアイデア提案レポートを受け取る（UC-007 / F-011）

### 正常系

```
Given: 月次分析（UC-005）が正常に完了している
When:  Analysis 画面の「アイデア」タブを開く
Then:  3〜5件のビジネスアイデアが表示される
  And: 各アイデアには「タイトル」「概要」「実現ヒント」が含まれる
  And: 「これは可能性の提示であり、実在のビジネスを推奨するものではありません」の注記が表示される
```

```
Given: 前回分析のアイデアが存在する
When:  今回の月次分析が完了する
Then:  前回との差分（新規アイデア・継続アイデア）が識別できる形で表示される
```

---

## 共通受け入れ基準（全機能共通）

### セキュリティ
- [ ] 未認証ユーザーは保護されたリソースにアクセスできない（401 / ログインページリダイレクト）
- [ ] 他ユーザーのリソースにアクセスした場合は 403 が返却される
- [ ] 入力バリデーションエラーは 422 を返す
- [ ] API キー・トークンはレスポンスボディに含まれない
- [ ] CSRF トークンが全ての state 変更リクエストで検証される

### パフォーマンス
- [ ] 一覧・詳細 API は 500ms 以内に応答する（P95）
- [ ] N+1 クエリが発生していない（`with()` で Eager Loading 済み）

### テスト
- [ ] Feature Test がすべての基準をカバーしている
- [ ] テストが CI で通過している
- [ ] テスト名は `docs/product/use-cases.md` のフロー・エラーケースに対応している
