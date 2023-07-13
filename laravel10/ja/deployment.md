# デプロイ

- [はじめに](#introduction)
- [サーバー要件](#server-requirements)
- [サーバー構成](#server-configuration)
     - [Nginx](#nginx)
- [最適化](#optimization)
     - [オートローダーの最適化](#autoloader-optimization)
     - [構成の読み込みの最適化](#optimizing-configuration-loading)
     - [ルート読み込みの最適化](#optimizing-route-loading)
     - [ビューの読み込みの最適化](#optimizing-view-loading)
- [デバッグモード](#debug-mode)
- [Forge / Vapor でのデプロイ](#deploying-with-forge-or-vapor)

<a name="introduction"></a>
## はじめに

Laravel アプリケーションを本番環境にデプロイする準備が整ったら、アプリケーションができる限り効率的に動作するようにするためにできることがいくつかあります。このドキュメントでは、Laravel アプリケーションが適切にデプロイされるようにするための素晴らしいスタート地点について説明します。

<a name="server-requirements"></a>
## サーバー要件

Laravel フレームワークにはいくつかのシステム要件があります。Web サーバーが以下の最低 PHP バージョンと拡張機能を持っていることを確認してください。

<div class="content-list" markdown="1">

- PHP >= 8.1
- Ctype PHP Extension
- cURL PHP Extension
- DOM PHP Extension
- Fileinfo PHP Extension
- Filter PHP Extension
- Hash PHP Extension
- Mbstring PHP Extension
- OpenSSL PHP Extension
- PCRE PHP Extension
- PDO PHP Extension
- Session PHP Extension
- Tokenizer PHP Extension
- XML PHP Extension

</div>

<a name="server-configuration"></a>
## サーバー構成

<a name="nginx"></a>
### Nginx

アプリケーションを Nginx が動作しているサーバーにデプロイする場合は、以下の設定ファイルをウェブサーバー設定の出発点として使用できます。ほとんどの場合、このファイルはサーバーの設定に応じてカスタマイズする必要があります。**サーバーの管理に関する支援が必要な場合は、[Laravel Forge](https://forge.laravel.com) などの Laravel の第一級のサーバー管理およびデプロイメントサービスを利用することを検討してください。**

以下の設定のように、Web サーバーがすべてのリクエストをアプリケーションの `public/index.php` ファイルにリダイレクトするようにしてください。プロジェクトのルートに `index.php` ファイルを移動しようとしないでください。プロジェクトのルートからアプリケーションを提供すると、多くの機密設定ファイルがインターネットに公開されることになります。

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name example.com;
    root /srv/example.com/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

<a name="optimization"></a>
## 最適化

<a name="autoloader-optimization"></a>
### オートローダーの最適化

本番環境にデプロイする際は、Composer のクラスオートローダーマップを最適化して、Composer が指定されたクラスのロードに適切なファイルをすばやく見つけられるようにすることを確認してください。

```shell
composer install --optimize-autoloader --no-dev
```

> **Note**
> オートローダーを最適化することに加えて、プロジェクトのソース管理リポジトリに常に `composer.lock` ファイルを含めるようにしてください。`composer.lock` ファイルが存在する場合、プロジェクトの依存関係がはるかに高速にインストールされます。

<a name="optimizing-configuration-loading"></a>
### 設定の読み込みの最適化

アプリケーションを本番環境にデプロイする際は、デプロイメントプロセス中に `config:cache` Artisan コマンドを実行していることを確認してください。

```shell
php artisan config:cache
```

このコマンドは、Laravel のすべての設定ファイルを 1 つのキャッシュされたファイルに結合し、フレームワークが設定値を読み込む際にファイルシステムへのアクセス回数を大幅に減らします。

> **Warning**
> デプロイプロセス中に `config:cache` コマンドを実行する場合は、設定ファイル内からのみ `env` 関数を呼び出していることを確認してください。設定がキャッシュされると `.env` ファイルは読み込まれなくなり `.env` 変数に対する `env` 関数の呼び出し結果はすべて `null` を返します。

<a name="optimizing-route-loading"></a>
### ルートの読み込みの最適化

多くのルートを持つ大規模なアプリケーションを構築している場合は、デプロイメントプロセス中に `route:cache` Artisan コマンドを実行していることを確認してください。

```shell
php artisan route:cache
```

このコマンドは、キャッシュファイルの１つのメソッド呼び出しにすべてのルート登録をまとめるので、数百のルートを登録した際、ルート登録のパフォーマンスを向上させます。

<a name="optimizing-view-loading"></a>
### ビューの読み込みの最適化

アプリケーションを本番環境にデプロイする際は、デプロイメントプロセス中に `view:cache` Artisan コマンドを実行していることを確認してください。

```shell
php artisan view:cache
```

このコマンドはすべての Blade ビューを事前にコンパイルし、オンデマンドでコンパイルされず、ビューを返す各リクエストのパフォーマンスが向上します。

<a name="debug-mode"></a>
＃＃ デバッグモード

`config/app.php` 設定ファイル内のデバッグオプションは、エラーに関する情報が実際にユーザーに表示される量を決定します。デフォルトでは、このオプションはアプリケーションの `.env` ファイルに格納されている `APP_DEBUG` 環境変数の値を尊重するように設定されています。

**本番環境では、この値は常に `false` `にする必要があります。APP_DEBUG` 変数が本番環境で `true` に設定されている場合、アプリケーションのエンドユーザーに機密性の高い設定値が晒されるリスクがあります。**

<a name="deploying-with-forge-or-vapor"></a>
## Forge / Vapor でのデプロイ

<a name="laravel-forge"></a>
#### Laravel Forge

自分でサーバー設定を管理する準備ができていない場合や、堅牢な Laravel アプリケーションを実行するために必要なさまざまなサービスの設定に慣れていない場合、[Laravel Forge](https://forge.laravel.com) は素晴らしい代替手段です。

Laravel Forge は、DigitalOcean、Linode、AWS などのさまざまなインフラストラクチャプロバイダでサーバーを作成することができます。さらに、Forge は Nginx、MySQL、Redis、Memcached、Beanstalk などの堅牢な Laravel アプリケーションを構築するために必要なすべてのツールをインストールおよび管理します。

> **Note**
> Laravel Forge でデプロイするための完全なガイドが必要ですか? [Laravel Bootcamp](https://bootcamp.laravel.com/deploying) と Forge  の [Laracasts で利用可能なビデオ シリーズ](https://laracasts.com/series/learn-laravel-forge-2022-edition) を確認してください。 ）。

<a name="laravel-vapor"></a>
#### Laravel Vapor

Laravel 用にチューニングされた、完全にサーバーレスで自動スケーリングするデプロイメントプラットフォームが欲しい場合は、[Laravel Vapor](https://vapor.laravel.com) を試してください。Laravel Vapor は、AWS を搭載した Laravel のサーバーレスデプロイメントプラットフォームです。Vapor 上で Laravel インフラを立ち上げ、サーバーレスのシンプルさをスケーラブルに実現しましょう。Laravel Vapor は、フレームワークとシームレスに連携できるように Laravel の開発者によって微調整されており、今まで通りの Laravel アプリケーションを書き続けることができます。
