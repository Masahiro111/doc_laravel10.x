# ディレクトリ構造

- [はじめに](#introduction)
- [プロジェクトルートディレクトリ](#the-root-directory)
     - [`app` ディレクトリ](#the-root-app-directory)
     - [`bootstrap` ディレクトリ](#the-bootstrap-directory)
     - [`config` ディレクトリ](#the-config-directory)
     - [`database` ディレクトリ](#the-database-directory)
     - [`public` ディレクトリ](#the-public-directory)
     - [`resources` ディレクトリ](#the-resources-directory)
     - [`routes` ディレクトリ](#the-routes-directory)
     - [`storage` ディレクトリ](#the-storage-directory)
     - [`tests` ディレクトリ](#the-tests-directory)
     - [`vendor` ディレクトリ](#the-vendor-directory)
- [App ディレクトリ](#the-app-directory)
     - [`Broadcasting` ディレクトリ](#the-broadcasting-directory)
     - [`Console` ディレクトリ](#the-console-directory)
     - [`Events` ディレクトリ](#the-events-directory)
     - [`Exceptions` ディレクトリ](#the-exceptions-directory)
     - [`Http` ディレクトリ](#the-http-directory)
     - [`Jobs` ディレクトリ](#the-jobs-directory)
     - [`Listeners` ディレクトリ](#the-listeners-directory)
     - [`Mail` ディレクトリ](#the-mail-directory)
     - [`Models` ディレクトリ](#the-models-directory)
     - [`Notifications` ディレクトリ](#the-notifications-directory)
     - [`Policies` ディレクトリ](#the-policies-directory)
     - [`Providers` ディレクトリ](#the-providers-directory)
     - [`Rules` ディレクトリ](#the-rules-directory)

<a name="introduction"></a>
＃＃ はじめに

デフォルトの Laravel アプリケーション構造は、大規模・小規模なアプリケーションのどちらにも適した優れた出発点を提供することを目的としています。しかし、アプリケーションの構成は自由に変更して構いません。Laravel は、Composer がクラスを自動読み込みできる限り、特定のクラスがどこにあるかにほとんど制限を設けていません。

> **Note**
> Laravel は初めてですか？[Laravel Bootcamp](https://bootcamp.laravel.com) でフレームワークを実際に使いながら最初の Laravel アプリケーションを作成するプロセスを学びましょう

<a name="the-root-directory"></a>
## プロジェクトルートディレクトリ

<a name="the-root-app-directory"></a>
#### app ディレクトリ

`app` ディレクトリには、アプリケーションのコアコードが含まれています。このディレクトリは後ほど詳しく説明しますが、アプリケーションのほぼすべてのクラスがこのディレクトリに存在します。

<a name="the-bootstrap-directory"></a>
#### bootstrap ディレクトリ

`bootstrap` ディレクトリには、フレームワークをブートストラップする `app.php` ファイルが含まれています。このディレクトリには、ルートキャッシュやサービスキャッシュファイルなどのパフォーマンス最適化のためのフレームワーク生成ファイルを格納する `cache` ディレクトリも含まれています。通常、このディレクトリ内のファイルを変更する必要はありません。

<a name="the-config-directory"></a>
#### config ディレクトリ

「config」ディレクトリには、その名前が示すようにアプリケーションの構成ファイルがすべて含まれています。これらのファイルをすべて読み込んで、利用可能なオプションを把握することが良いでしょう。

<a name="the-database-directory"></a>
#### database ディレクトリ

`database` データベースのマイグレーション、モデルファクトリ、およびシードが含まれています。必要に応じて、このディレクトリに `SQLite` データベースを格納することもできます。

<a name="the-public-directory"></a>
#### public ディレクトリ

`public` ディレクトリには `index.php` ファイルが含まれており、アプリケーションに入るすべてのリクエストのエントリーポイントであり、オートロードを設定します。このディレクトリには、画像、JavaScript、CSSなどのアセットも格納されています。

<a name="the-resources-directory"></a>
#### resources ディレクトリ

`resources` ディレクトリには、[views](/laravel10_ja/views) と、CSS や JavaScript などの未コンパイルのアセットが含まれています。

<a name="the-routes-directory"></a>
#### routes ディレクトリ

`routes` ディレクトリには、アプリケーションのすべてのルート定義が含まれています。 デフォルトで複数のルートファイルが含まれています。（`web.php`、`api.php`、`console.php`、`channels.php`）

`web.php` ファイルには、`RouteServiceProvider` が `web` ミドルウェアグループに配置するルートが含まれており、セッション状態、CSRF 保護、および Cookie 暗号化が提供されます。 アプリケーションがステートレスな RESTful API を提供しない場合、すべてのルートはおそらく `web.php` ファイルに定義されるでしょう。

`api.php` ファイルには、`RouteServiceProvider` が `api` ミドルウェアグループに配置するルートが含まれています。これらのルートはステートレスを意図しており、これらのルートを通じてアプリケーションに入るリクエストは、[トークン経由](/laravel10_ja/sanctum) で認証され、セッション状態にアクセスできません。

`console.php` ファイルは、クロージャベースのコンソールコマンドをすべて定義できます。各クロージャはコマンドインスタンスにバインドされており、各コマンドの IO メソッドとのやり取りを簡単にするアプローチが提供されます。このファイルは HTTP ルートを定義していませんが、アプリケーションへのコンソールベースのエントリーポイント（ルート）を定義しています。

`channels.php` ファイルは、アプリケーションがサポートするすべての [イベントブロードキャスト](/laravel10_ja/broadcasting) チャネルを登録できる場所です。

<a name="the-storage-directory"></a>
#### storage ディレクトリ

`storage` ディレクトリには、ログ、コンパイル済みの `Blade` テンプレート、ファイルベースのセッション、ファイルキャッシュなど、フレームワークが生成するその他のファイルが格納されています。このディレクトリは、`app`、`framework`、および `logs` ディレクトリに分割されています。`app` ディレクトリは、アプリケーションが生成するファイルを格納するために使用できます。`framework` ディレクトリは、フレームワークが生成するファイルとキャッシュを格納するために使用されます。最後に、`logs` ディレクトリにはアプリケーションのログファイルが格納されています。

`storage/app/public` ディレクトリは、プロフィールアバターなどのユーザーが生成したファイルを格納するために使用できます。これらのファイルは一般に公開されるべきです。`public/storage` にこのディレクトリを指すシンボリックリンクを作成する必要があります。`php artisan storage:link` Artisanコマンドを使ってリンクを作成できます。

<a name="the-tests-directory"></a>
#### tests ディレクトリ

`tests` ディレクトリには、自動化されたテストが含まれています。（例 [PHPUnit](https://phpunit.de/) ）単体テストと機能テストは、すぐに使用できます。 各テストクラスには、`Test` という単語を接尾辞として付ける必要があります。 `phpunit` または `php vendor/bin/phpunit` コマンドを使用してテストを実行できます。 また、テスト結果をより詳細で美しい表示で見たい場合は、`php artisan test` Artisan コマンドを使用してテストを実行できます。

<a name="the-vendor-directory"></a>
#### vendor ディレクトリ

`vendor` ディレクトリには、[Composer](https://getcomposer.org) の依存関係が格納されています。

<a name="the-app-directory"></a>
## app ディレクトリ

アプリケーションの大部分は `app` ディレクトリに収められています。デフォルトでは、このディレクトリは `App` の名前空間にあり、Composer によって [PSR-4 オートロード標準](https://www.php-fig.org/psr/psr-4/) を使用してオートロードされます。

`app` ディレクトリには、`Console`、`Http`、`Providers` などのさまざまな追加ディレクトリが含まれています。`Console` および `Http` ディレクトリは、アプリケーションのコアへの API を提供するものと考えてください。HTTP プロトコルと CLI は、アプリケーションとやり取りするためのメカニズムですが、アプリケーションのロジックを実際には含まれていません。言い換えれば、それらはアプリケーションにコマンドを発行する2つの方法です。`Console` ディレクトリにはすべての Artisan コマンドが含まれており、`Http` ディレクトリにはコントローラー、ミドルウェア、およびリクエストが含まれています。

`make` Artisan コマンドを使用してクラスを生成すると、`app` ディレクトリ内にさまざまな他のディレクトリが生成されます。たとえば、`app/Jobs` ディレクトリは、ジョブクラスを生成するために `make:job` Artisan コマンドを実行するまで存在しません。

> **Note**
> `app` ディレクトリ内の多くのクラスは、Artisan を介したコマンドで生成できます。利用可能なコマンドを確認するには、ターミナルで `php artisan list make` コマンドを実行してください。。

<a name="the-broadcasting-directory"></a>
#### Broadcasting ディレクトリ

`Broadcasting` ディレクトリには、アプリケーションのブロードキャスト チャネル クラスがすべて含まれています。これらのクラスは、`make:channel` コマンドを使用して生成されます。このディレクトリはデフォルトでは存在しませんが、最初のチャネルを作成するときに作成されます。チャネルの詳細については、[イベントブロードキャスト](/laravel10_ja/broadcasting) のドキュメントをご覧ください。

<a name="the-console-directory"></a>
#### Console ディレクトリ

`Console` ディレクトリには、アプリケーション用のすべてのカスタム Artisan コマンドが含まれています。 これらのコマンドは、`make:command` コマンドを使用して生成できます。 また、このディレクトリにはコンソールカーネルも格納されており、カスタム Artisan コマンドが登録されたり、[スケジュールされたタスク](/laravel10_ja/scheduling) が定義されたりします。

<a name="the-events-directory"></a>
#### Events ディレクトリ

このディレクトリはデフォルトでは存在しませんが、`event:generate` および `make:event` Artisan コマンドによって作成されます。 `Events` ディレクトリには、[イベントクラス](/laravel10_ja/events) が格納されています。 イベントは、特定のアクションが発生したことをアプリケーションの他の部分に通知するために使用でき、柔軟性とデカップリングを提供します。

<a name="the-exceptions-directory"></a>
#### Exceptions  ディレクトリ

`Exceptions` ディレクトリには、アプリケーションがスローする任意の例外を配置するのに適しています。例外のログやレンダリング方法をカスタマイズしたい場合は、このディレクトリの `Handler` クラスを変更してください。

<a name="the-http-directory"></a>
#### Http ディレクトリ

`Http` ディレクトリには、コントローラ、ミドルウェア、およびフォーム リクエストが含まれています。 アプリケーションに入るリクエストを処理するほとんどすべてのロジックは、このディレクトリに配置されます。

<a name="the-jobs-directory"></a>
#### Jobs ディレクトリ

このディレクトリはデフォルトでは存在しませんが、`make:job` Artisan コマンドを実行すると作成されます。`Jobs` ディレクトリには、アプリケーションの [キュー可能なジョブ](/laravel10_ja/queues) が格納されています。ジョブは、アプリケーションによってキューに入れられるか、現在のリクエストライフサイクル内で同期的に実行されます。現在のリクエスト中に同期的に実行されるジョブは、[コマンドパターン](https://en.wikipedia.org/wiki/Command_pattern) の実装であるため、「コマンド」と呼ばれることがあります。

<a name="the-listeners-directory"></a>
#### Listeners ディレクトリ

このディレクトリはデフォルトでは存在しませんが、`event:generate` または `make:listener` Artisan コマンドを実行すると作成されます。`Listeners` ディレクトリには、[イベント](/laravel10_ja/events) を処理するクラスが含まれています。イベントリスナーはイベントインスタンスを受け取り、イベントが発生したことに対するロジックを実行します。例えば、`UserRegistered` イベントは `SendWelcomeEmail` リスナーによって処理されることがあります。

<a name="the-mail-directory"></a>
#### Mail ディレクトリ

このディレクトリはデフォルトでは存在しませんが、`make:mail` Artisan コマンドを実行すると作成されます。`Mail` ディレクトリには、アプリケーションから送信される [メールを表すクラス](/laravel10_ja/mail) がすべて含まれています。メールオブジェクトを使用すると、メールの作成に関するすべてのロジックを単一のシンプルなクラスにカプセル化でき、`Mail::send` メソッドを使用して送信できます。

<a name="the-models-directory"></a>
#### Models ディレクトリ

`Models` ディレクトリには、すべての [Eloquent モデル クラス](/laravel10_ja/eloquent) が含まれています。 Laravel に含まれる Eloquent ORM は、データベースを操作するための美しくシンプルな ActiveRecord の実装を提供します。各データベーステーブルには、そのテーブルと対話するために使用される対応する「モデル」があります。モデルを使用すると、テーブル内のデータをクエリしたり、テーブルに新しいレコードを挿入したりできます。

<a name="the-notifications-directory"></a>
#### Notifications ディレクトリ

このディレクトリはデフォルトでは存在しませんが、`make:notification` Artisan コマンドを実行すると作成されます。`Notifications` ディレクトリには、アプリケーション内で発生するイベントに関するシンプルな通知など、アプリケーションによって送信されるすべての「トランザクション」[通知](/laravel10_ja/notifications) が含まれます。 Laravel の通知機能は、電子メール、Slack、SMS、またはデータベースに保存されたさまざまなドライバーを介して通知を送信することを抽象化します。

<a name="the-policies-directory"></a>
#### Policies ディレクトリ

このディレクトリはデフォルトでは存在しませんが、`make:policy` Artisan コマンドを実行すると作成されます。`Policies` ディレクトリには、アプリケーションの [認可ポリシークラス](/laravel10_ja/authorization) が含まれています。 ポリシーは、ユーザーがリソースに対して特定のアクションを実行できるかどうかを判断するために使用されます。

<a name="the-providers-directory"></a>
#### Providers ディレクトリ

`Providers` ディレクトリには、アプリケーションのすべての [サービスプロバイダ](/laravel10_ja/providers) が含まれています。 サービスプロバイダは、サービスコンテナでサービスの結合、イベントの登録、その他のタスクを実行したりなど、アプリケーションへのリクエストに備えた準備することで、アプリケーションの初期化を行います。

新規の Laravel アプリケーションでは、このディレクトリにはすでにいくつかのプロバイダーが含まれています。必要に応じて、このディレクトリに独自のプロバイダーを追加することができます。

<a name="the-rules-directory"></a>
#### Rules ディレクトリ

このディレクトリはデフォルトでは存在しませんが、`make:rule` Artisan コマンドを実行すると作成されます。`Rules` ディレクトリには、アプリケーションのカスタム検証ルールオブジェクトが含まれています。 ルールは、複雑な検証ロジックを単純なオブジェクトにカプセル化するために使用されます。 詳細については、[検証ドキュメント](/laravel10_ja/validation)をご覧ください。
