# Blabe テンプレート

- [はじめに](#introduction)
    - [Livewire による Blade の強化](#supercharge-blade-with-livewire)
- [データの表示](#displaying-data)
    - [HTML エンティティエンコーディング](#html-entity-encoding)
    - [Blade と JavaScript フレームワーク](#blade-and-javascript-frameworks)
- [Blade ディレクティブ](#blade-directives)
    - [if 文](#if-statements)
    - [Switch 文](#switch-statements)
    - [繰り返し](#loops)
    - [ループ変数](#the-loop-variable)
    - [条件付きクラスとスタイル](#conditional-classes)
    - [その他の属性](#Additional-attributes)
    - [サブビューの読み込み](#include-subviews)
    - [`@once` ディレクティブ](#the-once-directive)
    - [素の PHP](#raw-php)
    - [コメント](#comments)
- [コンポーネント](#components)
    - [コンポーネントのレンダリング](#rendering-components)
    - [コンポーネントにデータを渡す](#passing-data-to-components)
    - [コンポーネント属性](#component-attributes)
    - [予約語](#reserved-keywords)
    - [スロット](#slots)
    - [インラインコンポーネントビュー](#inline-component-views)
    - [動的コンポーネント](#dynamic-components)
    - [コンポーネントを手動で登録](#manually-registering-components)
- [匿名コンポーネント](#anonymous-components)
    - [匿名インデックスコンポーネント](#anonymous-index-components)
    - [データプロパティ / 属性](#data-properties-attributes)
    - [親データへのアクセス](#accessing-parent-data)
    - [匿名コンポーネントのパス](#anonymous-component-paths)
- [レイアウト構築](#building-layouts)
    - [コンポーネントを使用したレイアウト](#layouts-using-components)
    - [テンプレート継承を使用したレイアウト](#layouts-using-template-inheritance)
- [フォーム](#forms)
    - [CSRF フィールド](#csrf-field)
    - [Method フィールド](#method-field)
    - [バリデーションエラー](#validation-errors)
- [スタック](#stacks)
- [サービスの注入](#service-injection)
- [インライン Blade  テンプレートのレンダリング](#rendering-inline-blade-templates)
- [Blade フラグメントのレンダリング](#rendering-blade-fragments)
- [Blade の拡張](#extending-blade)
    - [カスタム Echo ハンドラ](#custom-echo-handlers)
    - [カスタム If 文](#custom-if-statements)

<a name="introduction"></a>
## はじめに

Blade は、Laravel に含まれているシンプルかつ強力なテンプレートエンジンです。一部の PHP テンプレートエンジンとは異なり、Blade では、テンプレート内でプレーンな PHP コードの使用を制限しません。実際、すべての Blade テンプレートはプレーンな PHP コードにコンパイルされ、変更されるまでキャッシュされます。つまり、Blade はアプリケーションへ実質的なオーバーヘッドの負荷をかけません。Blade テンプレート ファイルは `.blade.php` ファイル拡張子を使用し、通常は `resources/views` ディレクトリに保存されます。

Blade ビューは、グローバル `view` ヘルパを使用してルート、またはコントローラから返されます。もちろん、[views](/docs/{{version}}/views) のドキュメントで説明されているように、`view` ヘルパの第２引数を使用してデータを Blade ビューに渡すこともできます。

    Route::get('/', function () {
        return view('greeting', ['name' => 'Finn']);
    });

<a name="supercharging-blade-with-livewire"></a>
### Livewire による Blade の強化

Blade テンプレートを次のレベルに引き上げて、動的なインターフェイスを簡単に構築することも可能です。[Laravel Livewire](https://laravel-livewire.com) をチェックしてください。Livewireを使用すると、通常ReactやVueのようなフロントエンドフレームワークでのみ可能な動的機能で拡張されたBladeコンポーネントを書くことができます。JavaScript フレームワークのような複雑さや、クライアントサイドレンダリング、ビルドステップをなしに、モダンでリアクティブなフロントエンドを構築する素晴らしいアプローチを提供します。

<a name="displaying-data"></a>
## データの表示

変数を中括弧で囲むことにより、Blade ビューに渡されるデータを表示できます。たとえば、以下のルートがあるとします。

    Route::get('/', function () {
        return view('welcome', ['name' => 'Samantha']);
    });

次のように `name` 変数の内容を表示できます。

```blade
Hello, {{ $name }}.
```

> **Note**  
> Blade の `{{ }}` エコーステートメントは、XSS 攻撃を防ぐために PHP の `htmlspecialchars` 関数を通じて自動的に送信されます。

ビューに渡された変数の内容を表示することに限定されません。任意の PHP 関数の結果をエコーすることもできます。 実際、Blade のエコーステートメント内に任意の PHP コードを含めることができます。

```blade
The current UNIX timestamp is {{ time() }}.
```

<a name="html-entity-encoding"></a>
### HTML エンティティエンコーディング

デフォルトでは、Blade (および Laravel `e` ヘルパ) は HTML エンティティを二重エンコードします。 二重エンコーディングを無効にしたい場合は、`AppServiceProvider` の `boot` メソッドから `Blade::withoutDoubleEncoding` メソッドを呼び出します。

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Blade;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         */
        public function boot(): void
        {
            Blade::withoutDoubleEncoding();
        }
    }

<a name="displaying-unescaped-data"></a>
#### エスケープしないデータの表示

デフォルトでは、Blade `{{ }}` 文は、XSS 攻撃を防ぐために、PHP の `htmlspecialchars` 関数を通じて自動的に送信されます。データをエスケープしたくない場合は、次の構文を使用します。

```blade
Hello, {!! $name !!}.
```

> **Warning**
> アプリケーションのユーザーが投稿したコンテンツを出力する場合は、十分に注意してください。ユーザーが指定したデータを表示するときに XSS 攻撃を防ぐには、通常、エスケープ効果がある二重中括弧構文を使用してください。

<a name="blade-and-javascript-frameworks"></a>
### Blade と JavaScript フレームワーク

多くの JavaScript フレームワークでも「中括弧」を使用して特定の式をブラウザに表示することを示すために、`@` 記号を使用して式をそのままにしておくように Blade レンダリングエンジンへ通知できます。

```blade
<h1>Laravel</h1>

Hello, @{{ name }}.
```

この例では、`@` 記号が Blade によって削除されます。ただし、`{{ name }}` 式は Blade エンジンによって変更されないため、JavaScript フレームワークによってレンダリングできます。

`@` 記号は、Blade ディレクティブをエスケープするために使用することもできます。

```blade
{{-- Blade template --}}
@@if()

<!-- HTML output -->
@if()
```

<a name="rendering-json"></a>
#### JSON のレンダリング

JavaScript 変数を初期化するために、配列を JSON としてレンダリングする目的でビューに配列を渡すことがあります。以下はその例です。

```blade
<script>
    var app = <?php echo json_encode($array); ?>;
</script>
```

`json_encode` を呼び出す代わりに、`Illuminate\Support\Js::from` メソッドディレクティブも使用できます。`from` メソッドは、PHP の `json_encode` 関数と同じ引数を受け入れます。ただし、取得結果の JSON が HTML クオート内に含められるように適切にエスケープされることが保証されます。`from` メソッドは、指定されたオブジェクトや配列を有効な JavaScript オブジェクトに変換する `JSON.parse` JavaScript 文を文字列として返します。

```blade
<script>
    var app = {{ Illuminate\Support\Js::from($array) }};
</script>
```

最新バージョンの Laravel スケルトンには `Js` ファサードが含まれており、Blade テンプレート内でこの機能に簡単にアクセスできます。

```blade
<script>
    var app = {{ Js::from($array) }};
</script>
```

> **Warning**  
> 既存の変数を JSON としてレンダリングする場合は、`Js::from` メソッドのみを使用してください。Blade テンプレートは正規表現に基づいており、複雑な表現をディレクティブに渡そうとすると、予期しないエラーが発生する可能性があります。

<a name="the-at-verbatim-directive"></a>
#### `@verbatim` ディレクティブ

テンプレートの大部分で JavaScript 変数を表示している場合は、HTML を `@verbatim` ディレクティブでラップすると、各 Blade エコーステートメントの前に `@` 記号を付ける必要がなくなります。

```blade
@verbatim
    <div class="container">
        Hello, {{ name }}.
    </div>
@endverbatim
```

<a name="blade-directives"></a>
## Blade ディレクティブ

Blade は、テンプレートの継承とデータの表示に加えて、条件文やループなどの一般的な PHP 制御構造の便利なショートカットも提供しています。これらのショートカットは、PHP の制御構造を操作するための非常にクリーンで簡潔な方法を提供すると同時に、PHP の利用する人にとって馴染みのあるものです。

<a name="if-statements"></a>
### if 文

`@if`、`@elseif`、`@else`、`@endif` ディレクティブを使用して `if` ステートメントを構築できます。これらのディレクティブは、対応する PHP の構文と同様に機能します。

```blade
@if (count($records) === 1)
    I have one record!
@elseif (count($records) > 1)
    I have multiple records!
@else
    I don't have any records!
@endif
```

便宜上、Blade には `@unless` ディレクティブも用意されています。

```blade
@unless (Auth::check())
    You are not signed in.
@endunless
```

すでに説明した条件付きディレクティブに加えて、`@isset` および `@empty` ディレクティブは、それぞれの PHP 関数の便利なショートカットとして使用できます。

```blade
@isset($records)
    // $records is defined and is not null...
@endisset

@empty($records)
    // $records is "empty"...
@endempty
```

<a name="authentication-directives"></a>
#### 認証ディレクティブ

`@auth` および `@guest` ディレクティブを使用すると、現在のユーザーが [認証済み](/docs/{{version}}/authentication) であるか、またはゲストであるかをすぐに判断できます。

```blade
@auth
    // The user is authenticated...
@endauth

@guest
    // The user is not authenticated...
@endguest
```

必要に応じて、`@auth` および `@guest` ディレクティブを使用するときにチェックする必要がある認証ガードを指定できます。

```blade
@auth('admin')
    // The user is authenticated...
@endauth

@guest('admin')
    // The user is not authenticated...
@endguest
```

<a name="environment-directives"></a>
#### 環境ディレクティブ

`@production` ディレクティブを使用して、アプリケーションが実稼働環境で実行されているかどうかを確認できます。

```blade
@production
    // Production specific content...
@endproduction
```

または、`@env` ディレクティブを使用して、アプリケーションが特定の環境で実行されているかどうかを判断することもできます。

```blade
@env('staging')
    // The application is running in "staging"...
@endenv

@env(['staging', 'production'])
    // The application is running in "staging" or "production"...
@endenv
```

<a name="section-directives"></a>
#### セクションディレクティブ

`@hasSection` ディレクティブを使用して、テンプレート継承セクションにコンテンツがあるかどうかを判断できます。

```blade
@hasSection('navigation')
    <div class="pull-right">
        @yield('navigation')
    </div>

    <div class="clearfix"></div>
@endif
```

`sectionMissing` ディレクティブを使用して、セクションにコンテンツがないかどうかを判断できます。

```blade
@sectionMissing('navigation')
    <div class="pull-right">
        @include('default-navigation')
    </div>
@endif
```

<a name="switch-statements"></a>
### switch 文

switch ステートメントは、`@switch`、`@case`、`@break`、`@default`、`@endswitch` ディレクティブを使用して構築できます。

```blade
@switch($i)
    @case(1)
        First case...
        @break

    @case(2)
        Second case...
        @break

    @default
        Default case...
@endswitch
```

<a name="loops"></a>
### 繰り返し

条件文に加えて、Blade は PHP のループ構造を操作するための単純なディレクティブを提供します。繰り返しますが、これらの各ディレクティブは、対応する PHP 構文と同様に機能します。

```blade
@for ($i = 0; $i < 10; $i++)
    The current value is {{ $i }}
@endfor

@foreach ($users as $user)
    <p>This is user {{ $user->id }}</p>
@endforeach

@forelse ($users as $user)
    <li>{{ $user->name }}</li>
@empty
    <p>No users</p>
@endforelse

@while (true)
    <p>I'm looping forever.</p>
@endwhile
```

> **Note**  
> `foreach` ループを反復しているときに、[ループ変数](#the-loop-variable) を使用して、ループの最初の反復にいるのか最後の反復にいるのかなど、ループに役立つ貴重な情報を取得できます。

ループを使用する場合、`@ continue` および ` @break` ディレクティブを使用して、現在の反復をスキップしたり、ループを終了したりすることもできます。

```blade
@foreach ($users as $user)
    @if ($user->type == 1)
        @continue
    @endif

    <li>{{ $user->name }}</li>

    @if ($user->number == 5)
        @break
    @endif
@endforeach
```

ディレクティブ宣言内に継続条件または中断条件を含めることもできます。

```blade
@foreach ($users as $user)
    @continue($user->type == 1)

    <li>{{ $user->name }}</li>

    @break($user->number == 5)
@endforeach
```

<a name="the-loop-variable"></a>
### ループ変数

`foreach` ループを反復処理している間は、ループ内で `$loop` 変数が使用可能になります。この変数は、現在のループのインデックスや、ループの最初の反復であるか、最後の反復であるかなど、いくつかの有用な情報へのアクセスを提供します。

```blade
@foreach ($users as $user)
    @if ($loop->first)
        This is the first iteration.
    @endif

    @if ($loop->last)
        This is the last iteration.
    @endif

    <p>This is user {{ $user->id }}</p>
@endforeach
```

ネストされたループにいる場合は、`parent` プロパティを介して親ループの `$loop` 変数にアクセスできます。

```blade
@foreach ($users as $user)
    @foreach ($user->posts as $post)
        @if ($loop->parent->first)
            This is the first iteration of the parent loop.
        @endif
    @endforeach
@endforeach
```

`$loop` 変数には、他にも便利なプロパティが含まれています。

| プロパティ | 説明 |
|-----------------|---------------------------- ----------------------------|
| `$loop->index` | 現在の反復のインデックス (初期値０) |
| `$loop->iteration` | 現在の反復数 (初期値１)。 |
| `$loop->remaining` | 反復の残数 |
| `$loop->count` | 反復している配列内の項目の合計 |
| `$loop->first` | ループの最初の繰り返しか判定 |
| `$loop->last` | ループの最後の繰り返しか判定 |
| `$loop->even` | 現在のループが偶数回目の繰り返しか判定 |
| `$loop->odd` | 現在のループが奇数の反復であるかどうか。 |
| `$loop->depth` | 現在のループのネストレベル |
| `$loop->parent` | ネストしたループ内の場合、親のループ変数 |

<a name="conditional-classes"></a>
### 条件付きクラスとスタイル

`@class` ディレクティブは、CSS のクラス文字列を条件付きでコンパイルします。このディレクティブは 追加したい CSS のクラス名を指定して受け入れます。配列のキーには追加したいクラスを指定し、値には真偽値を指定します。配列のキーが数値の場合は、常にレンダリングされるクラスリストに含まれます。

```blade
@php
    $isActive = false;
    $hasError = true;
@endphp

<span @class([
    'p-4',
    'font-bold' => $isActive,
    'text-gray-500' => ! $isActive,
    'bg-red' => $hasError,
])></span>

<span class="p-4 text-gray-500 bg-red"></span>
```

`@style` ディレクティブを使用して、条件付きで HTML 要素にインラインの CSS スタイルを追加できます。

```blade
@php
    $isActive = true;
@endphp

<span @style([
    'background-color: red',
    'font-weight: bold' => $isActive,
])></span>

<span style="background-color: red; font-weight: bold;"></span>
```

<a name="additional-attributes"></a>
### その他の属性

`@checked` ディレクティブを使用すると、指定した HTMLの チェックボックス入力が「checked」かどうかを簡単に判定できます。このディレクティブは、引数内で指定された条件が `true` と評価された場合に `checked` をエコー出力します。

```blade
<input type="checkbox"
        name="active"
        value="active"
        @checked(old('active', $user->active)) />
```

同様に、`@selected` ディレクティブを使用して、指定のセレクトオプションが 「selected」 であるかを判定し、`true` と評価された場合に `selected` をエコー出力します。

```blade
<select name="version">
    @foreach ($product->versions as $version)
        <option value="{{ $version }}" @selected(old('version') == $version)>
            {{ $version }}
        </option>
    @endforeach
</select>
```

さらに、`@disabled` ディレクティブを使用して、特定の要素が「disabled」にするかどうかを示すことができます。

```blade
<button type="submit" @disabled($errors->isNotEmpty())>Submit</button>
```

さらに、`@readonly` ディレクティブを使用して、特定の要素を「readonly」にするかどうかを示すことができます。

```blade
<input type="email"
        name="email"
        value="email@laravel.com"
        @readonly($user->isNotAdmin()) />
```

さらに、`@required` ディレクティブを使用して、特定の要素を「required」にするかどうかを示すことができます。

```blade
<input type="text"
        name="title"
        value="title"
        @required($user->isAdmin()) />
```

<a name="include-subviews"></a>
### サブビューの読み込み

> **Note**  
> `@include` ディレクティブは自由に使用できますが、Blade [コンポーネント](#components) は同様の機能を提供しながら、データや属性のバインディングなど、`@include` ディレクティブよりも優れたいくつかの利点があります。

Blade の `@include` ディレクティブを使用すると、別のビュー内から Blade ビューを読み込めます。親ビューで使用できるすべての変数は、読み込まれたビューでも使用できるようになります。

```blade
<div>
    @include('shared.errors')

    <form>
        <!-- Form Contents -->
    </form>
</div>
```

読み込むビューは親ビューで使用できるすべてのデータを継承しますが、読み込むビューで利用できる追加データの配列を渡すこともできます。

```blade
@include('view.name', ['status' => 'complete'])
```

存在しないビューを `@include` しようとすると、Laravel はエラーをスローします。存在するかどうかわからないビューを含めたい場合は、`@includeIf` ディレクティブを使用する必要があります。

```blade
@includeIf('view.name', ['status' => 'complete'])
```

指定した論理値 `true` または `false` と評価された場合に、ビューを `@include` したい際は、`@includeWhen` および `@includeUnless` ディレクティブを使用できます。

```blade
@includeWhen($boolean, 'view.name', ['status' => 'complete'])

@includeUnless($boolean, 'view.name', ['status' => 'complete'])
```

指定したビューの配列から、存在する最初のビューを含めるには、`includeFirst` ディレクティブを使用します。

```blade
@includeFirst(['custom.admin', 'admin'], ['status' => 'complete'])
```

> **Warning**  
> Blade ビューでは `__DIR__` および `__FILE__` 定数を使用しないでください。キャッシュされコンパイルされたビューの場所を参照するためです。

<a name="rendering-views-for-collections"></a>
#### コレクションのビューのレンダリング

Blade の `@each` ディレクティブを使用して、ループと読み込みをコード１行で書くことが可能です。

```blade
@each('view.name', $jobs, 'job')
```

`@each` ディレクティブの第１引数は、配列またはコレクション内の各要素に対してレンダリングするビューです。第２引数は反復処理する配列またはコレクションで、第３引数はビュー内の現在の反復に割り当てられる変数名です。したがって、たとえば、`jobs` の配列を反復処理している場合、通常はビュー内の `job` 変数として各ジョブにアクセスする必要があります。現在の反復の配列キーは、ビュー内の `key` 変数として使用できます。

`@each` ディレクティブに第４引数を渡すこともできます。この引数は、指定された配列が空の場合にレンダリングされるビューを決定します。

```blade
@each('view.name', $jobs, 'job', 'view.empty')
```

> **Warning**  
> `@each` を介してレンダリングされたビューは、親ビューから変数を継承しません。子ビューでこれらの変数が必要な場合は、代わりに `@foreach` および `@include` ディレクティブを使用する必要があります。

<a name="the-once-directive"></a>
### `@once` ディレクティブ

`@once` ディレクティブを使用すると、レンダリングサイクルごとに１回だけ評価されるテンプレートの一部を定義できます。これは、[stacks](#stacks) を使用して、特定の JavaScript をページのヘッダに組み入れる場合に便利です。たとえば、ループ内で特定の [コンポーネント](#components) をレンダリングする場合、コンポーネントが初めてレンダリングされるときにのみ JavaScript をヘッダに組み入れしたい場合があります。

```blade
@once
    @push('scripts')
        <script>
            // Your custom JavaScript...
        </script>
    @endpush
@endonce
```

`@once` ディレクティブは、`@push` や `@prepend` ディレクティブと組み合わせて使用されることが多いため、使いやすいように `@pushOnce` および `@prependOnce` ディレクティブを利用すると良いでしょう。

```blade
@pushOnce('scripts')
    <script>
        // Your custom JavaScript...
    </script>
@endPushOnce
```

<a name="raw-php"></a>
### 素の PHP

状況によっては、PHP コードをビューに埋め込むと便利です。Blade の `@php` ディレクティブを使用して、テンプレート内のプレーン PHP のブロックを実行できます。

```blade
@php
    $counter = 1;
@endphp
```

PHP 文 １つのみを記述する場合は、`@php` ディレクティブ内に含めることができます。

```blade
@php($counter = 1)
```

<a name="comments"></a>
### コメント

Blade では、ビュー内にコメントを定義することもできます。ただし、HTML コメントとは異なり、Blade コメントはアプリケーションから返される HTML には含まれません。

```blade
{{-- This comment will not be present in the rendered HTML --}}
```

<a name="components"></a>
## コンポーネント

コンポーネントとスロットは、セクション、レイアウト、インクルードに同様の利点をもたらします。ただし、コンポーネントとスロットのメンタルモデルの方が理解しやすいと感じる人もいるかもしれません。コンポーネントを作成するには、クラスベースのコンポーネントと匿名コンポーネントの２つのアプローチがあります。

クラスベースのコンポーネントを作成するには、`make:component` Artisan コマンドを使用します。ここで単純な `Alert` コンポーネントを作成しましょう。`make:component` コマンドは、コンポーネントを `app/View/Components` ディレクトリに配置します。

```shell
php artisan make:component Alert
```

`make:component` コマンドは、コンポーネントのビューテンプレートも作成します。ビューは `resources/views/components` ディレクトリに配置されます。独自のアプリケーション用のコンポーネントを作成する場合、コンポーネントは `app/View/Components` ディレクトリおよび `resources/views/components` ディレクトリ内で自動的に検出されるため、通常は追加のコンポーネント登録は必要ありません。

サブディレクトリ内にコンポーネントを作成することもできます。

```shell
php artisan make:component Forms/Input
```

上記のコマンドは、`app/View/Components/Forms` ディレクトリに `Input` コンポーネントを作成し、ビューは `resources/views/components/forms` ディレクトリに配置されます。

匿名コンポーネント (Blade テンプレートのみでクラスを持たないコンポーネント) を作成したい場合は、`make:component` コマンドを呼び出すときに `--view` フラグを使用します。

```shell
php artisan make:component forms.input --view
```

上記のコマンドは、`resources/views/components/forms/input.blade.php` に Blade ファイルを作成します。これは、`<x-forms.input />` という記述によって、コンポーネントとしてレンダリングされます。

<a name="manually-registering-package-components"></a>
#### パッケージコンポーネントの手動登録

独自のアプリケーションのコンポーネントを作成する場合、コンポーネントは `app/View/Components` ディレクトリと `resources/views/components` ディレクトリ内で自動的に検出されます。

ただし、Blade コンポーネントを利用するパッケージを構築している場合は、コンポーネントクラスとその HTML タグ エイリアスを手動で登録する必要があります。 通常、コンポーネントはパッケージのサービスプロバイダの`boot`メソッドに登録する必要があります。

    use Illuminate\Support\Facades\Blade;

    /**
     * Bootstrap your package's services.
     */
    public function boot(): void
    {
        Blade::component('package-alert', Alert::class);
    }

コンポーネントを登録すると、タグエイリアスを使用してレンダリングできます。

```blade
<x-package-alert/>
```

あるいは、規約により `componentNamespace` メソッドを使用してコンポーネントクラスを自動ロードすることもできます。たとえば、`Nightshade` パッケージには、`Package\Views\Components` 名前空間内に存在する `Calendar` コンポーネントと `ColorPicker` コンポーネントが含まれているとします。

    use Illuminate\Support\Facades\Blade;

    /**
     * Bootstrap your package's services.
     */
    public function boot(): void
    {
        Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
    }

これにより、ベンダー名前空間で `package-name::` 構文を使用してパッケージコンポーネントを使用できるようになります。

```blade
<x-nightshade::calendar />
<x-nightshade::color-picker />
```

Blade は、コンポーネント名をパスカル文字に変換することで、このコンポーネントにリンクされているクラスを自動的に検出します。サブディレクトリもサポートしており「ドット」表記を使用します。

<a name="rendering-components"></a>
### コンポーネントのレンダリング

コンポーネントを表示するには、Blade テンプレートの１つの中で Blade コンポーネントタグを使用できます。Blade コンポーネントのタグは文字列 `x-` で始まり、その後にコンポーネントクラスのケバブケース名が続きます。

```blade
<x-alert/>

<x-user-profile/>
```

コンポーネントクラスが `app/View/Components` ディレクトリ内でさらに深くネストされている場合は、ディレクトリのネストを示すために `.` 文字を使用できます。たとえば、コンポーネントが `app/View/Components/Inputs/Button.php` にあるとすると、次のようにレンダリングできます。

```blade
<x-inputs.button/>
```

コンポーネントを条件付きでレンダリングしたい場合は、コンポーネントクラスで `shouldRender` メソッドを定義してください。`shouldRender` メソッドが `false` を返した場合、コンポーネントはレンダリングされません。

    use Illuminate\Support\Str;

    /**
     * Whether the component should be rendered
     */
    public function shouldRender(): bool
    {
        return Str::length($this->message) > 0;
    }

<a name="passing-data-to-components"></a>
### コンポーネントにデータを渡す

HTML 属性を使用してデータを Blade コンポーネントに渡せます。ハードコーディングされたプリミティブ値は、単純な HTML 属性文字列を使用してコンポーネントに渡すことができます。PHP の式と変数は、`:` 文字をプレフィックスとして使用する属性を介してコンポーネントに渡す必要があります。

```blade
<x-alert type="error" :message="$message"/>
```

コンポーネントの全データ属性を、そのクラスコンストラクタで定義する必要があります。コンポーネント上のすべてのパブリックプロパティは、コンポーネントのビューで自動的に利用できます。コンポーネントの `render` メソッドからビューにデータを渡す必要はありません。

    <?php

    namespace App\View\Components;

    use Illuminate\View\Component;
    use Illuminate\View\View;

    class Alert extends Component
    {
        /**
         * Create the component instance.
         */
        public function __construct(
            public string $type,
            public string $message,
        ) {}

        /**
         * Get the view / contents that represent the component.
         */
        public function render(): View
        {
            return view('components.alert');
        }
    }

コンポーネントがレンダリングされるとき、変数を名前でエコーすることによって、コンポーネントのパブリック変数の内容を表示できます。

```blade
<div class="alert alert-{{ $type }}">
    {{ $message }}
</div>
```

<a name="casing"></a>
#### ケース形式

コンポーネントのコンストラクタの引数はキャメルケース（ `camelCase` ）を使用して指定する必要がありますが、HTML 属性で引数名を参照する場合はケバブケース（ `kebab-case` ）を使用する必要があります。たとえば、次のコンポーネントのコンストラクタがあるとします。

    /**
     * Create the component instance.
     */
    public function __construct(
        public string $alertType,
    ) {}

`$alertType` 引数は次のようにコンポーネントに提供できます。

```blade
<x-alert alert-type="danger" />
```

<a name="short-attribute-syntax"></a>
#### 属性の短縮構文

コンポーネントに属性を渡すときは、「短縮」構文を使用できます。属性名は、対応する変数名と一致することが多いため、便利な手法です。

```blade
{{-- Short attribute syntax... --}}
<x-profile :$userId :$name />

{{-- Is equivalent to... --}}
<x-profile :user-id="$userId" :name="$name" />
```

<a name="escaping-attribute-rendering"></a>
#### 属性レンダリングのエスケープ

Alpine.js などの一部の JavaScript フレームワークもコロン接頭辞付きの属性を使用するため、二重コロン (`::`) 接頭辞を使用して属性が PHP 式ではないことを Blade に通知できます。たとえば、次のコンポーネントがあるとします。

```blade
<x-button ::class="{ danger: isDeleting }">
    Submit
</x-button>
```

次の HTML が Blade によってレンダリングされます。

```blade
<button :class="{ danger: isDeleting }">
    Submit
</button>
```

<a name="component-methods"></a>
#### コンポーネントメソッド

コンポーネントテンプレートで使用できるパブリック変数に加えて、コンポーネント上の任意のパブリックメソッドを呼び出すことができます。たとえば、`isSelected`メソッドを持つコンポーネントを想像してください。

    /**
     * Determine if the given option is the currently selected option.
     */
    public function isSelected(string $option): bool
    {
        return $option === $this->selected;
    }

メソッドの名前に一致する変数を呼び出すことで、コンポーネントテンプレートからこのメソッドを実行できます。

```blade
<option {{ $isSelected($value) ? 'selected' : '' }} value="{{ $value }}">
    {{ $label }}
</option>
```

<a name="using-attributes-slots-within-component-class"></a>
#### コンポーネントクラス内の属性とスロットへのアクセス

Blade コンポーネントを使用すると、クラスの render メソッド内のコンポーネント名、属性、スロットにアクセスすることもできます。ただし、このデータにアクセスするには、コンポーネントの `render` メソッドからクロージャを返す必要があります。クロージャは、唯一の引数として `$data` 配列を受け取ります。この配列には、コンポーネントに関する情報を提供するいくつかの要素が含まれます。

    use Closure;

    /**
     * Get the view / contents that represent the component.
     */
    public function render(): Closure
    {
        return function (array $data) {
            // $data['componentName'];
            // $data['attributes'];
            // $data['slot'];

            return '<div>Components content</div>';
        };
    }

`componentName` は、HTML タグ内で `x-` プレフィックスの後に使用される名前と等しいです。したがって、`<x-alert />` の `componentName` は `alert` になります。`attributes` 要素には、HTML タグに存在するすべての属性が含まれます。`slot` 要素は、コンポーネントのスロットの内容を含む `Illuminate\Support\HtmlString` インスタンスです。

クロージャは文字列を返す必要があります。返された文字列が既存のビューに対応する場合、そのビューがレンダリングされます。それ以外の場合、返された文字列はインライン Blade ビューとして評価されます。

<a name="additional-dependencies"></a>
#### 追加の依存関係

コンポーネントが Laravel の [サービスコンテナ](/docs/{{version}}/container) からの依存関係を必要とする場合、コンポーネントのデータ属性の前にそれらをリストすると、それらはコンテナによって自動的に注入されます。

```php
use App\Services\AlertCreator;

/**
 * Create the component instance.
 */
public function __construct(
    public AlertCreator $creator,
    public string $type,
    public string $message,
) {}
```

<a name="hiding-attributes-and-methods"></a>
#### 非表示属性 / メソッドの非表示

一部のパブリックメソッドまたはプロパティが変数としてコンポーネントテンプレートに公開されるのを防ぎたい場合は、それらをコンポーネントの `$except` 配列プロパティに追加します。

    <?php

    namespace App\View\Components;

    use Illuminate\View\Component;

    class Alert extends Component
    {
        /**
         * The properties / methods that should not be exposed to the component template.
         *
         * @var array
         */
        protected $except = ['type'];

        /**
         * Create the component instance.
         */
        public function __construct(
            public string $type,
        ) {}
    }

<a name="component-attributes"></a>
### コンポーネント属性

データ属性をコンポーネントに渡す方法については既に説明しました。しかし、`class` など、コンポーネントが機能するために必要なデータの一部ではない **追加のHTML属性** を指定する必要がある場合もあります。通常、これらの追加属性はコンポーネントテンプレートのルート要素に渡します。たとえば、次のように `alert` コンポーネントを表示したいとします。

```blade
<x-alert type="error" :message="$message" class="mt-4"/>
```

コンポーネントのコンストラクタの一部ではないすべての属性は、コンポーネントの「属性バッグ」に自動的に追加されます。この属性バッグは、`$attributes` 変数を介してコンポーネントで自動的に利用できるようになります。この変数をエコーすることで、すべての属性をコンポーネント内でレンダリングできます。

```blade
<div {{ $attributes }}>
    <!-- Component content -->
</div>
```

> **Warning**  
> 現時点では、コンポーネントタグ内での `@env` などのディレクティブの使用はサポートされていません。たとえば、`<x-alert :live="@env('production')"/>` はコンパイルされません。

<a name="default-merged-attributes"></a>
#### デフォルト / マージした属性

属性のデフォルト値を指定したり、コンポーネントの属性の一部に追加の値をマージしたりすることが必要な場合があります。これを実現するには、属性バッグの `merge` メソッドを使用します。このメソッドは、コンポーネントに常に適用する必要がある一連のデフォルト CSS クラスを定義する場合に特に役立ちます。

```blade
<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    {{ $message }}
</div>
```

このコンポーネントが以下のように利用されると仮定します。

```blade
<x-alert type="error" :message="$message" class="mb-4"/>
```

コンポーネントが最終的にレンダリングされた HTML は次のように表示されます。

```blade
<div class="alert alert-error mb-4">
    <!-- Contents of the $message variable -->
</div>
```

<a name="conditionally-merge-classes"></a>
#### 条件付きでクラスをマージ

特定の条件で `true` の場合にクラスをマージしたい際は `class` メソッドを使用して実行できます。このメソッドは引数に配列を受け取り、追加したいクラスを配列キーに、また値に論理式を指定します。配列要素に数値キーがある場合、その要素は常に表示されるクラスとなります。

```blade
<div {{ $attributes->class(['p-4', 'bg-red' => $hasError]) }}>
    {{ $message }}
</div>
```

他の属性をコンポーネントにマージする必要がある場合は、`merge` メソッドを `class` メソッドにチェーンさせることができます。

```blade
<button {{ $attributes->class(['p-4'])->merge(['type' => 'button']) }}>
    {{ $slot }}
</button>
```

> **Note**  
> マージした属性を受け取るべきでない他の HTML 要素のクラスを条件付きでコンパイルする必要がある場合は、[`@class` ディレクティブ](#conditional-classes) を使用してください。

<a name="non-class-attribute-merging"></a>
#### 非クラス属性のマージ

`class` 属性ではない属性をマージする場合、`merge` メソッドに指定された値は属性の「デフォルト」値とみなされます。ただし、`class` 属性とは異なり、これらの属性は挿入された属性値とマージされません。代わりに、上書きされます。たとえば、`button` コンポーネントの実装は次のようになります。

```blade
<button {{ $attributes->merge(['type' => 'button']) }}>
    {{ $slot }}
</button>
```

カスタム `type` を指定してボタンコンポーネントをレンダリングするには、コンポーネントを使用するときにそれを指定できます。タイプが指定されていない場合は、`button` タイプが使用されます。

```blade
<x-button type="submit">
    Submit
</x-button>
```

この例の `button` コンポーネントのレンダリングされた HTML は次のようになります。

```blade
<button type="submit">
    Submit
</button>
```

`class` 以外の属性にデフォルト値と挿入された値を結合させたい場合は、`prepends` メソッドを使用します。この例では、`data-controller` 属性は常に `profile-controller` で始まり、追加で挿入された `data-controller` 値はデフォルト値の後に配置されます。

```blade
<div {{ $attributes->merge(['data-controller' => $attributes->prepends('profile-controller')]) }}>
    {{ $slot }}
</div>
```

<a name="filtering-attributes"></a>
#### 属性の取得とフィルタリング

`filter` メソッドを使用して属性をフィルタリングできます。このメソッドは、属性バッグへ属性を保持したい際に `true` を返すクロージャを引数に取ります。

```blade
{{ $attributes->filter(fn (string $value, string $key) => $key == 'foo') }}
```

`whereStartsWith` メソッドを使用して、キーが指定された文字列で始まるすべての属性を取得できます。

```blade
{{ $attributes->whereStartsWith('wire:model') }}
```

逆に、`whereDoesntStartWith` メソッドを使用して、キーが指定された文字列で始まるすべての属性を除外することもできます。

```blade
{{ $attributes->whereDoesntStartWith('wire:model') }}
```

`first` メソッドを使用すると、指定された属性バッグ内の最初の属性をレンダリングできます。

```blade
{{ $attributes->whereStartsWith('wire:model')->first() }}
```

コンポーネントに属性が存在するかどうかを確認したい場合は、`has` メソッドを使用します。このメソッドは、属性名を唯一の引数として受け入れ、属性が存在するかどうかを示す論理値を返します。

```blade
@if ($attributes->has('class'))
    <div>Class attribute is present</div>
@endif
```

`get` メソッドを使用して特定の属性の値を取得できます。

```blade
{{ $attributes->get('class') }}
```

<a name="reserved-keywords"></a>
### 予約語

コンポーネントをレンダリングする Blade の内部使用のため、一部のキーワードは予約語として登録されています。次のキーワードは、コンポーネント内のパブリックプロパティまたはメソッド名として定義できません。

<div class="content-list" markdown="1">

- `data`
- `render`
- `resolveView`
- `shouldRender`
- `view`
- `withAttributes`
- `withName`

</div>

<a name="slots"></a>
### スロット

多くの場合、追加のコンテンツを「スロット」経由でコンポーネントに渡す必要があります。コンポーネントスロットは、`$slot` 変数をエコーしてレンダリングされます。この概念を詳しく調べるために、`alert` コンポーネントに次のマークアップがあると仮定してみましょう。

```blade
<!-- /resources/views/components/alert.blade.php -->

<div class="alert alert-danger">
    {{ $slot }}
</div>
```

コンポーネントにコンテンツを挿入することで、コンテンツを `slot` に渡すことができます。

```blade
<x-alert>
    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

コンポーネント内の異なる場所に複数の異なるスロットをレンダリングする必要がある場合があります。「タイトル（title）」スロットを挿入できるようにアラートコンポーネントを変更しましょう。

```blade
<!-- /resources/views/components/alert.blade.php -->

<span class="alert-title">{{ $title }}</span>

<div class="alert alert-danger">
    {{ $slot }}
</div>
```

`x-slot` タグを使用して、名前付きスロットの内容を定義できます。明示的な `x-slot` タグ内にないコンテンツは、`$slot` 変数のコンポーネントに渡されます。

```xml
<x-alert>
    <x-slot:title>
        Server Error
    </x-slot>

    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

<a name="scoped-slots"></a>
#### スコープ付きスロット

Vue などの JavaScript フレームワークを使用したことがある場合は、スロット内のコンポーネントからデータまたはメソッドにアクセスできる「スコープ付きスロット」に慣れているかもしれません。コンポーネント上でパブリックメソッドまたはプロパティを定義し、`$component` 変数を介してスロット内のコンポーネントにアクセスすることで、Laravel でも同様の動作を実現できます。この例では、`x-alert`コンポーネントのコンポーネントクラスにパブリックの `formatAlert` メソッドが定義されていると仮定します。

```blade
<x-alert>
    <x-slot:title>
        {{ $component->formatAlert('Server Error') }}
    </x-slot>

    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

<a name="slot-attributes"></a>
#### スロットの属性

Blade コンポーネントと同様に、CSS クラス名などのスロットに追加の [属性](#component-attributes) を割り当てることができます。

```xml
<x-card class="shadow-sm">
    <x-slot:heading class="font-bold">
        Heading
    </x-slot>

    Content

    <x-slot:footer class="text-sm">
        Footer
    </x-slot>
</x-card>
```

スロット属性を操作するには、スロットの変数の `attributes` プロパティにアクセスします。属性を操作する方法の詳細については、[コンポーネント属性](#component-attributes) に関するドキュメントを参照してください。

```blade
@props([
    'heading',
    'footer',
])

<div {{ $attributes->class(['border']) }}>
    <h1 {{ $heading->attributes->class(['text-lg']) }}>
        {{ $heading }}
    </h1>

    {{ $slot }}

    <footer {{ $footer->attributes->class(['text-gray-700']) }}>
        {{ $footer }}
    </footer>
</div>
```

<a name="inline-component-views"></a>
### インラインコンポーネントビュー

非常に小さなコンポーネントの場合、コンポーネントのクラスとビューテンプレート、両方を管理するのが面倒に感じる場合があります。このため、コンポーネントのマークアップを `render` メソッドから直接返すことができます。

    /**
     * Get the view / contents that represent the component.
     */
    public function render(): string
    {
        return <<<'blade'
            <div class="alert alert-danger">
                {{ $slot }}
            </div>
        blade;
    }

<a name="generate-inline-view-components"></a>
#### インラインビューコンポーネントの生成

インラインビューをレンダリングするコンポーネントを作成するには、`make:component` コマンドを実行するときに `inline` オプションを使用できます。

```shell
php artisan make:component Alert --inline
```

<a name="dynamic-components"></a>
### 動的コンポーネント

コンポーネントをレンダリングする必要があるが、実行時までどのコンポーネントをレンダリングすべきかわからない場合があります。この状況では、Laravel の組み込みコンポーネントの `dynamic-component` を使用して、実行時の値または変数に基づいてコンポーネントをレンダリングできます。

```blade
<x-dynamic-component :component="$componentName" class="mt-4" />
```

<a name="manually-registering-components"></a>
### コンポーネントを手動で登録

> **Warning**  
> コンポーネントの手動登録に関する以下のドキュメントは、主にビューコンポーネントを含む Laravel パッケージの開発者に当てはまります。パッケージを作成していない場合、この部分のドキュメントは関係ないかもしれません。

独自のアプリケーションのコンポーネントを作成する場合、コンポーネントは `app/View/Components` ディレクトリおよび `resources/views/components` ディレクトリ内で自動的に検出されます。

ただし、Blade コンポーネントを利用するパッケージを構築する場合、またはコンポーネントを従来とは異なるディレクトリに配置する場合は、Laravel がコンポーネントの場所を認識できるように、コンポーネントクラスとその HTML タグのエイリアスを手動で登録する必要があります。通常、コンポーネントはパッケージのサービスプロバイダの `boot` メソッドに登録する必要があります。

    use Illuminate\Support\Facades\Blade;
    use VendorPackage\View\Components\AlertComponent;

    /**
     * Bootstrap your package's services.
     */
    public function boot(): void
    {
        Blade::component('package-alert', AlertComponent::class);
    }

コンポーネントを登録すると、そのタグエイリアスを使用してレンダリングできます。

```blade
<x-package-alert/>
```

#### パッケージコンポーネントの自動ロード

規約に従って `componentNamespace` メソッドを使用してコンポーネントクラスを自動ロードすることもできます。たとえば、`Nightshade` パッケージには、`Package\Views\Components` 名前空間内に存在する `Calendar` コンポーネントと `ColorPicker` コンポーネントが含まれているとしましょう。

    use Illuminate\Support\Facades\Blade;

    /**
     * Bootstrap your package's services.
     */
    public function boot(): void
    {
        Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
    }

これにより、ベンダー名前空間で `package-name::` 構文を利用でき、パッケージコンポーネントを使用できるようになります。

```blade
<x-nightshade::calendar />
<x-nightshade::color-picker />
```

Blade は、コンポーネント名をパスカルケースに変換することで、このコンポーネントにリンクされているクラスを自動的に検出します。サブディレクトリは、「ドット」表記をサポートしています。

<a name="anonymous-components"></a>
## 匿名コンポーネント

インラインコンポーネントと同様に、匿名コンポーネントは、単一のファイルを介してコンポーネントを管理するメカニズムを提供します。ただし、匿名コンポーネントは単一のビューファイルを使用し、関連するクラスを持ちません。匿名コンポーネントを定義するには、`resources/views/components` ディレクトリ内に Blade テンプレートを配置するだけです。たとえば、`resources/views/components/alert.blade.php` でコンポーネントを定義したと仮定すると、次のようにレンダリングできます。

```blade
<x-alert/>
```

`.` 文字を使用して、コンポーネントが `components` ディレクトリのさらに深くにネストされているかを示すことができます。たとえば、コンポーネントが `resources/views/components/inputs/button.blade.php` で定義されていると仮定すると、次のようにレンダリングできます。

```blade
<x-inputs.button/>
```

<a name="anonymous-index-components"></a>
### 匿名インデックスコンポーネント

あるコンポーネントが多数の Blade テンプレートで構成されている場合、そのコンポーネントのテンプレートを１つのディレクトリ内にグループ化したい場合があります。たとえば、次のディレクトリ構造を持つ「アコーディオン」コンポーネントがあるとします。

```none
/resources/views/components/accordion.blade.php
/resources/views/components/accordion/item.blade.php
```

このディレクトリ構造により、アコーディオンコンポーネントとその項目を次のようにレンダリングできます。

```blade
<x-accordion>
    <x-accordion.item>
        ...
    </x-accordion.item>
</x-accordion>
```

しかし、`x-accordion` を介してアコーディオンコンポーネントをレンダリングするために、「index」アコーディオンコンポーネントテンプレートを、他のアコーディオン関連テンプレートと共に `accordion` ディレクトリ内に入れ子にするのではなく、`resources/views/components` ディレクトリ内に配置する必要がありました。

ありがたいことに、Blade ではコンポーネントのテンプレートディレクトリ内に `index.blade.php` ファイルを配置できます。コンポーネントに `index.blade.php` テンプレートが存在する場合、それはコンポーネントの「ルート」ノードとしてレンダリングされます。したがって、上記の例で示したのと同じ Blade 構文を引き続き使用できます。 ただし、ディレクトリ構造は次のように調整します。

```none
/resources/views/components/accordion/index.blade.php
/resources/views/components/accordion/item.blade.php
```

<a name="data-properties-attributes"></a>
### データプロパティ / 属性

匿名コンポーネントには関連付けられたクラスがないため、どのデータを変数としてコンポーネントに渡す必要があるのか、またどの属性をコンポーネントの [属性バッグ](#component-attributes) に配置する必要があるのかをどのように区別すればよいのか疑問に思うかもしれません。

コンポーネントの Blade テンプレートの先頭にある `@props` ディレクティブを使用して、どの属性をデータ変数と見なすかを指定できます。コンポーネントの他のすべての属性は、コンポーネントの属性バッグを介して利用可能になります。データ変数にデフォルト値を与えたい場合は、変数の名前を配列キーとして指定し、デフォルト値を配列値として指定できます。

```blade
<!-- /resources/views/components/alert.blade.php -->

@props(['type' => 'info', 'message'])

<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    {{ $message }}
</div>
```

上記のコンポーネント定義をすると、次のようにコンポーネントをレンダリングできます。

```blade
<x-alert type="error" :message="$message" class="mb-4"/>
```

<a name="accessing-parent-data"></a>
### 親データへのアクセス

子コンポーネントの内部で親コンポーネントのデータにアクセスしたい場合があります。このような場合には、`@aware` ディレクティブを使用します。たとえば、親コンポーネント `<x-menu>` と子コンポーネント `<x-menu.item>` からなる複雑なメニューコンポーネントを構築しているとします。

```blade
<x-menu color="purple">
    <x-menu.item>...</x-menu.item>
    <x-menu.item>...</x-menu.item>
</x-menu>
```

`<x-menu>` コンポーネントは、次のような実装となるでしょう。

```blade
<!-- /resources/views/components/menu/index.blade.php -->

@props(['color' => 'gray'])

<ul {{ $attributes->merge(['class' => 'bg-'.$color.'-200']) }}>
    {{ $slot }}
</ul>
```

`color` プロップは親 (`<x-menu>`) にのみ渡されるため、`<x-menu.item>` 内では使用できません。ただし、`@aware` ディレクティブを使用すると、`<x-menu.item>` 内でも使用できるようになります。

```blade
<!-- /resources/views/components/menu/item.blade.php -->

@aware(['color' => 'gray'])

<li {{ $attributes->merge(['class' => 'text-'.$color.'-800']) }}>
    {{ $slot }}
</li>
```

> **Warning**  
> `@aware` ディレクティブは、HTML 属性を介して親コンポーネントに明示的に渡されていない親データにはアクセスできません。親コンポーネントに明示的に渡されないデフォルトの `@props` 値には、`@aware` ディレクティブではアクセスできません。

<a name="anonymous-component-paths"></a>
### 匿名コンポーネントのパス

前に説明したように、匿名コンポーネントは通常、`resources/views/components` ディレクトリ内に Blade テンプレートを配置することによって定義されます。ただし、デフォルトのパスに加えて、他の匿名コンポーネントのパスを Laravel に登録したい場合もあります。

`anonymousComponentPath` メソッドは、匿名コンポーネントの場所への「パス」を第１引数に受け取り、コンポーネントを配置するオプションの「名前空間」を 第２引数にオプションとして受け取ります。通常、このメソッドは、アプリケーションの [サービス プロバイダ](/docs/{{version}}/providers) の `boot` メソッドから呼び出す必要があります。

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Blade::anonymousComponentPath(__DIR__.'/../components');
    }

上記の例のように、コンポーネントパスがプレフィックスを指定せずに登録した場合、Blade コンポーネントでも対応するプレフィックスなしでレンダリングされます。たとえば、上記で登録したパスに `panel.blade.php` コンポーネントが存在する場合、次のようにレンダリングされます。

```blade
<x-panel />
```

プレフィックスの「名前空間」は、`anonymousComponentPath` メソッドの 第２引数として指定できます。

    Blade::anonymousComponentPath(__DIR__.'/../components', 'dashboard');

プレフィックスを指定すると、その「名前空間」内のコンポーネントは、コンポーネントがレンダリングされるときに、そのコンポーネントの名前空間にプレフィックスを付けることにより、レンダリングされます。

```blade
<x-dashboard::panel />
```

<a name="building-layouts"></a>
## レイアウト構築

<a name="layouts-using-components"></a>
### コンポーネントを使用したレイアウト

ほとんどの Web アプリケーションは、さまざまなページにわたり同じレイアウトを共有します。作成するすべてのビューでレイアウトの HTML を繰り返し記述する必要がある場合、アプリケーションを保守するのは非常に面倒になります。ありがたいことに、このレイアウトを単一の [Blade コンポーネント](#components) として定義し、アプリケーション全体で使用すると便利です。

<a name="defining-the-layout-component"></a>
#### レイアウトコンポーネントの定義

たとえば、`todo` リストアプリケーションを構築していると仮定しましょう。次のような `layout` コンポーネントを定義するとします。

```blade
<!-- resources/views/components/layout.blade.php -->

<html>
    <head>
        <title>{{ $title ?? 'Todo Manager' }}</title>
    </head>
    <body>
        <h1>Todos</h1>
        <hr/>
        {{ $slot }}
    </body>
</html>
```

<a name="applying-the-layout-component"></a>
#### レイアウトコンポーネントの適用

`layout` コンポーネントを定義したら、そのコンポーネントを利用する Blade ビューを作成できます。この例では、タスクリストを表示する単純なビューを定義します。

```blade
<!-- resources/views/tasks.blade.php -->

<x-layout>
    @foreach ($tasks as $task)
        {{ $task }}
    @endforeach
</x-layout>
```

コンポーネントに挿入されるコンテンツは、`layout` コンポーネント内のデフォルトの `$slot` 変数として提供されます。お気づきかもしれませんが、この `layout` は、`$title` スロットが提供されている場合はそれも尊重します。それ以外の場合は、デフォルトのタイトルが表示されます。 [コンポーネントのドキュメント](#components) で説明されている標準スロット構文を使用して、タスクリストビューからカスタムタイトルを挿入できます。

```blade
<!-- resources/views/tasks.blade.php -->

<x-layout>
    <x-slot:title>
        Custom Title
    </x-slot>

    @foreach ($tasks as $task)
        {{ $task }}
    @endforeach
</x-layout>
```

レイアウトとタスクリストのビューを定義したので、あとはルートから `task` ビューを返すだけです。

    use App\Models\Task;

    Route::get('/tasks', function () {
        return view('tasks', ['tasks' => Task::all()]);
    });

<a name="layouts-using-template-inheritance"></a>
### テンプレートの継承を使用したレイアウト

<a name="defining-a-layout"></a>
#### レイアウトの定義

レイアウトは「テンプレート継承」によって作成することもできます。これは、[コンポーネント](#components) が導入される前に、アプリケーションを構築する主な方法でした。

まず、簡単な例を見てみましょう。ページレイアウトを見ていきます。ほとんどの Web アプリケーションはさまざまなページにわたって同じ一般的なレイアウトを維持するため、このレイアウトを単一の Blade ビューとして定義すると便利です。

```blade
<!-- resources/views/layouts/app.blade.php -->

<html>
    <head>
        <title>App Name - @yield('title')</title>
    </head>
    <body>
        @section('sidebar')
            This is the master sidebar.
        @show

        <div class="container">
            @yield('content')
        </div>
    </body>
</html>
```

ご覧のとおり、このファイルには典型的な HTML マークアップが含まれています。 ただし、`@section`ディレクティブと`@yield`ディレクティブに注意してください。 `@section` ディレクティブは、名前が示すとおり、コンテンツのセクションを定義します。一方、`@yield` ディレクティブは、指定されたセクションのコンテンツを表示するために使用されます。

アプリケーションのレイアウトを定義したので、そのレイアウトを継承する子ページを定義しましょう。

<a name="extending-a-layout"></a>
#### レイアウトの拡張

子ビューを定義するときは、`@extends`Blade ディレクティブを使用して、子ビューが`継承`するレイアウトを指定します。 Blade レイアウトを拡張するビューは、`@section`ディレクティブを使用してコンテンツをレイアウトのセクションに挿入できます。 上の例に見られるように、これらのセクションの内容は `@yield` を使用してレイアウトに表示されることに注意してください。

```blade
<!-- resources/views/child.blade.php -->

@extends('layouts.app')

@section('title', 'Page Title')

@section('sidebar')
    @@parent

    <p>This is appended to the master sidebar.</p>
@endsection

@section('content')
    <p>This is my body content.</p>
@endsection
```

この例では、`sidebar` セクションが `@@parent` ディレクティブを利用して、コンテンツを (上書きではなく) レイアウトのサイドバーに追加しています。 `@@parent` ディレクティブは、ビューがレンダリングされるときにレイアウトのコンテンツに置き換えられます。

> **Note**  
> 前の例とは異なり、この `sidebar` セクションは `@show` ではなく `@endsection` で終わります。 `@endsection` ディレクティブはセクションを定義するだけですが、`@show` はセクションを定義して **即座に生成**します。

`@yield` ディレクティブは、2 番目のパラメータとしてデフォルト値も受け入れます。 この値は、生成されるセクションが未定義の場合に表示されます。

```blade
@yield('content', 'Default content')
```

<a name="forms"></a>
## フォーム

<a name="csrf-field"></a>
### CSRF フィールド

アプリケーションで HTML フォームを定義するときは常に、[CSRF 保護](/docs/{{version}}/csrf) ミドルウェアがリクエストを検証できるように、フォームに非表示の CSRF トークン フィールドを含める必要があります。 `@csrf` Blade ディレクティブを使用してトークン フィールドを生成できます。

```blade
<form method="POST" action="/profile">
    @csrf

    ...
</form>
```

<a name="method-field"></a>
### Method フィールド

HTML フォームでは `PUT`、`PATCH`、または `DELETE` リクエストを作成できないため、これらの HTTP 動詞を偽装するには、非表示の `_method` フィールドを追加する必要があります。 `@method` Blade ディレクティブを使用すると、このフィールドを作成できます。

```blade
<form action="/foo/bar" method="POST">
    @method('PUT')

    ...
</form>
```

<a name="validation-errors"></a>
### バリデーションエラー

`@error` ディレクティブを使用すると、特定の属性に [検証エラー メッセージ](/docs/{{version}}/validation#quick-displaying-the-validation-errors) が存在するかどうかをすばやく確認できます。 `@error` ディレクティブ内で、`$message` 変数をエコーしてエラー メッセージを表示できます。

```blade
<!-- /resources/views/post/create.blade.php -->

<label for="title">Post Title</label>

<input id="title"
    type="text"
    class="@error('title') is-invalid @enderror">

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

`@error` ディレクティブは "if" ステートメントにコンパイルされるため、属性にエラーがない場合は `@else` ディレクティブを使用してコンテンツをレンダリングできます。

```blade
<!-- /resources/views/auth.blade.php -->

<label for="email">Email address</label>

<input id="email"
    type="email"
    class="@error('email') is-invalid @else is-valid @enderror">
```

[特定のエラー バッグの名前](/docs/{{version}}/validation#named-error-bags) を `@error` ディレクティブの 2 番目のパラメーターとして渡して、複数のエラー バッグを含むページ上の検証エラー メッセージを取得できます。 フォーム:

```blade
<!-- /resources/views/auth.blade.php -->

<label for="email">Email address</label>

<input id="email"
    type="email"
    class="@error('email', 'login') is-invalid @enderror">

@error('email', 'login')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

<a name="stacks"></a>
## スタック

Blade を使用すると、別のビューまたはレイアウトの別の場所にレンダリングできる名前付きスタックにプッシュできます。 これは、子ビューで必要な JavaScript ライブラリを指定する場合に特に役立ちます。

```blade
@push('scripts')
    <script src="/example.js"></script>
@endpush
```

If you would like to `@push` content if a given boolean expression evaluates to `true`, you may use the `@pushIf` directive:

```blade
@pushIf($shouldPush, 'scripts')
    <script src="/example.js"></script>
@endPushIf
```

必要に応じて何度でもスタックにプッシュできます。 完全なスタックの内容をレンダリングするには、スタックの名前を `@stack` ディレクティブに渡します。

```blade
<head>
    <!-- Head Contents -->

    @stack('scripts')
</head>
```

コンテンツをスタックの先頭に追加したい場合は、`@prepend` ディレクティブを使用する必要があります。

```blade
@push('scripts')
    This will be second...
@endpush

// Later...

@prepend('scripts')
    This will be first...
@endprepend
```

<a name="service-injection"></a>
## サービスの注入

`@inject` ディレクティブは、Laravel [サービスコンテナ](/docs/{{version}}/container) からサービスを取得するために使用できます。 `@inject` に渡される最初の引数はサービスが配置される変数の名前であり、2 番目の引数は解決したいサービスのクラス名またはインターフェイス名です。

```blade
@inject('metrics', 'App\Services\MetricsService')

<div>
    Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
</div>
```

<a name="rendering-inline-blade-templates"></a>
## インライン Blade テンプレートのレンダリング

場合によっては、生の Blade テンプレート文字列を有効な HTML に変換する必要があるかもしれません。 これは、`Blade`ファサードによって提供される`render`メソッドを使用して実行できます。 `render`メソッドは、Blade テンプレート文字列と、テンプレートに提供するオプションのデータ配列を受け入れます。

```php
use Illuminate\Support\Facades\Blade;

return Blade::render('Hello, {{ $name }}', ['name' => 'Julian Bashir']);
```

Laravel は、インライン Blade テンプレートを`storage/framework/views`ディレクトリに書き込むことによってレンダリングします。 Blade テンプレートのレンダリング後に Laravel にこれらの一時ファイルを削除させたい場合は、メソッドに `deleteCachedView` 引数を指定できます。

```php
return Blade::render(
    'Hello, {{ $name }}',
    ['name' => 'Julian Bashir'],
    deleteCachedView: true
);
```

<a name="rendering-blade-fragments"></a>
## Blade フラグメントのレンダリング

[Turbo](https://turbo.hotwired.dev/) や [htmx](https://htmx.org/) などのフロントエンド フレームワークを使用する場合、Blade テンプレートの一部のみを返す必要がある場合があります。 HTTP 応答。 Bladeの`フラグメント`を使用すると、まさにそれが可能になります。 まず、Blade テンプレートの一部を `@fragment` および `@endfragment` ディレクティブ内に配置します。

```blade
@fragment('user-list')
    <ul>
        @foreach ($users as $user)
            <li>{{ $user->name }}</li>
        @endforeach
    </ul>
@endfragment
```

次に、このテンプレートを利用するビューをレンダリングするときに、`fragment`メソッドを呼び出して、指定されたフラグメントのみが送信 HTTP 応答に含まれるように指定できます。

```php
return view('dashboard', ['users' => $users])->fragment('user-list');
```

`fragmentIf` メソッドを使用すると、指定された条件に基づいて条件付きでビューのフラグメントを返すことができます。 それ以外の場合は、ビュー全体が返されます。

```php
return view('dashboard', ['users' => $users])
    ->fragmentIf($request->hasHeader('HX-Request'), 'user-list');
```

`fragments` メソッドと `fragmentsIf` メソッドを使用すると、応答で複数のビュー フラグメントを返すことができます。 フラグメントは連結されます。

```php
view('dashboard', ['users' => $users])
    ->fragments(['user-list', 'comment-list']);

view('dashboard', ['users' => $users])
    ->fragmentsIf(
        $request->hasHeader('HX-Request'),
        ['user-list', 'comment-list']
    );
```

<a name="extending-blade"></a>
## Blade の拡張

Blade では、`directive`メソッドを使用して独自のカスタム ディレクティブを定義できます。 Blade コンパイラはカスタム ディレクティブを検出すると、ディレクティブに含まれる式を使用して提供されたコールバックを呼び出します。

次の例では、指定された `$var` をフォーマットする `@datetime($var)` ディレクティブを作成します。これは、`DateTime` のインスタンスである必要があります。

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Blade;
    use Illuminate\Support\ServiceProvider;

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
            Blade::directive('datetime', function (string $expression) {
                return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
            });
        }
    }

ご覧のとおり、ディレクティブに渡される式に`format`メソッドを連鎖させます。 したがって、この例では、このディレクティブによって生成される最終的な PHP は次のようになります。

    <?php echo ($var)->format('m/d/Y H:i'); ?>

> **Warning**  
> Blade ディレクティブのロジックを更新した後、キャッシュされた Blade ビューをすべて削除する必要があります。 キャッシュされたBlade ビューは、`view:clear`アーティザン コマンドを使用して削除できます。

<a name="custom-echo-handlers"></a>
### カスタム Echo ハンドラ

Blade を使用してオブジェクトを`エコー`しようとすると、オブジェクトの __toString メソッドが呼び出されます。 [`__toString`](https://www.php.net/manual/en/ language.oop5.magic.php#object.tostring) メソッドは、PHP の組み込み`マジック メソッド`の 1 つです。 ただし、対話しているクラスがサードパーティのライブラリに属している場合など、特定のクラスの __toString メソッドを制御できない場合があります。

このような場合、Blade では、その特定の種類のオブジェクトにカスタム エコー ハンドラーを登録できます。 これを実現するには、Blade の`stringable`メソッドを呼び出す必要があります。 `stringable` メソッドはクロージャを受け入れます。 このクロージャは、レンダリングを担当するオブジェクトのタイプをタイプヒントする必要があります。 通常、`stringable`メソッドは、アプリケーションの`AppServiceProvider`クラスの`boot`メソッド内で呼び出す必要があります。

    use Illuminate\Support\Facades\Blade;
    use Money\Money;

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Blade::stringable(function (Money $money) {
            return $money->formatTo('en_GB');
        });
    }

カスタム エコー ハンドラーを定義したら、Blade テンプレート内のオブジェクトをエコーするだけです。

```blade
Cost: {{ $money }}
```

<a name="custom-if-statements"></a>
### カスタム If 文

カスタム ディレクティブのプログラミングは、単純なカスタム条件文を定義する場合、必要以上に複雑になる場合があります。 そのため、Blade は、クロージャを使用してカスタム条件ディレクティブを迅速に定義できる `Blade::if` メソッドを提供します。 たとえば、アプリケーションに設定されたデフォルトの`ディスク`をチェックするカスタム条件を定義してみましょう。 これは、`AppServiceProvider` の `boot` メソッドで行うことができます。

    use Illuminate\Support\Facades\Blade;

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Blade::if('disk', function (string $value) {
            return config('filesystems.default') === $value;
        });
    }

カスタム条件を定義したら、それをテンプレート内で使用できます。

```blade
@disk('local')
    <!-- The application is using the local disk... -->
@elsedisk('s3')
    <!-- The application is using the s3 disk... -->
@else
    <!-- The application is using some other disk... -->
@enddisk

@unlessdisk('local')
    <!-- The application is not using the local disk... -->
@enddisk
```
