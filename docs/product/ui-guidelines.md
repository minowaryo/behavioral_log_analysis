# UI Guidelines

> このファイルはプロジェクト固有のUI/デザイン仕様を定義する。
> AIがモック生成・フロント実装を行う際は必ずこのファイルを参照すること。

---

## デザイン方針

### モバイルファースト

- **主要利用環境**: スマートフォンブラウザ（iOS Safari / Android Chrome）
- ネイティブアプリは作らない。ブラウザ上でスマートフォンに最適化する
- デスクトップ（PC）でも崩れずに表示できること（レスポンシブ対応）

### Tailwind CSS によるブレークポイント

| ブレークポイント | 対象 | 最小幅 |
|---|---|---|
| デフォルト（無印） | スマートフォン（縦） | — |
| `sm:` | スマートフォン（横）・小型タブレット | 640px |
| `md:` | タブレット | 768px |
| `lg:` | デスクトップ | 1024px |

**原則**: スタイルは常にモバイル（デフォルト）から書き始め、`md:` や `lg:` で上書きする。

---

## レイアウト

### 基本レイアウト（モバイル）

- 1カラム縦スクロール
- ボトムナビゲーション（タブバー）で主要画面を切り替える
- ヘッダーはシンプル（タイトル＋右上にアイコンメニュー）

### ボトムナビゲーション（タブ構成）

| タブ | アイコン | 遷移先 |
|---|---|---|
| ホーム | 🏠 | Dashboard（ログ概要） |
| 日記 | 🎤 | Diary（録音・一覧） |
| 分析 | 📊 | Analysis（レポート閲覧） |
| 設定 | ⚙️ | Settings（YouTube連携・アカウント） |

### デスクトップ（`lg:`）

- サイドバーナビゲーション（左固定）に切り替える
- コンテンツエリアは中央配置（最大幅 `max-w-3xl` 程度）

---

## 音声録音 UI

### 録音ボタン

- 中央に大きな円形ボタン（タップしやすいサイズ: 80px 以上）
- 状態:
  - **待機中**: グレー、マイクアイコン
  - **録音中**: 赤、点滅アニメーション、停止アイコン
  - **処理中（Whisper 送信）**: ローディングスピナー
  - **完了**: グリーン、チェックアイコン

### 録音フロー（1画面完結）

1. 録音ボタンをタップ → 録音開始
2. もう一度タップ → 録音停止 → Whisper API に送信
3. 文字起こし結果がその場に表示される
4. 保存ボタンで確定（編集可能）

---

## カラーパレット（Tailwind CSS カスタム設定）

| 役割 | カラー（Tailwind） | 用途 |
|---|---|---|
| Primary | `indigo-600` | ボタン・アクセント |
| Danger | `red-500` | 録音中・削除 |
| Success | `green-500` | 完了・保存済み |
| Background | `gray-50` | 全体背景 |
| Card | `white` | カード・パネル |
| Text Primary | `gray-900` | 本文 |
| Text Muted | `gray-500` | サブテキスト |

---

## タイポグラフィ

- フォント: システムデフォルト（`font-sans`）
- ベースフォントサイズ: `text-base`（16px）
- 見出し: `text-xl font-bold`（ページタイトル）、`text-lg font-semibold`（セクション）
- 小テキスト: `text-sm text-gray-500`（日付・メタ情報）

---

## コンポーネント方針

| コンポーネント | 配置場所 | 備考 |
|---|---|---|
| BottomNavigation | `Components/Layout/BottomNavigation.vue` | モバイルのみ表示（`lg:hidden`） |
| SidebarNavigation | `Components/Layout/SidebarNavigation.vue` | デスクトップのみ表示（`hidden lg:block`） |
| RecordButton | `Components/Diary/RecordButton.vue` | 録音ボタン・状態管理 |
| AnalysisCard | `Components/Analysis/AnalysisCard.vue` | 分析レポートカード（タイプ別） |
| DailyLogCard | `Components/Diary/DailyLogCard.vue` | 日次ログ一覧アイテム |

---

## アクセシビリティ

- タップターゲットは最低 44×44px 以上
- カラーコントラスト比は WCAG AA 準拠（4.5:1 以上）
- `aria-label` を録音ボタン等のアイコンのみのボタンに必ず付ける

---

## 注意事項

- iOS Safari では MediaRecorder API の対応状況を事前確認すること
  - iOS 14.3 以降で対応（WebM は非対応のため MP4 / AAC を使う）
- スマートフォンでは `fixed bottom-0` のボトムナビが safe-area に干渉する場合があるため `pb-safe` 等で対処する
