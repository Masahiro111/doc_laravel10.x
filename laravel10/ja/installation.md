# インストール

- [Laravel の紹介](#meet-laravel)
    - [なぜ Laravel なのか？](#why-laravel)
- [初めての Laravel プロジェクト](#your-first-laravel-project)
- [Laravel & Docker](#laravel-and-docker)
    - [macOS で始める](#getting-started-on-macos)
    - [Windowsで始める](#getting-started-on-windows)
    - [Linux で始める](#getting-started-on-linux)
    - [Sail サービスを選択する](#choosing-your-sail-services)
- [初期設定](#initial-configuration)
    - [環境ベースの初期設定](#environment-based-configuration)
    - [データベースとマイグレーション](#databases-and-migrations)
- [次のステップへ](#next-steps)
    - [Laravel フルスタックフレームワーク](#laravel-the-fullstack-framework)
    - [Laravel API バックエンド](#laravel-the-api-backend)

<a name="meet-laravel"></a>
## Laravel の紹介

Laravel は、表現力豊かでエレガントなシンタックスを持つ Web アプリケーションフレームワークです。Web フレームワークは、アプリケーションを作成するための構造と出発点を提供し、私たちが詳細に汗を流す間、あなたは素晴らしいものを作ることに集中することができます

Laravel は、徹底した依存性注入、表現力豊かなデータベース抽象化レイヤー、キューとスケジュールされたジョブ、ユニットテストと統合テストなどの強力な機能を提供しながら、素晴らしい開発者体験を提供しようと努力しています。

PHP の Web フレームワークが初めての方でも、長年の経験をお持ちの方でも、Laravel はあなたとともに成長できるフレームワークです。私たちは、あなたがウェブデベロッパーとしての第一歩を踏み出すお手伝いをし、また、あなたの専門知識を次のレベルに引き上げるための後押しをします。あなたがどんなアプリを作るのか待ちきれません。

> **Note**
> Laravel は初めてですか？[Laravel Bootcamp](https://bootcamp.laravel.com)では、フレームワークのハンズオンツアーを行いながら、初めての Laravel アプリケーションを構築する方法をご紹介します。

<a name="why-laravel"></a>
### なぜ Laravel なのか？

Web アプリケーションを構築する際には、様々なツールやフレームワークがあります。しかし、Laravel はモダンなフルスタック Web アプリケーションを構築するための最良の選択であると信じています。

#### プログレッシブフレームワーク

私たちは、Laravel を「プログレッシブ」フレームワークと呼びたいと考えています。これは、Laravel があなたと共に成長することを意味します。もしあなたがウェブ開発の第一歩を踏み出したばかりなら、Laravel の膨大なドキュメント、ガイド、[ビデオチュートリアル](https://laracasts.com) のライブラリは、あなたが圧倒されることなくノウハウを学ぶのを助けてくれるでしょう。

上級開発者であれば、Laravel は[依存性注入](/laravel10_ja/container)、[ユニットテスト](/laravel10_ja/testing)、[キュー](/laravel10_ja/queues)、[リアルタイムイベント](/laravel10_ja/broadcasting) など、強固なツールを提供しています。Laravelは、プロフェッショナルな Web アプリケーションを構築するために細かく調整されており、企業の作業負荷を処理する準備ができています。

#### スケーラブルなフレームワーク

Laravel は信じられないほどスケーラブルです。PHP のスケーリングに適した性質と、Redis のような高速な分散キャッシュシステムをビルトインでサポートしています。Laravel を使って簡単に水平スケーリングを行えるでしょう。実際、Laravel のアプリケーションは、1ヶ月あたり数億のリクエストを処理するように簡単にスケーリングされています。

大規模なスケーリングを必要としていますか？[Laravel Vapor](https://vapor.laravel.com) のようなプラットフォームでは、AWS の最新のサーバーレステクノロジー上で Laravel アプリケーションをほぼ無制限のスケールで実行することができます。

#### コミュニティの枠組み

Laravel は、PHP のエコシステムにおける最高のパッケージを組み合わせ、最も堅牢で開発者に優しいフレームワークを提供しています。さらに、世界中の何千人もの才能ある開発者が [フレームワークに貢献しています](https://github.com/laravel/framework) 。もしかしたら、あなたもLaravelのコントリビューターになれるかもしれませんよ。

<a name="your-first-laravel-project"></a>
## 初めてのLaravelプロジェクト

最初の Laravel プロジェクトを作成する前に、ローカルマシンにPHPと[Composer](https://getcomposer.org)がインストールされていることを確認する必要があります。macOS で開発している場合、PHP と Composer は[Homebrew](https://brew.sh/)を介してインストールすることができます。また、[Node と NPM のインストール](https://nodejs.org)もお勧めします。

PHP と Composer のインストールが完了したら、Composer の`create-project`コマンドで新しい Laravel プロジェクトを作成することができます

```nothing
composer create-project laravel/laravel example-app
```

または、Composer 経由で Laravel インストーラーをグローバルにインストールして、新しい Laravel プロジェクトを作成することもできます：

```nothing
composer global require laravel/installer

laravel new example-app
```

プロジェクト作成後、Laravel の Artisan CLI `serve` コマンドを使用して、Laravel のローカル開発サーバーを起動します。

```nothing
cd example-app

php artisan serve
```

Artisan 開発サーバーを起動すると、アプリケーションは Web ブラウザで `http://localhost:8000` にアクセスできるようになります。次に、[Laravelエコシステムへの次のステップを開始する](#next-steps) 準備が整いました。もちろん、[データベースの設定](#databases-and-migrations) も必要でしょう。

> **Note**  
> Laravel アプリケーションの開発を先取りしたい場合は、[スターターキット](docs/laravel10_ja/starter-kits)を使用することができます。Laravel スターターキットは、あなたの新しい Laravel アプリケーションのために、バックエンドとフロントエンドの認証スカフォールディングを提供します。

<a name="laravel-and-docker"></a>
## Laravel & Docker

あなたの好みのオペレーティングシステムに関係なく、Laravelをできるだけ簡単に始められるようにしたいと考えています。そのために、ローカルマシンでLaravelプロジェクトを開発し実行するための様々なオプションを提供しています。これらのオプションについては後で説明しますが、Laravelは、[Docker](https://www.docker.com)、[Sail](/laravel10_ja/sail)を提供したLaravelプロジェクトを実行するための組み込みソリューションを提供します。

Dockerは、アプリケーションやサービスを小さく軽量な「コンテナ」で実行するためのツールで、ローカルマシンのインストール済みソフトウェアや設定に干渉しません。つまり、ローカルマシンにWebサーバーやデータベースなどの複雑な開発ツールを設定したり、セットアップしたりする心配がありません。[Docker Desktop](https://www.docker.com/products/docker-desktop) を使えば、すぐに使い始めることができます。

Laravel Sail は、Laravel のデフォルトの Docker 設定と連携するための軽量なコマンドラインインターフェイスです。Sail は、PHP、MySQL、Redis でLaravel アプリケーションを構築するための素晴らしい出発点となり、事前の Docker の経験は必須ではありません。

> **Note**  
> すでにDockerのエキスパートですか？ご安心ください！Laravelに含まれる`docker-compose.yml`ファイルを使用して、Sailですべてをカスタマイズすることができます。

<a name="getting-started-on-macos"></a>
### macOS で始める

If you're developing on a Mac and [Docker Desktop](https://www.docker.com/products/docker-desktop) is already installed, you can use a simple terminal command to create a new Laravel project. For example, to create a new Laravel application in a directory named "example-app", you may run the following command in your terminal:
Mac で開発していて、[Docker Desktop](https://www.docker.com/products/docker-desktop) が既にインストールされていれば、簡単なターミナルコマンドで新しいLaravelプロジェクトを作成することができます。例えば、「example-app」という名前のディレクトリに新しい Laravel アプリケーションを作成するには、ターミナルで以下のコマンドを実行します

```shell
curl -s "https://laravel.build/example-app" | bash
```

もちろん、このURLの「example-app」を好きなものに変更することができます。ただし、アプリケーション名には英数字、ダッシュ、アンダースコアしか含まれていないことを確認してください。Laravelアプリケーションのディレクトリは、コマンドを実行したディレクトリの中に作成されます。

Sailのインストールは、Sailのアプリケーションコンテナがローカルマシンに構築される間、数分かかる場合があります。

After the project has been created, you can navigate to the application directory and start Laravel Sail. Laravel Sail provides a simple command-line interface for interacting with Laravel's default Docker configuration:
プロジェクトが作成されたら、アプリケーションのディレクトリに移動してLaravel Sailを起動します。Laravel Sailは、LaravelのデフォルトのDocker構成と連携するためのシンプルなコマンドラインインターフェイスを提供します：

```shell
cd example-app

./vendor/bin/sail up
```

Once the application's Docker containers have been started, you can access the application in your web browser at: http://localhost.
アプリケーションのDockerコンテナが起動したら、`http://localhost` に Web ブラウザでアプリケーションにアクセスできます。

> **Note**  
> To continue learning more about Laravel Sail, review its [complete documentation](/laravel10_ja/sail).
Laravel Sailの詳細については、その[完全なドキュメント](/laravel10_ja/sail)を参照してください。

<a name="getting-started-on-windows"></a>
### Windowsで始める

Before we create a new Laravel application on your Windows machine, make sure to install [Docker Desktop](https://www.docker.com/products/docker-desktop). Next, you should ensure that Windows Subsystem for Linux 2 (WSL2) is installed and enabled. WSL allows you to run Linux binary executables natively on Windows 10. Information on how to install and enable WSL2 can be found within Microsoft's [developer environment documentation](https://docs.microsoft.com/en-us/windows/wsl/install-win10).
Windowsマシンで新しいLaravelアプリケーションを作成する前に、[Docker Desktop](https://www.docker.com/products/docker-desktop)をインストールすることを確認してください。次に、Windows Subsystem for Linux 2 (WSL2)がインストールされ、有効になっていることを確認する必要があります。WSLを使用すると、Windows 10上でLinuxのバイナリ実行ファイルをネイティブに実行することができます。WSL2のインストールと有効化に関する情報は、Microsoftの[開発者環境ドキュメント](https://docs.microsoft.com/en-us/windows/wsl/install-win10)内に記載されています。

> **Note**  
> After installing and enabling WSL2, you should ensure that Docker Desktop is [configured to use the WSL2 backend](https://docs.docker.com/docker-for-windows/wsl/).
 WSL2をインストールして有効にした後、Docker Desktopが[WSL2バックエンドを使用するように設定されている](https://docs.docker.com/docker-for-windows/wsl/)ことを確認する必要があります。

Next, you are ready to create your first Laravel project. Launch [Windows Terminal](https://www.microsoft.com/en-us/p/windows-terminal/9n0dx20hk701?rtc=1&activetab=pivot:overviewtab) and begin a new terminal session for your WSL2 Linux operating system. Next, you can use a simple terminal command to create a new Laravel project. For example, to create a new Laravel application in a directory named "example-app", you may run the following command in your terminal:
次に、最初のLaravelプロジェクトを作成する準備が整いました。[Windows Terminal](https://www.microsoft.com/en-us/p/windows-terminal/9n0dx20hk701?rtc=1&activetab=pivot:overviewtab) を起動し、WSL2 Linuxオペレーティングシステム用の新しいターミナルセッションを開始します。次に、単純なターミナルコマンドを使用して、新しいLaravelプロジェクトを作成することができます。例えば、「example-app」という名前のディレクトリに新しいLaravelアプリケーションを作成するには、ターミナルで次のコマンドを実行します：

```shell
curl -s https://laravel.build/example-app | bash
```

Of course, you can change "example-app" in this URL to anything you like - just make sure the application name only contains alpha-numeric characters, dashes, and underscores. The Laravel application's directory will be created within the directory you execute the command from.
もちろん、このURLの「example-app」を好きなものに変更することができます。ただし、アプリケーション名には英数字、ダッシュ、アンダースコアしか含まれていないことを確認してください。Laravelアプリケーションのディレクトリは、コマンドを実行したディレクトリの中に作成されます。

Sail installation may take several minutes while Sail's application containers are built on your local machine.
Sailのインストールは、Sailのアプリケーションコンテナがローカルマシンに構築される間、数分かかる場合があります。

After the project has been created, you can navigate to the application directory and start Laravel Sail. Laravel Sail provides a simple command-line interface for interacting with Laravel's default Docker configuration:
プロジェクトが作成されたら、アプリケーションのディレクトリに移動してLaravel Sailを起動します。Laravel Sailは、LaravelのデフォルトのDocker構成と対話するためのシンプルなコマンドラインインターフェイスを提供します：

```shell
cd example-app

./vendor/bin/sail up
```

Once the application's Docker containers have been started, you can access the application in your web browser at: http://localhost.
アプリケーションのDockerコンテナが起動したら、ウェブブラウザでアプリケーションにアクセスできます: http://localhost.

> **Note**  
> Laravel Sailについてさらに学ぶには、その[完全なドキュメント](/laravel10_ja/sail)を確認してください。

#### WSL2内で開発する

Of course, you will need to be able to modify the Laravel application files that were created within your WSL2 installation. To accomplish this, we recommend using Microsoft's [Visual Studio Code](https://code.visualstudio.com) editor and their first-party extension for [Remote Development](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack).
もちろん、WSL2インストール内に作成されたLaravelアプリケーションファイルを変更できる必要があります。これを実現するには、Microsoftの[Visual Studio Code](https://code.visualstudio.com)エディターと、[リモート開発](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack)のファーストパーティ拡張機能を使うことをお勧めします。

Once these tools are installed, you may open any Laravel project by executing the `code .` command from your application's root directory using Windows Terminal.
これらのツールをインストールしたら、Windowsターミナルを使用して、アプリケーションのルートディレクトリから`code .`コマンドを実行することで、任意のLaravelプロジェクトを開くことができます。

<a name="getting-started-on-linux"></a>
### Linuxで始める

If you're developing on Linux and [Docker Compose](https://docs.docker.com/compose/install/) is already installed, you can use a simple terminal command to create a new Laravel project. For example, to create a new Laravel application in a directory named "example-app", you may run the following command in your terminal:
Linuxで開発していて、[Docker Compose](https://docs.docker.com/compose/install/)が既にインストールされている場合、簡単なターミナルコマンドで新しいLaravelプロジェクトを作成することができます。例えば、"example-app "というディレクトリに新しいLaravelアプリケーションを作成するには、ターミナルで次のコマンドを実行します：

```shell
curl -s https://laravel.build/example-app | bash
```

Of course, you can change "example-app" in this URL to anything you like - just make sure the application name only contains alpha-numeric characters, dashes, and underscores. The Laravel application's directory will be created within the directory you execute the command from.
もちろん、このURLの「example-app」を好きなものに変更することができます。ただし、アプリケーション名には英数字、ダッシュ、アンダースコアしか含まれていないことを確認してください。Laravelアプリケーションのディレクトリは、コマンドを実行したディレクトリの中に作成されます。

Sail installation may take several minutes while Sail's application containers are built on your local machine.
Sailのインストールは、Sailのアプリケーションコンテナがローカルマシンで構築される間、数分かかる場合があります。

After the project has been created, you can navigate to the application directory and start Laravel Sail. Laravel Sail provides a simple command-line interface for interacting with Laravel's default Docker configuration:
プロジェクトが作成されたら、アプリケーションのディレクトリに移動し、Laravel Sailを起動することができます。Laravel Sailは、LaravelのデフォルトのDocker構成と対話するためのシンプルなコマンドラインインターフェイスを提供します：

```shell
cd example-app

./vendor/bin/sail up
```

Once the application's Docker containers have been started, you can access the application in your web browser at: http://localhost.
アプリケーションのDockerコンテナが起動したら、ウェブブラウザでアプリケーションにアクセスできます: http://localhost.

> **Note**  
> To continue learning more about Laravel Sail, review its [complete documentation](/laravel10_ja/sail).
Laravel Sailについてさらに詳しく学ぶには、その[完全なドキュメント](/laravel10_ja/sail)を確認してください。

<a name="choosing-your-sail-services"></a>
### Choosing Your Sail Services
Sailのサービスを選択する

When creating a new Laravel application via Sail, you may use the `with` query string variable to choose which services should be configured in your new application's `docker-compose.yml` file. Available services include `mysql`, `pgsql`, `mariadb`, `redis`, `memcached`, `meilisearch`, `minio`, `selenium`, and `mailpit`:
Sail経由で新しいLaravelアプリケーションを作成する場合、`with`クエリ文字列変数を使用して、新しいアプリケーションの `docker-compose.yml` ファイルに設定するサービスを選択することができます。利用可能なサービスは `mysql`、`pgsql`、`mariadb`、`redis`、`memcached`、`meilisearch`、`minio`、`selenium`、`mailpit` です：

```shell
curl -s "https://laravel.build/example-app?with=mysql,redis" | bash
```

If you do not specify which services you would like configured, a default stack of `mysql`, `redis`, `meilisearch`, `mailpit`, and `selenium` will be configured.
どのサービスを設定したいかを指定しない場合、`mysql`, `redis`, `meilisearch`, `mailpit`, `selenium` のデフォルトスタックが設定されます。

You may instruct Sail to install a default [Devcontainer](/laravel10_ja/sail#using-devcontainers) by adding the `devcontainer` parameter to the URL:
URLに `devcontainer` パラメータを追加することで、デフォルトの [Devcontainer](/laravel10_ja/sail#using-devcontainers) をインストールするように Sail に指示することができます：
```shell
curl -s "https://laravel.build/example-app?with=mysql,redis&devcontainer" | bash
```

<a name="initial-configuration"></a>
## 初期設定

All of the configuration files for the Laravel framework are stored in the `config` directory. Each option is documented, so feel free to look through the files and get familiar with the options available to you.
Laravelフレームワークのすべての設定ファイルは、`config`ディレクトリに格納されています。各オプションはドキュメント化されているので、自由にファイルに目を通して、利用可能なオプションに慣れてください。

Laravel needs almost no additional configuration out of the box. You are free to get started developing! However, you may wish to review the `config/app.php` file and its documentation. It contains several options such as `timezone` and `locale` that you may wish to change according to your application.
Laravelは、箱から出してもほとんど追加設定は必要ありません。自由に開発を始めることができます！しかし、`config/app.php`ファイルとそのドキュメントを確認しておくとよいでしょう。このファイルには、`timezone`や`locale`など、あなたのアプリケーションに応じて変更したいオプションが含まれています。

<a name="environment-based-configuration"></a>
### 環境ベースのコンフィギュレーション

Laravel の設定オプションの多くは、アプリケーションがローカルマシン上で動作しているか、本番のWebサーバー上で動作しているかによって異なる場合があるので、多くの重要な設定値は、アプリケーションのルートに存在する `.env` ファイルを使用して定義します。

アプリケーションを使用する各開発者やサーバーは異なる環境構成を必要とする可能性があるため、`.env` ファイルはアプリケーションのソース管理にコミットしないでください。 さらに、侵入者がソース管理リポジトリへのアクセスを取得した場合、機密情報が漏洩してしまうためセキュリティリスクにもなります。

> **Note**  
> For more information about the `.env` file and environment based configuration, check out the full [configuration documentation](/laravel10_ja/configuration#environment-configuration).
.env`ファイルや環境ベースの設定についての詳細は、完全な [設定ドキュメント](/laravel10_ja/configuration#environment-configuration) を参照してください。

<a name="databases-and-migrations"></a>
### データベースとマイグレーション

Now that you have created your Laravel application, you probably want to store some data in a database. By default, your application's `.env` configuration file specifies that Laravel will be interacting with a MySQL database and will access the database at `127.0.0.1`. If you are developing on macOS and need to install MySQL, Postgres, or Redis locally, you may find it convenient to utilize [DBngin](https://dbngin.com/).
Laravelアプリケーションを作成した今、おそらくいくつかのデータをデータベースに保存したいと思うことでしょう。デフォルトでは、アプリケーションの `.env` 設定ファイルは、LaravelがMySQLデータベースと対話し、`127.0.0.1`でデータベースにアクセスすることを指定します。macOSで開発しており、MySQL、Postgres、Redisをローカルにインストールする必要がある場合、[DBngin](https://dbngin.com/) を利用すると便利でしょう。


If you do not want to install MySQL or Postgres on your local machine, you can always use a [SQLite](https://www.sqlite.org/index.html) database. SQLite is a small, fast, self-contained database engine. To get started, create a SQLite database by creating an empty SQLite file. Typically, this file will exist within the `database` directory of your Laravel application:
MySQLやPostgresをローカルマシンにインストールしたくない場合は、いつでも[SQLite](https://www.sqlite.org/index.html)データベースを使用することができます。SQLite は小さく、速く、自己完結型のデータベースエンジンです。始めるには、空の SQLite ファイルを作成して、SQLite データベースを作成します。通常、このファイルはLaravelアプリケーションの`database`ディレクトリ内に存在することになります：

```shell
touch database/database.sqlite
```

Next, update your `.env` configuration file to use Laravel's `sqlite` database driver. You may remove the other database configuration options:

```ini
DB_CONNECTION=sqlite # [tl! add]
DB_CONNECTION=mysql # [tl! remove]
DB_HOST=127.0.0.1 # [tl! remove]
DB_PORT=3306 # [tl! remove]
DB_DATABASE=laravel # [tl! remove]
DB_USERNAME=root # [tl! remove]
DB_PASSWORD= # [tl! remove]
```

Once you have configured your SQLite database, you may run your application's [database migrations](/laravel10_ja/migrations), which will create your application's database tables:
次に、Laravelの`sqlite`データベースドライバーを使用するために、`.env`設定ファイルを更新します。他のデータベース設定オプションは削除してもかまいません：

```shell
php artisan migrate
```

<a name="next-steps"></a>
## 次のステップ

Now that you have created your Laravel project, you may be wondering what to learn next. First, we strongly recommend becoming familiar with how Laravel works by reading the following documentation:
さて、Laravelプロジェクトを作成したところで、次に何を学べばいいのか悩むかもしれません。まず、以下のドキュメントを読んで、Laravelの仕組みに慣れることを強くお勧めします：

<div class="content-list" markdown="1">

- [Request Lifecycle](/laravel10_ja/lifecycle)
- [Configuration](/laravel10_ja/configuration)
- [Directory Structure](/laravel10_ja/structure)
- [Frontend](/laravel10_ja/frontend)
- [Service Container](/laravel10_ja/container)
- [Facades](/laravel10_ja/facades)

</div>

How you want to use Laravel will also dictate the next steps on your journey. There are a variety of ways to use Laravel, and we'll explore two primary use cases for the framework below.
Laravelをどのように使いたいかで、次のステップも決まります。Laravelには様々な使い方がありますが、ここではフレームワークの2つの主な使用例について説明します。

> **Note**
> New to Laravel? Check out the [Laravel Bootcamp](https://bootcamp.laravel.com) for a hands-on tour of the framework while we walk you through building your first Laravel application.
Laravelは初めてですか？[Laravel Bootcamp](https://bootcamp.laravel.com) をチェックして、最初のLaravelアプリケーションの構築を通して、フレームワークのハンズオンツアーを行っています。

<a name="laravel-the-fullstack-framework"></a>
### Laravel フルスタックフレームワーク

Laravel may serve as a full stack framework. By "full stack" framework we mean that you are going to use Laravel to route requests to your application and render your frontend via [Blade templates](/laravel10_ja/blade) or a single-page application hybrid technology like [Inertia](https://inertiajs.com). This is the most common way to use the Laravel framework, and, in our opinion, the most productive way to use Laravel.
Laravelはフルスタックフレームワークとして使用することができます。フルスタックフレームワークとは、Laravelを使用してアプリケーションへのリクエストをルーティングし、[Blade テンプレート](/laravel10_ja/blade) や [Inertia](https://inertiajs.com) のようなシングルページアプリケーションハイブリッドテクノロジーを使用してフロントエンドをレンダリングする方法を指します。これは、Laravelフレームワークを使用する最も一般的な方法であり、私たちの意見では、Laravel を使用する上で最も生産的な方法です。

If this is how you plan to use Laravel, you may want to check out our documentation on [frontend development](/laravel10_ja/frontend), [routing](/laravel10_ja/routing), [views](/laravel10_ja/views), or the [Eloquent ORM](/laravel10_ja/eloquent). In addition, you might be interested in learning about community packages like [Livewire](https://laravel-livewire.com) and [Inertia](https://inertiajs.com). These packages allow you to use Laravel as a full-stack framework while enjoying many of the UI benefits provided by single-page JavaScript applications.
Laravel をこのように使用する場合、[フロントエンド開発](/laravel10_ja//frontend)、[ルーティング](/laravel10_ja//routing)、[ビュー](/laravel10_ja//views)、[エロージェント ORM](/laravel10_ja//eloquent) などに関するドキュメントを参照することをお勧めします。また、[Livewire](https://laravel-livewire.com) や [Inertia](https://inertiajs.com) のようなコミュニティパッケージの学習にも興味を持つかもしれません。これらのパッケージを使用することで、シングルページ JavaScript アプリケーションが提供する多くの UI の利点を享受しながら、Laravel をフルスタックフレームワークとして使用することができます。

If you are using Laravel as a full stack framework, we also strongly encourage you to learn how to compile your application's CSS and JavaScript using [Vite](/laravel10_ja/vite).
Laravelをフルスタックフレームワークとして使用する場合、[Vite](/laravel10_ja/vite)を使用してアプリケーションのCSSとJavaScriptをコンパイルする方法を学ぶことも強くお勧めします。

> **Note**  
> If you want to get a head start building your application, check out one of our official [application starter kits](/laravel10_ja/starter-kits).
アプリケーションの構築を先取りしたい場合は、公式の[アプリケーションスターターキット](/laravel10_ja/starter-kits)をチェックしてみてください。

<a name="laravel-the-api-backend"></a>
### Laravel API バックエンド

Laravel may also serve as an API backend to a JavaScript single-page application or mobile application. For example, you might use Laravel as an API backend for your [Next.js](https://nextjs.org) application. In this context, you may use Laravel to provide [authentication](/laravel10_ja/sanctum) and data storage / retrieval for your application, while also taking advantage of Laravel's powerful services such as queues, emails, notifications, and more.
Laravelは、JavaScriptのシングルページアプリケーションやモバイルアプリケーションのAPIバックエンドとして機能することもあります。例えば、[Next.js](https://nextjs.org)アプリケーションのAPIバックエンドとしてLaravelを使用する場合があります。この文脈では、Laravelを使用して、アプリケーションの[認証](/laravel10_ja/sanctum)とデータの保存/検索を提供し、同時にキュー、メール、通知など、Laravelの強力なサービスを利用することもできます。

If this is how you plan to use Laravel, you may want to check out our documentation on [routing](/laravel10_ja/routing), [Laravel Sanctum](/laravel10_ja/sanctum), and the [Eloquent ORM](/laravel10_ja/eloquent).
このようにLaravelを使用する場合、[routing](/laravel10_ja/routing)、[Laravel Sanctum](/laravel10_ja/sanctum), [Eloquent ORM](/laravel10_ja/eloquent) に関するドキュメントをチェックするとよいでしょう。

> **Note**  
> Laravel のバックエンドと Next.js のフロントエンドのスキャフォールディングを有利に開始する必要がありますか？Laravel Breeze は、[API スタック](/laravel10_ja/starter-kits#breeze-and-next) と [Next.js フロントエンド実装](https://github.com/laravel/breeze-next) を提供しているので、数分で始めることができます。