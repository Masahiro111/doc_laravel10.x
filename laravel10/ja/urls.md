# URL 生成

- [はじめに](#introduction)
- [基礎](#the-basics)
    - [URL の生成](#generate-urls)
    - [現在の URL にアクセス](#accessing-the-current-url)
- [名前付きルートの URL](#urls-for-named-routes)
    - [署名付き URL](#signed-urls)
- [コントローラアクションの URL](#urls-for-controller-actions)
- [デフォルト値](#default-values)

<a name="introduction"></a>
## はじめに

Laravel には、アプリケーションの URL の生成を支援するいくつかのヘルパが用意されています。これらのヘルパは主に、テンプレートや API レスポンスでリンクを構築するとき、またはアプリケーションの別の部分へのリダイレクトレスポンスを生成するときに役立ちます。

<a name="the-basics"></a>
## 基礎

<a name="generating-urls"></a>
### URL の生成

`url` ヘルパは、アプリケーションの任意の URL を生成するために使用されます。生成された URL は、アプリケーションによって処理されている現在のリクエストのスキーム (HTTP または HTTPS) とホストを自動的に使用します。

    $post = App\Models\Post::find(1);

    echo url("/posts/{$post->id}");

    // http://example.com/posts/1

<a name="accessing-the-current-url"></a>
### 現在の URL にアクセス

`url` ヘルパにパスが指定されていない場合は、`Illuminate\Routing\UrlGenerator` インスタンスが返され、現在の URL に関する情報にアクセスできます。

    // Get the current URL without the query string...
    echo url()->current();

    // Get the current URL including the query string...
    echo url()->full();

    // Get the full URL for the previous request...
    echo url()->previous();

これらの各メソッドには、`URL` [ファサード](/docs/{{version}}/facades) 経由でアクセスすることもできます。

    use Illuminate\Support\Facades\URL;

    echo URL::current();

<a name="urls-for-named-routes"></a>
## 名前付きルートの URL

`route` ヘルパは、[名前付きルート](/docs/{{version}}/routing#named-routes) への URL を生成するために使用できます。名前付きルートを使用すると、ルート上で定義された実際の URL と結合せずに URL を生成できます。したがって、ルートの URL が変更された場合でも、`route` 関数の呼び出しを変更する必要はありません。たとえば、アプリケーションに次のように定義されたルートが含まれていると想像してください。

    Route::get('/post/{post}', function (Post $post) {
        // ...
    })->name('post.show');

このルートへの URL を生成するには、以下のように `route` ヘルパを使用します。

    echo route('post.show', ['post' => 1]);

    // http://example.com/post/1

もちろん、`route` ヘルパを使用して、複数のパラメータを持つルートの URL を生成することもできます。

    Route::get('/post/{post}/comment/{comment}', function (Post $post, Comment $comment) {
        // ...
    })->name('comment.show');

    echo route('comment.show', ['post' => 1, 'comment' => 3]);

    // http://example.com/post/1/comment/3

ルートの定義パラメータに対応しない追加の配列要素は、URL のクエリ文字列に追加されます。

    echo route('post.show', ['post' => 1, 'search' => 'rocket']);

    // http://example.com/post/1?search=rocket

<a name="eloquent-models"></a>
#### Eloquent モデル

多くの場合、[Eloquent モデル](/docs/{{version}}/eloquent) のルートキー (通常は主キー) を使用して URL を生成します。このため、パラメータ値として Eloquent モデルを渡すことができます。`route` ヘルパはモデルのルートキーを自動的に抽出します。

    echo route('post.show', ['post' => $post]);

<a name="signed-urls"></a>
### 署名付き URL

Laravel は、名前付きルートへの「署名付き URL」を簡単に作成できます。これらの URL にはクエリ文字列に「署名」ハッシュが追加されており、これにより Laravel は URL が作成されてから変更されていないことを確認できます。署名付き URL は、公にアクセス可能でありながら、URL 操作に対する保護レイヤが必要なルートに特に役立ちます。

たとえば、署名付き URL を使用して、顧客に電子メールで送信される公の「購読解除」リンクを実装できます。名前付きルートへの署名付き URL を作成するには、`URL` ファサードの `signedRoute` メソッドを使用します。

    use Illuminate\Support\Facades\URL;

    return URL::signedRoute('unsubscribe', ['user' => 1]);

指定した時間が経過すると期限切れになる一時的な署名付きルート URL を生成したい場合は、`temporarySignedRoute` メソッドを使用できます。Laravel は、一時的な署名付きルート URL を検証するときに、署名付き URL にエンコードされている有効期限のタイムスタンプが経過していないことを確認します。

    use Illuminate\Support\Facades\URL;

    return URL::temporarySignedRoute(
        'unsubscribe', now()->addMinutes(30), ['user' => 1]
    );

<a name="validating-signed-route-requests"></a>
#### 署名付きルートリクエストの検証

受信リクエストに有効な署名があることを確認するには、受信した `Illuminate\Http\Request` インスタンスで `hasValidSignature` メソッドを呼び出します。

    use Illuminate\Http\Request;

    Route::get('/unsubscribe/{user}', function (Request $request) {
        if (! $request->hasValidSignature()) {
            abort(401);
        }

        // ...
    })->name('unsubscribe');

クライアント側でページネーションを実行する場合など、アプリケーションのフロントエンドが署名付き URL にデータを追加できるようにする必要があります。その際は、`hasValidSignaturewhileIgnoring` メソッドを使用して署名付き URL を検証するときに無視するリクエストクエリパラメータを指定できます。パラメータの無視を許すと、誰でもリクエストでそれらのパラメータを変更できるようになることに注意してください。

    if (! $request->hasValidSignatureWhileIgnoring(['page', 'order'])) {
        abort(401);
    }

受信リクエストインスタンスを使用して署名付き URL を検証する代わりに、`Illuminate\Routing\Middleware\ValidateSignature` [ミドルウェア](/docs/{{version}}/middleware) をルートに割り当てることができます。まだ割り当てていない場合、HTTP カーネルの `$middlewareAliases` 配列でこのミドルウェアにエイリアスを割り当ててください。

    /**
     * アプリケーションのミドルウェアのエイリアス
     *
     * エイリアスを使用すると、ミドルウェアをルートやグループに簡単に割り当てることができます
     *
     * @var array<string, class-string|string>
     */
    protected $middlewareAliases = [
        'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
    ];

カーネルにミドルウェアを登録したら、それをルートにアタッチできます。受信リクエストに有効な署名がない場合、ミドルウェアは自動的に `403` HTTP レスポンスを返します。

    Route::post('/unsubscribe/{user}', function (Request $request) {
        // ...
    })->name('unsubscribe')->middleware('signed');

<a name="responding-to-invalid-signed-routes"></a>
#### 無効な署名付きルートへのレスポンス

有効期限が切れた署名付き URL にアクセスすると、`403` HTTP ステータスコードの一般的なエラーページが表示されます。ただし、例外ハンドラで `InvalidSignatureException` 例外のカスタム「renderable」クロージャを定義することで、この動作をカスタマイズできます。このクロージャは HTTP レスポンスを返す必要があります。

    use Illuminate\Routing\Exceptions\InvalidSignatureException;

    /**
     * Register the exception handling callbacks for the application.
     */
    public function register(): void
    {
        $this->renderable(function (InvalidSignatureException $e) {
            return response()->view('error.link-expired', [], 403);
        });
    }

<a name="urls-for-controller-actions"></a>
## コントローラアクションの URL

`action` 関数は、指定されたコントローラアクションの URL を生成します。

    use App\Http\Controllers\HomeController;

    $url = action([HomeController::class, 'index']);

コントローラメソッドがルートパラメータを受け取る場合、関数の第２引数にルートパラメータの連想配列を渡すことができます。

    $url = action([UserController::class, 'profile'], ['id' => 1]);

<a name="default-values"></a>
## デフォルト値

一部のアプリケーションでは、特定の URL パラメータに対してリクエスト全体のデフォルト値を指定したい場合があります。たとえば、ルートの多くが `{locale}` パラメータを定義していると想像してください。

    Route::get('/{locale}/posts', function () {
        // ...
    })->name('post.index');

`route` ヘルパを呼び出すたびに `locale` を渡すのは面倒です。したがって、`URL::defaults` メソッドを使用して、現在のリクエスト中に常に適用されるこのパラメータのデフォルト値を定義できます。現在のリクエストにアクセスできるように、[ルートミドルウェア](/docs/{{version}}/middleware#assigning-middleware-to-routes) からこのメソッドを呼び出すこともできます。

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\URL;
    use Symfony\Component\HttpFoundation\Response;

    class SetDefaultLocaleForUrls
    {
        /**
         * Handle an incoming request.
         *
         * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
         */
        public function handle(Request $request, Closure $next): Response
        {
            URL::defaults(['locale' => $request->user()->locale]);

            return $next($request);
        }
    }

`locale` パラメータのデフォルト値を設定すると、`route` ヘルパ経由で URL を生成するときにその値を渡す必要はなくなります。

<a name="url-defaults-middleware-priority"></a>
#### URL Defaults & Middleware Priority

Setting URL default values can interfere with Laravel's handling of implicit model bindings. Therefore, you should [prioritize your middleware](/docs/{{version}}/middleware#sorting-middleware) that set URL defaults to be executed before Laravel's own `SubstituteBindings` middleware. You can accomplish this by making sure your middleware occurs before the `SubstituteBindings` middleware within the `$middlewarePriority` property of your application's HTTP kernel.

The `$middlewarePriority` property is defined in the base `Illuminate\Foundation\Http\Kernel` class. You may copy its definition from that class and overwrite it in your application's HTTP kernel in order to modify it:

    /**
     * The priority-sorted list of middleware.
     *
     * This forces non-global middleware to always be in the given order.
     *
     * @var array
     */
    protected $middlewarePriority = [
        // ...
         \App\Http\Middleware\SetDefaultLocaleForUrls::class,
         \Illuminate\Routing\Middleware\SubstituteBindings::class,
         // ...
    ];
