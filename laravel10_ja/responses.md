# HTTP レスポンス

- [レスポンスの作成](#creating-responses)
    - [レスポンスにヘッダを付加](#attaching-headers-to-responses)
    - [レスポンスにクッキーを付加](#attaching-cookies-to-responses)
    - [クッキーと暗号化](#cookies-and-encryption)
- [リダイレクト](#redirects)
    - [名前付きルートへのリダイレクト](#redirecting-named-routes)
    - [コントローラアクションへのリダイレクト](#redirecting-controller-actions)
    - [外部ドメインへのリダイレクト](#redirecting-external-domains)
    - [一時保存したセッションデータを含むリダイレクト](#redirecting-with-flashed-session-data)
- [その他のレスポンスタイプ](#other-response-types)
    - [View レスポンス](#view-responses)
    - [JSON レスポンス](#json-responses)
    - [File ダウンロード](#file-downloads)
    - [File レスポンス](#file-responses)
- [レスポンスマクロ](#response-macros)

<a name="creating-responses"></a>
## レスポンスの作成

<a name="strings-arrays"></a>
#### 文字列と配列

すべてのルートとコントローラは、ユーザーのブラウザにレスポンスを送り返す必要があります。Laravel では、レスポンスを返す方法がいくつか提供されています。最も基本的なレスポンスは、ルートまたはコントローラから文字列を返すことです。フレームワークは、文字列を完全な HTTP レスポンスに自動的に変換します。

    Route::get('/', function () {
        return 'Hello World';
    });

ルートやコントローラから文字列を返すだけでなく、配列も返すこともできます。フレームワークは配列を JSON レスポンスに自動的に変換します。

    Route::get('/', function () {
        return [1, 2, 3];
    });

> **Note**  
> ルートまたはコントローラから [Eloquent collections](/docs/{{version}}/eloquent-collections) を返すこともできることをご存知ですか？これらは自動的に JSON に変換されます。試してみてくださいね！

<a name="response-objects"></a>
#### レスポンスオブジェクト

通常、ルートアクションから単純な文字列や配列を返すだけではありません。たとえば、完全な `Illuminate\Http\Response` インスタンス、または [ビュー](/docs/{{version}}/views) を返したりもします。

完全な `Response` インスタンスを返すと、レスポンスの HTTP ステータスコードとヘッダをカスタマイズできます。 `Response` インスタンスは、HTTP レスポンスを構築するためのさまざまなメソッドを提供する `Symfony\Component\HttpFoundation\Response` クラスを継承しています。

    Route::get('/home', function () {
        return response('Hello World', 200)
                      ->header('Content-Type', 'text/plain');
    });

<a name="eloquent-models-and-collections"></a>
#### Eloquent モデルとコレクション

[Eloquent ORM](/docs/{{version}}/eloquent) モデルとコレクションをルートとコントローラから直接返すこともできます。これを行うと、Laravel はモデルの [非表示属性](/docs/{{version}}/eloquent-serialization#hiding-attributes-from-json) を尊重しながら、モデルとコレクションを JSON レスポンスに自動変換します。

    use App\Models\User;

    Route::get('/user/{user}', function (User $user) {
        return $user;
    });

<a name="attaching-headers-to-responses"></a>
### レスポンスにヘッダを付加

大体のレスポンスメソッドはチェーン可能であり、レスポンスインスタンスをスムーズに構築できます。たとえば、`header` メソッドを使用して、レスポンスをユーザーに送り返す前に一連のヘッダを追加することもできます。

    return response($content)
                ->header('Content-Type', $type)
                ->header('X-Header-One', 'Header Value')
                ->header('X-Header-Two', 'Header Value');

または、`withHeaders` メソッドを使用して、レスポンスに追加するヘッダの配列を指定することもできます。

    return response($content)
                ->withHeaders([
                    'Content-Type' => $type,
                    'X-Header-One' => 'Header Value',
                    'X-Header-Two' => 'Header Value',
                ]);

<a name="cache-control-middleware"></a>
#### キャッシュ制御ミドルウェア

Laravel には `cache.headers` ミドルウェアが含まれており、ルートグループに `Cache-Control` ヘッダをすばやく設定するために使用できます。ディレクティブは、対応する cache-control ディレクティブと同等の「スネーク ケース」を使用し、セミコロンで区切ってください。ディレクティブのリストに `etag` が指定されている場合、レスポンスコンテンツの MD5 ハッシュが ETag 識別子として自動的に設定されます。

    Route::middleware('cache.headers:public;max_age=2628000;etag')->group(function () {
        Route::get('/privacy', function () {
            // ...
        });

        Route::get('/terms', function () {
            // ...
        });
    });

<a name="attaching-cookies-to-responses"></a>
### レスポンスにクッキーを付加

`cookie` メソッドを使用して、送信する `Illuminate\Http\Response` インスタンスにクッキーを付加できます。クッキーが有効であるとみなされる、名前、値、分数をこのメソッドに渡す必要があります。

    return response('Hello World')->cookie(
        'name', 'value', $minutes
    );

`cookie` メソッドは、使用頻度は低いですが、いくつかの引数も受け入れます。一般に、これらの引数は、PHP のネイティブ [setcookie](https://secure.php.net/manual/en/function.setcookie.php) メソッドに与える引数と同じ目的と意味を持っています。

    return response('Hello World')->cookie(
        'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
    );

送信レスポンスと一緒にクッキーを送信したくても、そのレスポンスのインスタンスがまだない場合は、`Cookie` ファサードを使用して、送信時にレスポンスに付加するクッキーを「キュー」に入れることができます。 `queue` メソッドは、クッキーインスタンスの作成に必要な引数を受け取ります。これらのクッキーは、送信レスポンスがブラウザに送信される前に付加されます。

    use Illuminate\Support\Facades\Cookie;

    Cookie::queue('name', 'value', $minutes);

<a name="generating-cookie-instances"></a>
#### クッキーインスタンスの生成

後でレスポンスインスタンスにアタッチできる `Symfony\Component\HttpFoundation\Cookie` インスタンスを生成したい場合は、グローバル `cookie` ヘルパを使用できます。このクッキーは、レスポンスインスタンスに付加されない限り、クライアントに返送されません。

    $cookie = cookie('name', 'value', $minutes);

    return response('Hello World')->cookie($cookie);

<a name="expiring-cookies-early"></a>
#### クッキーを期限切れにさせる

送信レスポンスの `withoutCookie` メソッドを使用してクッキーを期限切れにさせることで、クッキーを削除できます。

    return response('Hello World')->withoutCookie('name');

送信レスポンスのインスタンスがまだないない場合は、`Cookie` ファサードの `expire` メソッドを使用してクッキーを期限切れにさせることができます。

    Cookie::expire('name');

<a name="cookies-and-encryption"></a>
### クッキーと暗号化

デフォルトでは、Laravel によって生成されるすべてのクッキーは暗号化および署名されるため、クライアントによる変更や読み取りはできません。アプリケーションによって生成されたクッキーのサブセットの暗号化を無効にしたい場合は、`app/Http/Middleware` ディレクトリ内にある `App\Http\Middleware\EncryptCookies` ミドルウェアの `$except` プロパティを設定してください。

    /**
     * The names of the cookies that should not be encrypted.
     *
     * @var array
     */
    protected $except = [
        'cookie_name',
    ];

<a name="redirects"></a>
## リダイレクト

リダイレクトレスポンスは `Illuminate\Http\RedirectResponse` クラスのインスタンスであり、ユーザーを別の URL にリダイレクトするために必要な適切なヘッダが含まれています。`RedirectResponse` インスタンスを生成するにはいくつかの方法があります。最も簡単な方法は、グローバル `redirect` ヘルパを使用することです。

    Route::get('/dashboard', function () {
        return redirect('home/dashboard');
    });

送信されたフォームが無効な場合など、ユーザーを以前の場所にリダイレクトしたい場合があります。これを行うには、グローバル `back` ヘルパ関数を使用します。この機能は [セッション](/docs/{{version}}/session) を利用するため、`back` 関数を呼び出すルートが `web` ミドルウェアグループを使用していることを確認してください。

    Route::post('/user/profile', function () {
        // Validate the request...

        return back()->withInput();
    });

<a name="redirecting-named-routes"></a>
### 名前付きルートへのリダイレクト

`redirect` ヘルパをパラメータを指定せずに呼び出すと、`Illuminate\Routing\Redirector` のインスタンスが返され、`Redirector` インスタンスの任意のメソッドを呼び出すことができます。たとえば、名前付きルートへの `RedirectResponse` を生成したい場合は、`route` メソッドを使用できます。

    return redirect()->route('login');

ルートにパラメータがある場合は、それらを第２引数として `route` メソッドに渡すことができます。

    // For a route with the following URI: /profile/{id}

    return redirect()->route('profile', ['id' => 1]);

<a name="populating-parameters-via-eloquent-models"></a>
#### Eloquent モデルを介したパラメータの入力

Eloquent モデルの「ID」パラメータを、リダイレクト先のルートパラメータ情報とする場合は、モデル自体を渡すことができます。 ID は自動的に抽出されます。

    // For a route with the following URI: /profile/{id}

    return redirect()->route('profile', [$user]);

ルートパラメータに配置する値をカスタマイズしたい場合は、ルートパラメータ定義 (`/profile/{id:slug}`) で列を指定するか、Eloquent モデルの `getRouteKey` メソッドをオーバーライドすることができます。

    /**
     * Get the value of the model's route key.
     */
    public function getRouteKey(): mixed
    {
        return $this->slug;
    }

<a name="redirecting-controller-actions"></a>
### コントローラアクションへのリダイレクト

[コントローラアクション](/docs/{{version}}/controllers) へのリダイレクトを生成することもできます。これを行うには、コントローラとアクション名を `action` メソッドに渡します。

    use App\Http\Controllers\UserController;

    return redirect()->action([UserController::class, 'index']);

コントローラのルートにパラメータが必要な場合は、`action` メソッドの第２引数に渡すことができます。

    return redirect()->action(
        [UserController::class, 'profile'], ['id' => 1]
    );

<a name="redirecting-external-domains"></a>
### 外部ドメインへのリダイレクト

アプリケーションの外部のドメインにリダイレクトする必要がある場合は、`away` メソッドを呼び出します。このメソッドは、追加の URL エンコード、バリデーション、検証を行わずに `RedirectResponse` を作成します。

    return redirect()->away('https://www.google.com');

<a name="redirecting-with-flashed-session-data"></a>
### 一時保存したセッションデータを含むリダイレクト

通常、新しい URL へのリダイレクトと [セッションへのデータの一時保存](/docs/{{version}}/session#flash-data) は同時に行われます。一般的には、アクションが正常に実行された後で、成功メッセージをセッションに一時保存するときに行われます。便宜上、`RedirectResponse` インスタンスを作成し、メソッドチェーンを１つだけ書けば、セッションにデータを一時保存することができます。

    Route::post('/user/profile', function () {
        // ...

        return redirect('dashboard')->with('status', 'Profile updated!');
    });

ユーザーがリダイレクトされた後、[セッション](/docs/{{version}}/session) から一時保存されたメッセージを表示できます。たとえば、[Blade 記法](/docs/{{version}}/blade) を使用すると、次のようになります。

    @if (session('status'))
        <div class="alert alert-success">
            {{ session('status') }}
        </div>
    @endif

<a name="redirecting-with-input"></a>
#### 入力と一緒にリダイレクト

ユーザーを新しい場所にリダイレクトする前に、`RedirectResponse` インスタンスによって提供される `withInput` メソッドを使用して、現在のリクエストの入力データをセッションに一時保存できます。これは通常、ユーザーがバリデーションエラーに遭遇した場合に行われます。入力がセッションに一時保存されると、次のリクエスト中に簡単に [取得](/docs/{{version}}/requests#retrieving-old-input) してフォームに再入力できます。

    return back()->withInput();

<a name="other-response-types"></a>
## 他のレスポンスタイプ

`response`  ヘルパは、他のタイプのレスポンスインスタンスを生成するために使用できます。`response` ヘルパが引数なしで呼び出された場合、`Illuminate\Contracts\Routing\ResponseFactory` [契約](/docs/{{version}}/contracts) の実装が返されます。この契約は、レスポンスを生成するためのいくつかの便利なメソッドを提供します。

<a name="view-responses"></a>
### View レスポンス

レスポンスのステータスやヘッダを制御しながら、レスポンスのコンテンツとして [ビュー](/docs/{{version}}/views) を返す必要がある場合は、`view` メソッドを使用します。

    return response()
                ->view('hello', $data, 200)
                ->header('Content-Type', $type);

もちろん、カスタム HTTP ステータスコードやカスタムヘッダを渡す必要がない場合は、グローバル `view` ヘルパ関数を使用できます。

<a name="json-responses"></a>
### JSON レスポンス

`json` メソッドは、自動的に `Content-Type` ヘッダを `application/json` に設定し、`json_encode` PHP 関数を使用して指定された配列を JSON に変換します。

    return response()->json([
        'name' => 'Abigail',
        'state' => 'CA',
    ]);

JSONP レスポンスを生成したい場合は、`json` メソッドを `withCallback` メソッドと組み合わせて使用してください。

    return response()
                ->json(['name' => 'Abigail', 'state' => 'CA'])
                ->withCallback($request->input('callback'));

<a name="file-downloads"></a>
### File ダウンロード

`download` メソッドは、ユーザーのブラウザに、指定したパスのファイルをダウンロードさせるレスポンスを生成します。 `download` メソッドは、メソッドの第２引数にファイル名を指定することができます。これにより、ファイルをダウンロードするユーザーに表示されるファイル名が決まります。最後に、HTTP ヘッダの配列を第３引数としてメソッドに渡すことができます。

    return response()->download($pathToFile);

    return response()->download($pathToFile, $name, $headers);

> **Warning**  
> ファイルのダウンロードを管理する Symfony HttpFoundation では、ダウンロードされるファイルに ASCII のファイル名が付いている必要があります。

<a name="streamed-downloads"></a>
#### ストリームダウンロード

操作の内容をディスクに書き込むことなく、特定の操作の文字列レスポンスをダウンロード可能なレスポンスに変換したい場合があります。このシナリオでは `streamDownload` メソッドを使用します。このメソッドは、コールバック、ファイル名、およびオプションのヘッダ配列を引数として受け取ります。

    use App\Services\GitHub;

    return response()->streamDownload(function () {
        echo GitHub::api('repo')
                    ->contents()
                    ->readme('laravel', 'laravel')['contents'];
    }, 'laravel-readme.md');

<a name="file-responses"></a>
### File レスポンス

`file` メソッドは、ダウンロードをする代わりに、画像や PDF などのファイルをユーザーのブラウザに直接表示するために使用されます。このメソッドは、ファイルへのパスを第１引数に、ヘッダの配列を第２引数に指定します。

    return response()->file($pathToFile);

    return response()->file($pathToFile, $headers);

<a name="response-macros"></a>
## レスポンスマクロ

さまざまなルートやコントローラで再利用できるカスタムレスポンスを定義したい場合は、`Response` ファサードで `macro` メソッドを使用します。通常、このメソッドは、`App\Providers\AppServiceProvider` サービスプロバイダなど、アプリケーションの [サービスプロバイダ](/docs/{{version}}/providers) の１つの `boot` メソッドから呼び出す必要があります。

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Response;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         */
        public function boot(): void
        {
            Response::macro('caps', function (string $value) {
                return Response::make(strtoupper($value));
            });
        }
    }

`macro` 関数は、第１引数に名前を入れ、第２引数にクロージャーを記述します。マクロのクロージャは、`ResponseFactory` 実装または `response` ヘルパからマクロ名を呼び出すときに実行されます。

    return response()->caps('foo');
