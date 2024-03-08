# dockerを用いた環境構築


## コード使用者側
### 【共通】
1. $ git pull origin main
   - Gitからコードを取得

2. $ docker-compose up -d
   - ルートディレクトリ(docker-compose.yamlがあるディレクトリ)で上記コマンドを実行してください。


### 【Backend】
1. $ docker exec -it example-be sh
   - backendコンテナに接続します。

2. $ chmod -R 777 storage bootstrap/cache
   - 書き込み権限がないとキャッシュやログにエラーを書き込めないので、権限を付与しておきます。

3. $ composer install
   - 依存関係ライブラリのインストール

4. .env.example をコピーして.envファイルを作成します。
   DB_* APP_URL FRONTEND_URL を変更します。
    - DB内容はdocker-compose.yamlで設定している値に変更してください。
    - DB_HOSTの値はdbにしてください。（このdbはコンテナサービス名のdbです。ネットワークを共有しているコンテナ名で名前解決してくれます。）
    - APP_URL=http://localhost:9000
    - FRONTEND_URL=http://localhost (nginxを使わず各コンテナの開発サーバーで進める場合 http://localhost:3000)
   $ php artisan config:clear
    - envファイルを操作する度行ってください。

5. $ php artisan key:generate
   - .envにAPP_KEY= の値がない為新たに生成します。

6. $ php artisan storage:link
   - システムで生成したファイル等をブラウザからアクセスできるように public/storage から storage/app/public へのシンボリックリンクを張ります。

7. $ php artisan migrate
   - 最後にマイグレート行います。


### 【Frontend】
1. $ docker exec -it example-fe sh
   - frontendコンテナに接続します。

2. $ npm install
   - 依存関係パッケージをインストールします。

3. .env.example をコピーして.env.local を作成します。（開発環境では.env.localがロードされます。）
   .env.localで NEXT_PUBLIC_BACKEND_URL=http://localhost:8080 と変更します。(nginxを使わず各コンテナの開発サーバーで進める場合 :8000 fpm使うなら:9000)

4. $ npm run dev 
   - 開発サーバを起動し動作確認。 
   - http://localhost:80 にブラウザからアクセスしLaravelの初期画面が表示されればOK。
   - $ 本番環境を想定して$npm build 後に$ npm startでも動作するか確認。
    - 恐らくビルド時にESlintに邪魔されるので必要に応じてファイル修正したり、まだ使われない変数などには"// eslint-disable-line no-unused-vars" をつけて対応。



## コード作成者側手順
### 【共通】
1. $ docker-compose up -d
   - ルートディレクトリ(docker-compose.yamlがあるディレクトリ)で上記コマンドを実行してください。


### 【Backend】
1. $ docker exec -it example-be sh
   - backendコンテナに接続

2. $ composer create-project --prefer-dist "laravel/laravel=9.*" .
   $ chmod -R 777 storage bootstrap/cache
   - Laravelプロジェクトを作成します。

3. ブラウザでhttp://localhost:8080 へアクセスし動作確認してください。
   - Laravelのスタート画面が表示されればOKです。

4. $ backend/.env をエディタで開き、docker-compose.yamlで設定している値に変更してください。
   - DB_HOSTの値はdbにしてください。（このdbはコンテナサービス名のdbです。ネットワークを共有しているコンテナ名で名前解決してくれます。）
   $ php artisan config:clear
    - envファイルを操作する度行ってください。

5. $ php artisan migrate
   - コンテナに再度接続し、マイグレート行います。

### 以降breeze + React(Next)の設定
6. $ composer require laravel/breeze --dev
   $ php artisan breeze:install api
   - LaravelBreeze インストール
   - artisanコマンドを実行すると.envファイルに環境変数のFRONTEND_URLが設定されます。
   - フロントエンドとバックエンドのオリジンが異なっていても通信が行えるようにCORS(Cross-Origin Resource Sharing)の設定も行われます。
   - configフォルダのcors.phpファイルを確認すると.envファイルに設定したFRONTEND_URLがallowed_originsの配列に含まれていることが確認できます。

7. .envファイルを開き、DB_* APP_URL FRONTEND_URL を変更します。
    - DB内容はdocker-compose.yamlで設定している値に変更してください。
    - DB_HOSTの値はdbにしてください。（このdbはコンテナサービス名のdbです。ネットワークを共有しているコンテナ名で名前解決してくれます。）
    - APP_URL=http://localhost:9000
    - FRONTEND_URL=http://localhost (nginxを使わず各コンテナの開発サーバーで進める場合 http://localhost:3000)
   $ php artisan config:clear
    - envファイルを操作する度行ってください。

7. $ php artisan migrate
   - 再度マイグレート

8. ブラウザでhttp://localhost:8080 へアクセスし動作確認してください。
   - Laravel Breezeのルート(laravelのver情報)が表示されればOKです。（念のため/api/userにリクエストしてPOSTしてください～等の確認もした方が良いかもしれません。）


### 【Frontend】
1. $ docker exec -it example-fe sh
   - frontendコンテナに接続

2. $ npx create-react-app
   $ npx create-next-app . --typescript ($ npx create-next-app@11.1.3 . --use-npm のように必要に応じてver指定等, $ npm i next@12.3.2)
   $ git clone https://github.com/laravel/breeze-next.git .
   - 必要に応じてReactプロジェクト・Nextプロジェクトの作成。もしくはbreeze-nextのクローン。
   - breeze-nextをクローンした場合, 最新verはappルートなのでpageルートが使いたい場合はgithub等で古いバージョンでcheckoutすること。
      - https://github.com/laravel/breeze-next -> commit履歴 -> "convert to app router"の前のコミットのIDをコピー。
         - $ git checkout 81ce4caab419aaa3dc5f389a8cfd8afb33143bfa　

3. $ npm install
 - package.jsonが作成されるので依存関係ライブラリをインストール

4. .env.example をコピーして.env.local を作成します。（* 無い場合は作成して以下の変数を作成してください。開発環境では.env.localがロードされます。）
   .env.localで NEXT_PUBLIC_BACKEND_URL=http://localhost:8080 と変更します。(nginxを使わず各コンテナの開発サーバーで進める場合 :8000 fpm使うなら:9000)

*  (必要に応じて)TailWindCssのセットアップ  *breeze-nextをクローンした場合不要
   $ npm install -D tailwindcss postcss autoprefixer
   $ npx tailwindcss init -p
   tailwind.config.js のcontent: を以下の様に変更  - デフォルトだとbuid時クラスユーティリティを全て作ってしまいサイズが大きくなってしまう為、使用している物のみ作成するよう設定。
   """"""""""""""""""""""""""""""""""""""""""
     content: [
        "./app/**/*.{js,ts,jsx,tsx}",
        "./pages/**/*.{js,ts,jsx,tsx}",
        "./components/**/*.{js,ts,jsx,tsx}",
     ]
   """"""""""""""""""""""""""""""""""""""""""
   grobals.css を以下の3行で上書き
   """"""""""""""""""""""""""""""""""""""""""
     @tailwind base;
     @tailwind components;
     @tailwind utilities;
   """"""""""""""""""""""""""""""""""""""""""
   pages/_app.js でインポート
   """"""""""""""""""""""""""""""""""""""""""
     import '../styles/globals.css'
   """"""""""""""""""""""""""""""""""""""""""
   
### 以降TypeScriptの設定
5. $ npm install --save-dev typescript
   - 必要に応じてtypescriptをインストール。

6. $ touch tsconfig.json 
   - プロジェクトのルートディレクトリに作成します。
   - アプリケーションを起動したタイミングで中身を自動で書いてくれます。

7. $ cd src
   $ rename "s/js/tsx/;" components/*.js
   $ rename "s/js/tsx/;" components/Layouts/*.js
   $ rename "s/js/tsx/;" hooks/auth.tsx
   $ rename "s/js/tsx/;" lib/axios.js
   $ rename "s/js/tsx/;" pages/*.js
   $ rename "s/js/tsx/;" pages/password-reset/\\[token\\].js
   - 拡張子をjsからtsxへ変更します。

8. $ npm run dev 
   - 開発サーバを起動し動作確認。 
   - http://localhost:80 にブラウザからアクセスしLaravelの初期画面が表示されればOK。
   - $ 本番環境を想定して$npm build 後に$ npm startでも動作するか確認。
    - 恐らくビルド時にESlintに邪魔されるので必要に応じてファイル修正したり、まだ使われない変数などには"// eslint-disable-line no-unused-vars" をつけて対応。


* $ docker-compose down --rmi all --volumes --remove-orphans
   - コード化したインフラをテストする為に、今回作成したDocker環境を完全に消します。
   - コンテナの停止、ネットワーク・名前付きボリューム・コンテナイメージ、未定義コンテナを削除するコマンドです。
   - 最後にエディタを閉じ、プロジェクトフォルダを削除してコード使用者側の手順を実行してください。