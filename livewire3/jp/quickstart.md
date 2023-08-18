> [!warning] Livewire 3 はまだベータ版です
> 重大な変更を最小限に抑えるよう努めていますが、Livewire 3 がベータ版である間は変更が可能です。したがって、Livewire 3 を運用環境で使用する前に、アプリケーションを徹底的にテストすることをお勧めします。

Livewire の旅を始めるために、単純な「カウンター」コンポーネントを作成し、ブラウザーでレンダリングします。この例は、可能な限り簡単な方法で Livewire のライブネスを示すため、Livewire を初めて体験するのに最適な方法です。

## 前提条件

始める前に、以下がインストールされていることを確認してください。

- Laravel バージョン 9 以降
- PHP バージョン 8.1 以降

## Livewire をインストールする

Laravel アプリのルートディレクトリから、次の [Composer](https://getcomposer.org/) コマンドを実行します。

```shell
composer require livewire/livewire:^3.0@beta
```

## Livewire コンポーネントの作成

Livewire には、新しいコンポーネントを迅速に生成するための便利な Artisan コマンドが用意されています。以下のコマンドを実行して、新しい `Counter` コンポーネントを作成してみましょう。

```shell
php artisan make:livewire counter
```

このコマンドは、プロジェクト内に 2 つの新しいファイルを生成します。
* `App\Livewire\Counter.php`
* `resources/views/livewire/counter.blade.php`

## クラスの記述

`app/Livewire/Counter.php` を開き、内容を以下のように置き換えます。

```php
<?php

namespace App\Livewire;

use Livewire\Component;

class Counter extends Component
{
    public $count = 1;

    public function increment()
    {
        $this->count++;
    }

    public function decrement()
    {
        $this->count--;
    }

    public function render()
    {
        return view('livewire.counter');
    }
}
```

上記のコードの簡単な説明は以下のとおりです。
- `public $count = 1;` - `$count` という名前のパブリックプロパティを初期値 `1` で宣言します。
- `public function increment()` - 呼び出されるたびに `$count` プロパティをインクリメントする `increment()` という名前の パブリックメソッドを宣言します。このようなパブリックメソッドは、ユーザーがボタンをクリックしたときなど、さまざまな方法でブラウザから呼び出すことができます。
- `public function render()` - Blade ビューを返す `render()` メソッドを宣言します。この Blade ビューには、コンポーネントの HTML テンプレートが含まれます。

## ビューの作成

`resources/views/livewire/counter.blade.php` ファイルを開き、その内容を以下に置き換えます。

```blade
<div>
    <h1>{{ $count }}</h1>

    <button wire:click="increment">+</button>

    <button wire:click="decrement">-</button>
</div>
```

このコードは、`$count` プロパティの値と、`$count` プロパティをそれぞれ増加および減少させる２つのボタンを表示します。

## コンポーネントのルートを登録

Laravel アプリケーションで `routes/web.php` ファイルを開き、次のコードを追加します。

```php
use App\Livewire\Counter;

Route::get('/counter', Counter::class);
```

これで、`counter` コンポーネントが `/counter` ルートに割り当てられるようになり、ユーザーがアプリケーションの `/counter` エンドポイントにアクセスすると、このコンポーネントがブラウザによってレンダリングされます。

## テンプレートレイアウトの作成

ブラウザで `/counter` にアクセスする前に、内部でレンダリングするコンポーネントの HTML レイアウトが必要です。デフォルトでは、Livewire は自動的に `resources/views/components/layout.blade.php` という名前のレイアウトファイルを検索します。

このファイルが存在しない場合は、以下のコマンドを実行して作成できます。

```shell
php artisan livewire:layout
```

このコマンドは、以下の内容を含む `resources/views/components/layout.blade.php` というファイルを生成します。

```blade
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">

        <title>{{ $title ?? 'Page Title' }}</title>
    </head>
    <body>
        {{ $slot }}
    </body>
</html>
```

`counter` コンポーネントは、上記のテンプレートの `$slot` 変数の代わりにレンダリングされます。

Livewire では JavaScript や CSS アセットが提供されていないことに気づいたかもしれません。 これは、Livewire 3 以降では、必要なフロントエンドアセットが自動的に挿入されるためです。

## 試してみる

コンポーネント クラスとテンプレートを配置したら、コンポーネントをテストする準備が整いました。

ブラウザで `/counter` にアクセスすると、画面に数字が表示され、数字を増減する２つのボタンが表示されます。

いずれかのボタンをクリックすると、ページがリロードされずにカウントがリアルタイムで更新されることがわかります。これが Livewire の魔法です。すべて PHP で書かれた動的なフロントエンドアプリケーションです。

Livewire の機能のほんの表面をなぞっただけです。Livewire が提供するすべての機能を確認するには、ドキュメントを読み続けてください。
