# アセットの構築 (Vite)

- [はじめに](#introduction)
- [インストールとセットアップ](#installation)
  - [Node のインストール](#installing-node)
  - [Vite と Laravel プラグインのインストール](#installing-vite-and-laravel-plugin)
  - [Vite の設定](#cconfiguring-vite)
  - [スクリプトとスタイルの読み込み](#loading-your-scripts-and-styles)
- [Vite の実行](#running-vite)
- [JavaScript の操作](#working-with-scripts)
  - [エイリアス](#aliases)
  - [Vue](#vue)
  - [React](#react)
  - [Inertia](#inertia)
  - [URL 処理](#url-processing)
- [スタイルシートの操作](#working-with-stylesheets)
- [Blade とルートの操作](#working-with-blade-and-routes)
  - [Vite による静的アセット処理](#blade-processing-static-assets)
  - [保存時の再描画](#blade-refreshing-on-save)
  - [エイリアス](#blade-aliases)
- [ベース URL のカスタマイズ](#custom-base-urls)
- [環境変数](#environment-variables)
- [テスト時に Vite を無効化](#disabling-vite-in-tests)
- [サーバーサイドレンダリング (SSR)](#ssr)
- [Script と style タグ属性](#script-and-style-attributes)
  - [コンテンツセキュリティポリシー (CSP) ノンス](#content-security-policy-csp-nonce)
  - [サブリソース整合性 (SRI)](#subresource-integrity-sri)
  - [任意の属性](#arbitrary-attributes)
- [高度なカスタマイズ](#advanced-customization)
  - [開発サーバー URL の修正](#correcting-dev-server-urls)

<a name="introduction"></a>
## はじめに

[Vite](https://vitejs.dev) は、非常に高速な開発環境を提供し、実稼働用のコードを構築する最新のフロントエンドビルドツールです。Laravel でアプリケーションを構築する場合、通常は Vite を使用して、アプリケーションの CSS ファイルと JavaScript ファイルを本番環境に対応したアセットにバンドルします。

Laravel は、開発および本番用にアセットをロードするための公式プラグインと Blade ディレクティブを提供することで、Vite とシームレスに統合します。

> **Note**  
> Laravel Mix を実行していますか？新しい Laravel のインストールでは Laravel Mix を Vite に置き換えました。Mix のドキュメントについては、[Laravel Mix](https://laravel-mix.com/) の Web サイトをご覧ください。Vite に切り替えたい場合は、[移行ガイド](https://github.com/laravel/vite-plugin/blob/main/UPGRADE.md#migrated-from-laravel-mix-to-vite) をご覧ください。

<a name="vite-or-mix"></a>
#### Vite と Laravel Mix の選択

Vite に移行する前、新しい Laravel アプリケーションはアセットをバンドルするときに [webpack](https://webpack.js.org/) を利用した [Mix](https://laravel-mix.com/) を利用していました。Vite は、リッチな JavaScript アプリケーションを構築する際に、より高速で生産性の高いエクスペリエンスを提供することに重点を置いています。[Inertia](https://inertiajs.com) などのツールで開発されたものを含め、シングル ページ アプリケーション (SPA) を開発している場合は、Vite が最適です。

Vite は、[Livewire](https://laravel-livewire.com) を使用するアプリケーションなど、JavaScript の「スプリンクル（トッピングのようなもの）」を使用した従来のサーバーサイドでレンダリングされたアプリケーションでも適切に動作します。ただし、JavaScript アプリケーションで直接参照されない任意のアセットをビルドにコピーする機能など、Laravel Mix がサポートするいくつかの機能が欠けています。

<a name="migration-back-to-mix"></a>
#### Mix に戻す

Vite のスカフォールドを使用して新しい Laravel アプリケーションを開始しましたが、Laravel Mix と webpack に戻す際は、[Vite から Mix への移行に関する公式ガイド](https://github.com/laravel/vite-plugin/blob/main/UPGRADE.md#migrated-from-vite-to-laravel-mix) を参照してください。

<a name="installation"></a>
## インストールとセットアップ

> **Note**
> 以下のドキュメントでは、Laravel Vite プラグインを手動でインストールして構成する方法について説明します。ただし、Laravel の [スターター キット](/docs/{{version}}/starter-kits) にはこのスカフォールドがすべて含まれており、Laravel と Vite を始めるための最も早い方法です。

<a name="installing-node"></a>
### NNode のインストール

Vite と Laravel プラグインを実行する前に、Node.js (１６以降) と NPM がインストールされていることを確認する必要があります。

```sh
node -v
npm -v
```

[Node 公式 Web サイト](https://nodejs.org/en/download/) からシンプルなグラフィカルインストーラを使用して、Node と NPM の最新バージョンを簡単にインストールできます。または、[Laravel Sail](https://laravel.com/docs/{{version}}/sail) を使用している場合は、Sail を通じて Node と NPM を呼び出すことができます。

```sh
./vendor/bin/sail node -v
./vendor/bin/sail npm -v
```

<a name="installing-vite-and-laravel-plugin"></a>
### Vite と Laravel プラグインのインストール

Laravel を新規インストールすると、アプリケーションのディレクトリ構造のルートに `package.json` ファイルができます。デフォルトの `package.json` ファイルには、Vite と Laravel プラグインの利用するために必要なものがすべて含まれています。NPM 経由でアプリケーションのフロントエンド依存関係をインストールできます。

```sh
npm install
```

<a name="cconfiguring-vite"></a>
### Vite の設定

Vite は、プロジェクトのルートにある `vite.config.js` ファイルを介して設定します。このファイルは必要に応じて自由にカスタマイズできます。また、アプリケーションに必要な他のプラグイン (`@vitejs/plugin-vue` や `@vitejs/plugin-react` など) をインストールすることもできます。

Laravel Vite プラグインでは、アプリケーションのエントリポイントを指定する必要があります。これらは JavaScript または CSS ファイルであり、TypeScript、JSX、TSX、Sass などのプリプロセス言語が含まれます。

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel([
            'resources/css/app.css',
            'resources/js/app.js',
        ]),
    ],
});
```

Inertia を使用して構築されたアプリケーションを含む SPA を構築している場合、Vite は CSS エントリポイントなしで最適に動作します。

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel([
            'resources/css/app.css', // [tl! remove]
            'resources/js/app.js',
        ]),
    ],
});
```

代わりに、JavaScript 経由で CSS をインポートする必要があります。通常、これはアプリケーションの `resources/js/app.js` ファイルで行われます。

```js
import './bootstrap';
import '../css/app.css'; // [tl! add]
```

Laravel プラグインは、複数のエントリポイントと、[SSR エントリポイント](#ssr) などの高度な構成オプションもサポートしています。

<a name="working-with-a-secure-development-server"></a>
#### セキュアな開発サーバーの使用

ローカル開発 Web サーバーが HTTPS 経由でアプリケーションを提供している場合、Vite 開発サーバーへの接続で問題が発生する可能性があります。

ローカル開発に [Laravel Valet](/docs/{{version}}/valet) を使用しており、アプリケーションで [secure コマンド](/docs/{{version}}/valet#securing-sites) を実行した場合 、Valet が生成した TLS 証明書を Vite 開発サーバーで自動的に使用するように設定できます。

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            valetTls: 'my-app.test', // [tl! add]
        }),
    ],
});
```

別の Web サーバーを使用する場合は、信頼できる証明書を生成し、生成された証明書を使用するように Vite を手動で設定する必要があります。

```js
// ...
import fs from 'fs'; // [tl! add]

const host = 'my-app.test'; // [tl! add]

export default defineConfig({
    // ...
    server: { // [tl! add]
        host, // [tl! add]
        hmr: { host }, // [tl! add]
        https: { // [tl! add]
            key: fs.readFileSync(`/path/to/${host}.key`), // [tl! add]
            cert: fs.readFileSync(`/path/to/${host}.crt`), // [tl! add]
        }, // [tl! add]
    }, // [tl! add]
});
```

システムの信頼できる証明書を生成できない場合は、[`@vitejs/plugin-basic-ssl` プラグイン](https://github.com/vitejs/vite-plugin-basic-ssl) をインストールして設定してください。信頼できない証明書を使用する場合、`npm run dev` コマンドの実行時にコンソールの「Local」リンクをたどって、ブラウザで Vite の開発サーバーに対する証明書の警告を受け入れる必要があります。

<a name="loading-your-scripts-and-styles"></a>
### スクリプトとスタイルの読み込み

Vite エントリポイントを設定したら、アプリケーションのルートテンプレートの `<head>` に `@vite()` Blade ディレクティブを追加するだけで、JS と CSS を参照できるようになります。

```blade
<!doctype html>
<head>
    {{-- ... --}}

    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
```

JavaScript 経由で CSS をインポートする場合、 JavaScript エントリポイントのみを記述するだけです。

```blade
<!doctype html>
<head>
    {{-- ... --}}

    @vite('resources/js/app.js')
</head>
```

`@vite` ディレクティブは、Vite 開発サーバーを自動的に検出し、Vite クライアントを注入してホットモジュール置換を有効にします。このディレクティブは、ビルドモードでインポートされた CSS を含むコンパイルおよびバージョン管理されたアセットを読み込みます。

必要に応じて、`@vite` ディレクティブを呼び出すときに、コンパイル済みアセットのビルドパスを指定することもできます。

```blade
<!doctype html>
<head>
    {{-- Given build path is relative to public path. --}}

    @vite('resources/js/app.js', 'vendor/courier/build')
</head>
```

<a name="running-vite"></a>
## Vite の実行

Vite を実行するには２つの方法があります。`dev` コマンドで開発サーバを起動できます。これは、ローカルで開発するときに便利です。開発サーバはファイルへの変更を自動的に検出し、開いているブラウザウィンドウに即座に変更を反映します。

または、`build` コマンドを実行すると、アプリケーションのアセットがバージョン管理されてバンドルされ、本番環境にデプロイできるように準備されます。

```shell
# Run the Vite development server...
npm run dev

# Build and version the assets for production...
npm run build
```

<a name="working-with-scripts"></a>
## JavaScript の操作

<a name="aliases"></a>
### エイリアス

デフォルトでは、Laravel プラグインは、アプリケーションのアセットを簡単にインポートできるようにするための共通のエイリアスを提供します。

```js
{
    '@' => '/resources/js'
}
```

独自のエイリアスを `vite.config.js` 設定ファイルに追加することで、`'@'` エイリアスを上書きできます。

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel(['resources/ts/app.tsx']),
    ],
    resolve: {
        alias: {
            '@': '/resources/ts',
        },
    },
});
```

<a name="vue"></a>
### ビュー

[Vue](https://vuejs.org/) フレームワークを使用してフロントエンドを構築したい場合は、`@vitejs/plugin-vue` プラグインもインストールする必要があります。

```sh
npm install --save-dev @vitejs/plugin-vue
```

それから、プラグインを `vite.config.js` 設定ファイルに含めることができます。Laravel で Vue プラグインを使用する場合、必要となる追加オプションがいくつかあります。

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
    plugins: [
        laravel(['resources/js/app.js']),
        vue({
            template: {
                transformAssetUrls: {
                    // The Vue plugin will re-write asset URLs, when referenced
                    // in Single File Components, to point to the Laravel web
                    // server. Setting this to `null` allows the Laravel plugin
                    // to instead re-write asset URLs to point to the Vite
                    // server instead.
                    base: null,

                    // The Vue plugin will parse absolute URLs and treat them
                    // as absolute paths to files on disk. Setting this to
                    // `false` will leave absolute URLs un-touched so they can
                    // reference assets in the public directory as expected.
                    includeAbsolute: false,
                },
            },
        }),
    ],
});
```

> **Note**
> Laravel の [スターター キット](/docs/{{version}}/starter-kits) には、すでに適切な Laravel、Vue、Vite の構成が含まれています。Laravel、Vue、Vite を最速で始める方法については、[Laravel Breeze](/docs/{{version}}/starter-kits#breeze-and-inertia) をご確認ください。

<a name="react"></a>
### React

[React](https://reactjs.org/) フレームワークを使用してフロントエンドを構築したい場合は、`@vitejs/plugin-react` プラグインもインストールする必要があります。

```sh
npm install --save-dev @vitejs/plugin-react
```

次に、プラグインを `vite.config.js` 設定ファイルにインクルードしてください。

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import react from '@vitejs/plugin-react';

export default defineConfig({
    plugins: [
        laravel(['resources/js/app.jsx']),
        react(),
    ],
});
```

JSX を含むすべてのファイルに `.jsx` または `.tsx` 拡張子が付いていることを確認し、必要に応じて [上記のように](#cconfiguring-vite) エントリポイントの更新を忘れないでください。

また、既存の `@vite` ディレクティブに加えて、追加の `@viteReactRefresh` Blade ディレクティブをインクルードする必要もあります。

```blade
@viteReactRefresh
@vite('resources/js/app.jsx')
```

`@viteReactRefresh` ディレクティブは、`@vite` ディレクティブの前に呼び出す必要があります。

> **Note**
> Laravel の [スターター キット](/docs/{{version}}/starter-kits) には、すでに適切な Laravel、React、Vite の構成が含まれています。 Laravel、React、Vite を最も早く使い始める方法については、[Laravel Breeze](/docs/{{version}}/starter-kits#breeze-and-inertia) をご確認ください。

<a name="inertia"></a>
### Inertia

Laravel Vite プラグインは、Inertia ページコンポーネントの解決に役立つ便利な `resolvePageComponent` 関数を提供します。 以下は、Vue 3 で使用されるヘルパの例です。ただし、この関数は React などの他のフレームワークでも利用できます。

```js
import { createApp, h } from 'vue';
import { createInertiaApp } from '@inertiajs/vue3';
import { resolvePageComponent } from 'laravel-vite-plugin/inertia-helpers';

createInertiaApp({
  resolve: (name) => resolvePageComponent(`./Pages/${name}.vue`, import.meta.glob('./Pages/**/*.vue')),
  setup({ el, App, props, plugin }) {
    return createApp({ render: () => h(App, props) })
      .use(plugin)
      .mount(el)
  },
});
```

> **Note**
> Laravel の [スターター キット](/docs/{{version}}/starter-kits) には、適切な Laravel、Inertia、Vite の構成がすでに含まれています。Laravel、Inertia、Vite を最も早く使い始める方法については、[Laravel Breeze](/docs/{{version}}/starter-kits#breeze-and-inertia) をご確認ください。

<a name="url-processing"></a>
### URL処理

Vite を使用し、アプリケーションの HTML、CSS、または JS でアセットを参照する場合、考慮すべき注意事項がいくつかあります。まず、絶対パスでアセットを参照すると、Vite はビルドにアセットを含めません。したがって、アセットがパブリックディレクトリで利用可能であることを確認する必要があります。

相対パスで参照する場合、参照先のファイルに対する相対パスであることに注意してください。相対パス経由で参照されるアセットはすべて、Vite によって書き換えられ、バージョン管理され、バンドルされます。

以下のプロジェクト構造を考えてみましょう。

```nothing
public/
  taylor.png
resources/
  js/
    Pages/
      Welcome.vue
  images/
    abigail.png
```

以下の例は、Vite が相対 URL と絶対 URL をどのように扱うかを示しています。

```html
<!-- This asset is not handled by Vite and will not be included in the build -->
<img src="/taylor.png">

<!-- This asset will be re-written, versioned, and bundled by Vite -->
<img src="../../images/abigail.png">
```

<a name="working-with-stylesheets"></a>
## スタイルシートの操作

Vite の CSS サポートについて詳しくは、[Vite ドキュメント](https://vitejs.dev/guide/features.html#css) をご覧ください。[Tailwind](https://tailwindcss.com) など PostCSS プラグインを使用している場合、プロジェクトのルートに `postcss.config.js` ファイルを作成すると、Vite がそれを自動的に適用します。

```js
module.exports = {
    plugins: {
        tailwindcss: {},
        autoprefixer: {},
    },
};
```

> **Note**
> Laravel の [スターター キット](/docs/{{version}}/starter-kits) には、適切な Tailwind、PostCSS、Vite の構成がすでに含まれています。スターター キットを使用せずに Tailwind と Laravel を使用したい場合は、[Tailwind の Laravel インストール ガイド](https://tailwindcss.com/docs/guides/laravel) をご確認ください。

<a name="working-with-blade-and-routes"></a>
## Blade とルートの操作

<a name="blade-processing-static-assets"></a>
### Vite による静的アセット処理

JavaScript や CSS のアセットを参照すると、Vite はそれらを自動的に処理してバージョン付けします。さらに、Blade ベースのアプリケーションを構築する場合、Vite は Blade テンプレート内でのみ参照する静的アセットを処理、およびバージョン管理することもできます。

これを実現するには、静的アセットをアプリケーションのエントリポイントにインポートして、Vite にアセットを認識させる必要があります。たとえば、`resources/images` に保存されているすべての画像と、`resources/fonts` に保存されているすべてのフォントを処理してバージョン管理したい場合は、アプリケーションのエントリポイント `resources/js/app.js` に次の行を追加する必要があります。

```js
import.meta.glob([
  '../images/**',
  '../fonts/**',
]);
```

これらのアセットは、`npm run build` の実行時に Vite によって処理されるようになります。その後、`Vite::asset` メソッドを使用して、Blade テンプレートでこれらのアセットを参照できます。これにより、特定のアセットのバージョン管理された URL が返されます。

```blade
<img src="{{ Vite::asset('resources/images/logo.png') }}">
```

<a name="blade-refreshing-on-save"></a>
### 保存時の再描写

Blade での従来のサーバサイドレンダリングを使用してアプリケーションが構築されている場合、Vite はアプリケーション内のビューファイルを変更した際、ブラウザが自動的に更新されることで、開発ワークフローを改善します。この設定は `refresh` オプションを `true` に指定するだけです。

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            refresh: true,
        }),
    ],
});
```

`refresh` オプションが `true` の場合、以下のディレクトリにファイルを保存すると、`npm run dev` の実行中にブラウザがページ全体を更新します。

- `app/View/Components/**`
- `lang/**`
- `resources/lang/**`
- `resources/views/**`
- `routes/**`

[Ziggy](https://github.com/tighten/ziggy) を利用してアプリケーションのフロントエンドにルートリンクを生成する場合、`routes/**` ディレクトリを監視すると便利です。

これらのデフォルトのパスがニーズに合わない場合は、監視するパスの独自のリストを指定できます。

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            refresh: ['resources/views/**'],
        }),
    ],
});
```

Laravel Vite プラグイン内部では、[`vite-plugin-full-reload`](https://github.com/ElMassimo/vite-plugin-full-reload) パッケージを使用しており、この機能の動作を調整するための高度な設定オプションが提供されます。このレベルのカスタマイズが必要な場合は、`config` 定義を指定してください。

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            refresh: [{
                paths: ['path/to/watch/**'],
                config: { delay: 300 }
            }],
        }),
    ],
});
```

<a name="blade-aliases"></a>
### エイリアス

JavaScript アプリケーションでは、定期的に参照されるディレクトリに [エイリアスを作成](#aliases) するのが一般的です。ただし、`Illuminate\Support\Facades\Vite` クラスの `macro` メソッドを使用して、Blade で使用するエイリアスを作成することもできます。 通常、「マクロ」は [サービスプロバイダ](/docs/{{version}}/providers) の `boot` メソッド内で定義する必要があります。

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Vite::macro('image', fn (string $asset) => $this->asset("resources/images/{$asset}"));
    }

マクロを定義すると、テンプレート内でマクロを呼び出せます。たとえば、上記で定義した `image` マクロを使用して、`resources/images/logo.png` にあるアセットを参照してみましょう。

```blade
<img src="{{ Vite::image('logo.png') }}" alt="Laravel Logo">
```

<a name="custom-base-urls"></a>
## ベース URL のカスタマイズ

Vite でコンパイルをしたアセットが CDN 経由など、アプリケーションとは別のドメインにデプロイされている場合は、アプリケーションの `.env` ファイル内で `ASSET_URL` 環境変数を指定する必要があります。

```env
ASSET_URL=https://cdn.example.com
```

アセット URL を設定すると、書き換える URL の先頭に、設定した値が付きます。

```nothing
https://cdn.example.com/build/assets/app.9dce8d17.js
```

[絶対 URL は Vite によって書き換えられない](#url-processing) ため、プレフィックスは付加されないことに注意してください。

<a name="environment-variables"></a>
## 環境変数

アプリケーションの `.env` ファイル内で環境変数の先頭に `VITE_` を付けることで、JavaScript に環境変数を注入できます。

```env
VITE_SENTRY_DSN_PUBLIC=http://example.com
```

注入された環境変数には、`import.meta.env` オブジェクト経由でアクセスできます。

```js
import.meta.env.VITE_SENTRY_DSN_PUBLIC
```

<a name="disabling-vite-in-tests"></a>
## テスト時に Vite を無効化

Laravel の Vite 統合では、テストの実行中にアセットの解決が試行されるため、Vite 開発サーバーを実行するか、アセットをビルドする必要があります。

テスト中に Vite をモックしたい場合は、Laravel の `TestCase` クラスを拡張するあらゆるテストで利用できる `withoutVite` メソッドを呼び出すことができます。

```php
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_without_vite_example(): void
    {
        $this->withoutVite();

        // ...
    }
}
```

すべてのテストで Vite を無効にしたい場合は、ベースの `TestCase` クラスの `setUp` メソッドから `withoutVite` メソッドを呼び出します。

```php
<?php

namespace Tests;

use Illuminate\Foundation\Testing\TestCase as BaseTestCase;

abstract class TestCase extends BaseTestCase
{
    use CreatesApplication;

    protected function setUp(): void// [tl! add:start]
    {
        parent::setUp();

        $this->withoutVite();
    }// [tl! add:end]
}
```

<a name="ssr"></a>
## サーバーサイドレンダリング (SSR)

Laravel Vite プラグインを使用すると、Vite でのサーバサイドレンダリングを簡単にセットアップできます。まず、`resources/js/ssr.js` に SSR エントリポイントを作成し、構成オプションを Laravel プラグインに渡すことでエントリポイントを指定します。

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: 'resources/js/app.js',
            ssr: 'resources/js/ssr.js',
        }),
    ],
});
```

SSR エントリポイントの再構築を忘れないようにするために、アプリケーションの `package.json` 内の「build」スクリプトを拡張して SSR ビルドを作成することをお勧めします。

```json
"scripts": {
     "dev": "vite",
     "build": "vite build" // [tl! remove]
     "build": "vite build && vite build --ssr" // [tl! add]
}
```

次に、SSR サーバを構築して起動するには、以下のコマンドを実行します。

```sh
npm run build
node bootstrap/ssr/ssr.mjs
```

[Inertia で SSR](https://inertiajs.com/server-side-rendering) を使用している場合は、代わりに `inertia:start-ssr` Artisan コマンドを使用して SSR サーバを起動できます。

```sh
php artisan inertia:start-ssr
```

> **Note**
> Laravel の [スターター キット](/docs/{{version}}/starter-kits) には、適切な Laravel、Inertia SSR、Vite 構成がすでに含まれています。Laravel、Inertia SSR、Vite を最も早く使い始める方法については、[Laravel Breeze](/docs/{{version}}/starter-kits#breeze-and-inertia) をチェックしてください。

<a name="script-and-style-attributes"></a>
## Script と style タグ属性

<a name="content-security-policy-csp-nonce"></a>
### コンテンツセキュリティポリシー (CSP) ノンス

[コンテンツセキュリティポリシー](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) の一部としてスクリプトとスタイルタグに [`nonce` 属性](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/nonce) を含めたい場合、カスタム [ミドルウェア](/docs/{{version}}/middleware) の `useCspNonce` メソッドを使用して、ノンスを生成、指定できます。

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Vite;
use Symfony\Component\HttpFoundation\Response;

class AddContentSecurityPolicyHeaders
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next, string $role): Response
    {
        Vite::useCspNonce();

        return $next($request)->withHeaders([
            'Content-Security-Policy' => "script-src 'nonce-".Vite::cspNonce()."'",
        ]);
    }
}
```

`useCspNonce` メソッドを呼び出した後、Laravel は生成されたすべての script タグと style タグに `nonce` 属性を自動的に含めます。

Laravel の [スターターキット](/docs/{{version}}/starter-kits) に含まれる [Ziggy `@route` directive](https://github.com/tighten/ziggy#using-routes-with-a-content-security-policy) など、他の場所でノンスを指定する必要がある場合は、`cspNonce` メソッドを使用して取得できます。

```blade
@routes(nonce: Vite::cspNonce())
```

Laravel に使用を指示したいノンスがすでにある場合は、そのノンスを `useCspNonce` メソッドに渡すことができます。

```php
Vite::useCspNonce($nonce);
```

<a name="subresource-integrity-sri"></a>
### サブリソース完全性 (SRI)

Vite マニフェストにアセットの `integrity` ハッシュが含まれている場合、Laravel は [サブリソースの整合性](https://developer.mozilla.org/en) を強制するために、生成するスクリプトとスタイル タグに `integrity` 属性を自動的に追加します。 -US/docs/Web/Security/Subresource_Integrity)。 デフォルトでは、Vite のマニフェストには「integrity」ハッシュが含まれていませんが、[`vite-plugin-manifest-sri`](https://www.npmjs.com/package/vite-) をインストールすることで有効にすることができます。 plugin-manifest-sri) NPM プラグイン:

```shell
npm install --save-dev vite-plugin-manifest-sri
```

その後、「vite.config.js」ファイルでこのプラグインを有効にすることができます。

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import manifestSRI from 'vite-plugin-manifest-sri';// [tl! add]

export default defineConfig({
    plugins: [
        laravel({
            // ...
        }),
        manifestSRI(),// [tl! add]
    ],
});
```

必要に応じて、整合性ハッシュが見つかるマニフェスト キーをカスタマイズすることもできます。

```php
use Illuminate\Support\Facades\Vite;

Vite::useIntegrityKey('custom-integrity-key');
```

この自動検出を完全に無効にしたい場合は、`useIntegrityKey` メソッドに `false` を渡します。

```php
Vite::useIntegrityKey(false);
```

<a name="arbitrary-attributes"></a>
### 任意の属性

[`data-turbo-track`](https://turbo.hotwired.dev/handbook/drive#reloading-when-assets-change) 属性など、スクリプトとスタイル タグに追加の属性を含める必要がある場合 、`useScriptTagAttributes` および `useStyleTagAttributes` メソッドを介して指定できます。 通常、このメソッドは [サービス プロバイダー](/docs/{{version}}/providers) から呼び出す必要があります。

```php
use Illuminate\Support\Facades\Vite;

Vite::useScriptTagAttributes([
    'data-turbo-track' => 'reload', // Specify a value for the attribute...
    'async' => true, // Specify an attribute without a value...
    'integrity' => false, // Exclude an attribute that would otherwise be included...
]);

Vite::useStyleTagAttributes([
    'data-turbo-track' => 'reload',
]);
```

条件付きで属性を追加する必要がある場合は、アセットのソース パス、その URL、そのマニフェスト チャンク、およびマニフェスト全体を受け取るコールバックを渡すことができます。

```php
use Illuminate\Support\Facades\Vite;

Vite::useScriptTagAttributes(fn (string $src, string $url, array|null $chunk, array|null $manifest) => [
    'data-turbo-track' => $src === 'resources/js/app.js' ? 'reload' : false,
]);

Vite::useStyleTagAttributes(fn (string $src, string $url, array|null $chunk, array|null $manifest) => [
    'data-turbo-track' => $chunk && $chunk['isEntry'] ? 'reload' : false,
]);
```

> **警告**
> Vite 開発サーバーの実行中、`$chunk` および `$manifest` 引数は `null` になります。

<a name="advanced-customization"></a>
## 高度なカスタマイズ

Laravel の Vite プラグインは、そのままの状態で、ほとんどのアプリケーションで機能する賢明な規則を使用しています。 ただし、Vite の動作をカスタマイズする必要がある場合があります。 追加のカスタマイズ オプションを有効にするために、`@vite` Blade ディレクティブの代わりに使用できる次のメソッドとオプションが提供されています。

```blade
<!doctype html>
<head>
    {{-- ... --}}

    {{
        Vite::useHotFile(storage_path('vite.hot')) // Customize the "hot" file...
            ->useBuildDirectory('bundle') // Customize the build directory...
            ->useManifestFilename('assets.json') // Customize the manifest filename...
            ->withEntryPoints(['resources/js/app.js']) // Specify the entry points...
    }}
</head>
```

`vite.config.js` ファイル内で、同じ設定を指定する必要があります。

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            hotFile: 'storage/vite.hot', // Customize the "hot" file...
            buildDirectory: 'bundle', // Customize the build directory...
            input: ['resources/js/app.js'], // Specify the entry points...
        }),
    ],
    build: {
      manifest: 'assets.json', // Customize the manifest filename...
    },
});
```

<a name="correcting-dev-server-urls"></a>
### 開発サーバーの URL を修正する

Vite エコシステム内の一部のプラグインは、スラッシュで始まる URL が常に Vite dev サーバーを指すことを前提としています。 ただし、Laravel 統合の性質により、これは当てはまりません。

たとえば、「vite-imagetools」プラグインは、Vite がアセットを提供しているときに次のような URL を出力します。

```html
<img src="/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520">
```

`vite-imagetools` プラグインは、出力 URL が Vite によってインターセプトされることを期待しており、プラグインは `/@imagetools` で始まるすべての URL を処理できるようになります。 この動作を想定しているプラグインを使用している場合は、URL を手動で修正する必要があります。 これは、「vite.config.js」ファイルで「transformOnServe」オプションを使用して行うことができます。

この特定の例では、生成されたコード内のすべての `/@imagetools` に開発サーバー URL を追加します。

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import { imagetools } from 'vite-imagetools';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            transformOnServe: (code, devServerUrl) => code.replaceAll('/@imagetools', devServerUrl+'/@imagetools'),
        }),
        imagetools(),
    ],
});
```

これで、Vite がアセットを提供している間、Vite 開発サーバーを指す URL が出力されます。

```html
- <img src="/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520"><!-- [tl! remove] -->
+ <img src="http://[::1]:5173/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520"><!-- [tl! add] -->
```

