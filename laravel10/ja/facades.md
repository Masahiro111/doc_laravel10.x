# ファサード

- [はじめに](#introduction)
- [ファサードを使うタイミング](#when-to-use-facades)
     - [ファサードと依存性注入](#facades-vs-dependency-injection)
     - [ファサードとヘルパ関数](#facades-vs-helper-functions)
- [ファサードの仕組み](#how-facades-work)
- [リアルタイムファサード](#real-time-facades)
- [ファサードクラスリファレンス](#facade-class-reference)

<a name="introduction"></a>
## はじめに

Laravel のドキュメント全体で、「ファサード」を介して Laravel の機能とやりとりするコード例を見ることがあるでしょう。ファサードは、アプリケーションの [サービスコンテナ](/docs/{{version}}/container) で利用可能なクラスへの「静的」なインターフェースを提供します。Laravel には、ほぼすべての Laravel の機能にアクセスできる多くのファサードを搭載しています。

Laravel のファサードは、サービスコン「静的プロキシ」として機能し、従来の静的メソッドよりも高いテスト容易性と柔軟性を維持しながら、簡潔で表現力豊かな構文の利点を提供します。ファサードの仕組みが完全に理解できなくても大丈夫です。流れに沿って、Laravel について学習を続けてください。

Laravel のすべてのファサードは `Illuminate\Support\Facades` 名前空間で定義されています。そのため、以下のように簡単にファサードにアクセスできます。

    use Illuminate\Support\Facades\Cache;
    use Illuminate\Support\Facades\Route;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

Laravel ドキュメント全体を通じ、多くのコード例でファサードを使用して、フレームワークのさまざまな機能を実演しています。

<a name="helper-functions"></a>
#### ヘルパ関数

Laravel は、ファサードを補完するためのグローバルな「ヘルパ関数」を提供しており、Laravel 共通の機能を更に簡単に扱うことができます。 操作できる一般的なヘルパー関数として `view`、`response`、`url`、`config` などがあります。 Laravel が提供する各ヘルパ関数は、対応する機能とともにドキュメント化されています。完全なリストは専用の [ヘルパドキュメント](/docs/{{version}}/helpers) をご覧ください。

たとえば、`Illuminate\Support\Facades\Response` ファサードを使用して JSON レスポンスを生成する代わりに、単純に `response` 関数を使用することができます。ヘルパ関数はグローバルに利用可能なため、それらを使用するためにクラスをインポートする必要はありません。

    use Illuminate\Support\Facades\Response;

    Route::get('/users', function () {
        return Response::json([
            // ...
        ]);
    });

    Route::get('/users', function () {
        return response()->json([
            // ...
        ]);
    });

<a name="when-to-use-facades"></a>
## ファサードを使うタイミング

ファサードには多くの利点があります。手作業で挿入または設定する必要がある長いクラス名を覚えなくても、Laravel の機能を使用できる簡潔で覚えやすい構文を提供します。 さらに、PHP の動的メソッドを独自に使用しているため、テストが簡単です。

ただし、ファサードを使用する場合は注意が必要です。ファサードの主な危険性はクラスの「スコープクリープ」です。 ファサードは非常に使いやすく、依存性注入の必要がないため、クラス内のコードを成長させ続けてしまい 1つのクラスで多くのファサードを使用することが容易にになってしまうでしょう。 依存性注入を活用すると、大規模なコンストラクタによりクラスが大きくなっていないかを視覚的にフィードバックすることができるでしょう。したがって、ファサードを使用するときは、クラスの責任範囲が狭くならないように、クラスの規模に特に注意してください。クラスが大きくなりすぎる場合は、複数の小さなクラスに分割することを検討してください。

<a name="facades-vs-dependency-injection"></a>
### ファサードと依存性注入

依存性注入の主な利点の 1 つは、注入されたクラスの実装を交換できることです。これは、モックまたはスタブを挿入し、スタブでさまざまなメソッドが呼び出されたことを確認することができるため、テスト中に役立ちます。

通常、真に静的なクラスメソッドをモックしたりスタブしたりすることはできません。しかし、ファサードはサービスコンテナから解決されたオブジェクトへのメソッド呼び出しを動的メソッドでプロキシするため、注入されたクラスインスタンスのテストと同様にファサードをテストすることができます。 たとえば、次のルートがあるとします。

    use Illuminate\Support\Facades\Cache;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

Laravel のファサードテストメソッドを使用して、`Cache::get` メソッドが期待した引数で呼び出されたことを確認するテストを次のように書くことができます。

    use Illuminate\Support\Facades\Cache;

    /**
     * A basic functional test example.
     */
    public function test_basic_example(): void
    {
        Cache::shouldReceive('get')
             ->with('key')
             ->andReturn('value');

        $response = $this->get('/cache');

        $response->assertSee('value');
    }

<a name="facades-vs-helper-functions"></a>
### ファサードとヘルパ関数

ファサードに加えて、Laravel には、ビューの生成、イベントの起動、ジョブのディスパッチ、HTTP レスポンスの送信などの一般的なタスクを実行できるさまざまな「ヘルパ」関数が含まれています。これらのヘルパ関数の多くは、対応するファサードと同じ機能を実行しています。たとえば、次のファサード呼び出しとヘルパ呼び出しは同等です。

    return Illuminate\Support\Facades\View::make('profile');

    return view('profile');

ファサードとヘルパ関数の間には実質的な違いはまったくありません。ヘルパ関数を使用する場合でも、対応するファサードとまったく同じ方法でテストできます。たとえば、次のルートがあるとします。

    Route::get('/cache', function () {
        return cache('key');
    });

`cache` ヘルパは、`Cache` ファサードの基礎となるクラスの `get` メソッドを呼び出します。したがって、ヘルパ関数を使用している場合でも、次ようなテストを作成して、予期した引数でメソッドが呼び出されているかどうかを確認できます。

    use Illuminate\Support\Facades\Cache;

    /**
     * A basic functional test example.
     */
    public function test_basic_example(): void
    {
        Cache::shouldReceive('get')
             ->with('key')
             ->andReturn('value');

        $response = $this->get('/cache');

        $response->assertSee('value');
    }

<a name="how-facades-work"></a>
## ファサードの仕組み

Laravel アプリケーションでは、ファサードはコンテナからオブジェクトにアクセスするためのクラスです。この仕組みを実現するのが `Facade` クラスです。Laravel のファサード、および作成するカスタムファサードは、基底クラス `Illuminate\Support\Facades\Facade` を拡張します。

`Facade` 基底クラスは、`__callStatic()` マジックメソッドを使用して、ファサードからの呼び出しをコンテナから解決されたオブジェクトに委ねることができます。以下の例では、Laravel のキャッシュシステムを呼び出しています。このコードを見ると、静的な `get` メソッドが `Cache` クラスで呼び出されていると思うでしょう。

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\Cache;
    use Illuminate\View\View;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         */
        public function showProfile(string $id): View
        {
            $user = Cache::get('user:'.$id);

            return view('profile', ['user' => $user]);
        }
    }

ファイルの先頭近くで `Cache` ファサードを「インポート」していることに注意してください。このファサードは、 `Illuminate\Contracts\Cache\Factory` インターフェイスの基底となる実装にアクセスするためのプロキシとして機能します。ファサードを使って行う呼び出しはすべて、Laravel のキャッシュサービスの基礎となるインスタンスに渡されます。

その `Illuminate\Support\Facades\Cache` クラスを見ると、静的メソッド `get` がないことがわかります。

    class Cache extends Facade
    {
        /**
         * Get the registered name of the component.
         */
        protected static function getFacadeAccessor(): string
        {
            return 'cache';
        }
    }

代わりに、`Cache` ファサードは `Facade` 基底クラスを拡張し、メソッド `getFacadeAccessor()` を定義します。 このメソッドの役割は、サービスコンテナの結合名を返すことです。 ユーザーが `Cache` ファサード上で静的メソッドを参照すると、Laravel は [サービスコンテナ](/docs/{{version}}/container) からの `cache` 結合を解決し、要求されたメソッド(この場合は` get`) をそのオブジェクトに対して実行します。

<a name="real-time-facades"></a>
## リアルタイムファサード

リアルタイムファサードを使用すると、アプリケーション内の任意のクラスをファサードであるかのように扱うことができます。これがどのように使われるかを説明するために、まずリアルタイムファサードを使用しないコードを見てみましょう。たとえば、`Podcast` モデルに `publish` メソッドがあると仮定しましょう。ただし、ポッドキャストを公開するためには、`Publisher` インスタンスを依存性注入する必要があります。

    <?php

    namespace App\Models;

    use App\Contracts\Publisher;
    use Illuminate\Database\Eloquent\Model;

    class Podcast extends Model
    {
        /**
         * Publish the podcast.
         */
        public function publish(Publisher $publisher): void
        {
            $this->update(['publishing' => now()]);

            $publisher->publish($this);
        }
    }

Publisher の実装をメソッドに注入すると、注入されたPublisher をモックできるため、メソッドを分離して簡単にテストできます。ただし、`publish` メソッドを呼び出すたびに、常に Publisher  インスタンスを渡す必要があります。リアルタイムファサードを使用すると、`Publisher` インスタンスを明示的に渡す必要がなく、同じテストの容易性を維持できます。 リアルタイムファサードを生成するには、インポートされたクラスの名前空間に `Facades` をプレフィックスとして追加します。

    <?php

    namespace App\Models;

    use Facades\App\Contracts\Publisher;
    use Illuminate\Database\Eloquent\Model;

    class Podcast extends Model
    {
        /**
         * Publish the podcast.
         */
        public function publish(): void
        {
            $this->update(['publishing' => now()]);

            Publisher::publish($this);
        }
    }

リアルタイムファサードが使用されると、`Facades` プレフィックスの後に表示されるインターフェイスまたはクラス名の部分を使って、サービスコンテナが Publisher の実装を依存性解決させます。 テストするときは、Laravel の組み込みファサードテストヘルパを使用して、このメソッド呼び出しをモックできます。

    <?php

    namespace Tests\Feature;

    use App\Models\Podcast;
    use Facades\App\Contracts\Publisher;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Tests\TestCase;

    class PodcastTest extends TestCase
    {
        use RefreshDatabase;

        /**
         * A test example.
         */
        public function test_podcast_can_be_published(): void
        {
            $podcast = Podcast::factory()->create();

            Publisher::shouldReceive('publish')->once()->with($podcast);

            $podcast->publish();
        }
    }

<a name="facade-class-reference"></a>
## ファサードクラスリファレンス

以下に、すべてのファサードとその基礎となるクラスが表示されます。 これは、特定のファサード ルートの API ドキュメントをすばやく調べるのに便利なツールです。 該当する場合、[サービスコンテナ結合](/docs/{{version}}/container) キーも含まれます。

<div class="overflow-auto">

ファサード | クラス | サービスコンテナの結合キー
------------- | ------------- | -------------
App  |  [Illuminate\Foundation\Application](https://laravel.com/api/{{version}}/Illuminate/Foundation/Application.html)  |  `app`
Artisan  |  [Illuminate\Contracts\Console\Kernel](https://laravel.com/api/{{version}}/Illuminate/Contracts/Console/Kernel.html)  |  `artisan`
Auth  |  [Illuminate\Auth\AuthManager](https://laravel.com/api/{{version}}/Illuminate/Auth/AuthManager.html)  |  `auth`
Auth (Instance)  |  [Illuminate\Contracts\Auth\Guard](https://laravel.com/api/{{version}}/Illuminate/Contracts/Auth/Guard.html)  |  `auth.driver`
Blade  |  [Illuminate\View\Compilers\BladeCompiler](https://laravel.com/api/{{version}}/Illuminate/View/Compilers/BladeCompiler.html)  |  `blade.compiler`
Broadcast  |  [Illuminate\Contracts\Broadcasting\Factory](https://laravel.com/api/{{version}}/Illuminate/Contracts/Broadcasting/Factory.html)  |  &nbsp;
Broadcast (Instance)  |  [Illuminate\Contracts\Broadcasting\Broadcaster](https://laravel.com/api/{{version}}/Illuminate/Contracts/Broadcasting/Broadcaster.html)  |  &nbsp;
Bus  |  [Illuminate\Contracts\Bus\Dispatcher](https://laravel.com/api/{{version}}/Illuminate/Contracts/Bus/Dispatcher.html)  |  &nbsp;
Cache  |  [Illuminate\Cache\CacheManager](https://laravel.com/api/{{version}}/Illuminate/Cache/CacheManager.html)  |  `cache`
Cache (Instance)  |  [Illuminate\Cache\Repository](https://laravel.com/api/{{version}}/Illuminate/Cache/Repository.html)  |  `cache.store`
Config  |  [Illuminate\Config\Repository](https://laravel.com/api/{{version}}/Illuminate/Config/Repository.html)  |  `config`
Cookie  |  [Illuminate\Cookie\CookieJar](https://laravel.com/api/{{version}}/Illuminate/Cookie/CookieJar.html)  |  `cookie`
Crypt  |  [Illuminate\Encryption\Encrypter](https://laravel.com/api/{{version}}/Illuminate/Encryption/Encrypter.html)  |  `encrypter`
Date  |  [Illuminate\Support\DateFactory](https://laravel.com/api/{{version}}/Illuminate/Support/DateFactory.html)  |  `date`
DB  |  [Illuminate\Database\DatabaseManager](https://laravel.com/api/{{version}}/Illuminate/Database/DatabaseManager.html)  |  `db`
DB (Instance)  |  [Illuminate\Database\Connection](https://laravel.com/api/{{version}}/Illuminate/Database/Connection.html)  |  `db.connection`
Event  |  [Illuminate\Events\Dispatcher](https://laravel.com/api/{{version}}/Illuminate/Events/Dispatcher.html)  |  `events`
File  |  [Illuminate\Filesystem\Filesystem](https://laravel.com/api/{{version}}/Illuminate/Filesystem/Filesystem.html)  |  `files`
Gate  |  [Illuminate\Contracts\Auth\Access\Gate](https://laravel.com/api/{{version}}/Illuminate/Contracts/Auth/Access/Gate.html)  |  &nbsp;
Hash  |  [Illuminate\Contracts\Hashing\Hasher](https://laravel.com/api/{{version}}/Illuminate/Contracts/Hashing/Hasher.html)  |  `hash`
Http  |  [Illuminate\Http\Client\Factory](https://laravel.com/api/{{version}}/Illuminate/Http/Client/Factory.html)  |  &nbsp;
Lang  |  [Illuminate\Translation\Translator](https://laravel.com/api/{{version}}/Illuminate/Translation/Translator.html)  |  `translator`
Log  |  [Illuminate\Log\LogManager](https://laravel.com/api/{{version}}/Illuminate/Log/LogManager.html)  |  `log`
Mail  |  [Illuminate\Mail\Mailer](https://laravel.com/api/{{version}}/Illuminate/Mail/Mailer.html)  |  `mailer`
Notification  |  [Illuminate\Notifications\ChannelManager](https://laravel.com/api/{{version}}/Illuminate/Notifications/ChannelManager.html)  |  &nbsp;
Password  |  [Illuminate\Auth\Passwords\PasswordBrokerManager](https://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBrokerManager.html)  |  `auth.password`
Password (Instance)  |  [Illuminate\Auth\Passwords\PasswordBroker](https://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBroker.html)  |  `auth.password.broker`
Pipeline (Instance)  |  [Illuminate\Pipeline\Pipeline](https://laravel.com/api/{{version}}/Illuminate/Pipeline/Pipeline.html)  |  &nbsp;
Queue  |  [Illuminate\Queue\QueueManager](https://laravel.com/api/{{version}}/Illuminate/Queue/QueueManager.html)  |  `queue`
Queue (Instance)  |  [Illuminate\Contracts\Queue\Queue](https://laravel.com/api/{{version}}/Illuminate/Contracts/Queue/Queue.html)  |  `queue.connection`
Queue (Base Class)  |  [Illuminate\Queue\Queue](https://laravel.com/api/{{version}}/Illuminate/Queue/Queue.html)  |  &nbsp;
Redirect  |  [Illuminate\Routing\Redirector](https://laravel.com/api/{{version}}/Illuminate/Routing/Redirector.html)  |  `redirect`
Redis  |  [Illuminate\Redis\RedisManager](https://laravel.com/api/{{version}}/Illuminate/Redis/RedisManager.html)  |  `redis`
Redis (Instance)  |  [Illuminate\Redis\Connections\Connection](https://laravel.com/api/{{version}}/Illuminate/Redis/Connections/Connection.html)  |  `redis.connection`
Request  |  [Illuminate\Http\Request](https://laravel.com/api/{{version}}/Illuminate/Http/Request.html)  |  `request`
Response  |  [Illuminate\Contracts\Routing\ResponseFactory](https://laravel.com/api/{{version}}/Illuminate/Contracts/Routing/ResponseFactory.html)  |  &nbsp;
Response (Instance)  |  [Illuminate\Http\Response](https://laravel.com/api/{{version}}/Illuminate/Http/Response.html)  |  &nbsp;
Route  |  [Illuminate\Routing\Router](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html)  |  `router`
Schema  |  [Illuminate\Database\Schema\Builder](https://laravel.com/api/{{version}}/Illuminate/Database/Schema/Builder.html)  |  &nbsp;
Session  |  [Illuminate\Session\SessionManager](https://laravel.com/api/{{version}}/Illuminate/Session/SessionManager.html)  |  `session`
Session (Instance)  |  [Illuminate\Session\Store](https://laravel.com/api/{{version}}/Illuminate/Session/Store.html)  |  `session.store`
Storage  |  [Illuminate\Filesystem\FilesystemManager](https://laravel.com/api/{{version}}/Illuminate/Filesystem/FilesystemManager.html)  |  `filesystem`
Storage (Instance)  |  [Illuminate\Contracts\Filesystem\Filesystem](https://laravel.com/api/{{version}}/Illuminate/Contracts/Filesystem/Filesystem.html)  |  `filesystem.disk`
URL  |  [Illuminate\Routing\UrlGenerator](https://laravel.com/api/{{version}}/Illuminate/Routing/UrlGenerator.html)  |  `url`
Validator  |  [Illuminate\Validation\Factory](https://laravel.com/api/{{version}}/Illuminate/Validation/Factory.html)  |  `validator`
Validator (Instance)  |  [Illuminate\Validation\Validator](https://laravel.com/api/{{version}}/Illuminate/Validation/Validator.html)  |  &nbsp;
View  |  [Illuminate\View\Factory](https://laravel.com/api/{{version}}/Illuminate/View/Factory.html)  |  `view`
View (Instance)  |  [Illuminate\View\View](https://laravel.com/api/{{version}}/Illuminate/View/View.html)  |  &nbsp;
Vite  |  [Illuminate\Foundation\Vite](https://laravel.com/api/{{version}}/Illuminate/Foundation/Vite.html)  |  &nbsp;

</div>
