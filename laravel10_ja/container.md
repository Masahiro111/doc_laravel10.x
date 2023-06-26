# サービスコンテナ

- [はじめに](#introduction)
     - [設定なしでの依存解決](#zero-configuration-resolution)
     - [コンテナを使うタイミング](#when-to-use-the-container)
- [結合](#binding)
     - [結合の基本](#binding-basics)
     - [インターフェイスを実装に結合する](#binding-interfaces-to-implementations)
     - [コンテキスト結合](#contextual-binding)
     - [プリミティブ結合](#binding-primitives)
     - [型付き可変長引数の結合](#binding-typed-variadics)
     - [タグ付け](#tagging)
     - [結合の拡張](#extending-bindings)
- [依存の解決](#resolving)
     - [Make メソッド](#the-make-method)
     - [自動注入](#automatic-injection)
- [メソッドの呼び出しと依存注入](#method-invocation-and-injection)
- [コンテナイベント](#container-events)
- [PSR-11](#psr-11)

<a name="introduction"></a>
## はじめに

Laravel のサービスコンテナは、クラスの依存関係を管理し、依存性の注入を行う強力なツールです。依存性注入とは、基本的には以下の意味です。クラスの依存関係は、コンストラクタを通じてクラスに「注入」されるか、場合によっては「セッター」メソッドを使って行われます。

簡単な例を見てみましょう。

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Repositories\UserRepository;
    use App\Models\User;
    use Illuminate\View\View;

    class UserController extends Controller
    {
        /**
         * Create a new controller instance.
         */
        public function __construct(
            protected UserRepository $users,
        ) {}

        /**
         * Show the profile for the given user.
         */
        public function show(string $id): View
        {
            $user = $this->users->find($id);

            return view('user.profile', ['user' => $user]);
        }
    }

この例では、`UserController` はデータ ソースからユーザーを取得する必要があります。 そのため、ユーザーを取得できるサービスを**注入**します。 このコンテキストでは、`UserRepository` はおそらく [Eloquent](/docs/{{version}}/eloquent) を使用してデータベースからユーザー情報を取得します。 ただし、リポジトリは注入されているため、別の実装と簡単に交換できます。 アプリケーションをテストするときに、簡単に「モック」するか、`UserRepository` のダミー実装を作成することもできます。

Laravel コア自体に貢献するだけではなく、強力で大規模なアプリケーションを構築するには、Laravel のサービスコンテナーを深く理解することが不可欠です。

<a name="zero-configuration-resolution"></a>
### 設定なしでの依存解決

クラスに依存関係がないか、他の具体的なクラス (インターフェイスではない) にのみ依存している場合、そのクラスを解決する方法をコンテナーに指示する必要はありません。 たとえば、`routes/web.php` ファイルに次のコードを配置できます。

    <?php

    class Service
    {
        // ...
    }

    Route::get('/', function (Service $service) {
        die(get_class($service));
    });

この例では、アプリケーションの `/` ルートにアクセスすると、`Service` クラスが自動的に解決され、ルートのハンドラに注入されます。これは画期的です。これにより、設定ファイルが肥大化することなく、アプリケーションの開発を行い、依存性の注入を活用できます。

ありがたいことに、Laravel アプリケーションを構築する際に書く多くのクラスは、コンテナを介して自動的に依存関係を受け取ります。これには、[コントローラー](/docs/{{version}}/controllers)、[イベントリスナ](/docs/{{ version}}/events)、[ミドルウェア](/docs/{{version}}/middleware)  などが含まれます。さらに、[キューに入れられたジョブ](/docs/{{version}}/queues) の `handle` メソッドで依存関係を型指定することもできます。設定をせずとも、自動的に依存性注入の力を味わうと、それなしでは開発ができなくなる気がします。

<a name="when-to-use-the-container"></a>
### コンテナを使うタイミング

設定いらずで依存性の解決をすることができるため、ルートやコントローラ、イベントリスナなどで依存関係を型指定することで、コンテナと手動でやり取りすることなく作業を行うことがあると思います。例えば、現在のリクエストに簡単にアクセスできるように、ルート定義で `Illuminate\Http\Request` オブジェクトを型指定することができます。このコードを書くためにコンテナとやり取りする必要はありませんが、コンテナは依存関係の注入を裏で管理しています。

    use Illuminate\Http\Request;

    Route::get('/', function (Request $request) {
        // ...
    });

多くの場合、自動依存性注入や [ファサード](/docs/{{version}}/facades) のおかげで、コンテナから何かを手動でバインドしたり依存関係を解決したりすることなく、Laravel アプリケーションを構築することができます。 **では、いつコンテナを手動で操作するのでしょうか？** 2つのケースを見てみましょう。

１つ目のケースは、インターフェイスを実装したクラスを作成し、そのインターフェイスをルートやクラスコンストラクタに型指定する場合、コンテナ [そのインターフェイスを解決する方法をコンテナーに伝える](#binding-interfaces-to-implementations) 必要があります。２つ目のケースとしては、他の Laravel 開発者と共有する [Laravel パッケージを作成](/docs/{{version}}/packages) している場合、パッケージのサービスをコンテナにバインドする必要があるかもしれません。

<a name="binding"></a>
## 結合

<a name="binding-basics"></a>
### 結合の基本

<a name="simple-bindings"></a>
#### シンプルな結合

ほとんどのサービスコンテナの結合は [サービスプロバイダ](/docs/{{version}}/providers) 内で登録されるため、これらの例のほとんどはそのコンテキストでコンテナを使用する方法を示しています。

サービスプロバイダ内では、`$this->app` プロパティを介して常にコンテナにアクセスできます。 `bind` メソッドを使用して結合を登録できます。登録したいクラスまたはインターフェイス名、クラスのインスタンスを返すクロージャとともに渡します。

    use App\Services\Transistor;
    use App\Services\PodcastParser;
    use Illuminate\Contracts\Foundation\Application;

    $this->app->bind(Transistor::class, function (Application $app) {
        return new Transistor($app->make(PodcastParser::class));
    });

コンテナ自体をリゾルバへの引数として受け取ることに注意してください。その後、コンテナを使用して、構築中のオブジェクトのサブ依存関係を解決できます。

前述の通り、通常はサービスプロバイダ内でコンテナを操作しますが、サービスプロバイダの外でコンテナを操作したい場合は、`App` [ファサード](/docs/{{version}}/facades) を使用して行うことができます。

    use App\Services\Transistor;
    use Illuminate\Contracts\Foundation\Application;
    use Illuminate\Support\Facades\App;

    App::bind(Transistor::class, function (Application $app) {
        // ...
    });

> **Note**  
> インターフェイスに依存していないクラスをコンテナにバインドする必要はありません。コンテナは、リフレクションを使用してこれらのオブジェクトを自動的に解決できるため、これらのオブジェクトの構築方法を指示する必要はありません。

<a name="binding-a-singleton"></a>
#### シングルトン結合

`singleton` メソッドは、１回だけ解決する必要があるクラスやインターフェイスをコンテナに結合します。シングルトン結合の依存性の解決がされると、コンテナに対する後続の呼び出しで同じオブジェクトインスタンスが返されます。

    use App\Services\Transistor;
    use App\Services\PodcastParser;
    use Illuminate\Contracts\Foundation\Application;

    $this->app->singleton(Transistor::class, function (Application $app) {
        return new Transistor($app->make(PodcastParser::class));
    });

<a name="binding-scoped"></a>
#### スコープ付きのシングルトン結合

`scoped` メソッドは、特定の Laravel リクエストやジョブのライフサイクル内で 1 回だけ解決する必要があるクラスまたはインターフェイスをコンテナに結合します。 このメソッドは `singleton` メソッドに似ていますが、`scoped` メソッドを使って登録されたインスタンスは、Laravel アプリケーションが新しい「ライフサイクル」を開始するたびにフラッシュされます。たとえば、[Laravel Octane](/docs/{{version}}/octane) ワーカが新しいリクエストを処理する場合や、Laravel [キュー ワーカ](/docs/{{version}}/queues) が新しいジョブを処理する場合です。

    use App\Services\Transistor;
    use App\Services\PodcastParser;
    use Illuminate\Contracts\Foundation\Application;

    $this->app->scoped(Transistor::class, function (Application $app) {
        return new Transistor($app->make(PodcastParser::class));
    });

<a name="binding-instances"></a>
#### インスタンスの結合

`instance` メソッドを使用して、既存のオブジェクトインスタンスをコンテナに結合することもできます。 指定されたインスタンスは、コンテナへの後続の呼び出しで常に返されます。

    use App\Services\Transistor;
    use App\Services\PodcastParser;

    $service = new Transistor(new PodcastParser);

    $this->app->instance(Transistor::class, $service);

<a name="binding-interfaces-to-implementations"></a>
### インターフェースを実装に結合する

サービスコンテナの非常に強力な機能は、特定の実装にインターフェイスをバインドする機能です。 たとえば、 `EventPusher` インターフェイスと `RedisEventPusher` 実装があるとしましょう。このインターフェースの `RedisEventPusher` 実装を作成したら、次のようにサービスコンテナに登録できます。

    use App\Contracts\EventPusher;
    use App\Services\RedisEventPusher;

    $this->app->bind(EventPusher::class, RedisEventPusher::class);

このステートメントは、クラスが `EventPusher` の実装を必要とするときに `RedisEventPusher` を注入する必要があることをコンテナに指示しています。これで、コンテナによって解決されるクラスのコンストラクタで `EventPusher` インターフェイースをタイプヒントとして使用できます。Laravel アプリケーション内のコントローラー、イベントリスナー、ミドルウェア、およびその他のさまざまなタイプのクラスは、常にコンテナを使用して解決されることに注意してください。

    use App\Contracts\EventPusher;

    /**
     * Create a new class instance.
     */
    public function __construct(
        protected EventPusher $pusher
    ) {}

<a name="contextual-binding"></a>
### コンテキスト結合

場合によっては、同じインターフェイスを利用する２つのクラスに、異なる実装を注入したいことがあります。たとえば、２つのコントローラが `Illuminate\Contracts\Filesystem\Filesystem` [コントラクト](/docs/{{version}}/contracts) の異なる実装に依存している場合があります。Laravel は、この動作を定義するためのシンプルで流暢なインターフェースを提供します。

    use App\Http\Controllers\PhotoController;
    use App\Http\Controllers\UploadController;
    use App\Http\Controllers\VideoController;
    use Illuminate\Contracts\Filesystem\Filesystem;
    use Illuminate\Support\Facades\Storage;

    $this->app->when(PhotoController::class)
              ->needs(Filesystem::class)
              ->give(function () {
                  return Storage::disk('local');
              });

    $this->app->when([VideoController::class, UploadController::class])
              ->needs(Filesystem::class)
              ->give(function () {
                  return Storage::disk('s3');
              });

<a name="binding-primitives"></a>
### プリミティブ結合

場合によっては、いくつかの注入されたクラスを受け取るクラスがあり、整数のような注入されたプリミティブな値も必要とすることがあります。コンテキスト結合を使用して、クラスが必要とする任意の値を簡単に注入できます。

    use App\Http\Controllers\UserController;
    
    $this->app->when(UserController::class)
              ->needs('$variableName')
              ->give($value);

クラスが [タグ付けされた](#tagging) インスタンスの配列に依存している場合があります。`giveTagged` メソッドを使用すると、そのタグを持つすべてのコンテナバインディングを簡単に注入できます。

    $this->app->when(ReportAggregator::class)
        ->needs('$reports')
        ->giveTagged('reports');

アプリケーションの構成ファイルの 1 つから値を注入する必要がある場合は、`giveConfig` メソッドを使用できます。

    $this->app->when(ReportAggregator::class)
        ->needs('$timezone')
        ->giveConfig('app.timezone');

<a name="binding-typed-variadics"></a>
### 型付き可変引数の結合

場合によっては、可変コンストラクタ引数を使用して型付きオブジェクトの配列を受け取るクラスがある場合があります。

    <?php

    use App\Models\Filter;
    use App\Services\Logger;

    class Firewall
    {
        /**
         * The filter instances.
         *
         * @var array
         */
        protected $filters;

        /**
         * Create a new class instance.
         */
        public function __construct(
            protected Logger $logger,
            Filter ...$filters,
        ) {
            $this->filters = $filters;
        }
    }

文脈上の結合を使用して `give` メソッドに、依存性解決された `Filter` インスタンスの配列を返すクロージャを提供することで、この依存関係を解決できます。

    $this->app->when(Firewall::class)
              ->needs(Filter::class)
              ->give(function (Application $app) {
                    return [
                        $app->make(NullFilter::class),
                        $app->make(ProfanityFilter::class),
                        $app->make(TooLongFilter::class),
                    ];
              });

便宜上、`Firewall` が `Filter` インスタンスを必要とするたびに、コンテナによって依存性解決されるクラス名の配列を提供することもできます。

    $this->app->when(Firewall::class)
              ->needs(Filter::class)
              ->give([
                  NullFilter::class,
                  ProfanityFilter::class,
                  TooLongFilter::class,
              ]);

<a name="variadic-tag-dependencies"></a>
#### 可変長タグの依存関係

クラスには、特定のクラスとして型宣言された可変長引数の依存関係がある場合があります (`Report ...$reports`)。`needs` および `giveTagged` メソッドを使用すると、指定された依存関係の [tag](#tagging) を含むすべてのコンテナ結合を簡単に注入できます。

    $this->app->when(ReportAggregator::class)
        ->needs(Report::class)
        ->giveTagged('reports');

<a name="tagging"></a>
### タグ付け

場合によっては、特定の結合「カテゴリ」をすべて依存性解決する必要がある場合があります。 たとえば、多数の異なる `Report` インターフェイス実装の配列を受け取るレポート アナライザーを作成しているとします。`Report` の実装を登録したら、`tag` メソッドを使用してタグを割り当てることができます。

    $this->app->bind(CpuReport::class, function () {
        // ...
    });

    $this->app->bind(MemoryReport::class, function () {
        // ...
    });

    $this->app->tag([CpuReport::class, MemoryReport::class], 'reports');

サービスにタグが付けられたら、コンテナの `tagged` メソッドを使用して簡単に依存性解決ができます。

    $this->app->bind(ReportAnalyzer::class, function (Application $app) {
        return new ReportAnalyzer($app->tagged('reports'));
    });

<a name="extending-bindings"></a>
### 結合の拡張

`extend` メソッドを使用すると、依存性が解決済みのサービスを変更できます。 たとえば、サービスの依存性が解決されると、追加のコードを実行してサービスを装飾または構成できます。`extend` メソッドは、拡張するサービスクラスと、変更されたサービスを返すクロージャの２つの引数を受け取ります。 クロージャは、依存性解決されるサービスとコンテナインスタンスを受け取ります。

    $this->app->extend(Service::class, function (Service $service, Application $app) {
        return new DecoratedService($service);
    });

<a name="resolving"></a>
## 依存性解決

<a name="the-make-method"></a>
### `make`メソッド

`make` メソッドを使用して、コンテナからクラスインスタンスを依存性解決できます。 `make` メソッドは、依存性解決したいクラスまたはインターフェースの名前を受け取ります。

    use App\Services\Transistor;

    $transistor = $this->app->make(Transistor::class);

クラスの依存関係の一部がコンテナを介して依存性解決できない場合は、`makeWith` メソッドに連想配列を渡すことで、それらを依存注入できます。 たとえば、`Transistor` サービスに必要な `$id` コンストラクタ引数を手動で渡すことができます。

    use App\Services\Transistor;

    $transistor = $this->app->makeWith(Transistor::class, ['id' => 1]);

サービスプロバイダの外部で、`$app` 変数にアクセスできないコードの場所では、`App` [ファサード](/docs/{{version}}/facades) または、`app` [ヘルパー](/docs/{{version}}/helpers#method-app)を使用して、コンテナからクラスインスタンスを解決できます。

    use App\Services\Transistor;
    use Illuminate\Support\Facades\App;

    $transistor = App::make(Transistor::class);

    $transistor = app(Transistor::class);

コンテナで解決されるクラスに Laravel コンテナインスタンス自体を注入したい場合は、クラスのコンストラクタで `Illuminate\Container\Container` クラスをタイプヒントとして使用できます。

    use Illuminate\Container\Container;

    /**
     * Create a new class instance.
     */
    public function __construct(
        protected Container $container
    ) {}

<a name="automatic-injection"></a>
### 自動注入

依存性を解決するには他の方法もあります。コンテナによって依存性解決されるクラスのコンストラクタに依存関係をタイプヒントとして指定できます。これには、[コントローラ](/docs/{{version}}/controllers)、[イベントリスナ](/docs/ {{version}}/events)、[ミドルウェア](/docs/{{version}}/middleware) などが含まれます。さらに、[キューに入れられたジョブ](/docs/{{version}}/queues) の `handle` メソッドでも依存関係をタイプ実際には、これはほとんどのオブジェクトがコンテナによって依存性解決される方法です。

たとえば、コントローラのコンストラクタで、アプリケーションによって定義されたリポジトリをタイプヒントすることができます。 リポジトリは自動的に依存性が解決され、クラスに注入されます。

    <?php

    namespace App\Http\Controllers;

    use App\Repositories\UserRepository;
    use App\Models\User;

    class UserController extends Controller
    {
        /**
         * Create a new controller instance.
         */
        public function __construct(
            protected UserRepository $users,
        ) {}

        /**
         * Show the user with the given ID.
         */
        public function show(string $id): User
        {
            $user = $this->users->findOrFail($id);

            return $user;
        }
    }

<a name="method-invocation-and-injection"></a>
## メソッドの呼び出しと注入

コンテナがメソッドの依存関係を自動的に注入できるようにしながら、オブジェクトインスタンスでメソッドを呼び出したい場合があります。たとえば、以下のようなクラスがあるとします。

    <?php

    namespace App;

    use App\Repositories\UserRepository;

    class UserReport
    {
        /**
         * Generate a new user report.
         */
        public function generate(UserRepository $repository): array
        {
            return [
                // ...
            ];
        }
    }

コンテナを使って `generate` メソッドを以下のように呼び出すことができます。

    use App\UserReport;
    use Illuminate\Support\Facades\App;

    $report = App::call([new UserReport, 'generate']);

`call` メソッドは、任意の PHP callable を受け入れます。コンテナの `call` メソッドを使用して、依存関係を自動的に注入しながらクロージャを呼び出すこともできます。

    use App\Repositories\UserRepository;
    use Illuminate\Support\Facades\App;

    $result = App::call(function (UserRepository $repository) {
        // ...
    });

<a name="container-events"></a>
## コンテナイベント

サービスコンテナは、オブジェクトの依存性を解決するたびにイベントを発生させます。`resolving` メソッドを使用して、このイベントをリッスンできます。

    use App\Services\Transistor;
    use Illuminate\Contracts\Foundation\Application;

    $this->app->resolving(Transistor::class, function (Transistor $transistor, Application $app) {
        // Called when container resolves objects of type "Transistor"...
    });

    $this->app->resolving(function (mixed $object, Application $app) {
        // Called when container resolves object of any type...
    });

ご覧のように、依存性解決されるオブジェクトがコールバックに渡されます。利用者側に渡される前にオブジェクトに追加のプロパティを設定できます。

<a name="psr-11"></a>
## PSR-11

Laravel のサービスコンテナは [PSR-11](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container.md) インターフェイスを実装しています。 そのため、PSR-11 コンテナインターフェイスを型指定して、Laravel コンテナのインスタンスを取得できます。

    use App\Services\Transistor;
    use Psr\Container\ContainerInterface;

    Route::get('/', function (ContainerInterface $container) {
        $service = $container->get(Transistor::class);

        // ...
    });

指定された識別子が解決できない場合、例外がスローされます。識別子が一度も結合されなかった場合、例外は `Psr\Container\NotFoundExceptionInterface` のインスタンスになります。識別子がバインドされたが解決できなかった場合は、 `Psr\Container\ContainerExceptionInterface` のインスタンスがスローされます。
