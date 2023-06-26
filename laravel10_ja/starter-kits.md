# スターター キット

- [はじめに](#introduction)
- [Laravel Breeze](#laravel-breeze)
     - [インストール](#laravel-breeze-installation)
     - [Breeze & Blade](#breeze-and-blade)
     - [Breeze & React / Vue](#breeze-and-inertia)
     - [Breeze & Next.js / API](#breeze-and-next)
- [Laravel Jetstream](#laravel-jetstream)

<a name="introduction"></a>
## はじめに

新しい Laravel アプリケーションを構築するための手助けとして、認証とアプリケーションスターターキットを提供しています。これらのキットは、アプリケーションのユーザ登録・認証するために必要なルート、コントローラー、ビューを自動的に生成します。

スターターキットを使うことは歓迎しますが、必須ではありません。新しい Laravel のインストールから始めて、独自のアプリケーションをゼロから構築することもできます。どちらにせよ、素晴らしいものが作れることを確信しています！

<a name="laravel-breeze"></a>
## Laravel Breeze

[Laravel Breeze](https://github.com/laravel/breeze) は、ログイン、登録、パスワードリセット、メールアドレス確認、パスワード確認を含む Laravel の [認証機能](/laravel10_ja/authentication) をすべてシンプルかつ最小限に実装したものです。さらに、Breeze にはユーザーが名前、メールアドレス、パスワードを更新できるシンプルな「プロフィール」ページが含まれています。

Laravel Breeze のデフォルトのビューレイヤーは、シンプルな [Blade テンプレート](/laravel10_ja/blade) を使用して [Tailwind CSS](https://tailwindcss.com) でスタイルが適用されています。また、Breeze は Vue または React を使用した [Inertia](https://inertiajs.com) を使ったアプリケーションのスケルトンを作成することもできます。

Breeze は、新しい Laravel アプリケーションを始めるのに最適なスタート地点であり、[Laravel Livewire](https://laravel-livewire.com) を使って Blade テンプレートを次のレベルに引き上げるプロジェクトにも適しています。

<img src="https://laravel.com/img/docs/breeze-register.png">

#### Laravel Bootcamp

Laravel を初めて使用する場合は、[Laravel Bootcamp](https://bootcamp.laravel.com) に参加してみてください。。Laravel Bootcamp では、Breeze を使って最初の Laravel アプリケーションを構築する方法を学ぶことができます。Laravel と Breeze が提供するすべての機能を体験するのに最適な方法です。

<a name="laravel-breeze-installation"></a>
### インストール

まず、[新しい Laravel アプリケーションを作成](/laravel10_ja/installation) して、データベースを構成し、[データベースマイグレーション](/laravel10_ja/migrations) を実行してください。新しい Laravel アプリケーションを作成したら、Composer を使用して Laravel Breeze をインストールできます。

```shell
composer require laravel/breeze --dev
```

Breeze をインストールしたら、以下のドキュメントで説明されている Breeze の「スタック」のいずれかを使用して、アプリケーションのスケルトンを作成できます。

<a name="breeze-and-blade"></a>
### Breeze & Blade

Composer が Laravel Breeze パッケージをインストールしたら、`breeze:install` Artisan コマンドを実行できます。このコマンドは、認証ビュー、ルート、コントローラー、およびアプリケーションにその他のリソースを公開します。Laravel Breeze は、その機能と実装に関する完全な制御と可視性を持つために、すべてのコードをアプリケーションに公開します。

デフォルトの Breeze "スタック" は Blade スタックで、シンプルな [Blade テンプレート](/docs/{{version}}/blade) を使ってアプリケーションのフロントエンドをレンダリングします。Blade スタックは、`breeze:install` コマンドを追加の引数なしで呼び出すことでインストールできます。Breeze のスケルトンがインストールされた後、アプリケーションのフロントエンドアセットもコンパイルする必要があります。

```shell
php artisan breeze:install

php artisan migrate
npm install
npm run dev
```

次に、Web ブラウザでアプリケーションの `/login` または `/register` URL に移動できます。 Breeze のすべてのルートは、`routes/auth.php` ファイル内で定義されています。

<a name="ダークモード"></a>
#### ダークモード

アプリケーションのフロントエンドをスケルトン化する際に Breeze に「ダークモード」サポートを含めたい場合は、`breeze:install` コマンドを実行する際に `--dark` ディレクティブを指定してください。

```shell
php artisan breeze:install --dark
```

> **Note**
> アプリケーションの CSS と JavaScript のコンパイルについて詳しくは、Laravel の [Vite ドキュメント](/docs/{{version}}/vite#running-vite) をご覧ください。

<a name="breeze-and-inertia"></a>
### Breeze & React / Vue

Laravel Breeze は、[Inertia](https://inertiajs.com) フロントエンド実装を介して React と Vue の足場も提供します。 Inertia を使用すると、従来のサーバー側のルーティングとコントローラーを使用して、最新の単一ページの React および Vue アプリケーションを構築できます。

Inertia を使用すると、React と Vue のフロントエンドのパワーを、Laravel と超高速の [Vite](https://vitejs.dev) コンパイルの信じられないほどのバックエンドの生産性と組み合わせて楽しむことができます。 Inertia スタックを使用するには、`breeze:install` Artisan コマンドを実行するときに、目的のスタックとして `vue` または `react` を指定します。 Breeze のスキャフォールディングをインストールしたら、アプリケーションのフロントエンド アセットもコンパイルする必要があります。

```shell
php artisan breeze:install vue

# Or...

php artisan breeze:install react

php artisan migrate
npm install
npm run dev
```

次に、Web ブラウザでアプリケーションの `/login` または `/register` URL に移動できます。 Breeze のすべてのルートは、`routes/auth.php` ファイル内で定義されています。

<a name="server-side-rendering"></a>
#### サーバーサイドレンダリング

Breeze に [Inertia SSR](https://inertiajs.com/server-side-rendering) のサポートをスケルトン化して欲しい場合は、`breeze:install` コマンドを呼び出す際に `ssr` オプションを指定してください。

```shell
php artisan breeze:install vue --ssr
php artisan breeze:install react --ssr
```

<a name="breeze-and-next"></a>
### Breeze & Next.js / API

Laravel Breeze は、[Next](https://nextjs.org) や [Nuxt](https://nuxtjs.org) などの最新の JavaScript アプリケーションを認証するための認証 API もスケルトン化できます。始めるには、`breeze:install` Artisan コマンドを実行する際に、希望するスタックとして `api` スタックを指定してください。

```shell
php artisan breeze:install api

php artisan migrate
```

インストール中に、Breeze はアプリケーションの `.env` ファイルに `FRONTEND_URL` 環境変数を追加します。この URL は、JavaScript アプリケーションの URL にする必要があります。通常、ローカル開発時には `http://localhost:3000` になります。また、APP_URL を `http://localhost:8000` に設定しておく必要があります。これは、`serve` Artisan コマンドで使用されるデフォルトの URL です。

<a name="next-reference-implementation"></a>
#### Next.js リファレンス実装

最後に、このバックエンドを選択したフロントエンドとペアリングする準備が整いました。Breeze フロントエンドの Next リファレンス実装は [GitHub で入手可能](https://github.com/laravel/breeze-next) です。このフロントエンドは Laravel によってメンテナンスされており、Breeze が提供する従来の Blade および Inertia スタックと同じユーザーインターフェースが含まれています。

<a name="laravel-jetstream"></a>
## Laravel Jetstream

Laravel Breeze は Laravel アプリケーションを構築するためのシンプルで最小限なスタート地点を提供していますが、Jetstream はより堅牢な機能と追加のフロントエンド技術スタックでその機能を強化します。**Laravel を初めて使う方は、Laravel Breeze で基本を学んだ後、Laravel Jetstream に進むことをお勧めします。**

Jetstream は、Laravel 用の美しくデザインされたアプリケーションのスケルトンを提供し、ログイン、登録、メール認証、2要素認証、セッション管理、Laravel Sanctum を介した API サポート、およびオプションのチーム管理を含みます。Jetstream は [Tailwind CSS](https://tailwindcss.com) を使用してデザインされており、[Livewire](https://laravel-livewire.com) または [Inertia](https://inertiajs.com) ドリブンのフロントエンドスケルトンを選択できます。

Laravel Jetstream のインストールに関する完全なドキュメントは、[公式の Jetstream ドキュメント](https://jetstream.laravel.com/3.x/introduction.html) をご覧ください。
