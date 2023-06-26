# HTTP リクエスト

- [はじめに](#introduction)
- [リクエストの操作](#interacting-with-the-request)
    - [リクエストへのアクセス](#accessing-the-request)
    - [リクエストのパス、ホスト、メソッド](#request-path-and-method)
    - [リクエストヘッダ](#request-headers)
    - [リクエスト IP アドレス](#request-ip-address)
    - [コンテンツネゴシエーション](#content-negotiation)
    - [PSR-7 リクエスト](#psr7-requests)
- [入力](#input)
    - [入力の取得](#retrieving-input)
    - [入力の存在判定](#determining-if-input-is-present)
    - [追加入力のマージ](#merging-additional-input)
    - [直前の入力](#old-input)
    - [クッキー](#cookies)
    - [入力のトリミングと正規化](#input-trimming-and-normalization)
- [ファイル](#files)
    - [アップロードしたファイルの取得](#retrieving-uploaded-files)
    - [アップロードしたファイルの保存](#storing-uploaded-files)
- [信頼するプロキシの設定](#cconfigured-trusted-proxies)
- [信頼するホストの設定](#cconfigured-trusted-hosts)

<a name="introduction"></a>
## はじめに

Laravel の `Illuminate\Http\Request` クラスは、アプリケーションで処理されている現在の HTTP リクエストを操作し、リクエストとともに送信された入力、クッキー、およびファイルを取得するオブジェクト指向の方法を提供します。

<a name="interacting-with-the-request"></a>
## リクエストの操作

<a name="accessing-the-request"></a>
### リクエストへのアクセス

現在の HTTP リクエストのインスタンスを取得するには、ルートクロージャ、またはコントローラメソッドで `Illuminate\Http\Request` クラスをタイプヒントする必要があります。受信したリクエストのインスタンスは、Laravel [サービスコンテナ](/docs/{{version}}/container) によって自動的に依存性注入されます。

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Store a new user.
         */
        public function store(Request $request): RedirectResponse
        {
            $name = $request->input('name');

            // Store the user...

            return redirect('/users');
        }
    }

前述したように、ルートクロージャで `Illuminate\Http\Request` クラスをタイプヒントで指定することもできます。サービス コンテナは、実行時に受信リクエストを自動的にクロージャに依存性注入します。

    use Illuminate\Http\Request;

    Route::get('/', function (Request $request) {
        // ...
    });

<a name="dependency-injection-route-parameters"></a>
#### 依存性注入とルートパラメータ

コントローラメソッドがルート パラメータからの入力も期待している場合は、他の依存関係の後にルートパラメータをリストする必要があります。たとえば、ルートが次のように定義されているとします。

    use App\Http\Controllers\UserController;

    Route::put('/user/{id}', [UserController::class, 'update']);

以下のようにコントローラメソッドを定義することで、`Illuminate\Http\Request` をタイプヒントし、`id` ルートパラメータにアクセスできます。

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Update the specified user.
         */
        public function update(Request $request, string $id): RedirectResponse
        {
            // Update the user...

            return redirect('/users');
        }
    }

<a name="request-path-and-method"></a>
### リクエストのパス、ホスト、およびメソッド

`Illuminate\Http\Request` インスタンスは、`Symfony\Component\HttpFoundation\Request` クラスを拡張し、受信 HTTP リクエストを調査するためのさまざまなメソッドを提供します。以下では、最も重要な調査方法のいくつかを説明します。

<a name="retrieving-the-request-path"></a>
#### リクエストパスの取得

`path` メソッドはリクエストのパス情報を返します。したがって、受信リクエストが `http://example.com/foo/bar` をターゲットにしている場合、`path` メソッドは `foo/bar` を返します。

    $uri = $request->path();

<a name="inspecting-the-request-path"></a>
#### リクエストのパス / ルートの検査

`is` メソッドを使用すると、受信リクエストのパスが指定されたパターンと一致するかどうかを検証できます。このメソッドは、`*` 文字をワイルドカードとして使用できます。

    if ($request->is('admin/*')) {
        // ...
    }

`routeIs` メソッドでは、受信リクエストが [名前付きルート](/docs/{{version}}/routing#named-routes) と一致したかどうかを判定できます。

    if ($request->routeIs('admin.*')) {
        // ...
    }

<a name="retrieving-the-request-url"></a>
#### リクエスト URL の取得

受信リクエストの完全な URL を取得するには、`url` メソッドか `fullUrl` メソッドを使用します。`url` メソッドはクエリ文字列なしで URL を返しますが、`fullUrl` メソッドでは、クエリ文字列が含まれます。

    $url = $request->url();

    $urlWithQueryString = $request->fullUrl();

現在の URL にクエリ文字列データを追加したい場合は、`fullUrlWithQuery` メソッドを呼び出します。このメソッドは、指定されたクエリ文字列変数の配列を現在のクエリ文字列とマージします。

    $request->fullUrlWithQuery(['type' => 'phone']);

<a name="retrieving-the-request-host"></a>
#### リクエストホストの取得

`host`、`httpHost`、`schemeAndHttpHost` メソッドを使用して、受信リクエストの「ホスト」を取得できます。

    $request->host();
    $request->httpHost();
    $request->schemeAndHttpHost();

<a name="retrieving-the-request-method"></a>
#### リクエストメソッドの取得

`method` メソッドはリクエストの HTTP 動詞を返します。`isMethod` メソッドを使用すると、HTTP 動詞が指定された文字列と一致するかを判定できます。

    $method = $request->method();

    if ($request->isMethod('post')) {
        // ...
    }

<a name="request-headers"></a>
### リクエストヘッダ

`header` メソッドを使用して、`Illuminate\Http\Request` インスタンスからリクエストヘッダを取得できます。リクエストにヘッダが存在しない場合は、`null` が返されます。また `header` メソッドではオプションで、リクエストにヘッダが存在しない場合に返される値を  2 番目の引数で指定できます。

    $value = $request->header('X-Header-Name');

    $value = $request->header('X-Header-Name', 'default');

`hasHeader` メソッドは、リクエストに特定のヘッダが含まれているかどうかを判定するために使用できます。

    if ($request->hasHeader('X-Header-Name')) {
        // ...
    }

便宜上、`bearerToken` メソッドを使用して、`Authorization` ヘッダからベアラートークンを取得できます。そのようなヘッダーが存在しない場合は、空の文字列が返されます。

    $token = $request->bearerToken();

<a name="request-ip-address"></a>
### リクエスト IP アドレス

`ip` メソッドは、アプリケーションにリクエストしたクライアントの IP アドレスを取得できます。

    $ipAddress = $request->ip();

<a name="content-negotiation"></a>
### コンテンツネゴシエーション

Laravel は、`Accept` ヘッダを介して受信リクエストのリクエストされたコンテンツタイプを検査するためのメソッドをいくつか提供しています。まず、`getAcceptableContentTypes` メソッドは、リクエストによって受け入れられたすべてのコンテンツタイプを含む配列を返します。

    $contentTypes = $request->getAcceptableContentTypes();

`accepts` メソッドは、コンテンツタイプの配列を受け入れ、いずれかのコンテンツタイプがリクエストによって受け入れられた場合は `true` を返します。それ以外の場合は、`false` が返されます。

    if ($request->accepts(['text/html', 'application/json'])) {
        // ...
    }

`prefers` メソッドを使用すると、指定したコンテンツタイプの配列の中から、どのコンテンツタイプがリクエストで最も優先されるかを判断できます。提供されたコンテンツタイプがリクエスト内にない場合は、`null` が返されます。

    $preferred = $request->prefers(['text/html', 'application/json']);

多くのアプリケーションは HTML または JSON のみを提供するため、`expectsJson` メソッドを使用して、受信リクエストが JSON リクエストを期待しているかどうかを素早く判定できます。

    if ($request->expectsJson()) {
        // ...
    }

<a name="psr7-requests"></a>
### PSR-7 リクエスト

[PSR-7 標準](https://www.php-fig.org/psr/psr-7/) は、リクエストとレスポンスを含む HTTP メッセージのインターフェイスを指定します。Laravel リクエストではなく PSR-7 リクエストのインスタンスを取得したい場合は、まずいくつかのライブラリをインストールする必要があります。Laravel は *Symfony HTTP Message Bridge* コンポーネントを使用して、一般的な Laravel リクエストとレスポンスを PSR-7 互換の実装に変換します。

```shell
composer require symfony/psr-http-message-bridge
composer require nyholm/psr7
```

これらのライブラリをインストールしたら、ルートクロージャかコントローラメソッドでリクエストインターフェイスをタイプヒントすることで PSR-7 リクエストを取得できます。

    use Psr\Http\Message\ServerRequestInterface;

    Route::get('/', function (ServerRequestInterface $request) {
        // ...
    });

> **Note**  
> ルートまたはコントローラから PSR-7 レスポンスインスタンスを返すと、自動的に Laravel レスポンスインスタンスに変換され、フレームワークによって表示されます。

<a name="input"></a>
## 入力

<a name="retrieving-input"></a>
### 入力の取得

<a name="retrieving-input"></a>
#### 全入力データの取得

`all` メソッドを使用すると、受信リクエストのすべての入力データを `array` として取得できます。このメソッドは、受信リクエストが HTML フォームからのものであるか、XHR リクエストであるかに関係なく使用できます。

    $input = $request->all();

`collect` メソッドを使用すると、受信リクエストのすべての入力データを [コレクション](/docs/{{version}}/collections) として取得できます。

    $input = $request->collect();

加えて `collect` は受信リクエストの入力のサブセットをコレクションとして取得することもできます。

    $request->collect('users')->each(function (string $user) {
        // ...
    });

<a name="retrieving-an-input-value"></a>
#### 入力値の取得

いくつかの簡単な方法を使用すると、リクエストにどの HTTP 動詞が使用されたかを気にすることなく、`Illuminate\Http\Request` インスタンスからのすべてのユーザー入力にアクセスできます。HTTP 動詞に関係なく、`input` メソッドを使用してユーザー入力を取得できます。

    $name = $request->input('name');

`input` メソッドの 2 番目の引数にデフォルト値を渡すことができます。１番目に指定した入力値がリクエストに存在しない場合、この値が返されます。

    $name = $request->input('name', 'Sally');

配列入力を含むフォームを操作する場合は、「ドット」表記を使用して配列にアクセスします。

    $name = $request->input('products.0.name');

    $names = $request->input('products.*.name');

すべての入力値を連想配列として取得するには、引数なしで `input` メソッドを呼び出す方法があります。

    $input = $request->input();

<a name="retrieving-input-from-the-query-string"></a>
#### クエリ文字列から入力を取得

`input` メソッドはリクエストペイロード全体 (クエリ文字列を含む) から値を取得しますが、`query` メソッドはクエリ文字列からのみ値を取得します。

    $name = $request->query('name');

指定したクエリ文字列の値データが存在しない場合、このメソッドの 2 番目の引数が返されます。

    $name = $request->query('name', 'Helen');

すべてのクエリ文字列の値を連想配列として取得するには、引数なしで `query` メソッドを呼び出すことができます。

    $query = $request->query();

<a name="retrieving-json-input-values"></a>
#### JSON 入力値の取得

JSON リクエストをアプリケーションに送信する際は、リクエストの `Content-Type` ヘッダが `application/json` に適切に設定されている限り、`input` メソッド経由で JSON データにアクセスできます。「ドット」構文を使用して、JSON 配列 / オブジェクト内にネストされた値を取得することもできます。

    $name = $request->input('user.name');

<a name="retrieving-stringable-input-values"></a>
#### Stringable 入力値の取得

リクエストの入力データをプリミティブな `string` として取得する代わりに、`string` メソッドを使用してリクエスト データを [`Illuminate\Support\Stringable`](/docs/{{version}}/helpers#fluent-strings) のインスタンスとして取得することもできます。

    $name = $request->string('name')->trim();

<a name="retrieving-boolean-input-values"></a>
#### 論理入力値の取得

チェックボックスなどの HTML 要素を処理する場合、アプリケーションは文字列である「真」の値を受け取ることがあります。たとえば 「true」または「on」です。 便宜上、`boolean` メソッドを使用してこれらの値をブール値として取得できます。`boolean` メソッドは、1、"1"、true、"true"、"on"、"yes" の場合は `true` を返します。他のすべての値は `false` を返します。

    $archived = $request->boolean('archived');

<a name="retrieving-date-input-values"></a>
#### 日付入力値の取得

便宜上、日付や時刻を含む入力値は、`date` メソッドを使用して Carbon インスタンスとして取得できます。リクエストに指定された名前の入力値が含まれていない場合は、`null` が返されます。

    $birthday = $request->date('birthday');

`date` メソッドで受け取る２
番目と３番目の引数は、それぞれ日付の形式とタイムゾーンを指定するために使用できます。

    $elapsed = $request->date('elapsed', '!H:i', 'Europe/Madrid');

入力値が存在しても無効な形式の場合、`InvalidArgumentException` がスローされます。 したがって、`date` メソッドを呼び出す前に入力値を検証することをお勧めします。

<a name="retrieving-enum-input-values"></a>
#### 列挙型（Enum）入力値の取得

[PHP enums](https://www.php.net/manual/en/language.types.enumerations.php) に対応する入力値もリクエストから取得できます。リクエストに指定した名前の入力値が含まれていない場合、または列挙型に入力値と一致するバッキング値がない場合は、`null` が返されます。`enum` メソッドは、入力値の名前と enum クラスを１番目と２番目の引数として受け取ります。

    use App\Enums\Status;

    $status = $request->enum('status', Status::class);

<a name="retrieving-input-via-dynamic-properties"></a>
#### 動的プロパティを介した入力の取得

`Illuminate\Http\Request` インスタンスの動的プロパティを使用してユーザー入力にアクセスすることもできます。たとえば、アプリケーションのフォームの １つに `name` フィールドが含まれている場合、次のようにフィールドの値にアクセスできます。

    $name = $request->name;

動的プロパティを使用する場合、Laravel は最初にリクエストペイロード内のパラメータの値を探します。存在しない場合、Laravel は一致したルートのパラメータ内のフィールドを検索します。

<a name="retrieving-a-portion-of-the-input-data"></a>
#### 入力データの一部の取得

入力データのサブセットを取得する必要がある場合は、`only` メソッドと `except` メソッドを使用できます。これらのメソッドは両方とも、単一の「配列」または引数の動的なリストを受け取ります。

    $input = $request->only(['username', 'password']);

    $input = $request->only('username', 'password');

    $input = $request->except(['credit_card']);

    $input = $request->except('credit_card');

> **Warning**  
> `only` メソッドは、指定したすべてのキーと値のペアを返します。ただし、リクエストに存在しないキーと値のペアは返されません。

<a name="determining-if-input-is-present"></a>
### 入力値の存在を判定

`has` メソッドを使用して、リクエストに値が存在するかどうかを確認できます。値がリクエストに存在する場合、`has` メソッドは `true` を返します。

    if ($request->has('name')) {
        // ...
    }

`has` メソッドに配列を与え、その配列に指定された値がリクエストにすべて存在するかどうかを判定します。

    if ($request->has(['name', 'email'])) {
        // ...
    }

リクエストに指定した値が存在する場合、`whenHas` メソッドは指定されたクロージャを実行します。

    $request->whenHas('name', function (string $input) {
        // ...
    });

加えて `whenHas` メソッドには、指定された値がリクエストに存在しない場合に実行される処理を２番目のクロージャとして渡すことができます。

    $request->whenHas('name', function (string $input) {
        // The "name" value is present...
    }, function () {
        // The "name" value is not present...
    });

指定した値のいずれかが存在する場合、`hasAny` メソッドは `true` を返します。

    if ($request->hasAny(['name', 'email'])) {
        // ...
    }

値がリクエストに存在し、空の文字列でないかを確認したい場合は、`filled` メソッドを使用できます。

    if ($request->filled('name')) {
        // ...
    }

`whenFilled` メソッドは、リクエストに値が存在し、空の文字列ではない場合に、指定されたクロージャを実行します。

    $request->whenFilled('name', function (string $input) {
        // ...
    });

`whenFilled` メソッドには、指定された値が空だった場合に実行される２つ目のクロージャを設定することができます。

    $request->whenFilled('name', function (string $input) {
        // The "name" value is filled...
    }, function () {
        // The "name" value is not filled...
    });

指定されたキーがリクエストに存在しないかどうかを確認するには、`missing` メソッドと `whenMissing` メソッドを使用できます。

    if ($request->missing('name')) {
        // ...
    }

    $request->whenMissing('name', function (array $input) {
        // The "name" value is missing...
    }, function () {
        // The "name" value is present...
    });

<a name="merging-additional-input"></a>
### 追加入力のマージ

追加の入力をリクエストの既存の入力データに手動でマージするには `merge` メソッドを使用します。指定した入力キーがリクエストにすでに存在する場合は、`merge` メソッドに提供されたデータで上書きされます。

    $request->merge(['votes' => 0]);

対応するキーがリクエストの入力データ内に存在しない場合に、`mergeIfMissing` メソッドを使用して入力をリクエストにマージできます。

    $request->mergeIfMissing(['votes' => 0]);

<a name="old-input"></a>
### 直前の入力

現在のリクエストでの入力情報を、次のリクエストまで保持することができます。この機能は、バリデーションエラーを検出した後にフォームを再入力する場合に役立ちます。ただし、Laravel の [バリデーション機能](/docs/{{version}}/validation) を使用している場合は、Laravel の組み込みメソッドの一部のように、これらのセッション入力一時保存メソッドを手動で直接使用する必要がない可能性があります。バリデーション機能はそれらの一時保存機能を自動的に呼び出すからです。

<a name="flashing-input-to-the-session"></a>
#### 入力をセッションに一時保存

`Illuminate\Http\Request` クラスの `flash` メソッドは、現在の入力を [セッション](/docs/{{version}}/session) に一時保存し、ユーザーがアプリケーションに対し、次回のリクエストをする際に利用できます。

    $request->flash();

`flashOnly` メソッドと `flashExcept` メソッドを使用して、リクエストデータのサブセットをセッションに一時保存することも可能です。これらの方法は、パスワードなどの機密情報をセッションから除外させるのに役立ちます。

    $request->flashOnly(['username', 'email']);

    $request->flashExcept('password');

<a name="flashing-input-then-redirecting"></a>
#### 入力を一時保存してリダイレクト

入力情報をセッションへ一時保存して別ページにリダイレクトさせたい場合、`withInput` メソッドをリダイレクトインスタンスにチェーンさせて、入力の際の一時保存情報をリダイレクト先で簡単に受け取ることができます。

    return redirect('form')->withInput();

    return redirect()->route('user.create')->withInput();

    return redirect('form')->withInput(
        $request->except('password')
    );

<a name="retrieving-old-input"></a>
#### 直前の入力の取得

前のリクエストから一時保存された入力情報を取得するには、`Illuminate\Http\Request` インスタンスで `old` メソッドを呼び出します。`old` メソッドは、以前に一時保存された入力データを [セッション](/docs/{{version}}/session) から取得します。

    $username = $request->old('username');

Laravel はグローバルな `old` ヘルパも提供します。[Blade テンプレート](/docs/{{version}}/blade) 内で古い入力を表示する場合は、`old` ヘルパを使用してフォームに再入力する方が便利です。指定されたフィールドに古い入力が存在しない場合は、`null` が返されます。

    <input type="text" name="username" value="{{ old('username') }}">

<a name="cookies"></a>
＃＃＃ クッキー

<a name="retrieving-cookies-from-requests"></a>
#### リクエストからクッキーを取得

Laravel よって作成されたすべてのクッキーは暗号化され、認証コードで署名されます。つまり、クライアントによって変更された場合は無効とみなされます。リクエストからクッキー 値を取得するには、`Illuminate\Http\Request` インスタンスで `cookie` メソッドを使用します。

    $value = $request->cookie('name');

<a name="input-trimming-and-normalization"></a>
## 入力のトリミングと正規化

Laravel にはアプリケーションのグローバルミドルウェアスタックに `App\Http\Middleware\TrimStrings` と `Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull` ミドルウェアが含まれています。これらのミドルウェアは、グローバルミドルウェアスタックの `App\Http\Kernel` クラスによって登録されています。これらのミドルウェアは、リクエストの受信文字列フィールドをすべて自動的にトリミングし、空の文字列フィールドを `null` に変換します。これにより、ルートとコントローラにおけるこれらの正規化の問題を心配する必要がなくなります。

#### 入力の正規化の無効化

すべてのリクエストで、上記にて説明した入力の正規化を無効にしたい場合は、`App\Http\Kernel` クラスの `$middleware` プロパティから２つのミドルウェアを削除することで、アプリケーションのミドルウェアスタックから削除できます。

アプリケーションへのリクエストのサブセットに対して、文字列のトリミングと空の文字列の変換を無効にしたい場合は、両方のミドルウェアが提供する `skipWhen` メソッドを使用できます。このメソッドは、入力の正規化をスキップするかどうかを `true` または `false` を返すクロージャを受け取ります。通常、`skipWhen` メソッドは、アプリケーションの `AppServiceProvider` の `boot` メソッドで呼び出す必要があります。

```php
use App\Http\Middleware\TrimStrings;
use Illuminate\Http\Request;
use Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    TrimStrings::skipWhen(function (Request $request) {
        return $request->is('admin/*');
    });

    ConvertEmptyStringsToNull::skipWhen(function (Request $request) {
        // ...
    });
}
```

<a name="files"></a>
## ファイル

<a name="retrieving-uploaded-files"></a>
### アップロードしたファイルの取得

`file` メソッド、または動的プロパティを使用して、`Illuminate\Http\Request` インスタンスからアップロードされたファイルを取得できます。`file` メソッドは、`Illuminate\Http\UploadedFile` クラスのインスタンスを返します。このクラスは、PHP の `SplFileInfo` クラスを拡張し、ファイルと対話するためのさまざまなメソッドを提供します。

    $file = $request->file('photo');

    $file = $request->photo;

`hasFile` メソッドを使用して、リクエストにファイルが存在するかどうかを確認できます。

    if ($request->hasFile('photo')) {
        // ...
    }

<a name="validating-successful-uploads"></a>
#### 成功したアップロードのバリデーション

ファイルが存在するかどうかを確認するだけでなく、`isValid` メソッドを使用してファイルのアップロードに問題がなかったことを確認することもできます。

    if ($request->file('photo')->isValid()) {
        // ...
    }

<a name="file-paths-extensions"></a>
#### ファイルパスと拡張子

`UploadedFile` クラスには、ファイルの完全修飾パスとその拡張子にアクセスするためのメソッドも含まれています。`extension` メソッドは、ファイルの内容に基づいてファイルの拡張子を推測しようとします。この拡張子は、クライアントによって提供された拡張子とは異なる場合があります。

    $path = $request->photo->path();

    $extension = $request->photo->extension();

<a name="other-file-methods"></a>
#### 他のファイルメソッド

`UploadedFile` インスタンスではさまざまなメソッドが利用できます。 これらのメソッドの詳細については、[クラスの API ドキュメント](https://github.com/symfony/symfony/blob/6.0/src/Symfony/Component/HttpFoundation/File/UploadedFile.php) を確認してください。

<a name="storing-uploaded-files"></a>
### アップロードしたファイルの保存

アップロードしたファイルを保存するには、通常、設定済みの [ファイルシステム](/docs/{{version}}/filesystem) の 1 つを使用します。`UploadedFile` クラスには、アップロードしたファイルをディスクの１つに移動する `store` メソッドがあります。ディスクは、ローカルファイルシステム上の場所または Amazon S3 などのクラウドストレージの場所である可能性があります。

`store` メソッドは、ファイルシステムの設定されたルート ディレクトリを基準にしてファイルを保存するパスを引数に取ります。ファイル名として機能する一意の ID が自動的に生成されるため、このパスにはファイル名を含めないでください。

`store` メソッドは、ファイルの保存に使用するディスクの名前をオプションとして第２引数に取ることができます。このメソッドは、ディスクのルートを基準としたファイルの相対パスを返します。

    $path = $request->photo->store('images');

    $path = $request->photo->store('images', 's3');

ファイル名を自動的に生成したくない場合は、パス、ファイル名、ディスク名を引数として受け取る `storeAs` メソッドを使用できます。

    $path = $request->photo->storeAs('images', 'filename.jpg');

    $path = $request->photo->storeAs('images', 'filename.jpg', 's3');

> **Note**  
> Laravel のファイルストレージの詳細については、完全な [ファイルストレージドキュメント](/docs/{{version}}/filesystem) を確認してください。

<a name="configuring-trusted-proxies"></a>
## 信頼できるプロキシの設定

TLS / SSL 証明書を末端とするロードバランサの背後でアプリケーションを実行する場合、`url` ヘルパの使用時にアプリケーションが HTTPS リンクを生成しないことがあります。通常、これはアプリケーションがポート 80 上のロードバランサからトラフィックを転送されており、安全なリンクを生成する必要があることを認識していないことが理由です。

これを解決するには、`App\Http\Middleware\TrustProxies` ミドルウェアを使用します。これにより、アプリケーションが信頼するロードバランサまたはプロキシをすばやくカスタマイズできます。信頼するプロキシは、このミドルウェアの `$proxies` プロパティに配列として登録する必要があります。信頼できるプロキシの設定に加えて、信頼すべきプロキシ `$headers` を設定することもできます。

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Http\Middleware\TrustProxies as Middleware;
    use Illuminate\Http\Request;

    class TrustProxies extends Middleware
    {
        /**
         * The trusted proxies for this application.
         *
         * @var string|array
         */
        protected $proxies = [
            '192.168.1.1',
            '192.168.1.2',
        ];

        /**
         * The headers that should be used to detect proxies.
         *
         * @var int
         */
        protected $headers = Request::HEADER_X_FORWARDED_FOR | Request::HEADER_X_FORWARDED_HOST | Request::HEADER_X_FORWARDED_PORT | Request::HEADER_X_FORWARDED_PROTO;
    }

> **Note**  
> AWS Elastic Load Balancing を使用している場合、`$headers` の値は `Request::HEADER_X_FORWARDED_AWS_ELB` である必要があります。`$headers` プロパティで使用できる定数の詳細については、[信頼するプロキシ] に関する Symfony のドキュメント (https://symfony.com/doc/current/deployment/proxies.html) を参照してください。

<a name="trusting-all-proxies"></a>
#### すべてのプロキシを信頼する

Amazon AWS または別の「クラウド」ロードバランサプロバイダを使用している場合は、実際のバランサの IP アドレスがわからない可能性があります。この場合、`*` を使用してすべてのプロキシを信頼します。

    /**
     * The trusted proxies for this application.
     *
     * @var string|array
     */
    protected $proxies = '*';

<a name="configuring-trusted-hosts"></a>
## 信頼するホストの設定

デフォルトでは、Laravel は HTTP リクエストの `Host` ヘッダの内容に関係なく、受信したすべてのリクエストに応答します。さらに、`Host` ヘッダの値は、Web リクエスト中にアプリケーションへの絶対 URL を生成するときに使用されます。

通常、指定されたホスト名に一致するリクエストのみをアプリケーションに送信するように、Nginx や Apache などの Web サーバーを設定する必要があります。ただし、Web サーバーを直接カスタマイズする機能がなく、特定のホスト名にのみ応答するように Laravel に指示する必要がある場合は、アプリケーションの `App\Http\Middleware\TrustHosts` ミドルウェアを有効にしましょう。

`TrustHosts`ミドルウェアは、アプリケーションの `$middleware` スタックにすでに含まれています。ただし、アクティブになるようにコメントを解除する必要があります。このミドルウェアの `hosts` メソッド内で、アプリケーションが応答するホスト名を指定できます。他の `Host` の値ヘッダを持つ受信リクエストは拒否されます。

    /**
     * Get the host patterns that should be trusted.
     *
     * @return array<int, string>
     */
    public function hosts(): array
    {
        return [
            'laravel.test',
            $this->allSubdomainsOfApplicationUrl(),
        ];
    }

`allSubdomainsOfApplicationUrl` ヘルパメソッドは、アプリケーションの `app.url` 設定値のすべてのサブドメインに一致する正規表現を返します。このヘルパメソッドは、ワイルドカードのサブドメインを使用するアプリケーションを構築するときに、アプリケーションのすべてのサブドメインを許可する便利な方法を提供します。
