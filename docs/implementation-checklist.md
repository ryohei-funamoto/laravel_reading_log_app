# 読書記録アプリ 実装チェックリスト

`docs/requirements.md` の仕様に基づく実行計画。各ステップの完了時に必ずコミットする。

## ステップ0: 準備

- [x] `docs/requirements.md` に要件定義を作成
- [x] `docs/implementation-checklist.md`（本ファイル）を作成
- [x] Gitリポジトリを初期化し、最初のコミットを作成
- [x] GitHubにリポジトリを作成しpush（任意）

## ステップ1: 本の手入力CRUD

- [ ] `Book` の Enum（`app/Enums/BookStatus.php` 等）を作成: `Unread` / `Reading` / `Finished`
- [ ] `php artisan make:model Book -mfr` でモデル・マイグレーション・ファクトリ・コントローラを作成
- [ ] `books` マイグレーションにカラムを定義（`isbn`, `title`, `author`, `publisher`, `published_at`, `thumbnail_url`, `status`, `finished_at`）
- [ ] `Book` モデルに `$fillable`／`casts`（`status` を Enum、`published_at`・`finished_at` を `date`）を設定
- [ ] `BookFactory` を作成
- [ ] ルーティング（`resource` route）を `routes/web.php` に追加
- [ ] `BookController` に一覧・作成フォーム・保存・編集フォーム・更新・削除を実装
- [ ] バリデーション用の `StoreBookRequest` / `UpdateBookRequest` を作成（`title` 必須、他は任意）
- [ ] Bladeビュー作成: 一覧（`index`）、新規登録フォーム（`create`）、編集フォーム（`edit`）、詳細（`show`）
- [ ] フィーチャーテスト作成: 一覧表示・作成・更新・削除
- [ ] `vendor/bin/pint --dirty --format agent` を実行
- [ ] テスト実行（`php artisan test --compact`）
- [ ] コミット

## ステップ2: ISBN入力→openBD自動取得

- [ ] openBD連携用のサービスクラス（例: `app/Services/OpenBdClient.php`）を作成
- [ ] `Http::get('https://api.openbd.jp/v1/get', ['isbn' => $isbn])` でISBN検索を実装
- [ ] レスポンスの `summary.title` / `author` / `publisher` / `pubdate` / `cover` をパースする処理を実装
- [ ] 該当なし・通信エラー時のハンドリング（フォームにエラーメッセージを表示し手入力へフォールバック）
- [ ] ISBN登録用ルート・コントローラアクション（`/books/create-by-isbn` 相当）を追加
- [ ] ISBN入力→取得結果プレビュー→保存確認のBladeビューを作成
- [ ] `Http::fake()` を使ったフィーチャーテスト作成（成功時・該当なし時・通信エラー時）
- [ ] `vendor/bin/pint --dirty --format agent` を実行
- [ ] テスト実行
- [ ] コミット

## ステップ3: ステータス管理・読了日・絞り込み

- [ ] 一覧・詳細・編集画面でステータス（積読／読書中／読了）を切り替えられるUIを実装
- [ ] ステータスを `finished` に変更した際、読了日入力欄を表示（未入力時は当日日付をデフォルト）
- [ ] 一覧画面にステータスによる絞り込み（クエリパラメータ）を実装
- [ ] ステータス変更・絞り込みのフィーチャーテスト作成
- [ ] `vendor/bin/pint --dirty --format agent` を実行
- [ ] テスト実行
- [ ] コミット

## ステップ4: 感想メモ（1対多リレーション）

- [ ] `php artisan make:model Memo -mf` でモデル・マイグレーション・ファクトリを作成
- [ ] `memos` マイグレーションに `book_id`（外部キー、`cascade` delete）と `body` を定義
- [ ] `Book` に `memos(): HasMany`、`Memo` に `book(): BelongsTo` を実装
- [ ] `MemoFactory` を作成
- [ ] ネストしたルート（例: `books/{book}/memos`）とコントローラを作成
- [ ] 本の詳細画面に感想メモの一覧（作成日時順）・追加フォーム・編集・削除を実装
- [ ] `StoreMemoRequest` でバリデーション（`body` 必須）
- [ ] 感想メモのCRUDに対するフィーチャーテスト作成
- [ ] `vendor/bin/pint --dirty --format agent` を実行
- [ ] テスト実行
- [ ] コミット

## ステップ5: 検索・ページネーション・テスト強化

- [ ] 一覧画面に書名・著者名によるあいまい検索（`LIKE`）を実装
- [ ] 一覧クエリに `paginate(20)` を適用し、ページ送りリンクを表示
- [ ] 検索とステータス絞り込みを同時に使えるようにクエリを組み立て
- [ ] 検索・ページネーションのフィーチャーテスト作成
- [ ] 既存テストの見直し・不足しているエッジケースの追加
- [ ] `vendor/bin/pint --dirty --format agent` を実行
- [ ] テスト実行（`php artisan test --compact`）
- [ ] コミット

## ステップ6（任意）: ログイン機能

- [ ] `composer require laravel/breeze --dev`
- [ ] `php artisan breeze:install blade --no-interaction`
- [ ] `npm install && npm run build`
- [ ] マイグレーション実行（`users` テーブル確認）
- [ ] 認証ミドルウェアを本関連ルートに適用するか検討・実装
- [ ] 認証関連のデフォルトテストが通ることを確認
- [ ] コミット

## 完了後の確認

- [ ] `docs/requirements.md` と実装内容に差異がないか確認（あれば仕様書を更新）
- [ ] READMEに簡単なセットアップ手順を追記（任意）
