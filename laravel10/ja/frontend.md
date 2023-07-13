# フロントエンド

- [はじめに](#introduction)
- [PHP の利用](#using-php)
    - [PHP と Blade](#php-and-blade)
    - [Livewire](#livewire)
    - [スターターキット](#php-starter-kits)
- [Vue / React の使用](#using-vue-react)
    - [Inertia](#inertia)
    - [スターターキット](#inertia-starter-kits)
- [アセットの結合](#bundling-assets)

<a name="introduction"></a>
## はじめに

Laravelは、バックエンドフレームワークであり、[ルーティング](/laravel10_ja/routing)、[バリデーション](/laravel10_ja/validation)、[キャッシュ](/laravel10_ja/cache)、[キュー](/laravel10_ja/queues)、[ファイルストレージ](/laravel10_ja/filesystem) など、現代のWebアプリケーションを構築するために必要なすべての機能を提供しています。しかし、開発者に美しいフルスタックの経験を提供することも重要であり、アプリケーションのフロントエンドを構築するための強力なアプローチが含まれます。

Laravel でアプリケーションを構築する際、フロントエンド開発に取り組む主な方法は2つあります。どちらのアプローチを選ぶかは、フロントエンドを  PHP を活用して構築するか、Vue や React のような JavaScript フレームワークを使用して構築するかによって決まります。以下でこれらの選択肢について説明し、アプリケーションに最適なフロントエンド開発のアプローチを選択できるようにします。

<a name="using-php"></a>
## PHP の利用

<a name="php-and-blade"></a>
### PHP と Blade

これまで、ほとんどの PHP アプリケーションは、シンプルな HTML テンプレートを使用してブラウザに HTML をレンダリングし、リクエスト中にデータベースから取得されたデータを表示するために PHP の `echo` 文を使用していました。

```blade
<div>
    <?php foreach ($users as $user): ?>
        Hello, <?php echo $user->name; ?> <br />
    <?php endforeach; ?>
</div>
```

Laravel では、このようなHTMLのレンダリング方法は、[ビュー](/laravel10_ja/views) と [Blade](/laravel10_ja/blade) を使用して実現することができます。Blade は、データの表示、データの反復処理などに便利で短い構文を提供する非常に軽量なテンプレーティング言語です。

```blade
<div>
    @foreach ($users as $user)
        Hello, {{ $user->name }} <br />
    @endforeach
</div>
```

この方法でアプリケーションを構築する場合、通常、フォーム送信やその他のページ インタラクションは、サーバーからまったく新しい HTML ドキュメントを受け取り、ページ全体がブラウザーによって再レンダリングされます。 今日でも、多くのアプリケーションは、単純な Blade テンプレートを使用してこのようにフロントエンドを構築することに完全に適している場合があります。

<a name="growing-expectations"></a>
#### 期待の高まり

しかし、Web アプリケーションに対するユーザーの期待が成熟するにつれ、多くの開発者は、より洗練されたインタラクションを持つダイナミックなフロントエンドを構築する必要性を感じるようになりました。これを受けて、一部の開発者は、Vueや React のような JavaScript フレームワークを使用してアプリケーションのフロントエンドを構築し始めます。

一方、バックエンド言語に慣れている開発者は、主にバックエンド言語を使用しながら、現代の Web アプリケーション UI を構築するためのソリューションを開発しました。例えば [Rails](https://rubyonrails.org/) エコシステムでは、[Turbo](https://turbo.hotwired.dev/) [Hotwire](https://hotwired.dev/)、[Stimulus](https://stimulus.hotwired.dev/) などのライブラリが登場しています。

Laravel エコシステムでは、主に PHP を使用してモダンでダイナミックなフロントエンドを作成するためのニーズが、[Laravel Livewire](https://laravel-livewire.com) や [Alpine.js](https://alpinejs.dev/) の登場につながっています。

<a name="livewire"></a>
### Livewire

[Laravel Livewire](https://laravel-livewire.com) は、Vue や React のような JavaScript フレームワークで構築されたフロントエンドと同じように、ダイナミックでモダンで活気のあるフロントエンドを構築するための Laravel を活用したフレームワークです。

Livewire を使用する場合、UI の個別の部分をレンダリングし、アプリケーションのフロントエンドから呼び出され、操作されるメソッドやデータを公開する Livewire「コンポーネント」を作成します。例えば、シンプルな「Counter」コンポーネントは以下のようになります

```php
<?php

namespace App\Http\Livewire;

use Livewire\Component;

class Counter extends Component
{
    public $count = 0;

    public function increment()
    {
        $this->count++;
    }

    public function render()
    {
        return view('livewire.counter');
    }
}
```

そして、対応するカウンターのテンプレートは次のように記述されます。

```blade
<div>
    <button wire:click="increment">+</button>
    <h1>{{ $count }}</h1>
</div>
```

ご覧のとおり、Livewire を使用すると、Laravel アプリケーションのフロントエンドとバックエンドを接続する `wire:click` などの新しい HTML 属性を記述できます。 さらに、シンプルな Blade 式を使用して、コンポーネントの現在の状態をレンダリングできます。

多くの人にとって、Livewire は Laravel を使用したフロントエンド開発に革命をもたらし、最新の動的 Web アプリケーションを構築しながら Laravel の快適さを維持できるようにしました。 通常、Livewire を使用する開発者は、[Alpine.js](https://alpinejs.dev/) を利用して、ダイアログウィンドウをレンダリングする場合など、必要な場所にのみ JavaScript をフロントエンドに散りばめます。

Laravel が初めての方は、[ビュー](/laravel10_ja/views) と [Blade](/laravel10_ja/blade) の基本的な使用方法に慣れてから、[Laravel Livewire の公式ドキュメント](https://laravel-livewire.com/docs) を参照して、インタラクティブな Livewire コンポーネントを使ってアプリケーションを次のレベルに引き上げる方法を学んでください。

<a name="php-starter-kits"></a>
### スターター キット

PHP と Livewire を使用してフロントエンドを構築する場合は、Breeze または Jetstream の [スターター キット](/laravel10_ja/starter-kits) を活用して、アプリケーションの開発をすぐに開始できます。これらのスターターキットはどちらも、[Blade](/laravel10_ja/blade) と [Tailwind](https://tailwindcss.com) を使用して、アプリケーションのバックエンドとフロントエンドの認証フローを足場にします。 スキャフォールドし、次のビッグアイデアをすぐに構築できるようにします。

<a name="using-vue-react"></a>
## Vue / React の使用

Laravel と Livewire を使用して最新のフロントエンドを構築することは可能ですが、多くの開発者は依然として Vue や React などの JavaScript フレームワークの機能を活用することを好みます。 これにより、開発者は、NPM 経由で利用できる JavaScript パッケージとツールの豊富なエコシステムを活用できます。

ただし、追加のツールがなければ、Laravel を Vue または React と組み合わせると、クライアント側のルーティング、データ ハイドレーション、認証など、さまざまな複雑な問題を解決する必要が生じます。 クライアント側のルーティングは、多くの場合、[Nuxt](https://nuxtjs.org/) や [Next](https://nextjs.org/) などの独自の Vue / React フレームワークを使用して簡素化されます。ただし、Laravel のようなバックエンドフレームワークをこれらのフロントエンドフレームワークと組み合わせる場合、データのハイドレーションと認証は依然として複雑で面倒な問題です。

さらに、開発者は 2 つの別個のコードリポジトリを維持する必要があり、多くの場合、両方のリポジトリ間でメンテナンス、リリース、および展開を調整する必要があります。これらの問題は克服できないわけではありませんが、アプリケーションを開発するための生産的または楽しい方法ではないと考えています。

<a name="inertia"></a>
### Inertia

ありがたいことに、Laravel は両方の長所を提供します。[Inertia](https://inertiajs.com) は、Laravel アプリケーションと最新の Vue または React フロントエンドの間のギャップを埋め、Laravel のルートとコントローラーをルーティングに活用しながら、Vue または React を使用して本格的な最新のフロントエンドを構築できるようにします。データハイドレーションと認証 — すべてが単一のコード リポジトリ内にあります。このアプローチにより、Laravel と Vue / React の両方の機能を損なうことなく、両方の機能を最大限に活用できます。

Inertia を Laravel アプリケーションにインストールしたら、通常どおりルートとコントローラーを記述します。ただし、コントローラーから Blade テンプレートを返す代わりに、Inertia ページを返します。

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Models\User;
use Inertia\Inertia;
use Inertia\Response;

class UserController extends Controller
{
    /**
     * Show the profile for a given user.
     */
    public function show(string $id): Response
    {
        return Inertia::render('Users/Profile', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```

Inertia ページは、Vue または React コンポーネントに対応し、通常はアプリケーションの `resources/js/Pages` ディレクトリ内に保存されます。`Inertia::render` メソッドでページに渡されるデータは、ページコンポーネントの「props」をハイドレートするために使用されます。

```vue
<script setup>
import Layout from '@/Layouts/Authenticated.vue';
import { Head } from '@inertiajs/inertia-vue3';

const props = defineProps(['user']);
</script>

<template>
    <Head title="User Profile" />

    <Layout>
        <template #header>
            <h2 class="text-xl font-semibold leading-tight text-gray-800">
                Profile
            </h2>
        </template>

        <div class="py-12">
            Hello, {{ user.name }}
        </div>
    </Layout>
</template>
```

このように、Inertia を使えば、フロントエンドを構築する際に Vue や React の全機能を活用できるだけでなく、Laravel で動作するバックエンドと JavaScript で動作するフロントエンドとの間の軽量なブリッジを提供します。

#### サーバーサイドレンダリング

もし Inertia を使ってアプリケーションを構築する際に、サーバーサイドレンダリングが必要で心配な場合でも、大丈夫です。Inertia は [サーバーサイドレンダリングのサポート] (https://inertiajs.com/server-side-rendering) を提供します。また、[Laravel Forge](https://forge.laravel.com) を使ってアプリケーションをデプロイする際に、Inertia のサーバーサイドレンダリングプロセスが常に実行されていることを簡単に確認できます。

<a name="inertia-starter-kits"></a>
### スターター キット

もし Inertia と Vue / React を使ってフロントエンドを構築したい場合は、Breeze または Jetstream の [スターター キット](/laravel10_ja/starter-kits#breeze-and-inertia) を利用して、アプリケーションの開発をすくに開始できます。これらのスターターキットは、Inertia、Vue / React、[Tailwind](https://tailwindcss.com)、および [Vite](https://vitejs.dev) を使用してアプリケーションのバックエンドとフロントエンドの認証フローをスキャフォールドし、次のビッグアイデアをすぐに構築できるようにします。

<a name="bundling-assets"></a>
## アセットのバンドル

Blade と Livewire または Vue / React と Inertia を使ってフロントエンドを開発する場合に関係なく、アプリケーションの CSS をプロダクション用のアセットにバンドルする必要があります。もちろん、Vue や React を使ってアプリケーションのフロントエンドを構築する場合は、ブラウザで利用可能な JavaScript アセットにコンポーネントをバンドルする必要もあります。

デフォルトでは、Laravel は [Vite](https://vitejs.dev) を使ってアセットをバンドルします。Vite は、非常に高速なビルド時間と、ローカル開発時にほぼ瞬時に実行される Hot Module Replacement (HMR) を提供します。新しい Laravel アプリケーションでは、[スターターキット](/laravel10_ja/starter-kits) を含むすべてのアプリケーションで、Laravel Vite プラグインを読み込む `vite.config.js` ファイルが見つかります。このプラグインは軽量で、Laravel アプリケーションと Vite を使いやすくします。

Laravel と Vite を使ってアプリケーションの開発を始める最速の方法は、[Laravel Breeze](/laravel10_ja/starter-kits#laravel-breeze) を使って始めることです。これは、フロントエンドとバックエンドの認証スキャフォールドを提供してアプリケーションをジャンプスタートする、最もシンプルなスターターキットです。

> **Note**
> Laravel での Vite の利用に関する詳細なドキュメントについては、[アセットのバンドルとコンパイルに関する専用ドキュメント](/laravel10_ja/vite) を参照してください。
