# エラー処理

- [はじめに](#introduction)
- [設定](#configuration)
- [例外ハンドラ](#the-exception-handler)
    - [例外のレポート](#reporting-exceptions)
    - [例外のログレベル](#exception-log-levels)
    - [タイプごとの例外の無視](#ignoring-exceptions-by-type)
    - [例外のレンダリング](#rendering-exceptions)
    - [Reportable 例外と Renderable 例外](#renderable-exceptions)
- [レポートした例外の調節](#throttling-reported-exceptions)
- [HTTP 例外](#http-exceptions)
    - [HTTP エラーページのカスタマイズ](#custom-http-error-pages)

<a name="introduction"></a>
## はじめに

新しい Laravel プロジェクトを開始すれば、エラーと例外の処理がすでに設定されています。`App\Exceptions\Handler` クラスは、アプリケーションによってスローされたすべての例外がログに記録され、ユーザーに表示される場所です。このドキュメントでは、このクラスについてさらに詳しく説明します。

<a name="configuration"></a>
## 設定

`config/app.php` 設定ファイルの `debug` オプションは、エラーに関する情報が実際にユーザーにどの程度表示されるかを指定します。デフォルトでは、このオプションは `.env` ファイルに保存されている `APP_DEBUG` 環境変数の値を尊重するように設定されています。

ローカルでの開発中は、`APP_DEBUG` 環境変数を `true` に設定する必要があります。**実稼働環境では、この値は常に `false` である必要があります。運用環境で値が `true` に設定されている場合、機密の構成値がアプリケーションのエンドユーザーに公開される危険があります。**

<a name="the-exception-handler"></a>
## 例外ハンドラ

<a name="reporting-exceptions"></a>
### 例外の報告

すべての例外は `App\Exceptions\Handler` クラスによって処理されます。このクラスには、カスタム例外レポートとレンダリングコールバックを登録できる `register` メソッドが含まれています。これらの各概念を詳しく見ていきます。例外レポートは、例外を記録したり、[Flare](https://flareapp.io)、[Bugsnag](https://bugsnag.com)、[Sentry](https://github.com/getsentry/sentry-laravel) などの外部サービスに例外を送信したりするために使用されます。デフォルトでは、例外は [ログ](/docs/{{version}}/logging) 設定に基づいて記録されます。また、例外を自由にログに記録することもできます。

さまざまなタイプの例外をさまざまな方法で報告する必要がある場合は、`reportable` メソッドを使用して、特定のタイプの例外を報告する必要があるときに実行されるクロージャを登録できます。Laravel は、クロージャのタイプヒントを調べることで、クロージャが報告する例外のタイプを判断します。

    use App\Exceptions\InvalidOrderException;

    /**
     * アプリケーションの例外処理コールバックを登録
     */
    public function register(): void
    {
        $this->reportable(function (InvalidOrderException $e) {
            // ...
        });
    }

 `reportable` メソッドを使用してカスタム例外レポートコールバックを登録すると、Laravel はアプリケーションのデフォルトのログ設定を使用して例外をログに記録します。デフォルトのログスタックへの例外の伝播を停止したい場合は、レポートコールバックを定義するときに `stop` メソッドを使用するか、コールバックから `false` を返します。

    $this->reportable(function (InvalidOrderException $e) {
        // ...
    })->stop();

    $this->reportable(function (InvalidOrderException $e) {
        return false;
    });

> **Note**  
> 特定の例外の例外レポートをカスタマイズするには、[レポート可能な例外](/docs/{{version}}/errors#renderable-exceptions) を利用することもできます。

<a name="global-log-context"></a>
#### グローバル ログ コンテキスト

利用可能な場合、Laravel は現在のユーザーの ID をコンテキストデータとして、すべて例外のログメッセージに自動的に追加します。アプリケーションの `App\Exceptions\Handler` クラスで `context` メソッドを定義することにより、独自のグローバルコンテキストデータを定義できます。この情報は、アプリケーションによって書き込まれるすべての例外のログメッセージに含まれます。

    /**
     * ロギング用のデフォルトのコンテキスト変数を取得
     *
     * @return array<string, mixed>
     */
    protected function context(): array
    {
        return array_merge(parent::context(), [
            'foo' => 'bar',
        ]);
    }

<a name="exception-log-context"></a>
#### 例外ログのコンテキスト

すべてのログメッセージにコンテキストを追加すると便利ですが、場合によっては、特定の例外にログに含めたい固有のコンテキストが含まれる場合があります。アプリケーションの例外の1つで `context` メソッドを定義すると、例外のログエントリに追加する必要がある、その例外に関連するデータを指定できます。

    <?php

    namespace App\Exceptions;

    use Exception;

    class InvalidOrderException extends Exception
    {
        // ...

        /**
         * 例外のコンテキスト情報を取得
         *
         * @return array<string, mixed>
         */
        public function context(): array
        {
            return ['order_id' => $this->orderId];
        }
    }

<a name="the-report-helper"></a>
####  `レポート` ヘルパ

場合によっては、例外を報告しても現在のリクエストの処理を続行する必要がある場合があります。`report` ヘルパ関数を使用すると、ユーザーにエラーページを表示せずに、例外ハンドラを介して例外を迅速に報告できます。

    public function isValid(string $value): bool
    {
        try {
            // Validate the value...
        } catch (Throwable $e) {
            report($e);

            return false;
        }
    }

<a name="deduplicating-reported-exceptions"></a>
#### レポートされた例外の重複排除

アプリケーション全体で `report` 関数を使用している場合、同じ例外を複数回報告することがあり、ログに重複したエントリが作成されることがあります。

例外の単一インスタンスが一度だけ報告されるようにしたい場合は、アプリケーションの `App\Exceptions\Handler` クラス内で `$withoutDuplicates` プロパティを `true` に設定します。

```php
namespace App\Exceptions;

use Illuminate\Foundation\Exceptions\Handler as ExceptionHandler;

class Handler extends ExceptionHandler
{
    /**
     * 例外インスタンスは一度だけ報告する必要があることを示す
     *
     * @var bool
     */
    protected $withoutDuplicates = true;

    // ...
}
```

これで、同じ例外インスタンスで `report` ヘルパが呼び出された場合、最初の呼び出しのみが報告されます。

```php
$original = new RuntimeException('Whoops!');

report($original); // レポートされる

try {
    throw $original;
} catch (Throwable $caught) {
    report($caught); // 無視される
}

report($original); // 無視される
report($caught); // 無視される
```

<a name="exception-log-levels"></a>
### 例外のログレベル

メッセージがアプリケーションの [ログ](/docs/{{version}}/logging) に書き込まれる場合、そのメッセージは指定された [ログレベル](/docs/{{version}}/logging#log-levels) で書き込まれます。これはログに記録されるメッセージの重大度または重要性を表します。

上記で述べたように、`reportable` メソッドを使用してカスタム例外レポートコールバックを登録した場合でも、Laravel はアプリケーションのデフォルトのログ設定を使用して例外をログに記録します。ただし、ログレベルはメッセージが記録されるチャネルに影響を与える場合があるため、特定の例外を記録するログレベルを設定することもできます。

これを実現するには、アプリケーションの例外ハンドラで `$levels` プロパティを定義できます。このプロパティには、例外のタイプとそれに関連するログレベルの配列が含まれている必要があります。

    use PDOException;
    use Psr\Log\LogLevel;

    /**
     * 例外タイプとそれに対応するカスタムログレベルのリスト
     *
     * @var array<class-string<\Throwable>, \Psr\Log\LogLevel::*>
     */
    protected $levels = [
        PDOException::class => LogLevel::CRITICAL,
    ];

<a name="ignoring-exceptions-by-type"></a>
### タイプごとの例外の無視

アプリケーションを構築するとき、レポートをしたくないタイプの例外が発生することがあります。これらの例外を無視するには、アプリケーションの例外ハンドラで `$dontReport` プロパティを定義します。このプロパティに追加したクラスは決して報告されません。ただし、カスタムレンダリングロジックがまだ存在する場合があります。

    use App\Exceptions\InvalidOrderException;

    /**
     * レポートしない例外タイプのリスト
     *
     * @var array<int, class-string<\Throwable>>
     */
    protected $dontReport = [
        InvalidOrderException::class,
    ];

内部的には、Laravel はすでに、404 HTTP エラーや無効な CSRF トークンによって生成された 419 HTTP レスポンスに起因する例外など、一部のタイプのエラーを無視します。特定のタイプの例外の無視を停止するように Laravel に指示したい場合は、例外ハンドラの `register` メソッド内で `stopIgnoring` メソッドを呼び出します。

    use Symfony\Component\HttpKernel\Exception\HttpException;

    /**
     * アプリケーションの例外処理コールバックを登録
     */
    public function register(): void
    {
        $this->stopIgnoring(HttpException::class);

        // ...
    }

<a name="rendering-exceptions"></a>
### 例外のレンダリング

デフォルトでは、Laravel の例外ハンドラは例外を HTTP レスポンスに変換します。ただし、特定のタイプの例外に対してカスタムレンダリングクロージャを自由に登録することも可能です。これを行うには、例外ハンドラ内で `renderable` メソッドを呼び出します。

`renderable` メソッドに渡すクロージャは、`Illuminate\Http\Response` のインスタンスを返す必要があります。これは、`response` ヘルパを介して生成できます。Laravel は、クロージャのタイプヒントを調べることによって、クロージャがレンダリングする例外のタイプを決定します。

    use App\Exceptions\InvalidOrderException;
    use Illuminate\Http\Request;

    /**
     * アプリケーションの例外処理コールバックを登録
     */
    public function register(): void
    {
        $this->renderable(function (InvalidOrderException $e, Request $request) {
            return response()->view('errors.invalid-order', [], 500);
        });
    }

また、`renderable` メソッドを使用して、`NotFoundHttpException` などの Laravel や Symfony の組み込み例外のレンダリング動作をオーバーライドすることもできます。`renderable` メソッドに指定したクロージャが値を返さない場合、Laravel のデフォルトの例外レンダリングが利用されます。

    use Illuminate\Http\Request;
    use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;

    /**
     * アプリケーションの例外処理コールバックを登録
     */
    public function register(): void
    {
        $this->renderable(function (NotFoundHttpException $e, Request $request) {
            if ($request->is('api/*')) {
                return response()->json([
                    'message' => 'Record not found.'
                ], 404);
            }
        });
    }

<a name="renderable-exceptions"></a>
### Reportable 例外と Renderable 例外

例外ハンドラの `register` メソッドでカスタムレポートやレンダリング動作を定義する代わりに、アプリケーションの例外に `report` と `render` メソッドを直接定義できます。これらのメソッドが存在する場合、フレームワークによって自動的に呼び出されます。

    <?php

    namespace App\Exceptions;

    use Exception;
    use Illuminate\Http\Request;
    use Illuminate\Http\Response;

    class InvalidOrderException extends Exception
    {
        /**
         * 例外をレポート
         */
        public function report(): void
        {
            // ...
        }

        /**
         * 例外を HTTP レスポンスにレンダリング
         */
        public function render(Request $request): Response
        {
            return response(/* ... */);
        }
    }

Laravel や Symfony の組み込み例外など、すでにレンダリング可能な例外を拡張する場合、例外の `render` メソッドから `false` を返して、例外のデフォルトの HTTP レスポンスをレンダリングできます。

    /**
     * 例外を HTTP レスポンスにレンダリング
     */
    public function render(Request $request): Response|bool
    {
        if (/** この例外をレポートするか判断*/) {

            return response(/* ... */);
        }

        return false;
    }

特定の条件が満たされた場合にのみ必要なカスタムレポートロジックが例外に含まれている場合は、デフォルトの例外処理設定を使用して例外をレポートするように Laravel に指示する必要がある場合があります。これを実現するには、例外の `report` メソッドから `false` を返します。

    /**
     * 例外をレポート
     */
    public function report(): bool
    {
        if (/** この例外をレポートするか判断 */) {

            // ...

            return true;
        }

        return false;
    }

> **Note**  
> `report` メソッドの必要な依存関係をタイプヒントで指定すると、それらはLaravelの [サービスコンテナ](/docs/{{version}}/container) によってメソッドに自動的に依存性注入されます。

<a name="throttling-reported-exceptions"></a>
### レポートした例外の調整

アプリケーションが非常に多くの例外をレポートする場合、実際にログに記録される、またはアプリケーションの外部エラーを追跡するサービスに送信する例外の数量を調整することができます。

ランダムな例外のサンプルレートを取得するには、例外ハンドラの `throttle` メソッドから `Lottery` インスタンスを返す方法があります。`App\Exceptions\Handler` クラスにこのメソッドが含まれていない場合は、単にクラスに追加するだけで済みます。

```php
use Illuminate\Support\Lottery;
use Throwable;

/**
 * 受け取る例外を調整
 */
protected function throttle(Throwable $e): mixed
{
    return Lottery::odds(1, 1000);
}
```

例外タイプに基づいて条件付きでサンプリングすることも可能です。特定の例外クラスのインスタンスのみをサンプルしたい場合は、そのクラスの `Lottery` インスタンスのみを返します。

```php
use App\Exceptions\ApiMonitoringException;
use Illuminate\Support\Lottery;
use Throwable;

/**
 * 受け取る例外を調整
 */
protected function throttle(Throwable $e): mixed
{
    if ($e instanceof ApiMonitoringException) {
        return Lottery::odds(1, 1000);
    }
}
```

また `Lottery` の代わりに `Limit` インスタンスを返すことで、外部のエラー追跡サービスに記録、または送信される例外の制限レートを指定できます。これは、アプリケーションで使用されているサードパーティのサービスがダウンした場合など、ログに大量のレポートが登録されるような突然の例外バーストから保護したい場合に役立ちます。

```php
use Illuminate\Broadcasting\BroadcastException;
use Illuminate\Cache\RateLimiting\Limit;
use Throwable;

/**
 * 受け取る例外を調整
 */
protected function throttle(Throwable $e): mixed
{
    if ($e instanceof BroadcastException) {
        return Limit::perMinute(300);
    }
}
```

デフォルトで、リミットは例外のクラスをレートリミットキーとして使用します。これをカスタマイズするには `Limit` の `by` メソッドを使用して独自のキーを指定します。

```php
use Illuminate\Broadcasting\BroadcastException;
use Illuminate\Cache\RateLimiting\Limit;
use Throwable;

/**
 * 受信する例外を調節
 */
protected function throttle(Throwable $e): mixed
{
    if ($e instanceof BroadcastException) {
        return Limit::perMinute(300)->by($e->getMessage());
    }
}
```

もちろん、さまざまな例外に対して `Lottery` インスタンスと `Limit` インスタンスを組み合わせて返すこともできます。

```php
use App\Exceptions\ApiMonitoringException;
use Illuminate\Broadcasting\BroadcastException;
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Lottery;
use Throwable;

/**
 * 受信する例外を調節
 */
protected function throttle(Throwable $e): mixed
{
    return match (true) {
        $e instanceof BroadcastException => Limit::perMinute(300),
        $e instanceof ApiMonitoringException => Lottery::odds(1, 1000),
        default => Limit::none(),
    };
}
```

<a name="http-exceptions"></a>
## HTTP 例外

一部の例外は、サーバーからの HTTP エラーコードを示します。たとえば、「ページが見つかりません」エラー (404)、「不正エラー」(401)、または開発者が生成した 500 エラーである可能性もあります。アプリケーションのどこからでもそのようなレスポンスを生成したい場合は、`abort` ヘルパを使用できます。

    abort(404);

<a name="custom-http-error-pages"></a>
### HTTP エラーページのカスタマイズ

Laravel を使用すると、さまざまな HTTP ステータスコードのカスタムエラーページを簡単に表示できます。たとえば、404 HTTP ステータスコードのエラーページをカスタマイズするには、 `resources/views/errors/404.blade.php` ビューテンプレートを作成します。このビューは、アプリケーションによって生成されたすべての 404 エラーに対してレンダリングされます。このディレクトリ内のビューには、対応する HTTP ステータスコードと一致する名前を付ける必要があります。`abort` 関数によって生成された `Symfony\Component\HttpKernel\Exception\HttpException` インスタンスは、`$exception` 変数としてビューに渡されます。

    <h2>{{ $exception->getMessage() }}</h2>

Laravel のデフォルトのエラーページテンプレートは、`vendor:publish` Artisan コマンドを使用して公開することができます。テンプレートが公開されたら、好みに合わせてカスタマイズできます。

```shell
php artisan vendor:publish --tag=laravel-errors
```

<a name="fallback-http-error-pages"></a>
#### HTTP エラーページのフォールバック

特定の一連の HTTP ステータスコードに対応する「フォールバック」エラーページを定義することもできます。このページは、発生した特定の HTTP ステータスコードに対応するページがない場合に表示されます。これを実現するには、アプリケーションの `resources/views/errors` ディレクトリに `4xx.blade.php` テンプレートと `5xx.blade.php` テンプレートを定義します。