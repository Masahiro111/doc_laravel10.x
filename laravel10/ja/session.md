# HTTP セッション

- [はじめに](#introduction)
    - [設定](#configuration)
    - [ドライバの動作条件](#driver-prerequisites)
- [セッションの操作](#interacting-with-the-session)
    - [データの取得](#retrieving-data)
    - [データの保存](#storing-data)
    - [データの一時保存](#flash-data)
    - [データの削除](#deleting-data)
    - [セッション ID の再生成](#regenerating-the-session-id)
- [セッションブロッキング](#session-blocking)
- [カスタムセッション ドライバの追加](#adding-custom-session-drivers)
    - [ドライバの実装](#implementing-the-driver)
    - [ドライバの登録](#registering-the-driver)

<a name="introduction"></a>
## はじめに

HTTP 駆動のアプリケーションはステートレスであるため、セッションは複数のリクエストにわたってユーザーに関する情報を保存する方法を提供します。そのユーザー情報は通常、後続のリクエストからアクセスできる永続的な保存 / バックエンドに配置されます。

Laravel には、表現力豊かな統合 API を通じてアクセスされるさまざまなセッションバックエンドが用意されています。[Memcached](https://memcached.org)、[Redis](https://redis.io)、データベースなどの一般的なバックエンドをサポートしています。

<a name="configuration"></a>
### 設定

アプリケーションのセッション設定ファイルは `config/session.php` に保存されています。このファイルで使用できるオプションを必ず確認してください。デフォルトでは、Laravel は `file` セッションドライバを使用するように設定されており、これは多くのアプリケーションで適切に機能します。アプリケーションが複数の Web サーバー間で負荷分散される場合は、Redis やデータベースなど、すべてのサーバーがアクセスできる集中型保存領域を選択する必要があります。

セッションの `driver` 設定オプションは、各リクエストのセッションデータが保存される場所を定義します。Laravel には、すぐに使用できるいくつかの優れたドライバが同梱されています。

<div class="content-list" markdown="1">

- `file` - セッションを `storage/framework/sessions` に保存します。
- `cookie` - セッションを安全な暗号化された cookie に保存します。
- `database` - セッションをリレーショナルデータベースに保存します。
- `memcached` / `redis` - セッションを、これらの高速なキャッシュベースの保存領域のいずれかに保存します。
- `dynamodb` - セッションを AWS DynamoDB に保存します。
- `array` - セッションを PHP 配列に保存され、永続化されません。

</div>

> **Note**  
> array ドライバは主に [testing](/docs/{{version}}/testing) 中に使用され、セッションに保存されたデータが永続化されるのを防ぎます。

<a name="driver-prerequisites"></a>
### ドライバの動作条件

<a name="database"></a>
#### データベース

`database` セッションドライバを使用する場合は、セッションレコードを含むテーブルを作成する必要があります。テーブルの `Schema` 宣言の例を以下に示します。

    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Support\Facades\Schema;

    Schema::create('sessions', function (Blueprint $table) {
        $table->string('id')->primary();
        $table->foreignId('user_id')->nullable()->index();
        $table->string('ip_address', 45)->nullable();
        $table->text('user_agent')->nullable();
        $table->text('payload');
        $table->integer('last_activity')->index();
    });

このマイグレーションを生成するには、`session:table` Artisan コマンドを使用します。データベースのマイグレーションの詳細については、完全な [マイグレーションドキュメント](/docs/{{version}}/migrations) を参照してください。

```shell
php artisan session:table

php artisan migrate
```

<a name="redis"></a>
#### Redis

Laravel で Redis セッションを使用する前に、PECL 経由で PhpRedis PHP 拡張機能をインストールするか、Composer 経由で `predis/predis` パッケージ (~1.0) をインストールする必要があります。Redis の設定の詳細については、Laravel の [Redis ドキュメント](/docs/{{version}}/redis#configuration) を参照してください。

> **Note**
> `session` 設定ファイルでは、`connection` オプションを使用して、セッションで使用される Redis 接続を指定できます。

<a name="interacting-with-the-session"></a>
## セッションの操作

<a name="retrieving-data"></a>
### データの取得

Laravel でセッションデータを操作するには、主に２つの方法があります。グローバル `session` ヘルパと `Request` インスタンス経由です。まず、`Request` インスタンスを介してセッションにアクセスする方法を見てみましょう。これは、ルートクロージャまたはコントローラメソッドでタイプヒントを指定します。 コントローラメソッドの依存関係は、Laravel [サービスコンテナ](/docs/{{version}}/container) 経由で自動的に依存性注入されます。

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;
    use Illuminate\View\View;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         */
        public function show(Request $request, string $id): View
        {
            $value = $request->session()->get('key');

            // ...

            $user = $this->users->find($id);

            return view('user.profile', ['user' => $user]);
        }
    }

セッションからアイテムを取得するとき、デフォルト値を第２引数として `get` メソッドに渡すことができます。指定されたキーがセッションに存在しない場合、このデフォルト値が返されます。クロージャをデフォルト値として `get` メソッドに渡し、要求されたキーが存在しない場合、クロージャが実行され、その結果が返されます。

    $value = $request->session()->get('key', 'default');

    $value = $request->session()->get('key', function () {
        return 'default';
    });

<a name="the-global-session-helper"></a>
#### グローバルセッションヘルパ

グローバル `session` PHP 関数を使用して、セッション内のデータを取得および保存することもできます。`session` ヘルパが単一の文字列引数を指定して呼び出されると、そのセッションキーの値が返されます。キーと値のペアの配列を使用してヘルパを呼び出すと、それらの値はセッションに保存されます。

    Route::get('/home', function () {
        // セッションからデータを取得します...
        $value = session('key');

        // デフォルト値を指定しています...
        $value = session('key', 'default');

        // データをセッションに保存します...
        session(['key' => 'value']);
    });

> **Note**
> HTTP リクエストインスタンス経由でセッションを使用する場合と、グローバル `session` ヘルパを使用する場合には、実質的な違いはほとんどありません。どちらのメソッドも、すべてのテストケースで使用できる `assertSessionHas` メソッド経由で [テスト可能](/docs/{{version}}/testing) になります。

<a name="retrieving-all-session-data"></a>
#### 全セッションデータの取得

セッション内のすべてのデータを取得したい場合は、`all` メソッドを使用します。

    $data = $request->session()->all();

<a name="determining-if-an-item-exists-in-the-session"></a>
#### セッション内アイテムの存在判定

アイテムがセッションに存在するかを確認するには、`has` メソッドを使用します。アイテムが存在し、`null` でない場合、`has` メソッドは `true` を返します。

    if ($request->session()->has('users')) {
        // ...
    }

`exists` メソッドを使用することで、指定したセッション内のアイテムが `null` だとしても、存在するかどうか確認できます。

    if ($request->session()->exists('users')) {
        // ...
    }

アイテムがセッションに存在しないかどうかを判断するには、`missing` メソッドを使用します。アイテムが存在しない場合、`missing` メソッドは `true` を返します。

    if ($request->session()->missing('users')) {
        // ...
    }

<a name="storing-data"></a>
### データの保存

セッションにデータを保存するには、通常、リクエストインスタンスの `put` メソッドまたはグローバル `session` ヘルバを使用します。

    // リクエストインスタンス経由
    $request->session()->put('key', 'value');

    // グローバル session ヘルパ経由
    session(['key' => 'value']);

<a name="pushing-to-array-session-values"></a>
#### 配列セッション値への追加

`push` メソッドは、配列のセッション値に新しい値を追加できます。たとえば、`user.teams` キーにチーム名の配列が含まれている場合、次のように配列に新しい値を追加できます。

    $request->session()->push('user.teams', 'developers');

<a name="retrieving-deleting-an-item"></a>

#### アイテムの取得と削除

`pull` メソッドは、単一のステートメントでセッションからアイテムを取得して削除します。

    $value = $request->session()->pull('key', 'default');

<a name="#incrementing-and-decrementing-session-values"></a>
#### セッション値の増減

セッション データにインクリメントまたはデクリメントしたい整数が含まれている場合は、`increment` および `decrement` メソッドを使用できます。

    $request->session()->increment('count');

    $request->session()->increment('count', $incrementBy = 2);

    $request->session()->decrement('count');

    $request->session()->decrement('count', $decrementBy = 2);

<a name="flash-data"></a>
### データの一時保存

場合によっては、後継リクエストに備えてセッションにアイテムを保存したい際は、`flash` メソッドを使用します。このメソッドを使用してセッションに保存されたデータは、即時もしくは後続の HTTP リクエスト中で使用できるようになります。後続の HTTP リクエストの後、一時保存されたデータは削除されます。一時保存データは主に、短期間のステータスメッセージに役立ちます。

    $request->session()->flash('status', 'Task was successful!');

If you need to persist your flash data for several requests, you may use the `reflash` method, which will keep all of the flash data for an additional request. If you only need to keep specific flash data, you may use the `keep` method:

    $request->session()->reflash();

    $request->session()->keep(['username', 'email']);

To persist your flash data only for the current request, you may use the `now` method:

    $request->session()->now('status', 'Task was successful!');

<a name="deleting-data"></a>
### Deleting Data

The `forget` method will remove a piece of data from the session. If you would like to remove all data from the session, you may use the `flush` method:

    // Forget a single key...
    $request->session()->forget('name');

    // Forget multiple keys...
    $request->session()->forget(['name', 'status']);

    $request->session()->flush();

<a name="regenerating-the-session-id"></a>
### Regenerating The Session ID

Regenerating the session ID is often done in order to prevent malicious users from exploiting a [session fixation](https://owasp.org/www-community/attacks/Session_fixation) attack on your application.

Laravel automatically regenerates the session ID during authentication if you are using one of the Laravel [application starter kits](/docs/{{version}}/starter-kits) or [Laravel Fortify](/docs/{{version}}/fortify); however, if you need to manually regenerate the session ID, you may use the `regenerate` method:

    $request->session()->regenerate();

If you need to regenerate the session ID and remove all data from the session in a single statement, you may use the `invalidate` method:

    $request->session()->invalidate();

<a name="session-blocking"></a>
## Session Blocking

> **Warning**  
> To utilize session blocking, your application must be using a cache driver that supports [atomic locks](/docs/{{version}}/cache#atomic-locks). Currently, those cache drivers include the `memcached`, `dynamodb`, `redis`, and `database` drivers. In addition, you may not use the `cookie` session driver.

By default, Laravel allows requests using the same session to execute concurrently. So, for example, if you use a JavaScript HTTP library to make two HTTP requests to your application, they will both execute at the same time. For many applications, this is not a problem; however, session data loss can occur in a small subset of applications that make concurrent requests to two different application endpoints which both write data to the session.

To mitigate this, Laravel provides functionality that allows you to limit concurrent requests for a given session. To get started, you may simply chain the `block` method onto your route definition. In this example, an incoming request to the `/profile` endpoint would acquire a session lock. While this lock is being held, any incoming requests to the `/profile` or `/order` endpoints which share the same session ID will wait for the first request to finish executing before continuing their execution:

    Route::post('/profile', function () {
        // ...
    })->block($lockSeconds = 10, $waitSeconds = 10)

    Route::post('/order', function () {
        // ...
    })->block($lockSeconds = 10, $waitSeconds = 10)

The `block` method accepts two optional arguments. The first argument accepted by the `block` method is the maximum number of seconds the session lock should be held for before it is released. Of course, if the request finishes executing before this time the lock will be released earlier.

The second argument accepted by the `block` method is the number of seconds a request should wait while attempting to obtain a session lock. An `Illuminate\Contracts\Cache\LockTimeoutException` will be thrown if the request is unable to obtain a session lock within the given number of seconds.

If neither of these arguments is passed, the lock will be obtained for a maximum of 10 seconds and requests will wait a maximum of 10 seconds while attempting to obtain a lock:

    Route::post('/profile', function () {
        // ...
    })->block()

<a name="adding-custom-session-drivers"></a>
## Adding Custom Session Drivers

<a name="implementing-the-driver"></a>
#### Implementing The Driver

If none of the existing session drivers fit your application's needs, Laravel makes it possible to write your own session handler. Your custom session driver should implement PHP's built-in `SessionHandlerInterface`. This interface contains just a few simple methods. A stubbed MongoDB implementation looks like the following:

    <?php

    namespace App\Extensions;

    class MongoSessionHandler implements \SessionHandlerInterface
    {
        public function open($savePath, $sessionName) {}
        public function close() {}
        public function read($sessionId) {}
        public function write($sessionId, $data) {}
        public function destroy($sessionId) {}
        public function gc($lifetime) {}
    }

> **Note**  
> Laravel does not ship with a directory to contain your extensions. You are free to place them anywhere you like. In this example, we have created an `Extensions` directory to house the `MongoSessionHandler`.

Since the purpose of these methods is not readily understandable, let's quickly cover what each of the methods do:

<div class="content-list" markdown="1">

- The `open` method would typically be used in file based session store systems. Since Laravel ships with a `file` session driver, you will rarely need to put anything in this method. You can simply leave this method empty.
- The `close` method, like the `open` method, can also usually be disregarded. For most drivers, it is not needed.
- The `read` method should return the string version of the session data associated with the given `$sessionId`. There is no need to do any serialization or other encoding when retrieving or storing session data in your driver, as Laravel will perform the serialization for you.
- The `write` method should write the given `$data` string associated with the `$sessionId` to some persistent storage system, such as MongoDB or another storage system of your choice.  Again, you should not perform any serialization - Laravel will have already handled that for you.
- The `destroy` method should remove the data associated with the `$sessionId` from persistent storage.
- The `gc` method should destroy all session data that is older than the given `$lifetime`, which is a UNIX timestamp. For self-expiring systems like Memcached and Redis, this method may be left empty.

</div>

<a name="registering-the-driver"></a>
#### ドライバの登録

ドライバを実装したら、Laravel に登録する準備が整いました。Laravel のセッションバックエンドにドライバを追加するには、`Session` [ファサード](/docs/{{version}}/facades) によって提供される `extend` メソッドを使用します。[サービスプロバイダ](/docs/{{version}}/providers) の `boot` メソッドから `extend` メソッドを呼び出す必要があります。既存の `App\Providers\AppServiceProvider` からこれを行うことも、また、まったく新しいプロバイダを作成することもできます。

    <?php

    namespace App\Providers;

    use App\Extensions\MongoSessionHandler;
    use Illuminate\Contracts\Foundation\Application;
    use Illuminate\Support\Facades\Session;
    use Illuminate\Support\ServiceProvider;

    class SessionServiceProvider extends ServiceProvider
    {
        /**
         * 各種アプリケーションサービスを登録します。
         */
        public function register(): void
        {
            // ...
        }

        /**
         * あらゆるアプリケーションサービスを初期起動します。
         */
        public function boot(): void
        {
            Session::extend('mongo', function (Application $app) {
                // SessionHandlerInterface の実装を返します...
                return new MongoSessionHandler;
            });
        }
    }

セッションドライバを登録すると、`config/session.php` 設定ファイルで `mongo` ドライバを使用できます。
