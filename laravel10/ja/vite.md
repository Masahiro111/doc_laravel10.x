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
- [ベース URL のカスタム](#custom-base-urls)
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
## Working With Blade & Routes

<a name="blade-processing-static-assets"></a>
### Processing Static Assets With Vite

When referencing assets in your JavaScript or CSS, Vite automatically processes and versions them. In addition, when building Blade based applications, Vite can also process and version static assets that you reference solely in Blade templates.

However, in order to accomplish this, you need to make Vite aware of your assets by importing the static assets into the application's entry point. For example, if you want to process and version all images stored in `resources/images` and all fonts stored in `resources/fonts`, you should add the following in your application's `resources/js/app.js` entry point:

```js
import.meta.glob([
  '../images/**',
  '../fonts/**',
]);
```

These assets will now be processed by Vite when running `npm run build`. You can then reference these assets in Blade templates using the `Vite::asset` method, which will return the versioned URL for a given asset:

```blade
<img src="{{ Vite::asset('resources/images/logo.png') }}">
```

<a name="blade-refreshing-on-save"></a>
### Refreshing On Save

When your application is built using traditional server-side rendering with Blade, Vite can improve your development workflow by automatically refreshing the browser when you make changes to view files in your application. To get started, you can simply specify the `refresh` option as `true`.

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

When the `refresh` option is `true`, saving files in the following directories will trigger the browser to perform a full page refresh while you are running `npm run dev`:

- `app/View/Components/**`
- `lang/**`
- `resources/lang/**`
- `resources/views/**`
- `routes/**`

Watching the `routes/**` directory is useful if you are utilizing [Ziggy](https://github.com/tighten/ziggy) to generate route links within your application's frontend.

If these default paths do not suit your needs, you can specify your own list of paths to watch:

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

Under the hood, the Laravel Vite plugin uses the [`vite-plugin-full-reload`](https://github.com/ElMassimo/vite-plugin-full-reload) package, which offers some advanced configuration options to fine-tune this feature's behavior. If you need this level of customization, you may provide a `config` definition:

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
### Aliases

It is common in JavaScript applications to [create aliases](#aliases) to regularly referenced directories. But, you may also create aliases to use in Blade by using the `macro` method on the `Illuminate\Support\Facades\Vite` class. Typically, "macros" should be defined within the `boot` method of a [service provider](/docs/{{version}}/providers):

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Vite::macro('image', fn (string $asset) => $this->asset("resources/images/{$asset}"));
    }

Once a macro has been defined, it can be invoked within your templates. For example, we can use the `image` macro defined above to reference an asset located at `resources/images/logo.png`:

```blade
<img src="{{ Vite::image('logo.png') }}" alt="Laravel Logo">
```

<a name="custom-base-urls"></a>
## Custom Base URLs

If your Vite compiled assets are deployed to a domain separate from your application, such as via a CDN, you must specify the `ASSET_URL` environment variable within your application's `.env` file:

```env
ASSET_URL=https://cdn.example.com
```

After configuring the asset URL, all re-written URLs to your assets will be prefixed with the configured value:

```nothing
https://cdn.example.com/build/assets/app.9dce8d17.js
```

Remember that [absolute URLs are not re-written by Vite](#url-processing), so they will not be prefixed.

<a name="environment-variables"></a>
## Environment Variables

You may inject environment variables into your JavaScript by prefixing them with `VITE_` in your application's `.env` file:

```env
VITE_SENTRY_DSN_PUBLIC=http://example.com
```

You may access injected environment variables via the `import.meta.env` object:

```js
import.meta.env.VITE_SENTRY_DSN_PUBLIC
```

<a name="disabling-vite-in-tests"></a>
## Disabling Vite In Tests

Laravel's Vite integration will attempt to resolve your assets while running your tests, which requires you to either run the Vite development server or build your assets.

If you would prefer to mock Vite during testing, you may call the `withoutVite` method, which is is available for any tests that extend Laravel's `TestCase` class:

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

If you would like to disable Vite for all tests, you may call the `withoutVite` method from the `setUp` method on your base `TestCase` class:

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
## Server-Side Rendering (SSR)

The Laravel Vite plugin makes it painless to set up server-side rendering with Vite. To get started, create an SSR entry point at `resources/js/ssr.js` and specify the entry point by passing a configuration option to the Laravel plugin:

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

To ensure you don't forget to rebuild the SSR entry point, we recommend augmenting the "build" script in your application's `package.json` to create your SSR build:

```json
"scripts": {
     "dev": "vite",
     "build": "vite build" // [tl! remove]
     "build": "vite build && vite build --ssr" // [tl! add]
}
```

Then, to build and start the SSR server, you may run the following commands:

```sh
npm run build
node bootstrap/ssr/ssr.mjs
```

> **Note**  
> Laravel's [starter kits](/docs/{{version}}/starter-kits) already include the proper Laravel, Inertia SSR, and Vite configuration. Check out [Laravel Breeze](/docs/{{version}}/starter-kits#breeze-and-inertia) for the fastest way to get started with Laravel, Inertia SSR, and Vite.

<a name="script-and-style-attributes"></a>
## Script & Style Tag Attributes

<a name="content-security-policy-csp-nonce"></a>
### Content Security Policy (CSP) Nonce

If you wish to include a [`nonce` attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/nonce) on your script and style tags as part of your [Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP), you may generate or specify a nonce using the `useCspNonce` method within a custom [middleware](/docs/{{version}}/middleware):

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

After invoking the `useCspNonce` method, Laravel will automatically include the `nonce` attributes on all generated script and style tags.

If you need to specify the nonce elsewhere, including the [Ziggy `@route` directive](https://github.com/tighten/ziggy#using-routes-with-a-content-security-policy) included with Laravel's [starter kits](/docs/{{version}}/starter-kits), you may retrieve it using the `cspNonce` method:

```blade
@routes(nonce: Vite::cspNonce())
```

If you already have a nonce that you would like to instruct Laravel to use, you may pass the nonce to the `useCspNonce` method:

```php
Vite::useCspNonce($nonce);
```

<a name="subresource-integrity-sri"></a>
### Subresource Integrity (SRI)

If your Vite manifest includes `integrity` hashes for your assets, Laravel will automatically add the `integrity` attribute on any script and style tags it generates in order to enforce [Subresource Integrity](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity). By default, Vite does not include the `integrity` hash in its manifest, but you may enable it by installing the [`vite-plugin-manifest-sri`](https://www.npmjs.com/package/vite-plugin-manifest-sri) NPM plugin:

```shell
npm install --save-dev vite-plugin-manifest-sri
```

You may then enable this plugin in your `vite.config.js` file:

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

If required, you may also customize the manifest key where the integrity hash can be found:

```php
use Illuminate\Support\Facades\Vite;

Vite::useIntegrityKey('custom-integrity-key');
```

If you would like to disable this auto-detection completely, you may pass `false` to the `useIntegrityKey` method:

```php
Vite::useIntegrityKey(false);
```

<a name="arbitrary-attributes"></a>
### Arbitrary Attributes

If you need to include additional attributes on your script and style tags, such as the [`data-turbo-track`](https://turbo.hotwired.dev/handbook/drive#reloading-when-assets-change) attribute, you may specify them via the `useScriptTagAttributes` and `useStyleTagAttributes` methods. Typically, this methods should be invoked from a [service provider](/docs/{{version}}/providers):

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

If you need to conditionally add attributes, you may pass a callback that will receive the asset source path, its URL, its manifest chunk, and the entire manifest:

```php
use Illuminate\Support\Facades\Vite;

Vite::useScriptTagAttributes(fn (string $src, string $url, array|null $chunk, array|null $manifest) => [
    'data-turbo-track' => $src === 'resources/js/app.js' ? 'reload' : false,
]);

Vite::useStyleTagAttributes(fn (string $src, string $url, array|null $chunk, array|null $manifest) => [
    'data-turbo-track' => $chunk && $chunk['isEntry'] ? 'reload' : false,
]);
```

> **Warning**  
> The `$chunk` and `$manifest` arguments will be `null` while the Vite development server is running.

<a name="advanced-customization"></a>
## Advanced Customization

Out of the box, Laravel's Vite plugin uses sensible conventions that should work for the majority of applications; however, sometimes you may need to customize Vite's behavior. To enable additional customization options, we offer the following methods and options which can be used in place of the `@vite` Blade directive:

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

Within the `vite.config.js` file, you should then specify the same configuration:

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
### Correcting Dev Server URLs

Some plugins within the Vite ecosystem assume that URLs which begin with a forward-slash will always point to the Vite dev server. However, due to the nature of the Laravel integration, this is not the case.

For example, the `vite-imagetools` plugin outputs URLs like the following while Vite is serving your assets:

```html
<img src="/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520">
```

The `vite-imagetools` plugin is expecting that the output URL will be intercepted by Vite and the plugin may then handle all URLs that start with `/@imagetools`. If you are using plugins that are expecting this behaviour, you will need to manually correct the URLs. You can do this in your `vite.config.js` file by using the `transformOnServe` option. 

In this particular example, we will append the dev server URL to all occurrences of `/@imagetools` within the generated code:

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

Now, while Vite is serving Assets, it will output URLs that point to the Vite dev server:

```html
- <img src="/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520"><!-- [tl! remove] -->
+ <img src="http://[::1]:5173/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520"><!-- [tl! add] -->
```

