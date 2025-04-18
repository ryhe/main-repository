# 概要

ワークスペース一覧画面は、ユーザーが所属する企業のワークスペースを確認・管理できる画面です。

企業ごとにタブで切り替えられ、各ワークスペースはカード形式で表示されます。ワークスペースの検索や、非表示ワークスペースの表示切り替え機能も提供します。

## 表示項目

-   企業選択タブ
-   ワークスペース一覧
    -   各ワークスペースカード（番組名、放送年、サムネイル画像など）
    -   ワークスペース詳細へのリンク
-   新規ワークスペース作成ボタン（権限がある場合）
-   非表示ワークスペース表示ボタン
-   検索フィルター（番組名、放送年）

## 操作内容

-   **企業タブ切り替え:** 選択された企業のワークスペース一覧を表示します。
    -   実現メカニズム:
        -   `CompanyListTab` コンポーネントで企業リストを表示します。
        -   タブをクリックすると、選択された企業のIDが状態として保持されます。
        -   状態が変更されると、対応する企業のワークスペース一覧をAPIから再取得し、表示を更新します。
-   **ワークスペース検索（番組名、放送年）:** 入力された条件に一致するワークスペースのみを表示します。
    -   実現メカニズム:
        -   番組名と放送年の入力値を状態として保持します。
        -   入力値が変更されると、APIリクエスト時に検索パラメータとして付与し、結果を再取得・表示更新します。
-   **非表示ワークスペースの表示切り替え:** 通常表示されているワークスペースと、非表示設定のワークスペースの表示を切り替えます。
    -   実現メカニズム:
        -   表示状態（通常/非表示）を状態として保持します。
        -   ボタンクリックで状態を切り替えます。
        -   状態変更に応じて、異なるAPIエンドポイント（通常用/非表示用）にリクエストを送り、結果を表示更新します。
-   **新規ワークスペース作成:** 新規ワークスペース作成ページへ遷移します。（権限チェックあり）
-   **ワークスペース詳細表示:** 各ワークスペースカードをクリックすると、そのワークスペースの詳細ページへ遷移します。

## API

### `GET /current/user`

現在ログインしているユーザーの情報を取得します。（ページ初期表示時）

### `GET /companies`

ユーザーが所属する、または管理権限を持つ企業の一覧を取得します。

### `GET /companies/{companyId}/workspaces`

指定された企業の、表示設定されているワークスペース一覧を取得します。番組名や放送年でのフィルターが可能です。

### `GET /companies/{companyId}/workspace/hidden`

指定された企業の、非表示設定されているワークスペース一覧を取得します。番組名や放送年でのフィルターが可能です。

### `GET /workspaces/{workspaceId}`

特定のワークスペースの詳細情報を取得します。（主に他ページからの遷移や内部処理で使用）

### `DELETE /sign_out`

ユーザーをログアウトさせます。（認証トークンが無効な場合などに実行）

## コンポーネント構成（Next.js）

-   `Index (page.tsx)`：ページコンテナ。初期のユーザー認証チェックを行います。
-   `WorkspacesContent`: ワークスペース一覧関連コンポーネントのラッパー。テーマやモーダルを提供します。
-   `WorkspacesList`: ワークスペース一覧表示の主要コンポーネント。企業タブ切り替え、検索、表示/非表示切り替え、API通信などのロジックを担当します。
-   `CompanyListTab`: 企業選択のためのタブUIコンポーネント。
-   `WorkspaceCard`: 個々のワークスペース情報を表示するカードUIコンポーネント。
-   `WorkspaceModal`: 画面全体を覆うモーダル表示のための汎用コンポーネント。
-   `CompletedDialog`: 特定のアクション完了時に表示されるダイアログ。

## 状態管理

-   React Hooks (`useState`, `useEffect`, `useRef`) を使用して、選択中の企業ID、ワークスペースリスト、検索条件、表示/非表示状態などを管理します。
-   `useRouter`, `useSearchParams` (Next.js Hooks) を用いて、ページ遷移やURLクエリパラメータの読み書きを行います。
-   カスタムフック (`useRequest`, `useCurrentUser`) を利用して、APIリクエスト処理や現在のユーザー情報の取得を抽象化しています。
-   ブラウザの `localStorage` を使用して、最後に選択された企業IDを永続化します。
-   ブラウザの `sessionStorage` を使用して、認証トークンやユーザーロールなどのセッション情報を保持します。

## 権限管理

-   ページアクセス時、未ログインユーザーはログインページへリダイレクトされます。
-   ユーザーが所属する企業が存在しない場合（かつシステム管理者でない場合）、アクセスが制限されログインページへ誘導されることがあります。
-   新規ワークスペース作成ボタンは、選択中の企業に対する管理者権限（ワークスペース管理者、企業管理者、またはシステム管理者）を持つユーザーにのみ表示されます。

## エラーハンドリング

-   APIリクエスト時のエラーは、`useRequest` カスタムフック内で処理されます（具体的な処理内容はフックの実装に依存）。
-   企業情報の取得に失敗した場合、コンソールにエラーが出力されます。
-   認証トークンが無効、または存在しない場合は、自動的にログアウト処理が実行されます。 