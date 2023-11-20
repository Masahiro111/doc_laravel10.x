# ログ

- [はじめに](#introduction)
- [設定](#configuration)
    - [利用可能なチャンネルドライバ](#available-channel-drivers)
    - [チャンネルの事前設定](#channel-prerequisites)
    - [非推奨ワーニングのログ](#logging-deprecation-warnings)
- [ログスタックの構築](#building-log-stacks)
- [ログメッセージの書き込み](#writing-log-messages)
    - [コンテキスト情報](#contextual-information)
    - [特定チャンネルへの書き込み](#writing-to-specified-channels)
- [Monolog チャンネルのカスタマイズ](#monolog-channel-customization)
    - [チャンネル向け Monolog のカスタマイズ](#customizing-monolog-for-channels)
    - [Monolog ハンドラチャンネルの作成](#creating-monolog-handler-channels)
    - [ファクトリ経由でカスタムチャンネル作成](#creating-custom-channels-via-factories)
- [Pail を使用したログメッセージの追跡](#tailing-log-messages-using-pail)
    - [インストール](#paill-installation)
    - [使い方](#pail-usage)
    - [ログのフィルタリング](#pail-filtering-logs)

<a name="introduction"></a>
## はじめに

アプリケーション内で何が起こっているかを詳しく知るために、Laravel はメッセージをファイル、システムエラーログ、さらには Slack に記録してチーム全体に通知できる堅牢なログサービスを提供します。

Laravelのログは「チャンネル」に基づいています。各チャンネルは、ログ情報を書き込む特定の方法を表します。たとえば、`single` チャンネルはログファイルを単一のログファイルに書き込みますが、`slack` チャンネルはログメッセージを Slack に送信します。ログメッセージは、重大度に基づいて複数のチャンネルに書き込まれる場合があります。

Laravel は内部的に、さまざまな強力なログハンドラのサポートを提供する [Monolog](https://github.com/Seldaek/monolog) ライブラリを利用しています。Laravel を使用すると、これらのハンドラの設定が簡単になり、それらを組み合わせてアプリケーションのログ処理をカスタマイズできるようになります。

<a name="configuration"></a>
## 設定

アプリケーションのログ動作の設定オプションはすべて `config/logging.php` 設定ファイルに格納されています。このファイルを使用すると、アプリケーションのログチャンネルを設定できるため、使用可能な各チャンネルとそのオプションを必ず確認してください。以下でいくつかの一般的なオプションを確認します。

デフォルトでは、Laravel はメッセージをログに記録するときに `stack` チャンネルを使用します。`stack` チャンネルは、複数のログチャンネルを1つのチャンネルに集約するために使用されます。スタックの構築の詳細については、[以下のドキュメント](#building-log-stacks) を参照してください。

<a name="configuring-the-channel-name"></a>
#### チャンネル名の設定

デフォルトで、Monolog は、`production` や `local` など、現在の環境に一致する「チャンネル名」でインスタンス化されます。この値を変更するには、チャンネル設定に `name` オプションを追加します。

    'stack' => [
        'driver' => 'stack',
        'name' => 'channel-name',
        'channels' => ['single', 'slack'],
    ],

<a name="available-channel-drivers"></a>
### 利用可能なチャンネルドライバ

各ログチャンネルは「ドライバ」によって駆動されます。ドライバは、ログメッセージが実際にどのように、またどこに記録されるかを決定します。次のログチャンネルドライバは、すべての Laravel アプリケーションで利用できます。これらのドライバのほとんどのエントリはアプリケーションの `config/logging.php` 設定ファイルにすでに用意されているため、必ずこのファイルを確認して内容をよく理解してください。

<div class="overflow-auto">

名前 | 説明
------------- | -------------
`custom` | 指定のファクトリを呼び出してチャンネルを作成するドライバ
`daily` | 毎日ファイルが切り替わる `RotatingFileHandler` ベースの Monolog ドライバ
`errorlog` | `ErrorLogHandler` ベースの Monolog ドライバ
`monolog` | サポートされている Monolog ハンドラを使用できる Monolog ファクトリ ドライバ
`papertrail` | `SyslogUdpHandler` ベースの Monolog ドライバ
`single` | 単一のファイルまたはパスベースのロガーチャンネル (`StreamHandler`)
'slack' | `SlackWebhookHandler` ベースの Monolog ドライバ
`stack` | 「マルチチャンネル」チャンネルの作成を容易にするラッパ
`syslog` | `SyslogHandler` ベースの Monolog ドライバ

</div>

> **Note**  
> `monolog` ドライバと`custom` ドライバの詳細については、[高度なチャンネルのカスタマイズ](#monolog-channel-customization) に関するドキュメントを参照してください。

<a name="channel-prerequisites"></a>
### チャンネルの事前設定

<a name="configuring-the-single-and-daily-channels"></a>
#### Single チャンネルと Daily チャンネルの設定

`single` チャンネルと `daily` チャンネルには、`bubble`、`permission`、`locking` の3オプションの設定オプションがあります。

<div class="overflow-auto">

名前 | 説明 | デフォルト
------------- | ------------- | -------------
`bubble` | メッセージが処理された後に、他のチャンネルにバブルアップする必要があるか | `true`
`locking` | ログファイルに書き込む前に、ログファイルのロックをするか | `false`
`permission` | ログファイルのパーミッション | `0644`

</div>

さらに、`daily` チャンネルの保持ポリシーは、`days` オプションを使用して設定できます。

<div class="overflow-auto">

名前 | 説明 | デフォルト
------------- |-------------------------------------------------------------------| -------------
`days` | デイリーログファイルを保持する日数 | `7`

</div>

<a name="configuring-the-papertrail-channel"></a>
#### Papertrail チャンネルの設定

`papertrail` チャンネルには、`host` と `port` 設定オプションが必要です。これらの値は、[Papertrail](https://help.papertrailapp.com/kb/configuration/cconfiguring-centralized-logging-from-php-apps/#send-events-from-php-app) から取得できます。

<a name="configuring-the-slack-channel"></a>
#### Slack チャンネルの設定

`slack` チャンネルには `url` 構成オプションが必要です。この URL は、Slack チーム用に設定した [受信 Webhook](https://slack.com/apps/A0F7XDUAZ-incoming-webhooks) の URL と一致する必要があります。

デフォルトでは、Slack は `critical` レベル以上のログのみを受信します。ただし `config/logging.php` 設定ファイルでSlack ログチャンネルの設定配列内の `level` 設定オプションを変更することで調整できます。

<a name="logging-deprecation-warnings"></a>
### 非推奨ワーニングのログ

PHP、Laravel、およびその他のライブラリは、機能の一部が非推奨になり、将来のバージョンで削除されることをユーザーに通知することがよくあります。これらの非推奨の警告をログに記録したい場合は、アプリケーションの `config/logging.php` 設定ファイルで希望の `deprecations` ログチャンネルを指定できます。

    'deprecations' => env('LOG_DEPRECATIONS_CHANNEL', 'null'),

    'channels' => [
        ...
    ]

または、`deprecations` という名前のログチャンネルを定義することもできます。この名前のログチャンネルが存在する場合は、非推奨のログを記録するために常に使用されます。

    'channels' => [
        'deprecations' => [
            'driver' => 'single',
            'path' => storage_path('logs/php-deprecation-warnings.log'),
        ],
    ],

<a name="building-log-stacks"></a>
## ログスタックの構築

前述したように、`stack` ドライバを使用すると、便宜上、複数のチャンネルを1つのログチャンネルに結合できます。ログスタックの使用方法を説明するために、本番の運用アプリケーションで見られる構成例を見てみましょう。

    'channels' => [
        'stack' => [
            'driver' => 'stack',
            'channels' => ['syslog', 'slack'],
        ],

        'syslog' => [
            'driver' => 'syslog',
            'level' => 'debug',
        ],

        'slack' => [
            'driver' => 'slack',
            'url' => env('LOG_SLACK_WEBHOOK_URL'),
            'username' => 'Laravel Log',
            'emoji' => ':boom:',
            'level' => 'critical',
        ],
    ],

この構成を詳しく見てみましょう。まず、`stack` チャンネルが `channels` オプションを介して他の2つのチャンネル `syslog` と `slack` を集約していることに注目してください。したがって、メッセージをログに記録する場合、これらのチャンネルの両方にメッセージをログに記録する機会があります。ただし、以下で説明するように、これらのチャンネルが実際にメッセージをログに記録するかどうかは、メッセージの重大度/「レベル」によって決定される場合があります。

<a name="log-levels"></a>
#### ログレベル

上記の例の `syslog` および `slack` チャンネル設定に存在する `level` 設定オプションに注目してください。このオプションは、チャンネルによってログに記録されるメッセージの最小「レベル」を決定します。Laravel のログサービスを強化する Monolog は、[RFC5424 仕様](https://tools.ietf.org/html/rfc5424) で定義されているすべてのログレベルを提供します。ログレベルの重要度の高い順に **emergency**、**alert**、**critical**、**error**、**warning**、**notice**、**info**、**debug** となります。

では、`debug` メソッドを使用してメッセージをログに記録するとしましょう。

    Log::debug('An informational message.');

この構成では、`syslog` チャンネルがメッセージをシステムログに書き込みます。ただし、エラーメッセージは`critical` 以上ではないため、Slack には送信されません。ただし、`emergency` メッセージをログに記録すると、`emergency` レベルが両方のチャンネルの最小レベルしきい値を超えているため、メッセージはシステムログと Slack の両方に送信されます。

    Log::emergency('The system is down!');

<a name="writing-log-messages"></a>
## ログメッセージの書き込み

`Log` [ファサード](/docs/{{version}}/facades) を使用して、ログに情報を書き込むことができます。前述したように、ロガーは [RFC5424 仕様](https://tools.ietf.org/html/rfc5424) で定義されている8つのログレベル（**emergency**、**alert**、**critical**、**error**、**warning**、**notice**、**info**、**debug**）を提供します。

    use Illuminate\Support\Facades\Log;

    Log::emergency($message);
    Log::alert($message);
    Log::critical($message);
    Log::error($message);
    Log::warning($message);
    Log::notice($message);
    Log::info($message);
    Log::debug($message);

これらのメソッドのいずれかを呼び出して、対応するレベルのメッセージをログに記録できます。デフォルトでは、メッセージは `logging` 設定ファイルで設定されたデフォルトのログチャンネルに書き込まれます。

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\User;
    use Illuminate\Support\Facades\Log;
    use Illuminate\View\View;

    class UserController extends Controller
    {
        /**
         * 指定ユーザーのプロフィールを表示
         */
        public function show(string $id): View
        {
            Log::info('Showing the user profile for user: {id}', ['id' => $id]);

            return view('user.profile', [
                'user' => User::findOrFail($id)
            ]);
        }
    }

<a name="contextual-information"></a>
### コンテキスト情報

コンテキストデータの配列を log メソッドに渡すことができます。このコンテキストデータはフォーマットされて、ログメッセージとともに表示されます。

    use Illuminate\Support\Facades\Log;

    Log::info('User {id} failed to login.', ['id' => $user->id]);

場合によっては、特定チャンネルの後に続くすべてのログエントリに含める必要があるコンテキスト情報を指定したい場合があります。たとえば、アプリケーションへの各受信リクエストに関連付けられたリクエスト ID をログに記録したい場合があります。これを実現するには、`Log` ファサードの `withContext` メソッドを呼び出します。

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Log;
    use Illuminate\Support\Str;
    use Symfony\Component\HttpFoundation\Response;

    class AssignRequestId
    {
        /**
         * 受信リクエストの処理
         *
         * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
         */
        public function handle(Request $request, Closure $next): Response
        {
            $requestId = (string) Str::uuid();

            Log::withContext([
                'request-id' => $requestId
            ]);

            $response = $next($request);

            $response->headers->set('Request-Id', $requestId);

            return $response;
        }
    }

すべてのログチャンネル間でコンテキスト情報を共有したい場合は、`Log::shareContext()` メソッドを呼び出します。このメソッドは、作成されたすべてのチャンネルと、その後に作成されるすべてのチャンネルにコンテキスト情報を提供します。通常、`shareContext` メソッドは、アプリケーションサービスプロバイダーの `boot` メソッドから呼び出す必要があります。

    use Illuminate\Support\Facades\Log;
    use Illuminate\Support\Str;

    class AppServiceProvider
    {
        /**
         * アプリケーションサービスの初期起動処理
         */
        public function boot(): void
        {
            Log::shareContext([
                'invocation-id' => (string) Str::uuid(),
            ]);
        }
    }

<a name="writing-to-specific-channels"></a>
### 特定チャンネルへの書き込み

場合によっては、アプリケーションのデフォルトチャンネル以外のチャンネルにメッセージを記録したい場合があります。`Log` ファサードの `channel` メソッドを使用して、設定ファイルで定義されている任意のチャンネルを取得してログに記録できます。

    use Illuminate\Support\Facades\Log;

    Log::channel('slack')->info('Something happened!');

複数のチャンネルで構成されるログスタックを要求に応じて作成したい場合は、`stack` メソッドを使用します。

    Log::stack(['single', 'slack'])->info('Something happened!');

<a name="on-demand-channels"></a>
#### オンデマンドチャンネル

アプリケーションの `logging` 設定ファイルに設定が存在しなくても、実行時に設定を提供することでオンデマンドチャンネルを作成することもできます。これを実現するには、設定の配列を `Log` ファサードの `build` メソッドに渡してください。

    use Illuminate\Support\Facades\Log;

    Log::build([
      'driver' => 'single',
      'path' => storage_path('logs/custom.log'),
    ])->info('Something happened!');

オンデマンドログスタックにオンデマンドチャンネルを含めることもできます。これは、`stack` メソッドに渡す配列にオンデマンドチャンネルのインスタンスを含めることによって実現できます。

    use Illuminate\Support\Facades\Log;

    $channel = Log::build([
      'driver' => 'single',
      'path' => storage_path('logs/custom.log'),
    ]);

    Log::stack(['slack', $channel])->info('Something happened!');

<a name="monolog-channel-customization"></a>
## Monolog チャンネルのカスタマイズ

<a name="customizing-monolog-for-channels"></a>
### チャンネル向けの Monolog のカスタマイズ

場合によっては、既存のチャンネルに対して Monolog を設定する方法を完全に制御する必要があることもあります。たとえば、Laravel の組み込み `single` チャンネル用にカスタム Monolog `FormatterInterface` 実装を設定したい場合です。

まず、チャンネルの設定で `tap` 配列を定義します。`tap` 配列には、Monolog インスタンスの作成後にカスタマイズする機会を持つクラスのリストが含まれている必要があります。これらのクラスを配置する規定の場所はないため、アプリケーション内にこれらのクラスを含むディレクトリを自由に作成できます。

    'single' => [
        'driver' => 'single',
        'tap' => [App\Logging\CustomizeFormatter::class],
        'path' => storage_path('logs/laravel.log'),
        'level' => 'debug',
    ],

チャンネルで `tap` オプションを設定したら、Monolog インスタンスをカスタマイズするクラスを定義する準備が整います。このクラスに必要なのは、`Illuminate\Log\Logger` インスタンスを受け取る `__invoke` という1つのメソッドだけです。`Illuminate\Log\Logger` インスタンスは、基礎となっている Monolog インスタンスへの、全メソッド呼び出しをプロキシします。

    <?php

    namespace App\Logging;

    use Illuminate\Log\Logger;
    use Monolog\Formatter\LineFormatter;

    class CustomizeFormatter
    {
        /**
         * 指定のロガーインスタンスをカスタマイズ
         */
        public function __invoke(Logger $logger): void
        {
            foreach ($logger->getHandlers() as $handler) {
                $handler->setFormatter(new LineFormatter(
                    '[%datetime%] %channel%.%level_name%: %message% %context% %extra%'
                ));
            }
        }
    }

> **Note**  
> すべての「tap」クラスは [サービスコンテナ](/docs/{{version}}/container) によって解決されるため、必要なコンストラクタの依存関係は自動的に依存性注入されます。

<a name="creating-monolog-handler-channels"></a>
### Monolog ハンドラチャンネルの作成

Monolog にはさまざまな [利用可能なハンドラ](https://github.com/Seldaek/monolog/tree/main/src/Monolog/Handler) があり、Laravel にはそれぞれのハンドラが組み込まれていません。場合によっては、対応する Laravel ログドライバを持たない特定の Monolog ハンドラのインスタンスにすぎないカスタムチャンネルを作成したい場合があります。これらのチャンネルは、`monolog` ドライバを使用して簡単に作成できます。

`monolog` ドライバを使用する場合、`handler` 設定オプションを使用して、インスタンス化するハンドラを指定します。オプションで、ハンドラに必要なコンストラクタパラメータは、`with` 設定オプションを使用して指定できます。

    'logentries' => [
        'driver'  => 'monolog',
        'handler' => Monolog\Handler\SyslogUdpHandler::class,
        'with' => [
            'host' => 'my.logentries.internal.datahubhost.company.com',
            'port' => '10000',
        ],
    ],

<a name="monolog-formatters"></a>
#### Monolog フォーマッタ

`monolog` ドライバを使用する場合、Monolog `LineFormatter` がデフォルトのフォーマッタとして使用されます。ただし、`formatter` および `formatter_with` 設定オプションを使用して、ハンドラに渡すフォーマッタのタイプをカスタマイズできます。

    'browser' => [
        'driver' => 'monolog',
        'handler' => Monolog\Handler\BrowserConsoleHandler::class,
        'formatter' => Monolog\Formatter\HtmlFormatter::class,
        'formatter_with' => [
            'dateFormat' => 'Y-m-d',
        ],
    ],

独自のフォーマッタを提供できる Monolog ハンドラを使用している場合は、`formatter` 設定オプションの値を `default` に設定できます。

    'newrelic' => [
        'driver' => 'monolog',
        'handler' => Monolog\Handler\NewRelicHandler::class,
        'formatter' => 'default',
    ],


 <a name="monolog-processors"></a>
 #### Monolog プロセッサ

 Monolog は、メッセージをログに記録する前にメッセージを処理することもできます。独自のプロセッサを作成することも、[Monolog が提供する既存のプロセッサ](https://github.com/Seldaek/monolog/tree/main/src/Monolog/Processor) を使用することもできます。

`monolog` ドライバのプロセッサをカスタマイズしたい場合は、`processors` 設定値をチャンネルの設定に追加します。

     'memory' => [
         'driver' => 'monolog',
         'handler' => Monolog\Handler\StreamHandler::class,
         'with' => [
             'stream' => 'php://stderr',
         ],
         'processors' => [
             // シンプルな構文
             Monolog\Processor\MemoryUsageProcessor::class,

             // オプションあり
             [
                'processor' => Monolog\Processor\PsrLogMessageProcessor::class,
                'with' => ['removeUsedContextFields' => true],
            ],
         ],
     ],


<a name="creating-custom-channels-via-factories"></a>
### ファクトリ経由でカスタムチャンネル作成

Monolog のインスタンス化と設定を完全に制御できる完全なカスタムチャンネルを定義したい場合は、`config/logging.php` 設定ファイルで `custom` ドライバタイプを指定します。設定には、Monolog インスタンスを作成するために呼び出されるファクトリクラスの名前を含む `via` オプションを含める必要があります。

    'channels' => [
        'example-custom-channel' => [
            'driver' => 'custom',
            'via' => App\Logging\CreateCustomLogger::class,
        ],
    ],

`custom` ドライバチャンネルを設定したら、Monolog インスタンスを作成するクラスを定義する準備が整います。このクラスには、Monolog ロガーインスタンスを返す単一の `__invoke` メソッドのみが必要です。このメソッドは、チャンネル設定配列を唯一の引数として受け取ります。

    <?php

    namespace App\Logging;

    use Monolog\Logger;

    class CreateCustomLogger
    {
        /**
         * カスタム Monolog インスタンスを作成
         */
        public function __invoke(array $config): Logger
        {
            return new Logger(/* ... */);
        }
    }

<a name="tailing-log-messages-using-pail"></a>
## Pail を使用したログメッセージの追跡

多くの場合、アプリケーションのログをリアルタイムで追跡する必要がある場合があります。たとえば、問題をデバッグする場合や、アプリケーションのログで特定の種類のエラーを監視する場合などです。

Laravel Pail は、コマンドラインから直接 Laravel アプリケーションのログファイルに簡単にアクセスできるパッケージです。標準の `tail` コマンドとは異なり、Pail は Sentry や Flare を含むあらゆるログドライバで動作するように設計されています。さらに Pail は探しているログをすばやく見つけるのに役立つ一連の便利なフィルターを提供します。

<img src="https://laravel.com/img/docs/pail-example.png">

<a name="pail-installation"></a>
### インストール

> **Warning**
> Laravel Pail には [PHP 8.2 以上](https://php.net/releases/) と [PCNTL](https://www.php.net/manual/en/book.pcntl.php) 拡張機能が必要です。

まず、Composer パッケージマネージャーを使用して Pail をプロジェクトにインストールします。

```bash
composer require laravel/pail
```

<a name="pail-usage"></a>
### 使用方法

ログの追跡を開始するには、`pail` コマンドを実行します。

```bash
php artisan pail
```

出力の冗長性を高め、切り捨て (…) を避けるには、`-v` オプションを使用します。

```bash
php artisan pail -v
```

冗長性を最大限に高め、例外スタックトレースを表示するには、`-vv` オプションを使用します。

```bash
php artisan pail -vv
```

ログの追跡を停止するには、`Ctrl+C` を押してください。

<a name="pail-filtering-logs"></a>
### ログのフィルタリング

<a name="pail-filtering-logs-filter-option"></a>
#### `--filter`

`--filter` オプションを使用すると、タイプ、ファイル、メッセージ、スタックトレースの内容によってログをフィルタリングできます。

```bash
php artisan pail --filter="QueryException"
```

<a name="pail-filtering-logs-message-option"></a>
#### `--message`

ログのメッセージのみをフィルタリングするには、`--message` オプションを使用します。

```bash
php artisan pail --message="User created"
```

<a name="pail-filtering-logs-level-option"></a>
#### `--level`

`--level` オプションは、[log レベル](#log-levels) によってログをフィルタリングするために使用します。

```bash
php artisan pail --level=error
```

<a name="pail-filtering-logs-user-option"></a>
#### `--user`

特定のユーザーが認証されている間に書き込まれたログのみを表示するには、ユーザーの ID を `--user` オプションに指定します。

```bash
php artisan pail --user=1
```