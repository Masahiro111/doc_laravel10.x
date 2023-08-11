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

Now, our _counter_ component is assigned to the `/counter` route so that when a user visits the `/counter` endpoint in your application, this component will be rendered by the browser.

## Create a template layout

Before you can visit `/counter` in the browser, we need an HTML layout for our component to render inside. By default, Livewire will automatically look for a layout file named: `resources/views/components/layout.blade.php`

You may create this file if it doesn't already exist by running the following command:

```shell
php artisan livewire:layout
```

This command will generate a file called `resources/views/components/layout.blade.php` with the following contents:

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

The _counter_ component will be rendered in place of the `$slot` variable in the template above.

You may have noticed there is no JavaScript or CSS assets provided by Livewire. That is because Livewire 3 and above automatically injects any frontend assets it needs.

## Test it out

With our component class and templates in place, our component is ready to test!

Visit `/counter` in your browser, and you should see a number displayed on the screen with two buttons to increment and decrement the number.

After clicking one of the buttons, you will notice that the count updates in real time without the page reloading. This is the magic of Livewire: dynamic frontend applications written entirely in PHP.

We've barely scratched the surface of what Livewire is capable of. Keep reading the documentation to see everything Livewire has to offer.
