# サービスプロバイダー

- [はじめに](#introduction)
- [サービスプロバイダの作成](#writing-service-providers)
     - [register メソッド](#the-register-method)
     - [boot メソッド](#the-boot-method)
- [プロバイダの登録](#registering-providers)
- [遅延プロバイダ](#deferred-providers)

<a name="introduction"></a>
## はじめに

サービスプロバイダは、Laravel アプリケーションの起動を処理する中心的な場所です。あなたが作成するアプリケーション同様に、Laravel のすべてのコアサービスは、サービスプロバイダを経由して **初期起動処理** を行っています。

しかし、「**初期起動処理**」とは何を意味するのでしょうか？ 一般的には、サービスコンテナによる結合、イベントリスナ、ミドルウェア、さらにはルートの登録など、**登録する** ことを意味します。サービスプロバイダは、アプリケーションを設定するための中心的な場所です。

Laravel に含まれている `config/app.php` ファイルを開くと、`providers` 配列がありあす。これは、アプリケーションにロードされるすべてのサービスプロバイダクラスです。デフォルトでは、Laravelコアサービスプロバイダの一連のセットがこの配列にリストされています。これらのプロバイダは、メーラー、キュー、キャッシュなどの Laravel コアコンポーネントを処理起動処理します。これらのプロバイダの多くは「遅延」プロバイダです。つまり、すべてのリクエストでロードされるわけではなく、提供されるサービスが実際に必要になったときにのみロードされます。

この概要では、独自のサービス プロバイダーを作成し、それらを Laravel アプリケーションに登録する方法を学習します。
この概要では、独自のサービスプロバイダを作成し、Laravelアプリケーションに登録する方法を学びます。

> **Note**
> Laravel がリクエストを処理し、内部でどのように機能するかについて詳しく知りたい場合は、Laravel [リクエスト ライフサイクル](/docs/{{version}}/lifecycle) に関するドキュメントをご覧ください。

<a name="writing-service-providers"></a>
## サービスプロバイダの作成

すべてのサービスプロバイダは、`Illuminate\Support\ServiceProvider` クラスを拡張します。 ほとんどのサービス プロバイダには、「register」メソッドと「boot」メソッドが含まれています。 `register` メソッド内では、**[サービスコンテナー](/docs/{{version}}/container)** にバインドするものだけを記述してください。`register` メソッド内で、イベントリスナ、ルート、またはその他の機能を登録しようとしないでください。

Artisan CLI は、`make:provider` コマンドを使って新しいプロバイダを生成できます。

```shell
php artisan make:provider RiakServiceProvider
```

<a name="the-register-method"></a>
### register メソッド

前述の通り、`register `メソッド内では、[サービスコンテナ](/docs/{{version}}/container) にのみバインドするものだけを記述する必要があります。`register` メソッド内で、イベントリスナやルート、またはその他の機能を登録しないでください。理由として、まだロードされていないサービスプロバイダによって提供されるサービスを誤って使用してしまう可能性があります。

基本的なサービスプロバイダを見てみましょう。どのサービスプロバイダメソッド内でも、サービスコンテナへのアクセスを提供する `$app` プロパティに常にアクセスできます。

    <?php

    namespace App\Providers;

    use App\Services\Riak\Connection;
    use Illuminate\Contracts\Foundation\Application;
    use Illuminate\Support\ServiceProvider;

    class RiakServiceProvider extends ServiceProvider
    {
        /**
         * Register any application services.
         */
        public function register(): void
        {
            $this->app->singleton(Connection::class, function (Application $app) {
                return new Connection(config('riak'));
            });
        }
    }

このサービスプロバイダは `register` メソッドのみを定義し、そのメソッドを使用してサービスコンテナ内で  `App\Services\Riak\Connection` の実装を定義しています。Laravel のサービスコンテナにまだ慣れていない場合は、[ドキュメント](/docs/{{version}}/container) を確認してください。

<a name="the-bindings-and-singletons-properties"></a>
#### `bindings` と `singletons` プロパティ

サービスプロバイダがシンプルな結合を多く登録している場合は、各コンテナ結合を手動で登録する代わりに、`bindings` および `singletons` プロパティを使用することをお勧めします。フレームワークによってサービスプロバイダがロードされると、自動的にこれらのプロパティをチェックし、結合情報を登録します。

    <?php

    namespace App\Providers;

    use App\Contracts\DowntimeNotifier;
    use App\Contracts\ServerProvider;
    use App\Services\DigitalOceanServerProvider;
    use App\Services\PingdomDowntimeNotifier;
    use App\Services\ServerToolsProvider;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * All of the container bindings that should be registered.
         *
         * @var array
         */
        public $bindings = [
            ServerProvider::class => DigitalOceanServerProvider::class,
        ];

        /**
         * All of the container singletons that should be registered.
         *
         * @var array
         */
        public $singletons = [
            DowntimeNotifier::class => PingdomDowntimeNotifier::class,
            ServerProvider::class => ServerToolsProvider::class,
        ];
    }

<a name="the-boot-method"></a>
### boot メソッド

サービスプロバイダ内で [ビューコンポーザ](/docs/{{version}}/views#view-composers) を登録する必要がある場合はどうすればよいでしょうか？これは `boot` メソッド内で行う必要があります。**このメソッドは、他のすべてのサービスプロバイダが登録された後に呼び出されます。** つまり、フレームワークによって登録された他のすべてのサービスにアクセスできます。

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\View;
    use Illuminate\Support\ServiceProvider;

    class ComposerServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         */
        public function boot(): void
        {
            View::composer('view', function () {
                // ...
            });
        }
    }

<a name="boot-method-dependency-injection"></a>
#### boot メソッドの依存性注入

サービスプロバイダの `boot` メソッドには、依存関係をタイプヒントとして指定できます。[サービスコンテナ](/docs/{{version}}/container) は、必要な依存関係を自動的に注入します。

    use Illuminate\Contracts\Routing\ResponseFactory;

    /**
     * Bootstrap any application services.
     */
    public function boot(ResponseFactory $response): void
    {
        $response->macro('serialized', function (mixed $value) {
            // ...
        });
    }

<a name="registering-providers"></a>
## プロバイダの登録

すべてのサービスプロバイダは `config/app.php` 設定ファイルに登録されています。このファイルには `providers` 配列が含まれており、サービスプロバイダのクラス名を登録できます。この配列は、デフォルトで、Laravel コアサービスプロバイダの一連のセットが登録されています。これらのプロバイダは、メーラー、キュー、キャッシュなどの主要な Laravel コンポーネントを初期起動させます。

プロバイダを登録するには、この配列に追加します。

    'providers' => [
        // Other Service Providers

        App\Providers\ComposerServiceProvider::class,
    ],

<a name="deferred-providers"></a>
## 遅延プロバイダ

プロバイダが [サービスコンテナ](/docs/{{version}}/container) にコンテナ結合を登録する **のみ** の場合は、登録された結合のいずれかが実際に必要になるまで、その登録を遅延させることができます。このようなプロバイダの読み込みを遅らせることによって、リクエストごとにファイルシステムから読み込まれる必要がなくなり、アプリケーションのパフォーマンスが向上します。

Laravel は、遅延サービスプロバイダにより提供されるすべてのサービスのリストと、そのサービスプロバイダのクラス名をコンパイルして保存します。その後、登録済みサービスのいずれかを依存解決しようした際に、Laravel はそのサービスプロバイダをロードします。

プロバイダのロードを遅延させるには、`\Illuminate\Contracts\Support\DeferrableProvider` インターフェイスを実装し、`provides` メソッドを定義します。 `provides` メソッドは、プロバイダによって登録されたサービスコンテナ結合を返します。

    <?php

    namespace App\Providers;

    use App\Services\Riak\Connection;
    use Illuminate\Contracts\Foundation\Application;
    use Illuminate\Contracts\Support\DeferrableProvider;
    use Illuminate\Support\ServiceProvider;

    class RiakServiceProvider extends ServiceProvider implements DeferrableProvider
    {
        /**
         * Register any application services.
         */
        public function register(): void
        {
            $this->app->singleton(Connection::class, function (Application $app) {
                return new Connection($app['config']['riak']);
            });
        }

        /**
         * Get the services provided by the provider.
         *
         * @return array<int, string>
         */
        public function provides(): array
        {
            return [Connection::class];
        }
    }
