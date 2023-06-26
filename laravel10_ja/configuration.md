# 設定

- [はじめに](#introduction)
- [環境設定](#environment-configuration)
     - [環境変数の種類](#environment-variable-types)
     - [環境設定の取得](#retrieving-environment-configuration)
     - [現在の環境の判別](#determining-the-current-environment)
     - [環境ファイルの暗号化](#encrypting-environment-files)
- [設定値へのアクセス](#accessing-configuration-values)
- [設定のキャッシュ](#configuration-caching)
- [デバッグモード](#debug-mode)
- [メンテナンスモード](#maintenance-mode)

<a name="introduction"></a>
## はじめに

Laravel フレームワークの設定ファイルはすべて `config` ディレクトリに格納されています。各オプションにはドキュメントが付属しているので、ファイルを見て、利用可能なオプションに慣れ親しんでください。

これらの設定ファイルを使用して、データベース接続情報、メールサーバー情報、およびアプリケーションのタイムゾーンや暗号化キーなどのさまざまなコア設定値を設定できます。

<a name="application-overview"></a>
#### アプリケーションの概要

急いでいますか？`about` Artisanコマンドを使って、アプリケーションの設定、ドライバ、環境の簡単な概要を取得できます。

```shell
php artisan about
```

アプリケーションの概要出力の特定のセクションのみに関心がある場合は、`--only` オプションを使用してそのセクションをフィルタリングできます。

```shell
php artisan about --only=environment
```

<a name="environment-configuration"></a>
## 環境設定

アプリケーションが実行されている環境に基づいて異なる設定値を持つことがよく役立ちます。たとえば、ローカルではキャッシュドライバーが本番サーバーとは異なるものを使用する場合があります。

これを簡単にするために、Laravel は [DotEnv](https://github.com/vlucas/phpdotenv) PHP ライブラリを利用しています。 新しい Laravel のインストールでは、アプリケーションのルートディレクトリに、多くの一般的な環境変数を定義した `.env.example` ファイルが含まれます。 Laravel のインストールプロセス中に、このファイルは自動的に `.env` としてコピーされます。

`Laravel` のデフォルトの `.env` ファイルには、アプリケーションがローカルで実行されているか本番 Web サーバーで実行されているかに応じて異なる可能性があるいくつかの一般的な設定値が含まれています。これらの値は、`config` ディレクトリ内のさまざまな Laravel 設定ファイルから Laravel の `env` 関数を使用して取得されます。

チームで開発している場合は、アプリケーションに `.env.example` ファイルを含め続けることが望ましいでしょう。例の設定ファイルにプレースホルダー値を入れることで、チームの他の開発者がアプリケーションを実行するために必要な環境変数を明確に確認できます。

> **Note**
> `.env` ファイル内の任意の変数は、サーバーレベルやシステムレベルの環境変数などの外部環境変数によって上書きされることがあります。

<a name="environment-file-security"></a>
#### 環境ファイルのセキュリティ

アプリケーションを使用する各開発者/サーバーは異なる環境構成を必要とする可能性があるため、`.env` ファイルはアプリケーションのソース管理にコミットしないでください。 さらに、侵入者がソース管理リポジトリへのアクセスを取得した場合、機密性の高い資格情報が公開されるため、これはセキュリティ リスクになります。

ただし、Laravel の組み込みの [環境の暗号化](#encrypting-environment-files) を使用して環境ファイルを暗号化することができます。暗号化された環境ファイルは、ソースコントロールに安全に配置できます。

<a name="追加の環境ファイル"></a>
#### 追加の環境ファイル

アプリケーションの環境変数をロードする前に、Laravel は `APP_ENV` 環境変数が外部から提供されているかどうか、または `--env` CLI 引数が指定されているかどうかを判断します。 その場合、Laravel は `.env.[APP_ENV]` ファイルが存在する場合はそれをロードしようとします。 存在しない場合は、デフォルトの `.env` ファイルがロードされます。

<a name="environment-variable-types"></a>
### 環境変数の種類

`.env` ファイル内のすべての変数は、通常、文字列として解析されるため、`env()` 関数からより広い範囲の型を返すことができるように、いくつかの予約値が作成されています。

| `.env` 値 | `env()` 値 |
|--------------|---------------|
| true         | (bool) true   |
| (true)       | (bool) true   |
| false        | (bool) false  |
| (false)      | (bool) false  |
| empty        | (string) ''   |
| (empty)      | (string) ''   |
| null         | (null) null   |
| (null)       | (null) null   |

スペースを含む値で環境変数を定義する必要がある場合は、値を二重引用符で囲んでください。

```ini
APP_NAME="My Application"
```

<a name="retrieving-environment-configuration"></a>
### 環境設定の取得

`.env` ファイルにリストされたすべての変数は、アプリケーションがリクエストを受信したときに `$_ENV` PHPスーパーグローバルにロードされます。ただし、設定ファイル内でこれらの変数から値を取得するために `env` 関数を使用することができます。実際、Laravel の設定ファイルを見ると、多くのオプションがすでにこの関数を使用していることに気付くでしょう。

    'debug' => env('APP_DEBUG', false),

`env` 関数に渡される 2つ目の値は「デフォルト値」です。指定されたキーに環境変数が存在しない場合、この値が返されます。

<a name="determining-the-current-environment"></a>
### 現在の環境の判別

現在のアプリケーション環境は、`.env` ファイルからの `APP_ENV` 変数によって決定されます。この値にアクセスするには、`App` [ファサード](/laravel10_ja/facades) の `environment` メソッドを使用します。

    use Illuminate\Support\Facades\App;

    $environment = App::environment();

環境が指定の値と一致するかどうかを判断するために、`environment` メソッドに引数を渡すこともできます。環境が指定された値のいずれかと一致する場合、メソッドは `true` を返します。

    if (App::environment('local')) {
        // The environment is local
    }

    if (App::environment(['local', 'staging'])) {
        // The environment is either local OR staging...
    }

> **Note**  
> 現在のアプリケーション環境の検出は、サーバレベルの `APP_ENV` 環境変数を定義することで上書きすることができます。

<a name="encrypting-environment-files"></a>
### 環境ファイルの暗号化

暗号化されていない環境ファイルは、ソース管理に保存するべきではありません。しかし、Laravelでは環境ファイルを暗号化して、アプリケーションの他の部分と一緒に安全にソース管理に追加できるようになっています。

<a name="encryption"></a>
#### 暗号化

環境ファイルを暗号化するには、`env:encrypt` コマンドを使用します。

```shell
php artisan env:encrypt
```

`env:encrypt` コマンドを実行すると、`.env` ファイルが暗号化され、暗号化された内容が `.env.encrypted` ファイルに格納されます。復号化キーはコマンドの出力に表示され、安全なパスワードマネージャに保存する必要があります。独自の暗号化キーを提供したい場合は、コマンドを呼び出す際に `--key` オプションを使用します。

```shell
php artisan env:encrypt --key=3UVsEgGVK36XN82KKeyLFMhvosbZN1aF
```

> **Note**  
> 提供されるキーの長さは、使用される暗号化暗号に必要なキー長と一致する必要があります。デフォルトでは、Laravel は `AES-256-CBC` 暗号を使用し、32 文字のキーが必要です。Laravel の [encrypter](/laravel10_ja/encryption) でサポートされている任意の暗号を使用することができます。コマンドを呼び出す際に `--cipher` オプションを渡すことで行います。

アプリケーションに複数の環境ファイルがある場合（`.env` と `.env.staging` など）、`--env` オプションで暗号化するべき環境ファイルを指定できます。

```shell
php artisan env:encrypt --env=staging
```

<a name="decryption"></a>
#### 復号化

環境ファイルを復号化するには、`env:decrypt` コマンドを使用します。このコマンドは復号化キーが必要で、Laravel は  `LARAVEL_ENV_ENCRYPTION_KEY` 環境変数から取得します。

```shell
php artisan env:decrypt
```

また、キーをコマンドに直接 `--key` オプションで指定することもできます。

```shell
php artisan env:decrypt --key=3UVsEgGVK36XN82KKeyLFMhvosbZN1aF
```

`env:decrypt` コマンドが呼び出されると、Laravel は `.env.encrypted` ファイルの内容を復号化し、復号化された内容を `.env` ファイルに格納します。

カスタム暗号化暗号を使用するために、`env:decrypt` コマンドに `--cipher` オプションを提供することができます。

```shell
php artisan env:decrypt --key=qUWuNRdfuImXcKxZ --cipher=AES-128-CBC
```

アプリケーションに複数の環境ファイルがある場合（`.env` と `.env.staging` など）、`--env` オプションで復号化するべき環境ファイルを指定できます。

```shell
php artisan env:decrypt --env=staging
```

既存の環境ファイルを上書きするには、`env:decrypt` コマンドに `--force` オプションを提供します。

```shell
php artisan env:decrypt --force
```

<a name="accessing-configuration-values"></a>
## 設定値へのアクセス

アプリケーションのどこからでもグローバル `config` 関数を使用して簡単に設定値にアクセスできます。設定値は 「ドット」構文を使用してアクセスされます。これには、アクセスしたいファイルとオプションの名前が含まれます。デフォルト値も指定でき、設定オプションが存在しない場合はその値が返されます。

    $value = config('app.timezone');

    // Retrieve a default value if the configuration value does not exist...
    $value = config('app.timezone', 'Asia/Seoul');

実行時に設定値を設定するには、`config` 関数に配列を渡します。

    config(['app.timezone' => 'America/Chicago']);

<a name="configuration-caching"></a>
## 設定のキャッシュ

アプリケーションの速度を向上させるために、`config:cache` Artisan コマンドを使用して、すべての設定ファイルを単一のファイルにキャッシュする必要があります。これにより、アプリケーションの設定オプションがすべて含まれた単一のファイルがフレームワークによって素早く読み込まれます。

通常、本番環境のデプロイメントプロセスの一部として `php artisan config:cache` コマンドを実行する必要があります。このコマンドは、ローカル開発中には実行しないでください。これは、アプリケーションの開発中に設定オプションを頻繁に変更する必要があるためです。

設定がキャッシュされると、アプリケーションの `.env` ファイルはリクエストや Artisan コマンド中にフレームワークによって読み込まれません。そのため、`env` 関数は外部のシステムレベルの環境変数のみを返します。

この理由から、アプリケーションの設定（`config`）ファイル内からのみ `env` 関数を呼び出すようにしてください。Laravel のデフォルトの設定ファイルを調べることで、これに関する多くの例を見ることができます。設定値は、[上記で説明した](#accessing-configuration-values) `config` 関数を使用してアプリケーション内の任意の場所からアクセスできます。

> **Warning**  
> デプロイメントプロセス中に `config:cache` コマンドを実行する場合は、設定ファイル内からのみ `env` 関数を呼び出していることを確認してください。設定がキャッシュされると、`.env` ファイルは読み込まれず、`env` 関数は外部のシステムレベルの環境変数のみを返します。

<a name="debug-mode"></a>
## デバッグモード

`config/app.php` 設定ファイルの `debug` オプションは、エラーに関する情報がユーザーに実際に表示される量を決定します。デフォルトでは、このオプションは `.env` ファイルに格納されている` APP_DEBUG` 環境変数の値を尊重するように設定されています。

ローカル開発では、`APP_DEBUG` 環境変数を `true` に設定する必要があります。**本番環境では、この値は常に `false` にする必要があります。本番環境でこの変数が `true` に設定されている場合、アプリケーションの最終ユーザーに機密設定値が漏れるリスクがあります。**

<a name="maintenance-mode"></a>
## メンテナンスモード

アプリケーションがメンテナンスモードのとき、すべてのリクエストに対してカスタムビューが表示されます。これにより、アプリケーションを更新している間やメンテナンスを行っている間、アプリケーションを簡単に停止状態とすることができます。デフォルトのミドルウェアスタックにはメンテナンスモードのチェックが含まれています。アプリケーションがメンテナンスモードにある場合、`Symfony\Component\HttpKernel\Exception\HttpException` インスタンスがステータスコード 503 でスローされます。

メンテナンスモードを有効にするには、`down` Artisan コマンドを実行します

```shell
php artisan down
```

すべてのメンテナンスモードのレスポンスに `Refresh` HTTP ヘッダーを送信したい場合は、`down` コマンドを呼び出す際に `refresh` オプションを提供できます。`Refresh` ヘッダーは、指定された秒数後にブラウザがページを自動的に更新するように指示します

```shell
php artisan down --refresh=15
```

また、`down` コマンドに `retry` オプションを提供することもできます。これは、`Retry-After` HTTP ヘッダーの値として設定されますが、通常ブラウザはこのヘッダーを無視します

```shell
php artisan down --retry=60
```

<a name="bypassing-maintenance-mode"></a>
#### メンテナンスモードのバイパス

シークレットトークンを使用してメンテナンスモードをバイパスできるようにするには、`secret` オプションを使用してメンテナンスモードバイパストークンを指定できます

```shell
php artisan down --secret="1630542a-246b-4b66-afa1-dd72a4c43515"
```

アプリケーションをメンテナンスモードに設定した後、このトークンに一致するアプリケーションの URL にアクセスすると、メンテナンスモードバイパスクッキーをブラウザに発行します

```shell
https://example.com/1630542a-246b-4b66-afa1-dd72a4c43515
```

この隠しルートにアクセスすると、アプリケーションの `/` ルートにリダイレクトされます。クッキーがブラウザに発行されると、メンテナンスモードでないかのように、アプリケーションを通常どおり閲覧できるようになります。

> **Note**  
> メンテナンスモードのシークレットは通常、英数字とダッシュで構成されるべきです。URL で特別な意味を持つ文字（`?` など）は使用しないでください。

<a name="pre-rendering-the-maintenance-mode-view"></a>
#### メンテナンスモードビューの事前レンダリング

デプロイ中に `php artisan down` コマンドを使用する場合、Composer の依存関係や他のインフラストラクチャコンポーネントが更新されている間にアプリケーションにアクセスすると、まれにエラーに遭遇することがあります。これは、アプリケーションがメンテナンス モードにあることを確認し、テンプレートエンジンを使用してメンテナンス モードのビューをレンダリングするために、Laravel フレームワークの大部分を起動する必要があるために発生します。

このため、Laravel では、リクエストサイクルの最初に返されるメンテナンスモードビューを事前にレンダリングすることができます。このビューは、アプリケーションの依存関係が読み込まれる前にレンダリングされます。`down` コマンドの `render` オプションを使用して、選択したテンプレートを事前にレンダリングできます。

```shell
php artisan down --render="errors::503"
```

<a name="redirecting-maintenance-mode-requests"></a>
#### メンテナンスモードリクエストのリダイレクト

メンテナンスモード中は、Laravel はユーザーがアクセスしようとするすべてのアプリケーションの URL に対して、メンテナンスモードビューを表示します。必要に応じて、すべてのリクエストを特定の URL にリダイレクトするように Laravel に指示することができます。これは `redirect` オプションを使って実現できます。たとえば、すべてのリクエストを `/` URI にリダイレクトさせることができます

```shell
php artisan down --redirect=/
```

<a name="disabling-maintenance-mode"></a>
#### メンテナンスモードの無効化

メンテナンスモードを無効にするには、`up` コマンドを使用します。

```shell
php artisan up
```

> **Note**  
> `resources/views/errors/503.blade.php` に独自のテンプレートを定義することで、デフォルトのメンテナンスモードテンプレートをカスタマイズできます。

<a name="maintenance-mode-queues"></a>
#### メンテナンスモードとキュー

アプリケーションがメンテナンスモードの間、[キューに入れられたジョブ](/laravel10_ja/queues) は処理されません。アプリケーションがメンテナンスモードを終了すると、ジョブは引き続き通常どおり処理されます。

<a name="alternatives-to-maintenance-mode"></a>
#### メンテナンスモードの代替手段

メンテナンスモードではアプリケーションに数秒間のダウンタイムが必要なため、[Laravel Vapor](https://vapor.laravel.com) や [Envoyer](https://envoyer.io) などの代替手段を検討してゼロダウンタイムのデプロイを実現してください。
