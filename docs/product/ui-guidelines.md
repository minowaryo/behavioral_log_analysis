# UI Guidelines

> このファイルはプロジェクト固有のUI/デザイン仕様を定義する。
> AIがモック生成・フロント実装を行う際は必ずこのファイルを参照すること。

---

## カラーパレット

| 用途 | カラーコード | 備考 |
|---|---|---|
| Primary | `#[COLOR]` | メインアクション・ボタン |
| Secondary | `#[COLOR]` | サブアクション |
| Danger | `#[COLOR]` | 削除・エラー |
| Warning | `#[COLOR]` | 注意・警告 |
| Success | `#[COLOR]` | 成功・完了 |
| Background | `#[COLOR]` | ページ背景 |
| Surface | `#[COLOR]` | カード・パネル背景 |
| Text primary | `#[COLOR]` | 本文 |
| Text secondary | `#[COLOR]` | 補足テキスト |

---

## タイポグラフィ

| 用途 | フォント | サイズ | ウェイト |
|---|---|---|---|
| 見出し H1 | [FONT] | [SIZE] | Bold |
| 見出し H2 | [FONT] | [SIZE] | SemiBold |
| 本文 | [FONT] | [SIZE] | Regular |
| ラベル | [FONT] | [SIZE] | Medium |
| キャプション | [FONT] | [SIZE] | Regular |

---

## レイアウト原則

- **グリッド**: [例: 12カラム / 8px基準グリッド]
- **ブレークポイント**:
  - Mobile: `< 768px`
  - Tablet: `768px〜1024px`
  - Desktop: `> 1024px`
- **コンテナ最大幅**: `[例: 1280px]`
- **標準余白**: `[例: 16px / 24px / 32px]`

---

## コンポーネント方針

### ボタン
- Primary: メインアクション（1画面に1つまで）
- Secondary: サブアクション
- Danger: 削除・取り消し（確認ダイアログを必ず挟む）
- Ghost: ナビゲーション系

### モーダル
- 利用シーン: 確認ダイアログ・簡易フォーム入力
- 全画面遷移が必要な複雑操作にはモーダルを使わない
- モーダル内にモーダルを重ねない

### テーブル
- ページネーション: [例: 20件/ページ]
- ソート可能カラムには矢印アイコンを表示
- 空状態（0件）には必ずメッセージを表示

### フォーム
- バリデーションエラーはフィールド直下にインライン表示
- 必須項目には `*` マークを付ける
- Submit後の成功/失敗はトースト通知で伝える

---

## アイコン

- ライブラリ: [例: Heroicons / FontAwesome / Material Icons]
- サイズ規則: [例: 16px（インライン） / 20px（ボタン） / 24px（ナビ）]

---

## モック作成ルール（AI向け）

AIがモックを生成する際の指示：

1. このファイル（`ui-guidelines.md`）を読んでからHTMLを生成する
2. 対応するUC番号をファイル名に含める（例: `screen-UC006-filter.html`）
3. `docs/product/mockups/` に配置する
4. `docs/product/mockups/README.md` の画面一覧に追記する
5. 実データは使わず、ダミーデータで描画する
6. インタラクションは不要（静的HTMLで可）

### モック生成プロンプトテンプレート

```
docs/product/use-cases.md の [UC-XXX] と docs/product/ui-guidelines.md を読み、
[画面名]のHTMLモックを docs/product/mockups/screen-[UC-XXX]-[screen-name].html として生成してください。
実データは不要です。ダミーデータを使用してください。
```
