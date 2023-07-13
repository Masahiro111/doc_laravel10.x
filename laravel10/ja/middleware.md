# ミドルウェア

- [はじめに](#introduction)
- [ミドルウェアの定義](#defining-middleware)
- [ミドルウェアの登録](#registering-middleware)
    - [グローバルミドルウェア](#global-middleware)
    - [ルートへのミドルウェアの割り当て](#assigning-middleware-to-routes)
    - [ミドルウェアグループ](#middleware-groups)
    - [ミドルウェアの順序](#sorting-middleware)
- [ミドルウェアのパラメータ](#middleware-parameters)
- [ミドルウェアの修了処理](#terminable-middleware)

<a name="introduction"></a>
##  はじめに

ミドルウェアは、アプリケーションに入る HTTP リクエストを検査およびフィルタリングするための便利なメカニズムを提供します。たとえば、Laravel には、アプリケーションのユーザーが認証されていることを確認するミドルウェアが含まれています。ユーザーが認証されていない場合、ミドルウェアはユーザーをアプリケーションのログイン画面にリダイレクトします。ただし、ユーザーが認証されている場合、ミドルウェアはリクエストがアプリケーション内にさらに進むことを許可します。

追加のミドルウェアを作成して、認証以外のさまざまなタスクを実行できます。たとえば、ログミドルウェアは、アプリケーションが受信したすべてのリクエストをログに記録できるようになります。Laravel フレームワークには、認証や CSRF 保護用のミドルウェアなど、いくつかのミドルウェアが含まれています。これらのミドルウェアはすべて `app/Http/Middleware` ディレクトリにあります。

<a name="defining-middleware"></a>
## ミドルウェアの定義

新しいミドルウェアを作成するには、`make:middleware` Artisan コマンドを使用します。

```shell
php artisan make:middleware EnsureTokenIsValid
```

このコマンドは、新しい `EnsureTokenIsValid` クラスを `app/Http/Middleware` ディレクトリ内に配置します。このミドルウェアでは、指定された `token` 入力が指定された値と一致する場合にのみ、ルートへのアクセスを許可します。 それ以外の場合は、ユーザーを `home` URI にリダイレクトさせます。

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Http\Request;
    use Symfony\Component\HttpFoundation\Response;

    class EnsureTokenIsValid
    {
        /**
         * Handle an incoming request.
         *
         * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
         */
        public function handle(Request $request, Closure $next): Response
        {
            if ($request->input('token') !== 'my-secret-token') {
                return redirect('home');
            }

            return $next($request);
        }
    }

ご覧のとおり、指定された `token` がシークレットトークンと一致しない場合、ミドルウェアはクライアントに HTTP リダイレクトを返します。それ以外の場合、リクエストはさらにアプリケーションに渡されます。リクエストをアプリケーションのさらに奥深くに渡すには (ミドルウェアが「渡す」ことができるように)、`$request` を使用して `$next` コールバックを呼び出す必要があります。

ミドルウェアは、HTTP リクエストがアプリケーションに到達する前に通過する必要がある一連の「レイヤー」であると考えるのが最善です。各層はリクエストを検査したり、完全に拒否したりすることもできます。

> **Note**  
> すべてのミドルウェアは [サービスコンテナ](/docs/{{version}}/container) を介して解決されるため、ミドルウェアのコンストラクタ内で必要な依存関係をタイプヒントで指定できます。

<a name="before-after-middleware"></a>
<a name="middleware-and-responses"></a>
#### ミドルウェアとレスポンス

ミドルウェアは、リクエストをアプリケーションの奥深くに渡す前、または後にタスクを実行できます。たとえば、次のミドルウェアは、アプリケーションによってリクエストが処理される **前に** 何らかのタスクを実行します。

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Http\Request;
    use Symfony\Component\HttpFoundation\Response;

    class BeforeMiddleware
    {
        public function handle(Request $request, Closure $next): Response
        {
            // Perform action

            return $next($request);
        }
    }

一方で、以下のミドルウェアは、リクエストがアプリケーションによって処理された **後** にタスクを実行します。

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Http\Request;
    use Symfony\Component\HttpFoundation\Response;

    class AfterMiddleware
    {
        public function handle(Request $request, Closure $next): Response
        {
            $response = $next($request);

            // Perform action

            return $response;
        }
    }

<a name="registering-middleware"></a>
## ミドルウェアの登録

<a name="global-middleware"></a>
### グローバルミドルウェア

アプリケーションへのすべての HTTP リクエスト中にミドルウェアを実行したい場合は、`app/Http/Kernel.php` クラスの `$middleware` プロパティにミドルウェアクラスを登録します。

<a name="assigning-middleware-to-routes"></a>
### ルートへのミドルウェアの割り当て

ミドルウェアを特定のルートに割り当てたい場合は、ルートを定義するときに `middleware` メソッドを呼び出してください。

    use App\Http\Middleware\Authenticate;

    Route::get('/profile', function () {
        // ...
    })->middleware(Authenticate::class);

ミドルウェア名の配列を `middleware` メソッドに渡すことで、複数のミドルウェアをルートに割り当てることができます。

    Route::get('/', function () {
        // ...
    })->middleware([First::class, Second::class]);

便宜上、アプリケーションの `app/Http/Kernel.php` ファイル内のミドルウェアにエイリアスを割り当てることができます。デフォルトでは、このクラスの `$middlewareAliases` プロパティには、Laravel に含まれるミドルウェアのエントリが含まれています。独自のミドルウェアをこのリストに追加し、好きな別名を割り当てることができます。

    // Within App\Http\Kernel class...

    protected $middlewareAliases = [
        'auth' => \App\Http\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
        'cache.headers' => \Illuminate\Http\Middleware\SetCacheHeaders::class,
        'can' => \Illuminate\Auth\Middleware\Authorize::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
        'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
        'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
        'verified' => \Illuminate\Auth\Middleware\EnsureEmailIsVerified::class,
    ];

ミドルウェアのエイリアスを HTTP カーネルで定義されたら、ミドルウェアをルートに割り当てるときにそのエイリアスを使用できます。

    Route::get('/profile', function () {
        // ...
    })->middleware('auth');

<a name="excluding-middleware"></a>
#### ミドルウェアの除外

ルートグループにミドルウェアを割り当てる場合、グループ内の個々のルートに、指定したミドルウェアを適用しないようにするには、`withoutMiddleware` メソッドを使用してください。

    use App\Http\Middleware\EnsureTokenIsValid;

    Route::middleware([EnsureTokenIsValid::class])->group(function () {
        Route::get('/', function () {
            // ...
        });

        Route::get('/profile', function () {
            // ...
        })->withoutMiddleware([EnsureTokenIsValid::class]);
    });

また、特定のミドルウェアのセットをルート定義の [グループ](/docs/{{version}}/routing#route-groups) 全体から除外することもできます。

    use App\Http\Middleware\EnsureTokenIsValid;

    Route::withoutMiddleware([EnsureTokenIsValid::class])->group(function () {
        Route::get('/profile', function () {
            // ...
        });
    });

`withoutMiddleware` メソッドはルートミドルウェアのみを削除でき、[グローバルミドルウェア](#global-middleware) には適用されません。

<a name="middleware-groups"></a>
### ミドルウェアグループ

ルートへの割り当てを容易にするために、複数のミドルウェアを 1 つのキーの下にグループ化したい場合があります。これは、HTTP カーネルの `$middlewareGroups` プロパティを使用して実現できます。

Laravel では、Web および API ルートに適用できるミドルウェアは `web` および `api` ミドルウェアグループとして予め事前定義されています。これらのミドルウェアグループは、アプリケーションの `App\Providers\RouteServiceProvider` サービスプロバイダによって、対応する `web` および `api` ルート ファイル内のルートに自動的に適用されることに注意してください。

    /**
     * The application's route middleware groups.
     *
     * @var array
     */
    protected $middlewareGroups = [
        'web' => [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
            \App\Http\Middleware\VerifyCsrfToken::class,
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],

        'api' => [
            \Illuminate\Routing\Middleware\ThrottleRequests::class.':api',
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],
    ];

ミドルウェアグループは、個々のミドルウェアと同じ構文を使用してルートとコントローラ アクションに割り当てることができます。繰り返しになりますが、ミドルウェアグループを使用すると、多くのミドルウェアを一度にルートに割り当てることができ、より便利になります。

    Route::get('/', function () {
        // ...
    })->middleware('web');

    Route::middleware(['web'])->group(function () {
        // ...
    });

> **Note**  
> デフォルトで `web` と `api` ミドルウェアグループは、`App\Providers\RouteServiceProvider` によってアプリケーションの対応する `routes/web.php` および `routes/api.php` ファイルに自動的に適用されています。

<a name="sorting-middleware"></a>
### ミドルウェアの順序

ミドルウェアを特定の順序で実行する必要がある場合、ルートに割り当てられたときに、その順序を制御できないことがあります。この場合、`app/Http/Kernel.php` ファイルの `$middlewarePriority` プロパティを使用してミドルウェアの優先順位を指定できます。 このプロパティは、デフォルトでは HTTP カーネルに存在しないかもしれません。もし存在しない場合は、以下のデフォルトの定義をコピーしてください。

    /**
     * The priority-sorted list of middleware.
     *
     * This forces non-global middleware to always be in the given order.
     *
     * @var string[]
     */
    protected $middlewarePriority = [
        \Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests::class,
        \Illuminate\Cookie\Middleware\EncryptCookies::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \Illuminate\Contracts\Auth\Middleware\AuthenticatesRequests::class,
        \Illuminate\Routing\Middleware\ThrottleRequests::class,
        \Illuminate\Routing\Middleware\ThrottleRequestsWithRedis::class,
        \Illuminate\Contracts\Session\Middleware\AuthenticatesSessions::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
        \Illuminate\Auth\Middleware\Authorize::class,
    ];

<a name="middleware-parameters"></a>
## ミドルウェアのパラメータ

ミドルウェアは追加のパラメータを受け取ることもできます。 たとえば、特定のアクションを実行する前に、認証されたユーザーが特定の「役割り（ロール）」を持っていることをアプリケーションで確認する必要がある場合、追加の引数としてロール名を受け取る `EnsureUserHasRole` ミドルウェアを作成できます。

追加のミドルウェアパラメータは、`$next` 引数の後にミドルウェアに渡されます。

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Http\Request;
    use Symfony\Component\HttpFoundation\Response;

    class EnsureUserHasRole
    {
        /**
         * Handle an incoming request.
         *
         * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
         */
        public function handle(Request $request, Closure $next, string $role): Response
        {
            if (! $request->user()->hasRole($role)) {
                // Redirect...
            }

            return $next($request);
        }

    }

ミドルウェアのパラメータは、ルートを定義するときにミドルウェア名とパラメータを「:」で区切って指定できます。複数のパラメータはカンマで区切る必要があります。

    Route::put('/post/{id}', function (string $id) {
        // ...
    })->middleware('role:editor');

<a name="terminable-middleware"></a>
## ミドルウェアの修了処理

場合によっては、HTTP レスポンスがブラウザに送信された後に、ミドルウェアが何らかの作業を行う必要がある場合があります。ミドルウェアで `terminate` メソッドを定義し、Web サーバーが FastCGI を使用している場合、レスポンスがブラウザに送信された後 `terminate` メソッドが自動的に呼び出されます。

    <?php

    namespace Illuminate\Session\Middleware;

    use Closure;
    use Illuminate\Http\Request;
    use Symfony\Component\HttpFoundation\Response;

    class TerminatingMiddleware
    {
        /**
         * Handle an incoming request.
         *
         * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
         */
        public function handle(Request $request, Closure $next): Response
        {
            return $next($request);
        }

        /**
         * Handle tasks after the response has been sent to the browser.
         */
        public function terminate(Request $request, Response $response): void
        {
            // ...
        }
    }

`terminate` メソッドはリクエストとレスポンスの両方を受信する必要があります。終了処理を行うミドルウェアを定義したら、それを `app/Http/Kernel.php` ファイル内のルートまたはグローバルミドルウェアのリストに追加する必要があります。

ミドルウェアで `terminate` メソッドを呼び出すと、Laravel は [サービスコンテナ](/docs/{{version}}/container) からミドルウェアの新しいインスタンスを依存性解決します。`handle` メソッドと `terminate` メソッドが呼び出されるときに同じミドルウェアインスタンスを使用したい場合は、コンテナの `singleton` メソッドを使用してミドルウェアをコンテナに登録します。 通常、これは `AppServiceProvider` の `register` メソッドで行う必要があります。

    use App\Http\Middleware\TerminatingMiddleware;

    /**
     * Register any application services.
     */
    public function register(): void
    {
        $this->app->singleton(TerminatingMiddleware::class);
    }
