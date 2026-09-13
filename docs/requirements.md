# 読書記録アプリ 仕様書

## 1. 概要

ISBN（本の裏のバーコードにある13桁の番号）を入力すると、書誌情報API「openBD」からタイトル・著者・出版社・書影（表紙画像）を自動取得して登録できる、個人用の読書記録アプリ。

- リポジトリ: `laravel_todo_app` の後継として新規作成
- 開発者: 初心者〜初中級。Todoアプリ（CRUD・Blade・SQLite）の経験あり
- 目的: 段階的に成功体験を積みながら、Todoアプリにはなかった「外部API連携」「1対多リレーション」「ページネーション」を学ぶこと

## 2. 背景・狙い

- 2026年10月から書籍レーベルサイトの保守・運用を担当予定。書誌データ（ISBN・書名・著者・版元・刊行日）の構造に事前に触れておく
- 個人開発の実績としても提示しやすい題材
- 挫折を避けるため、技術要素は最小限の追加にとどめる

## 3. 技術スタック

| 項目 | 選定 | 備考 |
|---|---|---|
| フレームワーク | Laravel（Blade） | Todoアプリと同一構成を維持 |
| DB | SQLite | ファイル1つで完結。MySQL導入コストを避ける |
| CSS | Tailwind CSS | Todoアプリと同一 |
| 開発環境 | Laravel Herd | Dockerは導入しない |
| 外部API | openBD (`https://api.openbd.jp/v1/get`) | APIキー不要、日本の書誌情報に特化 |
| HTTPクライアント | Laravel `Http` ファサード | 新規習得要素 |
| 認証 | Laravel Breeze（任意・最終ステップ） | Blade版を選択 |

**あえて導入しないもの**: Vue / React / Livewire / Inertia、Docker、本番デプロイ、画像アップロード機能、凝ったデザイン。

## 4. データモデル

### 4.1 books テーブル

| カラム | 型 | 説明 |
|---|---|---|
| id | bigint, PK | |
| isbn | string, nullable, unique | 13桁ISBN。手入力登録時はnull許容 |
| title | string | 書名（必須） |
| author | string, nullable | 著者 |
| publisher | string, nullable | 出版社 |
| published_at | date, nullable | 刊行日 |
| thumbnail_url | string, nullable | 書影URL（openBDから取得） |
| status | enum(`unread`,`reading`,`finished`) | 積読／読書中／読了。デフォルト `unread` |
| finished_at | date, nullable | 読了日。`status=finished` の時に設定 |
| created_at / updated_at | timestamp | |

`status` は PHP の Enum として実装し、キーは `Unread` / `Reading` / `Finished` とする。

### 4.2 memos テーブル（books と1対多）

| カラム | 型 | 説明 |
|---|---|---|
| id | bigint, PK | |
| book_id | bigint, FK → books.id (cascade delete) | |
| body | text | 感想メモ本文（必須） |
| created_at / updated_at | timestamp | |

1冊の本（`Book`）に複数の感想メモ（`Memo`）がぶら下がる `hasMany` / `belongsTo` の関係。

## 5. 機能一覧

### 5.1 本の登録・編集・削除（CRUD）
- 手入力での新規登録（タイトルのみ必須、他は任意）
- ISBN入力による自動取得登録（openBD連携）
  - openBDから情報が取得できた場合、フォームに自動反映してから保存確認
  - 取得できなかった場合はエラーメッセージを表示し、手入力にフォールバック
- 編集・削除

### 5.2 ステータス管理
- 積読／読書中／読了の3状態を切り替え
- 読了に変更した際、読了日を入力（未入力なら当日日付をデフォルト）
- ステータスによる一覧の絞り込み

### 5.3 感想メモ（1対多リレーション）
- 本の詳細画面から複数の感想メモを追加・編集・削除
- 作成日時順に一覧表示

### 5.4 検索・一覧
- 書名・著者名によるあいまい検索
- 1ページ20件のページネーション（`paginate(20)`）

### 5.5 認証（任意・最終ステップ）
- Laravel Breeze（Blade版）によるログイン機能
- 個人利用が前提のため、優先度は最も低い

## 6. 画面一覧

| 画面 | パス例 | 内容 |
|---|---|---|
| 本棚一覧 | `/books` | 検索・ステータス絞り込み・ページネーション付き一覧 |
| 本の詳細 | `/books/{book}` | 書誌情報＋感想メモ一覧・追加フォーム |
| 本の新規登録（手入力） | `/books/create` | タイトル等を直接入力 |
| ISBN登録 | `/books/create-by-isbn` | ISBN入力→openBD取得→内容確認→保存 |
| 本の編集 | `/books/{book}/edit` | 書誌情報・ステータス編集 |

## 7. 外部API仕様（openBD）

- エンドポイント: `GET https://api.openbd.jp/v1/get?isbn={ISBN}`
- 認証: 不要
- レスポンス: JSON配列（該当なしの場合は `[null]`）
- 主に使用するフィールド:
  - `summary.title` → タイトル
  - `summary.author` → 著者
  - `summary.publisher` → 出版社
  - `summary.pubdate` → 刊行日
  - `summary.cover` → 書影URL
- 取得失敗（該当なし・通信エラー）時はフォームにエラーメッセージを表示し、手入力を促す

## 8. 非機能要件・やらないことリスト

- Docker、本番デプロイ、画像アップロード、凝ったデザインは対象外
- テストは主要なCRUD・API連携部分にフィーチャーテストを作成（`php artisan make:test --phpunit`）
- 各開発ステップの終了時に必ずコミットする

## 9. 開発ステップ

| ステップ | 内容 | 目安時間 |
|---|---|---|
| 0 | 要件定義（本ドキュメント）／リポジトリ作成 | 30分 |
| 1 | 本の手入力CRUD（登録・一覧・編集・削除） | 2時間 |
| 2 | ISBN入力→openBD自動取得での登録 | 2時間 |
| 3 | ステータス（積読／読書中／読了）・読了日・絞り込み | 2時間 |
| 4 | 感想メモ（1対多リレーション） | 2時間 |
| 5 | 書名・著者検索＋ページネーション＋テスト追加 | 2時間 |
| 6 | （任意）Laravel Breezeによるログイン機能 | 1時間 |

ステップ1で手入力版を先に完成させ、問題発生時に「APIの問題か自コードの問題か」を切り分けやすくする。

## 10. 参考リンク

- openBD（書誌情報・書影API）: https://openbd.jp/
- Laravel HTTPクライアント: https://readouble.com/laravel/master/ja/http-client.html
- Laravel Breeze: https://readouble.com/laravel/master/ja/starter-kits.html
