# ルーティング

- [基本的なルーティング](#basic-routing)
    - [リダイレクトルート](#redirect-routes)
    - [ビュールート](#view-routes)
    - [ルートリスト](#the-route-list)
- [ルートパラメータ](#route-parameters)
    - [必須パラメータ](#required-parameters)
    - [オプションパラメータ](#parameters-optional-parameters)
    - [正規表現の制約](#parameters-regular-expression-constraints)
- [名前付きルート](#named-routes)
- [ルートグループ](#route-groups)
    - [ミドルウェア](#route-group-middleware)
    - [コントローラ](#route-group-controllers)
    - [サブドメインルーティング](#route-group-subdomain-routing)
    - [ルートプレフィックス](#route-group-prefixes)
    - [ルート名プレフィックス](#route-group-name-prefixes)
- [ルートモデル結合](#route-model-binding)
    - [暗黙的な結合](#implicit-binding)
    - [暗黙的な列挙型（Enum）結合](#implicit-enum-binding)
    - [明示的な結合](#explicit-binding)
- [フォールバックルート](#fallback-routes)
- [レート制限](#rate-limiting)
    - [レート制限の定義](#defining-rate-limiters)
    - [ルートへのレート制限の割り当て](#attaching-rate-limiters-to-routes)
- [疑似敵フォームメソッド](#form-method-spoofing)
- [現在のルートへのアクセス](#accessing-the-current-route)
- [オリジン間でのリソース共有 (CORS)](#cors)
- [ルートのキャッシュ](#route-caching)

<a name="basic-routing"></a>
## 基本的なルーティング

最も基本的な Laravel のルートは URI とクロージャを引数にとり、複雑なルーティング設定ファイルなしに、ルートと動作を定義する、非常にシンプルで表現力豊かな方法を提供します。

    use Illuminate\Support\Facades\Route;

    Route::get('/greeting', function () {
        return 'Hello World';
    });

<a name="the-default-route-files"></a>
#### デフォルトのルートファイル

Laravel のすべてのルートは、`routes` ディレクトリにあるルートファイルで定義されています。これらのファイルは、アプリケーションの `App\Providers\RouteServiceProvider` によって自動的に読み込まれます。`routes/web.php` ファイルは、Web インターフェース用のルートを定義します。これらのルートは `web` ミドルウェアグループに割り当てられ、セッション状態や CSRF 保護などの機能が提供されます。`routes/api.php` にあるルートはステートレスで、`api` ミドルウェアグループが割り当てられています。

ほとんどのアプリケーションでは、`routes/web.php` ファイルでルートを定義することから始めます。`routes/web.php` で定義されたルートは、ブラウザで定義されたルートの URL を入力することでアクセスできます。たとえば、ブラウザで `http://example.com/user` に移動すると、次のルートにアクセスできます。

    use App\Http\Controllers\UserController;

    Route::get('/user', [UserController::class, 'index']);

`routes/api.php` ファイルで定義されたルートは、`RouteServiceProvider` によってルートグループ内にネストされています。このグループ内では、`/api` URI プレフィックスが自動的に適用されるため、ファイル内のすべてのルートに手作業で適用する必要はありません。`RouteServiceProvider` クラスを変更することで、プレフィックスや他のルートグループオプションを変更できます。

<a name="available-router-methods"></a>
#### 利用可能なルーターメソッド

ルーターを使用すると、任意の HTTP 動詞に応答するルートを登録できます。

    Route::get($uri, $callback);
    Route::post($uri, $callback);
    Route::put($uri, $callback);
    Route::patch($uri, $callback);
    Route::delete($uri, $callback);
    Route::options($uri, $callback);

複数の HTTP 動詞に対応するルートを登録する必要がある場合があります。これには `match` メソッドを使用できます。また、`any` メソッドを使用して、すべての HTTP 動詞に対応するルートを登録することもできます。

    Route::match(['get', 'post'], '/', function () {
        // ...
    });

    Route::any('/', function () {
        // ...
    });

> **Note**
> 同じ URI を共有する複数のルートを定義する場合、`get`、`post`、`put`、`patch`、`delete`、および `options` メソッドを使用するルートは、`any`、`match`、および `redirect` メソッドを使用するルートよりも先に定義する必要があります。これにより、受信リクエストが正しいルートと一致するようになります

<a name="dependency-injection"></a>
#### 依存性の注入

ルートのコールバック引数に、ルートに必要な依存関係を型ヒントとして指定することができます。宣言された依存関係は、Laravel の [サービスコンテナ](/docs/{{version}}/container) によって自動的に解決され、コールバックに注入されます。たとえば、現在の HTTP リクエストをルートコールバックに自動的に注入させるには、`Illuminate\Http\Request` クラスを型ヒントとして指定できます。

    use Illuminate\Http\Request;

    Route::get('/users', function (Request $request) {
        // ...
    });

<a name="csrf-protection"></a>
#### CSRF 保護

`web` ルートファイルで定義されている `POST`、`PUT`、`PATCH`、または `DELETE` ルートを指す HTML フォームには、CSRF トークンフィールドが含まれている必要があることに注意してください。それ以外の場合、リクエストは拒否されます。CSRF 保護の詳細については、[CSRF ドキュメント](/docs/{{version}}/csrf) を参照してください。

    <form method="POST" action="/profile">
        @csrf
        ...
    </form>

<a name="redirect-routes"></a>
### リダイレクトルート

別の URI にリダイレクトするルートを定義する場合は、`Route::redirect` メソッドを使用できます。このメソッドは便利なショートカットを提供するため、単純なリダイレクトを実行するために完全なルートまたはコントローラーを定義する必要はありません。

    Route::redirect('/here', '/there');

デフォルトでは、`Route::redirect` は `302` ステータスコードを返します。オプションの 3 番目のパラメータを使用してステータスコードをカスタマイズできます。

    Route::redirect('/here', '/there', 301);

または、`Route::permanentRedirect` メソッドを使用して `301` ステータス コードを返すこともできます。

    Route::permanentRedirect('/here', '/there');

> **Warning**
> リダイレクトルートでルートパラメータを使用する場合、`destination` と `status` パラメータは Laravel によって予約されているため使用できません。

<a name="view-routes"></a>
### ビュールート

ルートが [ビュー](/docs/{{version}}/views) のみを返すだけでよい場合は、`Route::view` メソッドを使用できます。`redirect` メソッドと同様に、このメソッドは単純なショートカットを提供するため、完全なルートまたはコントローラーを定義する必要はありません。`view` メソッドは、最初の引数として URI を、2 番目の引数としてビュー名を受け取ります。 さらに、オプションの 3 番目の引数としてビューに渡すデータの配列を指定できます。

    Route::view('/welcome', 'welcome');

    Route::view('/welcome', 'welcome', ['name' => 'Taylor']);

> **Warning**
> ビュールートでルートパラメータを使用する場合、`view`、`data`、`status`、および `headers`のパラメータはLaravelによって予約されているため使用できません。

<a name="the-route-list"></a>
### ルートリスト

`route:list` Artisan コマンドを使用すると、アプリケーションによって定義されているすべてのルートの概要を簡単に提供できます。

```shell
php artisan route:list
```

デフォルトでは、各ルートに割り当てられたルートミドルウェアは `route:list` 出力には表示されません。 ただし、コマンドに `-v` オプションを追加することで、Laravel にルートミドルウェアを表示するように指示できます。

```shell
php artisan route:list -v
```

特定の URI で始まるルートのみを表示するように Laravel に指示することもできます。

```shell
php artisan route:list --path=api
```

さらに、`route:list` コマンドを実行するときに `--excel-vendor` オプションを指定することで、サードパーティのパッケージによって定義されたルートを非表示にするよう Laravel に指示することもできます。

```shell
php artisan route:list --except-vendor
```

同様に、`route:list` コマンドを実行する際に `--only-vendor` オプションを指定することで、サードパーティのパッケージによって定義されているルートのみを表示するよう Laravel に指示できます。

```shell
php artisan route:list --only-vendor
```

<a name="ルートパラメータ"></a>
## ルートパラメータ

<a name="必須パラメータ"></a>
### 必須パラメータ

ルート内で URI のセグメントを取得する必要がある場合があります。たとえば、URL からユーザー ID を取得する必要があるかもしれません。これを行うには、ルートパラメーターを定義します。

    Route::get('/user/{id}', function (string $id) {
        return 'User '.$id;
    });

ルートに必要なルートパラメータをいくつでも定義できます。

    Route::get('/posts/{post}/comments/{comment}', function (string $postId, string $commentId) {
        // ...
    });

ルートパラメータは常に `{}` 中括弧で囲まれ、アルファベット文字で構成されている必要があります。ルートパラメータ名ではアンダースコア ( `_` ) も使用できます。 ルートパラメータは、その順序に基づいてルートのコールバック / コントローラに注入されます。ルート のコールバック / コントローラ引数の名前は関係ありません。

<a name="parameters-and-dependency-injection"></a>
#### パラメータと依存関係の注入

Laravel サービスコンテナによって、ルートのコールバックへ自動的に注入したい依存関係がある際、依存関係の後にルートパラメータを記述する必要があります。

    use Illuminate\Http\Request;

    Route::get('/user/{id}', function (Request $request, string $id) {
        return 'User '.$id;
    });

<a name="パラメータ-オプションパラメータ"></a>
### オプションのパラメータ

場合によっては、URI に存在するとは限らないルートパラメータの指定が必要になることがあります。その際は、パラメータ名の後に `?` マークを付けます。ルートの対応する変数にデフォルト値を指定してください。

    Route::get('/user/{name?}', function (string $name = null) {
        return $name;
    });

    Route::get('/user/{name?}', function (string $name = 'John') {
        return $name;
    });

<a name="パラメータ-正規表現-制約"></a>
### 正規表現制約

ルートインスタンスの `where` メソッドを使用して、ルートパラメータの形式を制約できます。`where` メソッドは、パラメータ名と、パラメータがどのように制約されるべきかを定義する正規表現を引数に受け取ります。

    Route::get('/user/{name}', function (string $name) {
        // ...
    })->where('name', '[A-Za-z]+');

    Route::get('/user/{id}', function (string $id) {
        // ...
    })->where('id', '[0-9]+');

    Route::get('/user/{id}/{name}', function (string $id, string $name) {
        // ...
    })->where(['id' => '[0-9]+', 'name' => '[a-z]+']);

便宜上、一般的に使用される一部の正規表現パターンには、パターン制約をルートにすばやく追加できるヘルパメソッドが用意されています。

    Route::get('/user/{id}/{name}', function (string $id, string $name) {
        // ...
    })->whereNumber('id')->whereAlpha('name');

    Route::get('/user/{name}', function (string $name) {
        // ...
    })->whereAlphaNumeric('name');

    Route::get('/user/{id}', function (string $id) {
        // ...
    })->whereUuid('id');

    Route::get('/user/{id}', function (string $id) {
        //
    })->whereUlid('id');

    Route::get('/category/{category}', function (string $category) {
        // ...
    })->whereIn('category', ['movie', 'song', 'painting']);

受信リクエストがルートパターン制約と一致しない場合、404 HTTP レスポンスを返します。

<a name="parameters-global-constraints"></a>
#### グローバル制約

ルートパラメータを常に指定された正規表現によって制約したい場合は、`pattern` メソッドを使用できます。 これらのパターンは、 `App\Providers\RouteServiceProvider` クラスの `boot` メソッドで定義する必要があります。

    /**
     * Define your route model bindings, pattern filters, etc.
     */
    public function boot(): void
    {
        Route::pattern('id', '[0-9]+');
    }

パターンが定義されると、そのパラメータ名を使用するすべてのルートに自動的に適用されます。

    Route::get('/user/{id}', function (string $id) {
        // Only executed if {id} is numeric...
    });

<a name="parameters-encoded-forward-slashes"></a>
#### エンコードされたスラッシュ

Laravel ルーティングコンポーネントでは、`/` を除くすべての文字をルートパラメータ値内に含めることができます。 `/` をプレースホルダの一部として明示的に許可するには、`where` 条件の正規表現を使用する必要があります。

    Route::get('/search/{search}', function (string $search) {
        return $search;
    })->where('search', '.*');

> **警告**
> エンコードされたスラッシュは、最後のルートセグメント内でのみサポートされています。

<a name="named-routes"></a>
## 名前付きルート

名前付きルートを使用すると、特定のルートの URL やリダイレクトを便利に生成できます。ルート定義に `name` メソッドをチェーンしてルートに名前を指定できます。

    Route::get('/user/profile', function () {
        // ...
    })->name('profile');

コントローラアクションのルート名も指定できます。

    Route::get(
        '/user/profile',
        [UserProfileController::class, 'show']
    )->name('profile');

> **警告**
> ルート名は常に一意である必要があります。

<a name="generating-urls-to-named-routes"></a>
#### 名前付きルートの URL 生成

特定のルートに名前を割り当てたら、Laravel の `route` および `redirect` ヘルパ関数を使用して URL やリダイレクトを生成する際にルートの名前を使用できます。

    // Generating URLs...
    $url = route('profile');

    // Generating Redirects...
    return redirect()->route('profile');

    return to_route('profile');

名前付きルートがパラメータを定義している場合、パラメータを 2 番目の引数として `route` 関数に渡すことができます。 指定されたパラメータは、生成された URL の正しい位置に自動的に挿入されます。

    Route::get('/user/{id}/profile', function (string $id) {
        // ...
    })->name('profile');

    $url = route('profile', ['id' => 1]);

追加のパラメーターを配列で渡すと、それらのキーと値のペアが、生成された URL のクエリ文字列に自動的に追加されます。

    Route::get('/user/{id}/profile', function (string $id) {
        // ...
    })->name('profile');

    $url = route('profile', ['id' => 1, 'photos' => 'yes']);

    // /user/1/profile?photos=yes

> **注意**
> 現在のロケールなど、URL パラメータにリクエスト全体のデフォルト値を指定したい場合があります。これを実現するには、[`URL::defaults` メソッド](/docs/{{version}}/urls#default-values) を使用できます。

<a name="inspecting-the-current-route"></a>
#### 現在のルートの検査

現在のリクエストが指定された名前付きルートにルーティングされたかどうかを確認したい場合は、Route インスタンスで `named` メソッドを使用してください。たとえば、ルートミドルウェアから現在のルート名を確認できます。

    use Closure;
    use Illuminate\Http\Request;
    use Symfony\Component\HttpFoundation\Response;

    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        if ($request->route()->named('profile')) {
            // ...
        }

        return $next($request);
    }

<a name="route-groups"></a>
## ルートグループ

ルートグループを使用すると、個別のルートごとにミドルウェアなどのルート属性を定義することなく、多数のルート間でルート属性を共有できます。

ネストされたグループは、属性を親グループとインテリジェントに「マージ」しようとします。 ミドルウェアと `where` 条件はマージされ、名前とプレフィックス（接頭辞）が追加されます。 名前空間の区切り文字と URI プレフィックスのスラッシュは、必要に応じて自動的に追加されます。

<a name="route-group-middleware"></a>
### ミドルウェア

[ミドルウェア](/docs/{{version}}/middleware) をグループ内のすべてのルートに割り当てるには、グループを定義する前に `middleware` メソッドを使用します。 ミドルウェアは、配列にリストされている順序で実行されます。

    Route::middleware(['first', 'second'])->group(function () {
        Route::get('/', function () {
            // Uses first & second middleware...
        });

        Route::get('/user/profile', function () {
            // Uses first & second middleware...
        });
    });

<a name="route-group-controllers"></a>
### コントローラー

ルートのグループがすべて同じ [コントローラ](/docs/{{version}}/controllers) を利用する場合、`controller` メソッドを使用して、グループ内のすべてのルートに共通のコントローラを定義できます。ルートを定義するときに、ルートが呼び出すコントローラメソッドを指定するだけで済みます。

    use App\Http\Controllers\OrderController;

    Route::controller(OrderController::class)->group(function () {
        Route::get('/orders/{id}', 'show');
        Route::post('/orders', 'store');
    });

<a name="route-group-subdomain-routing"></a>
### サブドメインルーティング

ルートグループは、サブドメインルーティングを処理するためにも使用できます。サブドメインには、ルート URI のようにルートパラメータを割り当てることができます。これにより、ルートやコントローラでサブドメインの一部をキャプチャできます。`domain` メソッドを呼び出してグループを定義する前にサブドメインを指定します。

    Route::domain('{account}.example.com')->group(function () {
        Route::get('user/{id}', function (string $account, string $id) {
            // ...
        });
    });

> **警告**
> サブドメインルートに確実に到達できるようにするには、ドメインルートを登録する前にサブドメインルートを登録する必要があります。これにより、ドメインルートが同じ URI パスを持つサブドメインルートを上書きすることがなくなります。

<a name="route-group-prefixes"></a>
### ルートプレフィックス

`prefix` メソッドは、グループ内の各ルートに指定された URI をプレフィックスとして追加するために使用できます。たとえば、グループ内のすべてのルート URI に `admin` をプレフィックスとして追加することができます。

    Route::prefix('admin')->group(function () {
        Route::get('/users', function () {
            // Matches The "/admin/users" URL
        });
    });

<a name="route-group-name-prefixes"></a>
### ルート名のプレフィックス

`name` メソッドを使用すると、グループ内の各ルート名に特定の文字列をプレフィックスとして付けることができます。 たとえば、グループ内のすべてのルートの名前に `admin` というプレフィックスを付けることができます。指定された文字列は、指定されたとおりにルート名にプレフィックスとして付けられるため、プレフィックスの末尾に `.` 文字を必ず指定してください。

    Route::name('admin.')->group(function () {
        Route::get('/users', function () {
            // Route assigned name "admin.users"...
        })->name('users');
    });

<a name="route-model-binding"></a>
## ルートモデル結合

モデル ID をルートまたはコントローラアクションに挿入する場合、多くの場合、データベースにクエリを実行して、その ID に対応するモデルを取得します。 Laravel ルートモデル 結合では、モデルインスタンスをルートに直接自動的に注入する便利な方法を提供します。 たとえば、ユーザーの ID を注入する代わりに、指定された ID に一致する `User` モデルインスタンス全体を注入できます。

<a name="implicit-binding"></a>
### 暗黙的な結合

Laravel は、ルートセグメント名と一致する型付けされた変数名を持つルート、またはコントローラアクションで定義された Eloquent モデルを自動的に解決します。以下のような例があります。

    use App\Models\User;

    Route::get('/users/{user}', function (User $user) {
        return $user->email;
    });

`$user` 変数は `App\Models\User` Eloquent モデルとして型付けされています。変数名は `{user}` セグメントと一致するため、Laravel は自動的に、リクエスト URI からの対応する値に一致する ID を持つモデルインスタンスを注入します。一致するモデルインスタンスがデータベース内に見つからない場合、404 HTTP 応答が自動的に生成されます。

もちろん、コントローラメソッドを使用する場合は、暗黙的な結合も可能です。 再度、`{user}` URI セグメントが、`App\Models\User` タイプヒントを含むコントローラの `$user` 変数と一致することに注意してください。

    use App\Http\Controllers\UserController;
    use App\Models\User;

    // Route definition...
    Route::get('/users/{user}', [UserController::class, 'show']);

    // Controller method definition...
    public function show(User $user)
    {
        return view('user.profile', ['user' => $user]);
    }

<a name="implicit-soft-deleted-models"></a>
#### ソフトデリートされたモデル

通常、暗黙的な結合では、[ソフトデリート](/docs/{{version}}/eloquent#soft-deleting) されたモデルは取得されません。ただし、ルートの定義に `withTrashed` メソッドをチェーンすることで、暗黙的な結合であったとしても、ソフトデリート済みの情報を含むモデルを取得するできるようになります。

    use App\Models\User;

    Route::get('/users/{user}', function (User $user) {
        return $user->email;
    })->withTrashed();

<a name="customizing-the-key"></a>
<a name="customizing-the-default-key-name"></a>
#### キーのカスタマイズ

`id` 以外のカラムでEloquent モデルを解決したい場合があります。その場合、ルートパラメータ定義でカラムを指定できます。

    use App\Models\Post;

    Route::get('/posts/{post:slug}', function (Post $post) {
        return $post;
    });

特定のモデル クラスを取得するときに、モデル結合で常に `id` 以外のデータベースカラムを使用するようにしたい場合は、Eloquent モデルの `getRouteKeyName` メソッドをオーバーライドできます。

    /**
     * Get the route key for the model.
     */
    public function getRouteKeyName(): string
    {
        return 'slug';
    }

<a name="implicit-model-binding-scoping"></a>
#### カスタムキーとスコープ

単一のルート定義で複数の Eloquent モデルを暗黙的に結合する場合、前の Eloquent モデルの子である必要があるように 2 番目の Eloquent モデルのスコープを設定したい場合があります。たとえば、特定のユーザーのブログ投稿のスラッグによってを取得する次のルート定義について考えてみましょう。

    use App\Models\Post;
    use App\Models\User;

    Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
        return $post;
    });

カスタムキー付きの暗黙的な結合をネストしたルートパラメータとして使用する場合、Laravel は親の関連名を推測するための規約を使用して、親からネストされたモデルを取得するためのクエリを自動的にスコープします。この場合、`User` モデルには、`Post` モデルを取得するために使用できる `posts`（ルートパラメータ名の複数形）という名前のリレーションシップがあると想定されます。

必要に応じて、カスタムキーが提供されていない場合でも、Laravel に「子」結合のスコープを設定するように指示できます。 これを行うには、ルートを定義するときに `scopeBindings` メソッドを呼び出します。

    use App\Models\Post;
    use App\Models\User;

    Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
        return $post;
    })->scopeBindings();

または、グループのルート定義全体にスコープ付きの結合を使用するように指示することもできます。

    Route::scopeBindings()->group(function () {
        Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
            return $post;
        });
    });

同様に、`withoutScopedBindings` メソッドを呼び出すことで、Laravel に明示的な結合をスコープしないように指示することもできます

    Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
        return $post;
    })->withoutScopedBindings();

<a name="customizing-missing-model-behavior"></a>
#### モデルが見つからない場合の振る舞いのカスタマイズ

通常、暗黙的に結合されたモデルが見つからない場合は、404 HTTP レスポンスが生成されます。ただし、ルートを定義するときに `missing` メソッドを呼び出すことで、この動作をカスタマイズできます。 `missing` メソッドは、暗黙的に結合されたモデルが見つからない場合に呼び出されるクロージャを引数に取ります。

    use App\Http\Controllers\LocationsController;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Redirect;

    Route::get('/locations/{location:slug}', [LocationsController::class, 'show'])
            ->name('locations.view')
            ->missing(function (Request $request) {
                return Redirect::route('locations.index');
            });

<a name="implicit-enum-binding"></a>
### 暗黙的な列挙型（Enum）結合

PHP 8.1 では、[Enums](https://www.php.net/manual/en/ language.enumerations.backed.php) のサポートが導入されました。この機能を補完するために、Laravel ではルート定義で [値に依存した列挙型](https://www.php.net/manual/en/language.enumerations.backed.php) をタイプヒントで指定できます。ルートセグメントが有効な Enum 値に対応する場合、ルートを呼び出します。それ以外の場合は、404 HTTP レスポンスが自動的に返されます。たとえば、次の列挙型があるとします。

```php
<?php

namespace App\Enums;

enum Category: string
{
    case Fruits = 'fruits';
    case People = 'people';
}
```

この場合、`{category}` ルートセグメントが `fruits` または `people` である場合にのみ呼び出されるルートを定義できます。それ以外の場合は、Laravel が 404 HTTP レスポンスを返します

```php
use App\Enums\Category;
use Illuminate\Support\Facades\Route;

Route::get('/categories/{category}', function (Category $category) {
    return $category->value;
});
```

<a name="explicit-binding"></a>
### 明示的な結合

モデル結合を活用するために、Laravel の暗黙的な規約ベースのモデル解決を使用する必要はありません。ルートパラメーターがモデルにどのように対応するかを明示的に定義することもできます。明示的な結合を登録するには、ルーターの `model` メソッドを使用して、特定のパラメーターのクラスを指定します。`RouteServiceProvider`  クラスの `boot` メソッドの先頭で明示的なモデル結合を定義する必要があります。

    use App\Models\User;
    use Illuminate\Support\Facades\Route;

    /**
     * Define your route model bindings, pattern filters, etc.
     */
    public function boot(): void
    {
        Route::model('user', User::class);

        // ...
    }

次に `{user}` パラメータを含むルートを定義します。

    use App\Models\User;

    Route::get('/users/{user}', function (User $user) {
        // ...
    });

すべての `{user}` パラメータを `App\Models\User` モデルに結合しているため、そのクラスのインスタンスがルートに注入されます。たとえば、`users/1` へのリクエストは、ID が `1` であるデータベースから `User` インスタンスを注入します。

一致するモデルインスタンスがデータベース内に見つからない場合、404 HTTP レスポンスが自動的に生成されます。

<a name="customizing-the-resolution-logic"></a>
#### モデル結合の解決ロジックのカスタマイズ

独自のモデル結合の解決ロジックを定義したい場合は、`Route::bind` メソッドを使用できます。`bind` メソッドに渡すクロージャは、URI セグメントの値を受け取り、ルートに注入されるクラスのインスタンスを返す必要があります。繰り返しますが、このカスタマイズはアプリケーションの `RouteServiceProvider` の `boot` メソッドで行う必要があります。

    use App\Models\User;
    use Illuminate\Support\Facades\Route;

    /**
     * Define your route model bindings, pattern filters, etc.
     */
    public function boot(): void
    {
        Route::bind('user', function (string $value) {
            return User::where('name', $value)->firstOrFail();
        });

        // ...
    }

あるいは、Eloquent モデルの `resolveRouteBinding` メソッドをオーバーライドする方法もあります。このメソッドは URI セグメントの値を受け取り、ルートに注入されるクラスのインスタンスを返す必要があります。

    /**
     * Retrieve the model for a bound value.
     *
     * @param  mixed  $value
     * @param  string|null  $field
     * @return \Illuminate\Database\Eloquent\Model|null
     */
    public function resolveRouteBinding($value, $field = null)
    {
        return $this->where('name', $value)->firstOrFail();
    }

ルートが [暗黙的な結合のスコープ](#implicit-model-binding-scoping) を利用している場合、親モデルの子結合を解決するために `resolveChildRouteBinding` メソッドを使用することができます。

    /**
     * Retrieve the child model for a bound value.
     *
     * @param  string  $childType
     * @param  mixed  $value
     * @param  string|null  $field
     * @return \Illuminate\Database\Eloquent\Model|null
     */
    public function resolveChildRouteBinding($childType, $value, $field)
    {
        return parent::resolveChildRouteBinding($childType, $value, $field);
    }

<a name="fallback-routes"></a>
## フォールバックルート

`Route::fallback` メソッドを使用すると、受信リクエストに一致するルートが他にない場合に実行されるルートを定義できます。通常、未処理のリクエストは、アプリケーションの例外ハンドラを介して自動的に「404」ページをレンダリングします。ただし、通常は `routes/web.php` ファイル内で `fallback` ルートを定義するため `web` ミドルウェアグループ内のすべてのミドルウェアがルートに適用されます。必要に応じて、このルートにミドルウェアを自由に追加できます。

    Route::fallback(function () {
        // ...
    });

> **Warning**
> フォールバックルートは常にアプリケーションで最後に登録されるルートである必要があります。

<a name="rate-limiting"></a>
## レート制限

<a name="defining-rate-limiters"></a>
### レート制限の定義

Laravel には、特定のルートまたはルートのグループのトラフィック量を制限するために利用できる、強力でカスタマイズ可能なレート制限サービスが含まれています。まず、アプリケーションのニーズを満たすレート制限の設定を定義する必要があります。 通常、これはアプリケーションの `App\Providers\RouteServiceProvider` クラスの `configureRateLimiting` メソッド内で行う必要があります。このメソッドには、アプリケーションの `routes/api.php` ファイル内のルートに適用されるレート制限定義がすでに含まれています。

```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;

/**
 * Configure the rate limiters for the application.
 */
protected function configureRateLimiting(): void
{
    RateLimiter::for('api', function (Request $request) {
        return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
    });
}
```

レート制限は、`RateLimiter` ファサードの `for` メソッドを使用して定義されます。`for` メソッドは、レート制限名と、レート制限に割り当てられたルートに適用される制限設定を返すクロージャを引数に受け入れます。制限設定は、`Illuminate\Cache\RateLimiting\Limit` クラスのインスタンスです。このクラスには、制限をすばやく定義できる便利な「ビルダ」メソッドが含まれています。レート制限名には、任意の文字列を指定できます。

    use Illuminate\Cache\RateLimiting\Limit;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\RateLimiter;

    /**
     * Configure the rate limiters for the application.
     */
    protected function configureRateLimiting(): void
    {
        RateLimiter::for('global', function (Request $request) {
            return Limit::perMinute(1000);
        });
    }

受信リクエストが指定したレート制限を超える場合、Laravel は自動的に 429 のHTTP ステータスコードのレスポンスを返します。レート制限によって返す独自のレスポンスを定義したい場合は、`response` メソッドを使用できます

    RateLimiter::for('global', function (Request $request) {
        return Limit::perMinute(1000)->response(function (Request $request, array $headers) {
            return response('Custom response...', 429, $headers);
        });
    });

レート制限コールバックは受信 HTTP リクエストインスタンスを受け取るため、受信リクエストや認証済みのユーザに基づいて適切なレート制限を動的に構築できます。

    RateLimiter::for('uploads', function (Request $request) {
        return $request->user()->vipCustomer()
                    ? Limit::none()
                    : Limit::perMinute(100);
    });

<a name="segmenting-rate-limits"></a>
#### レート制限のセグメント化

場合によっては、レート制限を任意の値で分割したい場合があります。たとえば、ユーザーが IP アドレスごとに 1 分あたり 100 回、特定のルートにアクセスできるようにしたい場合、これを実現するには、レート制限を構築するときに `by` メソッドを使用できます。

    RateLimiter::for('uploads', function (Request $request) {
        return $request->user()->vipCustomer()
                    ? Limit::none()
                    : Limit::perMinute(100)->by($request->ip());
    });

この機能を別の例で説明すると、認証済みユーザー ID ごとに 1 分間に 100 回、またはゲストの IP アドレスごとに 1 分間に 10 回、ルートへのアクセスを制限することができます

    RateLimiter::for('uploads', function (Request $request) {
        return $request->user()
                    ? Limit::perMinute(100)->by($request->user()->id)
                    : Limit::perMinute(10)->by($request->ip());
    });

<a name="multiple-rate-limits"></a>
#### 複数のレート制限

必要に応じて、特定のレート制限設定のレート制限の配列を返すことができます。各レート制限は、配列内に配置された順序に基づいてルートに対して評価されます。

    RateLimiter::for('login', function (Request $request) {
        return [
            Limit::perMinute(500),
            Limit::perMinute(3)->by($request->input('email')),
        ];
    });

<a name="attaching-rate-limiters-to-routes"></a>
### レート制限をルートに付加する

レート制限は、`throttle` [ミドルウェア](/docs/{{version}}/middleware) を使用してルートまたはルートグループに付加できます。スロットルミドルウェアは、ルートに割り当てるレート制限名を引数に受け入れます。

    Route::middleware(['throttle:uploads'])->group(function () {
        Route::post('/audio', function () {
            // ...
        });

        Route::post('/video', function () {
            // ...
        });
    });

<a name="throttling-with-redis"></a>
#### Redis を使用したスロットリング

通常、`throttle` ミドルウェアは`Illuminate\Routing\Middleware\ThrottleRequests` クラスにマップされます。このマッピングは、アプリケーションの HTTP カーネル (`App\Http\Kernel`) で定義されます。ただし、アプリケーションのキャッシュドライバーとして Redis を使用している場合は、`Illuminate\Routing\Middleware\ThrottleRequestsWithRedis` クラスを使用するように変更することをお勧めします。このクラスは、Redis を使用したレート制限の管理においてより効率的です。

    'throttle' => \Illuminate\Routing\Middleware\ThrottleRequestsWithRedis::class,

<a name="form-method-spoofing"></a>
## 疑似フォームメソッド

HTML フォームは、`PUT`、`PATCH`、`DELETE` アクションをサポートしていません。したがって、HTML フォームから呼び出される `PUT`、`PATCH`、または `DELETE` ルートを定義する場合は、非表示の `_method` フィールドをフォームに追加する必要があります。`_method` フィールドで送信された値は、HTTP リクエストメソッドとして使用されます。

    <form action="/example" method="POST">
        <input type="hidden" name="_method" value="PUT">
        <input type="hidden" name="_token" value="{{ csrf_token() }}">
    </form>

便宜上、`@method` [Blade ディレクティブ](/docs/{{version}}/blade) を使用して、`_method` 入力フィールドを生成できます。

    <form action="/example" method="POST">
        @method('PUT')
        @csrf
    </form>

<a name="accessing-the-current-route"></a>
## 現在のルートへのアクセス

`Route` ファサードで`current`、`currentRouteName`、`urrentRouteAction`メソッドを使用して、受信リクエストを処理するルートに関する情報にアクセスできます。

    use Illuminate\Support\Facades\Route;

    $route = Route::current(); // Illuminate\Routing\Route
    $name = Route::currentRouteName(); // string
    $action = Route::currentRouteAction(); // string

ルートとルートクラスで利用可能なすべてのメソッドについては、[Route ファサードの基礎となるクラス](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html) と [Route インスタンス](https ://laravel.com/api/{{version}}/Illuminate/Routing/Route.html) のAPI ドキュメントを参照してください。

<a name="cors"></a>
## クロスオリジンリソース共有 (CORS)

Laravel は、設定した値に基づいて CORS `OPTIONS` HTTP リクエストに自動的に応答できます。すべての CORS 設定は、アプリケーションの `config/cors.php` ファイルで設定できます。`OPTIONS` リクエストは、グローバルミドルウェアスタックにデフォルトで含まれている `HandleCors` [ミドルウェア](/docs/{{version}}/middleware) によって自動的に処理されます。グローバルミドルウェアスタックは、アプリケーションの HTTP カーネル (`App\Http\Kernel`) にあります。

> **Note**
> CORS および CORS ヘッダの詳細については、[CORS に関する MDN Web ドキュメント](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#The_HTTP_response_headers) を参照してください。

<a name="route-caching"></a>
## ルートのキャッシュ

アプリケーションを運用環境にデプロイするときは、Laravel のルートキャッシュを利用すると良いでしょう。ルートキャッシュを使用すると、アプリケーションのすべてのルートを登録するのにかかる時間が大幅に短縮されます。ルート キャッシュを生成するには、`route:cache` Artisan コマンドを実行します。

```shell
php artisan route:cache
```

このコマンドを実行すると、キャッシュされたルートファイルはすべてのリクエストで読み込まれます。新しいルートを追加すると、新しいルートキャッシュを生成する必要があります。このため、`route:cache` コマンドはプロジェクトのデプロイメント中にのみ実行する必要があります。

`route:clear` コマンドを使用してルートキャッシュをクリアできます。

```shell
php artisan route:clear
```
