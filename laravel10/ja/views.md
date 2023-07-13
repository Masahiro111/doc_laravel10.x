# ビュー

- [はじめに](#introduction)
    - [React / Vue でのビューの記述](#writing-views-in-react-or-vue)
- [ビューの作成とレンダリング](#creating-and-rendering-views)
    - [ネストしたビューディレクトリ](#nested-view-directories)
    - [最初に利用できるビュー](#creating-the-first-available-view)
    - [ビューの存在を判定](#determining-if-a-view-exists)
- [ビューにデータを渡す](#passing-data-to-views)
    - [すべてのビューでデータを共有](#sharing-data-with-all-views)
- [ビューコンポーザ](#view-composers)
    - [ビュークリエイタ](#view-creators)
- [ビューの最適化](#optimizing-views)

<a name="introduction"></a>
## はじめに

HTML 文書の文字列全体をルートやコントローラから直接返してしまうのは現実的ではありません。ビューでは、すべての HTML を別々のファイルとして配置できるような便利な方法を提供しています。

ビューは、コントローラ / アプリケーションロジックをプレゼンテーションロジックから分離し、`resources/views` ディレクトリに保存されます。Laravel を使用する場合、ビューテンプレートは通常 [Blade テンプレート言語](/docs/{{version}}/blade) を使用して記述されます。単純なビューは次のようになります。

```blade
<!-- View stored in resources/views/greeting.blade.php -->

<html>
    <body>
        <h1>Hello, {{ $name }}</h1>
    </body>
</html>
```

このビューは `resources/views/greeting.blade.php` に保存したら、次のようにグローバル `view` ヘルパを使用して結果を返します。

    Route::get('/', function () {
        return view('greeting', ['name' => 'James']);
    });

> **Note**  
> Blade テンプレートの作成方法に関する詳細情報は [Blade ドキュメント](/docs/{{version}}/blade) を確認してください。

<a name="writing-views-in-react-or-vue"></a>
### React / Vue でのビューの作成

多くの開発者は、Blade を介して PHP でフロントエンドのテンプレートを作成する代わりに、React または Vue を使用してテンプレートを作成することを好み始めています。Laravel では、[Inertia](https://inertiajs.com/) のおかげで、これを簡単に行うことができます。これは、SPA の構築によくある複雑な作業を行わずに、React / Vue のフロントエンドを Laravel のバックエンドに簡単に接続できるようにするライブラリです。

Breeze と Jetstream の [スターターキット](/docs/{{version}}/starter-kits) は、Inertia を利用した次の Laravel アプリケーションの優れた出発点となります。さらに、[Laravel Bootcamp](https://bootcamp.laravel.com) では、Vue や React の例を含む、Inertia を利用した Laravel アプリケーションの構築に関する完全なデモンストレーションが提供されています。

<a name="creating-and-rendering-views"></a>
## ビューの作成とレンダリング

アプリケーションの `resources/views` ディレクトリに `.blade.php` 拡張子を持つファイルを配置することで、ビューを作成できます。`.blade.php` 拡張子は、ファイルに [Blade テンプレート](/docs/{{version}}/blade) が含まれていることをフレームワークに通知します。Blade テンプレートには、HTML と Blade ディレクティブが含まれており、値のエコー、「if」文の作成、データの反復処理などを簡単に行うことができます。

ビューを作成したら、グローバル `view` ヘルパを使用して、アプリケーションのルートまたはコントローラからビューを返すことができます。

    Route::get('/', function () {
        return view('greeting', ['name' => 'James']);
    });

ビューは、`View` ファサードを使用して返すこともできます。

    use Illuminate\Support\Facades\View;

    return View::make('greeting', ['name' => 'James']);

ご覧のとおり、`view` ヘルパに渡される第１引数は、`resources/views` ディレクトリ内のビューファイルの名前に対応します。第２引数は、ビューで使用できるようにするデータの配列です。上記では、[Blade 構文](/docs/{{version}}/blade) を使用してビューに表示する、`name` 変数を渡しています。

<a name="nested-view-directories"></a>
### ネストされたビューのディレクトリ

ビューは、`resources/views` ディレクトリのサブディレクトリ内にネストすることもできます。「ドット」表記は、ネストされたビューを参照するために使用できます。たとえば、ビューが `resources/views/admin/profile.blade.php` に保存されている場合、次のようにアプリケーションのルート / コントローラからビューを返すことができます。

    return view('admin.profile', $data);

> **Warning**  
> ビューのディレクトリ名には `.` 文字を含めないでください。

<a name="creating-the-first-available-view"></a>
### 最初に利用できるビュー

`View` ファサードの`first` メソッドを使用すると、指定されたビューの配列に存在する最初のビューを作成できます。これは、アプリケーションまたはパッケージでビューのカスタマイズまたは上書きが許可されている場合に便利です。

    use Illuminate\Support\Facades\View;

    return View::first(['custom.admin', 'admin'], $data);

<a name="determining-if-a-view-exists"></a>
### ビューの存在を判定

ビューの存在を確認する必要がある場合、`View` ファサードの `exists` メソッドを使用します。ビューが存在する場合は、`true` を返します。

    use Illuminate\Support\Facades\View;

    if (View::exists('emails.customer')) {
        // ...
    }

<a name="passing-data-to-views"></a>
## ビューにデータを渡す

前の例で見たように、データの配列をビューに渡して、そのデータをビューで使用できるようにすることができます。

    return view('greetings', ['name' => 'Victoria']);

この方法で情報を渡す場合、データはキーと値のペアを含む配列である必要があります。ビューにデータを渡した後、`<?php echo $name; ?>` などのデータキーを使用して、ビュー内の各値にアクセスできます。

データの完全な配列を `view` ヘルパ関数に渡す代わりに、`with` メソッドを使用して個々のデータをビューに追加することもできます。`with` メソッドはビューオブジェクトのインスタンスを返すため、ビューを返す前にメソッドチェーンを続けることができます。

    return view('greeting')
                ->with('name', 'Victoria')
                ->with('occupation', 'Astronaut');

<a name="sharing-data-with-all-views"></a>
### すべてのビューでデータを共有

アプリケーションでレンダリングされる、すべてのビューとデータを共有したい場合は、`View` ファサードの `share` メソッドを使用します。通常、サービスプロバイダの `boot` メソッド内で `share` メソッドを呼び出します。これらを `App\Providers\AppServiceProvider` クラスに自由に追加するか、またはそれらを収容する別のサービスプロバイダを生成することもできます。

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\View;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Register any application services.
         */
        public function register(): void
        {
            // ...
        }

        /**
         * Bootstrap any application services.
         */
        public function boot(): void
        {
            View::share('key', 'value');
        }
    }

<a name="view-composers"></a>
## ビューコンポーザ

ビューコンポーザは、ビューのレンダリング時に呼び出されるコールバック、またはクラスメソッドです。ビューがレンダリングされるたびに、ビューへ結合したいデータがある場合、ビューコンポーザを使用すると、そのロジックを１つの場所にまとめることが可能です。ビューコンポーザは、アプリケーション内の複数のルート、またはコントローラから同じビューが返され、常に特定のデータを必要とする場合に特に便利です。

通常、ビューコンポーザはアプリケーションの [サービスプロバイダ](/docs/{{version}}/providers) に登録します。この例では、このロジックを格納するために新しい `App\Providers\ViewServiceProvider` を作成したと仮定します。

`View` ファサードの `composer` メソッドを使用して、ビューコンポーザを登録します。Laravel にはクラスベースのビューコンポーザ用のデフォルトディレクトリが含まれていないため、必要に応じて自由に編成できます。たとえば、アプリケーションのすべてのビューコンポーザを格納する `app/View/Composers` ディレクトリを作成できます。

    <?php

    namespace App\Providers;

    use App\View\Composers\ProfileComposer;
    use Illuminate\Support\Facades;
    use Illuminate\Support\ServiceProvider;
    use Illuminate\View\View;

    class ViewServiceProvider extends ServiceProvider
    {
        /**
         * Register any application services.
         */
        public function register(): void
        {
            // ...
        }

        /**
         * Bootstrap any application services.
         */
        public function boot(): void
        {
            // Using class based composers...
            Facades\View::composer('profile', ProfileComposer::class);

            // Using closure based composers...
            Facades\View::composer('welcome', function (View $view) {
                // ...
            });

            Facades\View::composer('dashboard', function (View $view) {
                // ...
            });
        }
    }

> **Warning**  
> ビューコンポーザの登録するために新しいサービスプロバイダを作成する場合は、そのサービスプロバイダを `config/app.php` 設定ファイルの `providers` 配列に追加する必要があります。

コンポーザーを登録したので、`profile` ビューがレンダリングされるたびに `App\View\Composers\ProfileComposer` クラスの `compose` メソッドが実行されます。コンポーザクラスの例を見てみましょう。

    <?php

    namespace App\View\Composers;

    use App\Repositories\UserRepository;
    use Illuminate\View\View;

    class ProfileComposer
    {
        /**
         * Create a new profile composer.
         */
        public function __construct(
            protected UserRepository $users,
        ) {}

        /**
         * Bind data to the view.
         */
        public function compose(View $view): void
        {
            $view->with('count', $this->users->count());
        }
    }

ご覧のとおり、すべてのビューコンポーザは [サービスコンテナ](/docs/{{version}}/container) を介して依存性解決されるため、コンポーザのコンストラクタ内で必要な依存関係をタイプヒントで指定できます。

<a name="attaching-a-composer-to-multiple-views"></a>
#### 複数のビューへのコンポーザ適用

ビューコンポーザを複数のビューに一度に適応したい場合、`composer` メソッドの第１引数にビューの配列を渡します。

    use App\Views\Composers\MultiComposer;
    use Illuminate\Support\Facades\View;

    View::composer(
        ['profile', 'dashboard'],
        MultiComposer::class
    );

すべてのビューにコンポーザを適応する場合、`composer` メソッドにワイルドカードとして `*` 文字を使用することができます。

    use Illuminate\Support\Facades;
    use Illuminate\View\View;

    Facades\View::composer('*', function (View $view) {
        // ...
    });

<a name="view-creators"></a>
### ビュークリエイタ

ビューの「クリエイタ」はビューのコンポーザと非常によく似ています。ビュークリエイタは、ビューがレンダリングされる直前まで待機するのではなく、ビューがインスタンス化された直後に実行されます。ビュービュークリエイタを登録するには、`creator` メソッドを使用します。

    use App\View\Creators\ProfileCreator;
    use Illuminate\Support\Facades\View;

    View::creator('profile', ProfileCreator::class);

<a name="optimizing-views"></a>
## ビューの最適化

デフォルトでは、Blade テンプレートビューはオンデマンドでコンパイルされます。ビューをレンダリングするリクエストが実行されると、Laravel はビューのコンパイル済みバージョンが存在するかどうかを判断します。ファイルが存在する場合、Laravel は、コンパイルされていないビューがコンパイルされたビューよりも最近に変更されたかどうかを判断します。コンパイル済みビューが存在しない、もしくは、未コンパイルのビューが変更されている場合、Laravel はビューを再コンパイルします。

リクエスト中にビューをコンパイルすると、パフォーマンスにわずかな悪影響を及ぼす可能性があるため、Laravel では、アプリケーションで使用されるすべてのビューをプリコンパイルするための `view:cache` Artisan コマンドが提供されています。パフォーマンスを向上させるために、展開プロセスの一部として次のコマンドを実行するとよいでしょう。

```shell
php artisan view:cache
```

ビューキャッシュを削除するには、`view:clear` コマンドを使用してください。

```shell
php artisan view:clear
```
