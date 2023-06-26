# Blabe テンプレート

- [はじめに](#はじめに)
     - [ライブワイヤー付きスーパーチャージングブレード](#supercharge-blade-with-livewire)
- [データの表示](#displaying-data)
     - [HTMLエンティティエンコーディング](#html-entity-encoding)
     - [ブレードと JavaScript フレームワーク](#blade-and-javascript-frameworks)
- [ブレードディレクティブ](#blade-directives)
     - [if文](#if文)
     - [Switch ステートメント](#switch-statements)
     - [ループ](#ループ)
     - [ループ変数](#the-loop-variable)
     - [条件付きクラス](#conditional-classes)
     - [追加属性](#Additional-attributes)
     - [サブビューを含む](#include-subviews)
     - [`@once` ディレクティブ](#the-once-directive)
     - [生のPHP](#raw-php)
     - [コメント](#コメント)
- [コンポーネント](#components)
     - [レンダリングコンポーネント](#rendering-components)
     - [コンポーネントにデータを渡す](#passing-data-to-components)
     - [コンポーネントの属性](#component-attributes)
     - [予約済みキーワード](#reserved-keywords)
     - [スロット](スロット数)
     - [インラインコンポーネントビュー](#inline-component-views)
     - [動的コンポーネント](#dynamic-components)
     - [コンポーネントを手動で登録する](#manually-registering-components)
- [匿名コンポーネント](#anonymous-components)
     - [匿名インデックスコンポーネント](#anonymous-index-components)
     - [データのプロパティ/属性](#data-properties-attributes)
     - [親データへのアクセス](#accessing-parent-data)
     - [匿名コンポーネントのパス](#anonymous-component-paths)
- [建物のレイアウト](#building-layouts)
     - [コンポーネントを使用したレイアウト](#layouts-using-components)
     - [テンプレート継承を使用したレイアウト](#layouts-using-template-inheritance)
- [フォーム](#forms)
     - [CSRFフィールド](#csrf-field)
     - [メソッドフィールド](#method-field)
     - [検証エラー](#validation-errors)
- [スタック](#スタック)
- [サービスインジェクション](#service-injection)
- [レンダリング インライン ブレード テンプレート](#rendering-inline-blade-templates)
- [レンダリングブレードフラグメント](#rendering-blade-fragments)
- [拡張ブレード](#extending-blade)
     - [カスタム エコー ハンドラー](#custom-echo-handlers)
     - [カスタム If ステートメント](#custom-if-statements)

<a name="はじめに"></a>
＃＃ 序章

Blade は、Laravel に含まれるシンプルかつ強力なテンプレート エンジンです。 一部の PHP テンプレート エンジンとは異なり、Blade では、テンプレート内でプレーンな PHP コードを使用することが制限されません。 実際、すべての Blade テンプレートはプレーンな PHP コードにコンパイルされ、変更されるまでキャッシュされます。つまり、Blade はアプリケーションに本質的にオーバーヘッドを追加しません。 ブレード テンプレート ファイルは `.blade.php` ファイル拡張子を使用し、通常は `resources/views` ディレクトリに保存されます。

ブレード ビューは、グローバル `view` ヘルパーを使用してルートまたはコントローラーから返される場合があります。 もちろん、[views](/docs/{{version}}/views) のドキュメントで説明されているように、`view` ヘルパーの 2 番目の引数を使用してデータを Blade ビューに渡すこともできます。

     Route::get('/', function () {
         return view('挨拶', ['名前' => 'フィン']);
     });

<a name="supercharging-blade-with-livewire"></a>
### Livewire 付きブレードを過給する

Blade テンプレートを次のレベルに引き上げて、動的なインターフェイスを簡単に構築したいですか? [Laravel Livewire](https://laravel-livewire.com) をチェックしてください。 Livewire を使用すると、通常は React や Vue などのフロントエンド フレームワークを介してのみ可能となる動的機能で拡張された Blade コンポーネントを作成でき、複雑な作業、クライアント側のレンダリング、ビルド手順を必要とせずに最新のリアクティブ フロントエンドを構築するための優れたアプローチを提供します。 多くの JavaScript フレームワーク。

<a name="データの表示"></a>
## データの表示

変数を中括弧で囲むことにより、Blade ビューに渡されるデータを表示できます。 たとえば、次のルートがあるとします。

     Route::get('/', function () {
         return view('ようこそ', ['名前' => 'サマンサ']);
     });

次のように `name` 変数の内容を表示できます。

「」ブレード
こんにちは、{{ $name }}。
「」

> **注意**
> Blade の `{{ }}` echo ステートメントは、PHP の `htmlspecialchars` 関数を通じて自動的に送信され、XSS 攻撃を防ぎます。

ビューに渡された変数の内容を表示することに限定されません。 任意の PHP 関数の結果をエコーすることもできます。 実際、Blade echo ステートメント内に任意の PHP コードを含めることができます。

「」ブレード
現在の UNIX タイムスタンプは {{ time() }} です。
「」

<a name="html-entity-encoding"></a>
### HTML エンティティのエンコーディング

デフォルトでは、Blade (および Laravel `e` ヘルパー) は HTML エンティティを二重エンコードします。 二重エンコーディングを無効にしたい場合は、`AppServiceProvider` の `boot` メソッドから `Blade::withoutDoubleEncoding` メソッドを呼び出します。

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
#### アンエスケープされたデータの表示

デフォルトでは、Blade `{{ }}` ステートメントは、XSS 攻撃を防ぐために、PHP の `htmlspecialchars` 関数を通じて自動的に送信されます。 データをエスケープしたくない場合は、次の構文を使用できます。

「」ブレード
こんにちは、 {！！ $name !!}。
「」

> **警告**
> アプリケーションのユーザーが提供したコンテンツをエコーする場合は、十分に注意してください。 ユーザーが指定したデータを表示するときに XSS 攻撃を防ぐには、通常、エスケープされた二重中括弧構文を使用する必要があります。

<a name="blade-and-javascript-frameworks"></a>
### ブレードおよび JavaScript フレームワーク

多くの JavaScript フレームワークでも「中括弧」を使用して特定の式をブラウザーに表示する必要があることを示すため、「@」記号を使用して式をそのままにしておく必要があることを Blade レンダリング エンジンに通知できます。 例えば：

「」ブレード
<h1>ララベル</h1>

こんにちは、@{{ 名前 }}。
「」

この例では、「@」記号が Blade によって削除されます。 ただし、`{{ name }}` 式は Blade エンジンによって変更されないため、JavaScript フレームワークによってレンダリングできます。

「@」記号は、Blade ディレクティブをエスケープするために使用することもできます。

「」ブレード
{{-- ブレード テンプレート --}}
@@もしも（）

<!-- HTML 出力 -->
@もしも（）
「」

<a name="rendering-json"></a>
#### JSON のレンダリング

JavaScript 変数を初期化するために、配列を JSON としてレンダリングする目的でビューに配列を渡すことがあります。 例えば：

「」ブレード
<スクリプト>
     var app = <?php echo json_encode($array); ?>;
</script>
「」

ただし、「json_encode」を手動で呼び出す代わりに、「Illuminate\Support\Js::from」メソッド ディレクティブを使用することもできます。 「from」メソッドは、PHP の「json_encode」関数と同じ引数を受け入れます。 ただし、結果の JSON が HTML 引用符内に含められるように適切にエスケープされることが保証されます。 「from」メソッドは、指定されたオブジェクトまたは配列を有効な JavaScript オブジェクトに変換する文字列「JSON.parse」JavaScript ステートメントを返します。

「」ブレード
<スクリプト>
     var app = {{ Illuminate\Support\Js::from($array) }};
</script>
「」

Laravel アプリケーション スケルトンの最新バージョンには「Js」ファサードが含まれており、これにより Blade テンプレート内のこの機能に簡単にアクセスできます。

「」ブレード
<スクリプト>
     var app = {{ Js::from($array) }};
</script>
「」

> **警告**
> 既存の変数を JSON としてレンダリングする場合は、`Js::from` メソッドのみを使用してください。 Blade テンプレートは正規表現に基づいており、複雑な表現をディレクティブに渡そうとすると、予期しないエラーが発生する可能性があります。

<a name="the-at-verbatim-directive"></a>
#### `@verbatim` ディレクティブ

テンプレートの大部分で JavaScript 変数を表示している場合は、HTML を `@verbatim` ディレクティブでラップすると、各 Blade echo ステートメントの前に `@` 記号を付ける必要がなくなります。

「」ブレード
@逐語的に
     <div class="コンテナ">
         こんにちは、{{ 名前 }}。
     </div>
@endverbatim
「」

<a name="ブレードディレクティブ"></a>
## ブレードディレクティブ

Blade は、テンプレートの継承とデータの表示に加えて、条件ステートメントやループなどの一般的な PHP 制御構造の便利なショートカットも提供します。 これらのショートカットは、PHP の制御構造を操作するための非常にクリーンで簡潔な方法を提供すると同時に、PHP の対応するものにとっても馴染みのあるものです。

<a name="if-statements"></a>
### if ステートメント

`@if`、`@elseif`、`@else`、および `@endif` ディレクティブを使用して `if` ステートメントを構築できます。 これらのディレクティブは、対応する PHP ディレクティブと同様に機能します。

「」ブレード
@if (カウント($レコード) === 1)
     レコードが1枚あります！
@elseif (カウント($レコード) > 1)
     複数の記録を持っています！
@それ以外
     記録がないんです！
@endif
「」

便宜上、Blade には `@unless` ディレクティブも用意されています。

「」ブレード
@unless (認証::check())
     サインインしていません。
@endunless
「」

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

`@auth` および `@guest` ディレクティブを使用すると、現在のユーザーが [認証済み](/docs/{{version}}/authentication) であるかゲストであるかを迅速に判断できます。

「」ブレード
@認証
     // ユーザーは認証されています...
@endauth

@ゲスト
     // ユーザーは認証されていません...
@エンドゲスト
「」

必要に応じて、`@auth` および `@guest` ディレクティブを使用するときにチェックする必要がある認証ガードを指定できます。

「」ブレード
@auth('管理者')
     // ユーザーは認証されています...
@endauth

@ゲスト('管理者')
     // ユーザーは認証されていません...
@エンドゲスト
「」

<a name="環境ディレクティブ"></a>
#### 環境指令

`@production` ディレクティブを使用して、アプリケーションが実稼働環境で実行されているかどうかを確認できます。

「」ブレード
@製造
     // プロダクション固有のコンテンツ...
@endproduction
「」

または、`@env` ディレクティブを使用して、アプリケーションが特定の環境で実行されているかどうかを判断することもできます。

「」ブレード
@env('ステージング')
     // アプリケーションは「ステージング」で実行されています...
@endenv

@env(['ステージング', '本番'])
     // アプリケーションは「ステージング」または「本番」で実行されています...
@endenv
「」

<a name="セクションディレクティブ"></a>
#### セクションディレクティブ

`@hasSection` ディレクティブを使用して、テンプレート継承セクションにコンテンツがあるかどうかを判断できます。

「」ブレード
@hasSection('ナビゲーション')
     <div class="プルライト">
         @yield('ナビゲーション')
     </div>

     <div class="clearfix"></div>
@endif
「」

`sectionMissing` ディレクティブを使用して、セクションにコンテンツがないかどうかを判断できます。

「」ブレード
@sectionMissing('ナビゲーション')
     <div class="プルライト">
         @include('デフォルトナビゲーション')
     </div>
@endif
「」

<a name="switch-statements"></a>
### switch ステートメント

switch ステートメントは、`@switch`、`@case`、`@break`、`@default`、および `@endswitch` ディレクティブを使用して構築できます。

「」ブレード
@スイッチ($i)
     @ケース(1)
         最初のケース...
         @壊す

     @ケース(2)
         2番目のケース...
         @壊す

     @デフォルト
         デフォルトのケース...
@エンドスイッチ
「」

<a name="ループ"></a>
### ループ

条件文に加えて、Blade は PHP のループ構造を操作するための単純なディレクティブを提供します。 繰り返しますが、これらの各ディレクティブは、対応する PHP ディレクティブと同様に機能します。

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

> **注意**
> `foreach` ループを反復しているときに、[ループ変数](#the-loop-variable) を使用して、ループの最初の反復にいるのか最後の反復にいるのかなど、ループに関する貴重な情報を取得できます。

ループを使用する場合、`@ continue` および ` @break` ディレクティブを使用して、現在の反復をスキップしたり、ループを終了したりすることもできます。

「」ブレード
@foreach ($users として $user)
     @if ($user->type == 1)
         @続く
     @endif

     <li>{{ $user->名前 }}</li>

     @if ($user->number == 5)
         @壊す
     @endif
@endforeach
「」

ディレクティブ宣言内に継続条件または中断条件を含めることもできます。

「」ブレード
@foreach ($users として $user)
     @Continue($user->type == 1)

     <li>{{ $user->名前 }}</li>

     @break($user->number == 5)
@endforeach
「」

<a name="ループ変数"></a>
### ループ変数

「foreach」ループを反復している間、ループ内で「$loop」変数が使用可能になります。 この変数は、現在のループ インデックスや、これがループの最初の反復であるか最後の反復であるかなど、いくつかの有用な情報へのアクセスを提供します。

「」ブレード
@foreach ($users として $user)
     @if ($loop->first)
         これが最初の反復です。
     @endif

     @if ($loop->last)
         これが最後の反復です。
     @endif

     <p>これはユーザー {{ $user->id }} です</p>
@endforeach
「」

ネストされたループにいる場合は、「parent」プロパティを介して親ループの「$loop」変数にアクセスできます。

「」ブレード
@foreach ($users として $user)
     @foreach ($user->$post として投稿)
         @if ($loop->parent->first)
             これは親ループの最初の反復です。
         @endif
     @endforeach
@endforeach
「」

`$loop` 変数には、他にもさまざまな便利なプロパティが含まれています。

| プロパティ | 説明 |
|-----------------|---------------------------- ----------------------------|
| `$loop->index` | 現在のループ反復のインデックス (0 から始まります)。 |
| `$ループ->反復` | 現在のループの繰り返し (1 から始まります)。 |
| `$loop->残り` | ループ内に残っている反復数。 |
| `$loop->count` | 反復される配列内の項目の合計数。 |
| `$loop->first` | これがループの最初の反復であるかどうか。 |
| `$loop->last` | これがループの最後の反復であるかどうか。 |
| `$loop->even` | これがループの均等な反復であるかどうか。 |
| `$loop->odd` | これがループ全体での奇数の反復であるかどうか。 |
| `$ループ->深さ` | 現在のループのネスト レベル。 |
| `$loop->親` | ネストされたループ内の場合、親のループ変数。 |

<a name="条件クラス"></a>
### 条件付きクラスとスタイル

`@class` ディレクティブは、CSS クラス文字列を条件付きでコンパイルします。 このディレクティブは、追加するクラスを配列キーに含み、値がブール式であるクラスの配列を受け入れます。 配列要素に数値キーがある場合、その要素は常に表示されるクラス リストに含まれます。

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

同様に、「@style」ディレクティブを使用して、条件付きでインライン CSS スタイルを HTML 要素に追加できます。

「」ブレード
@php
     $isActive = true;
@endphp

<スパン @style([
     '背景色: 赤',
     'font-weight: 太字' => $isActive,
])></スパン>

<span style="background-color: red; font-weight:bold;"></span>
「」

<a name="追加属性"></a>
### 追加の属性

便宜上、「@checked」ディレクティブを使用して、特定の HTML チェックボックス入力が「チェックされている」かどうかを簡単に示すことができます。 このディレクティブは、指定された条件が「true」と評価された場合に「checked」をエコーします。

「」ブレード
<input type="チェックボックス"
         名前="アクティブ"
         値 = "アクティブ"
         @checked(old('active', $user->active)) />
「」

同様に、`@selected` ディレクティブを使用して、特定の選択オプションを「選択」する必要があるかどうかを示すことができます。

「」ブレード
<select name="バージョン">
     @foreach ($product->versions として $version)
         <option value="{{ $version }}" @selected(old('version') == $version)>
             {{ $バージョン }}
         </オプション>
     @endforeach
</選択>
「」

さらに、`@disabled` ディレクティブを使用して、特定の要素を「無効」にするかどうかを示すことができます。

「」ブレード
<button type="submit" @disabled($errors->isNotEmpty())>送信</button>
「」

さらに、`@readonly` ディレクティブを使用して、特定の要素を「読み取り専用」にするかどうかを示すことができます。

「」ブレード
<input type="電子メール"
         名前 = "電子メール"
         value="email@laravel.com"
         @readonly($user->isNotAdmin()) />
「」

さらに、`@required` ディレクティブを使用して、特定の要素を「必須」にするかどうかを示すことができます。

「」ブレード
<input type="テキスト"
         名前 = "タイトル"
         値 = "タイトル"
         @required($user->isAdmin()) />
「」

<a name="含む-サブビュー"></a>
### サブビューを含む

> **注意**
> `@include` ディレクティブは自由に使用できますが、Blade [components](#components) は同様の機能を提供し、データや属性のバインディングなど、`@include` ディレクティブよりも優れたいくつかの利点を提供します。

Blade の `@include` ディレクティブを使用すると、別のビュー内から Blade ビューを含めることができます。 親ビューで使用できるすべての変数は、組み込まれたビューでも使用できるようになります。

「」ブレード
<div>
     @include('shared.errors')

     <フォーム>
         <!-- フォームの内容 -->
     </form>
</div>
「」

含まれるビューは親ビューで使用可能なすべてのデータを継承しますが、含まれるビューで使用できるようにする必要がある追加データの配列を渡すこともできます。

「」ブレード
@include('view.name', ['status' => 'complete'])
「」

存在しないビューを「@include」しようとすると、Laravel はエラーをスローします。 存在するかどうかわからないビューを含めたい場合は、`@includeIf` ディレクティブを使用する必要があります。

「」ブレード
@includeIf('view.name', ['ステータス' => '完了'])
「」

指定されたブール式が「true」または「false」に評価された場合にビューを「@include」したい場合は、「@includeWhen」および「@includeUnless」ディレクティブを使用できます。

```blade
@includeWhen($boolean, 'view.name', ['status' => 'complete'])

@includeUnless($boolean, 'view.name', ['status' => 'complete'])
```

指定されたビューの配列から存在する最初のビューを含めるには、`includeFirst` ディレクティブを使用できます。

「」ブレード
@includeFirst(['custom.admin', 'admin'], ['status' => 'complete'])
「」

> **警告**
> ブレード ビューでは `__DIR__` および `__FILE__` 定数を使用しないでください。これらは、キャッシュされコンパイルされたビューの場所を参照するためです。

<a name="rendering-views-for-collections"></a>
#### コレクションのビューのレンダリング

Blade の `@each` ディレクティブを使用して、ループとインクルードを 1 行に結合できます。

「」ブレード
@each('view.name', $jobs, 'job')
「」

`@each` ディレクティブの最初の引数は、配列またはコレクション内の各要素に対してレンダリングするビューです。 2 番目の引数は反復処理する配列またはコレクションで、3 番目の引数はビュー内の現在の反復に割り当てられる変数名です。 したがって、たとえば、「ジョブ」の配列を反復処理している場合、通常はビュー内の「ジョブ」変数として各ジョブにアクセスする必要があります。 現在の反復の配列キーは、ビュー内の `key` 変数として使用できます。

`@each` ディレクティブに 4 番目の引数を渡すこともできます。 この引数は、指定された配列が空の場合にレンダリングされるビューを決定します。

「」ブレード
@each('view.name', $jobs, 'job', 'view.empty')
「」

> **警告**
> `@each` を介してレンダリングされたビューは、親ビューから変数を継承しません。 子ビューでこれらの変数が必要な場合は、代わりに `@foreach` および `@include` ディレクティブを使用する必要があります。

<a name="the-once-directive"></a>
### `@once` ディレクティブ

`@once` ディレクティブを使用すると、レンダリング サイクルごとに 1 回だけ評価されるテンプレートの一部を定義できます。 これは、[stacks](#stacks) を使用して、特定の JavaScript をページのヘッダーにプッシュする場合に便利です。 たとえば、ループ内で特定の [コンポーネント](#components) をレンダリングする場合、コンポーネントが初めてレンダリングされるときにのみ JavaScript をヘッダーにプッシュしたい場合があります。

「」ブレード
@一度
     @push('スクリプト')
         <スクリプト>
             // カスタム JavaScript...
         </script>
     @endpush
@endence
「」

`@once` ディレクティブは、`@push` または `@prepend` ディレクティブと組み合わせて使用されることが多いため、便宜上、`@pushOnce` および `@prependOnce` ディレクティブを利用できます。

「」ブレード
@pushOnce('スクリプト')
     <スクリプト>
         // カスタム JavaScript...
     </script>
@endPushOnce
「」

<a name="raw-php"></a>
### 生の PHP

状況によっては、PHP コードをビューに埋め込むと便利です。 Blade の `@php` ディレクティブを使用して、テンプレート内のプレーン PHP のブロックを実行できます。

「」ブレード
@php
     $カウンター = 1;
@endphp
「」

単一の PHP ステートメントのみを記述する必要がある場合は、そのステートメントを `@php` ディレクティブ内に含めることができます。

「」ブレード
@php($counter = 1)
「」

<a name="コメント"></a>
### コメント

Blade では、ビュー内にコメントを定義することもできます。 ただし、HTML コメントとは異なり、Blade コメントはアプリケーションから返される HTML には含まれません。

「」ブレード
{{-- このコメントはレンダリングされた HTML には表示されません --}}
「」

<a name="コンポーネント"></a>
## コンポーネント

コンポーネントとスロットは、セクション、レイアウト、インクルードに同様の利点をもたらします。 ただし、コンポーネントとスロットのメンタル モデルの方が理解しやすいと感じる人もいるかもしれません。 コンポーネントを作成するには、クラスベースのコンポーネントと匿名コンポーネントの 2 つのアプローチがあります。

クラスベースのコンポーネントを作成するには、「make:component」Artisan コマンドを使用できます。 コンポーネントの使用方法を説明するために、単純な「Alert」コンポーネントを作成します。 `make:component` コマンドは、コンポーネントを `app/View/Components` ディレクトリに配置します。

```シェル
php 職人の make:component アラート
「」

`make:component` コマンドは、コンポーネントのビュー テンプレートも作成します。 ビューは「resources/views/components」ディレクトリに配置されます。 独自のアプリケーション用のコンポーネントを作成する場合、コンポーネントは「app/View/Components」ディレクトリおよび「resources/views/components」ディレクトリ内で自動的に検出されるため、通常は追加のコンポーネント登録は必要ありません。

サブディレクトリ内にコンポーネントを作成することもできます。

```シェル
php 職人 make:component フォーム/入力
「」

上記のコマンドは、「app/View/Components/Forms」ディレクトリに「Input」コンポーネントを作成し、ビューは「resources/views/components/forms」ディレクトリに配置されます。

匿名コンポーネント (Blade テンプレートのみでクラスを持たないコンポーネント) を作成したい場合は、`make:component` コマンドを呼び出すときに `--view` フラグを使用できます。

```shell
php artisan make:component forms.input --view
```

上記のコマンドは、`resources/views/components/forms/input.blade.php` に Blade ファイルを作成します。これは、`<x-forms.input />` を介してコンポーネントとしてレンダリングできます。

<a name="手動登録パッケージコンポーネント"></a>
#### パッケージコンポーネントの手動登録

独自のアプリケーションのコンポーネントを作成する場合、コンポーネントは `app/View/Components` ディレクトリおよび `resources/views/components` ディレクトリ内で自動的に検出されます。

ただし、Blade コンポーネントを利用するパッケージを構築している場合は、コンポーネント クラスとその HTML タグ エイリアスを手動で登録する必要があります。 通常、コンポーネントはパッケージのサービスプロバイダーの「boot」メソッドに登録する必要があります。

     Illuminate\Support\Facades\Blade を使用します。

     /**
      * パッケージのサービスをブートストラップします。
      */
     パブリック関数 boot(): void
     {
         Blade::component('パッケージアラート', Alert::class);
     }

コンポーネントが登録されると、そのタグ エイリアスを使用してレンダリングできます。

「」ブレード
<x-package-alert/>
「」

あるいは、慣例に従って `componentNamespace` メソッドを使用してコンポーネント クラスを自動ロードすることもできます。 たとえば、「Nightshade」パッケージには、「Package\Views\Components」名前空間内に存在する「Calendar」コンポーネントと「ColorPicker」コンポーネントが含まれる場合があります。

     Illuminate\Support\Facades\Blade を使用します。

     /**
      * パッケージのサービスをブートストラップします。
      */
     パブリック関数 boot(): void
     {
         Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
     }

これにより、ベンダー名前空間で「package-name::」構文を使用してパッケージ コンポーネントを使用できるようになります。

「」ブレード
<x-nightshade::calendar />
<x-nightshade::color-picker />
「」

Blade は、コンポーネント名をパスカル文字に変換することで、このコンポーネントにリンクされているクラスを自動的に検出します。 サブディレクトリは、「ドット」表記を使用してサポートされています。

<a name="rendering-components"></a>
### レンダリングコンポーネント

コンポーネントを表示するには、Blade テンプレートの 1 つ内で Blade コンポーネント タグを使用できます。 ブレード コンポーネントのタグは文字列「x-」で始まり、その後にコンポーネント クラスのケバブ ケース名が続きます。

「」ブレード
<x-アラート/>

<x-ユーザープロファイル/>
「」

コンポーネント クラスが `app/View/Components` ディレクトリ内でさらに深くネストされている場合は、ディレクトリのネストを示すために `.` 文字を使用できます。 たとえば、コンポーネントが `app/View/Components/Inputs/Button.php` にあると仮定すると、次のようにレンダリングできます。

「」ブレード
<x-inputs.button/>
「」

コンポーネントを条件付きでレンダリングしたい場合は、コンポーネント クラスで ` shouldRender` メソッドを定義できます。 「 shouldRender 」メソッドが「 false 」を返した場合、コンポーネントはレンダリングされません。

     Illuminate\Support\Str を使用します。

     /**
      * コンポーネントをレンダリングするかどうか
      */
     パブリック関数 shouldRender(): bool
     {
         戻り Str::length($this->message) > 0;
     }

<a name="コンポーネントへのデータの受け渡し"></a>
### データをコンポーネントに渡す

HTML 属性を使用してデータを Blade コンポーネントに渡すことができます。 ハードコーディングされたプリミティブ値は、単純な HTML 属性文字列を使用してコンポーネントに渡すことができます。 PHP 式と変数は、「:」文字をプレフィックスとして使用する属性を介してコンポーネントに渡す必要があります。

「」ブレード
<x-alert type="error" :message="$message"/>
「」

コンポーネントのすべてのデータ属性をそのクラス コンストラクターで定義する必要があります。 コンポーネント上のすべてのパブリック プロパティは、コンポーネントのビューで自動的に利用できるようになります。 コンポーネントの「render」メソッドからビューにデータを渡す必要はありません。

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

「」ブレード
<div class="アラート アラート-{{ $type }}">
     {{ $メッセージ }}
</div>
「」

<a name="ケーシング"></a>
#### ケーシング

コンポーネントのコンストラクター引数は `camelCase` を使用して指定する必要がありますが、HTML 属性で引数名を参照する場合は `kebab-case` を使用する必要があります。 たとえば、次のコンポーネント コンストラクターがあるとします。

     /**
      * コンポーネントのインスタンスを作成します。
      */
     パブリック関数 __construct(
         パブリック文字列 $alertType、
     ) {}

`$alertType` 引数は次のようにコンポーネントに提供できます。

「」ブレード
<x-alert アラートタイプ="危険" />
「」

<a name="short-attribute-syntax"></a>
#### 短い属性構文

コンポーネントに属性を渡すときは、「短い属性」構文を使用することもできます。 属性名は、対応する変数名と一致することが多いため、これは便利なことがよくあります。

「」ブレード
{{-- 短い属性構文... --}}
<x-プロファイル :$userId :$name />

{{-- と同等です... --}}
<x-profile :user-id="$userId" :name="$name" />
「」

<a name="エスケープ-属性-レンダリング"></a>
#### 属性レンダリングのエスケープ

Alpine.js などの一部の JavaScript フレームワークもコロン接頭辞付きの属性を使用するため、二重コロン (`::`) 接頭辞を使用して属性が PHP 式ではないことを Blade に通知できます。 たとえば、次のコンポーネントがあるとします。

「」ブレード
<x-button ::class="{ 危険性: 削除中 }">
     送信
</xボタン>
「」

次の HTML が Blade によってレンダリングされます。

「」ブレード
<button :class="{ 危険性: 削除中 }">
     送信
</ボタン>
「」

<a name="コンポーネントメソッド"></a>
#### コンポーネントのメソッド

コンポーネント テンプレートで使用できるパブリック変数に加えて、コンポーネント上の任意のパブリック メソッドを呼び出すことができます。 たとえば、「isSelected」メソッドを持つコンポーネントを想像してください。

     /**
      * 指定されたオプションが現在選択されているオプションかどうかを確認します。
      */
     パブリック関数 isSelected(string $option): bool
     {
         return $option === $this->selected;
     }

メソッドの名前に一致する変数を呼び出すことで、コンポーネント テンプレートからこのメソッドを実行できます。

「」ブレード
<オプション {{ $isSelected($value) ? '選択済み' : '' }} value="{{ $value }}">
     {{ $ラベル }}
</オプション>
「」

<a name="using-attributes-slots-within-component-class"></a>
#### コンポーネント クラス内の属性とスロットへのアクセス

ブレード コンポーネントを使用すると、クラスの render メソッド内のコンポーネント名、属性、スロットにアクセスすることもできます。 ただし、このデータにアクセスするには、コンポーネントの `render` メソッドからクロージャを返す必要があります。 クロージャは、唯一の引数として `$data` 配列を受け取ります。 この配列には、コンポーネントに関する情報を提供するいくつかの要素が含まれます。

     クロージャを使用します。

     /**
      * コンポーネントを表すビュー/コンテンツを取得します。
      */
     パブリック関数 render(): クロージャ
     {
         戻り関数 (配列 $data) {
             // $data['コンポーネント名'];
             // $data['属性'];
             // $data['slot'];

             '<div>コンポーネントのコンテンツ</div>' を返します。
         };
     }

`componentName` は、HTML タグ内で使用される名前の `x-` プレフィックスの後に等しいです。 したがって、`<x-alert />` の `componentName` は `alert` になります。 「attributes」要素には、HTML タグに存在するすべての属性が含まれます。 `slot` 要素は、コンポーネントのスロットの内容を含む `Illuminate\Support\HtmlString` インスタンスです。

クロージャは文字列を返す必要があります。 返された文字列が既存のビューに対応する場合、そのビューがレンダリングされます。 それ以外の場合、返された文字列はインライン Blade ビューとして評価されます。

<a name="additional-dependencies"></a>
#### 追加の依存関係

コンポーネントが Laravel の [サービス コンテナ](/docs/{{version}}/container) からの依存関係を必要とする場合、コンポーネントのデータ属性の前にそれらをリストすると、それらはコンテナによって自動的に挿入されます。

```php
App\Services\AlertCreator を使用します。

/**
  * コンポーネントのインスタンスを作成します。
  */
パブリック関数 __construct(
     public AlertCreator $creator,
     パブリック文字列 $type、
     パブリック文字列 $message、
) {}
「」

<a name="hiding-attributes-and-methods"></a>
#### 属性/メソッドの非表示

一部のパブリック メソッドまたはプロパティが変数としてコンポーネント テンプレートに公開されるのを防ぎたい場合は、それらをコンポーネントの `$exc` 配列プロパティに追加できます。

     <?php

     名前空間 App\View\Components;

     Illuminate\View\Component を使用します。

     クラス Alert はコンポーネントを拡張します
     {
         /**
          * コンポーネント テンプレートに公開すべきではないプロパティ/メソッド。
          *
          * @var 配列
          */
         protected $excel = ['type'];

         /**
          * コンポーネントのインスタンスを作成します。
          */
         パブリック関数 __construct(
             パブリック文字列 $type、
         ) {}
     }

<a name="コンポーネント属性"></a>
### コンポーネントの属性

データ属性をコンポーネントに渡す方法はすでに検討しました。 ただし、コンポーネントが機能するために必要なデータの一部ではない、「class」などの追加の HTML 属性を指定する必要がある場合があります。 通常、これらの追加属性はコンポーネント テンプレートのルート要素に渡します。 たとえば、次のように「alert」コンポーネントをレンダリングしたいとします。

「」ブレード
<x-alert type="error" :message="$message" class="mt-4"/>
「」

コンポーネントのコンストラクターの一部ではないすべての属性は、コンポーネントの「属性バッグ」に自動的に追加されます。 この属性バッグは、`$attributes` 変数を介してコンポーネントで自動的に利用できるようになります。 この変数をエコーすることで、すべての属性をコンポーネント内でレンダリングできます。

「」ブレード
<div {{ $attributes }}>
     <!-- コンポーネントの内容 -->
</div>
「」

> **警告**
> 現時点では、コンポーネントタグ内での「@env」などのディレクティブの使用はサポートされていません。 たとえば、`<x-alert :live="@env('production')"/>` はコンパイルされません。

<a name="default-merged-attributes"></a>
#### デフォルト/結合された属性

場合によっては、属性のデフォルト値を指定したり、コンポーネントの属性の一部に追加の値をマージしたりすることが必要な場合があります。 これを実現するには、属性バッグの `merge` メソッドを使用できます。 このメソッドは、コンポーネントに常に適用する必要がある一連のデフォルト CSS クラスを定義する場合に特に役立ちます。

「」ブレード
<div {{ $attributes->merge(['class' => 'alert アラート-'.$type]) }}>
     {{ $メッセージ }}
</div>
「」

このコンポーネントが次のように利用されると仮定すると、次のようになります。

「」ブレード
<x-alert type="error" :message="$message" class="mb-4"/>
「」

コンポーネントの最終的にレンダリングされた HTML は次のように表示されます。

「」ブレード
<div class="mb-4 アラート アラート-エラー">
     <!-- $message 変数の内容 -->
</div>
「」

<a name="conditionally-merge-classes"></a>
#### 条件付きクラスのマージ

特定の条件が「true」の場合にクラスをマージしたい場合があります。 これは、「class」メソッドを使用して実行できます。このメソッドは、追加したいクラスを配列キーに含み、値がブール式であるクラスの配列を受け入れます。 配列要素に数値キーがある場合、その要素は常に表示されるクラス リストに含まれます。

「」ブレード
<div {{ $attributes->class(['p-4', 'bg-red' => $hasError]) }}>
     {{ $メッセージ }}
</div>
「」

他の属性をコンポーネントにマージする必要がある場合は、「merge」メソッドを「class」メソッドに連鎖させることができます。

「」ブレード
<button {{ $attributes->class(['p-4'])->merge(['type' => 'button']) }}>
     {{ $スロット }}
</ボタン>
「」

> **注意**
> マージされた属性を受け取るべきでない他の HTML 要素のクラスを条件付きでコンパイルする必要がある場合は、[`@class` ディレクティブ](#conditional-classes) を使用できます。

<a name="非クラス属性マージ"></a>
#### 非クラス属性の結合

「class」属性ではない属性をマージする場合、「merge」メソッドに指定された値は属性の「デフォルト」値とみなされます。 ただし、「class」属性とは異なり、これらの属性は挿入された属性値とマージされません。 代わりに、上書きされます。 たとえば、「ボタン」コンポーネントの実装は次のようになります。

```blade
<button {{ $attributes->merge(['type' => 'button']) }}>
    {{ $slot }}
</button>
```

カスタム `type` を使用してボタン コンポーネントをレンダリングするには、コンポーネントを使用するときにそれを指定できます。 タイプが指定されていない場合は、`button` タイプが使用されます。

「」ブレード
<x-button type="送信">
     送信
</xボタン>
「」

この例の「button」コンポーネントのレンダリングされた HTML は次のようになります。

「」ブレード
<button type="送信">
     送信
</ボタン>
「」

「class」以外の属性にそのデフォルト値と注入された値を結合させたい場合は、「prepends」メソッドを使用できます。 この例では、`data-controller` 属性は常に `profile-controller` で始まり、追加で挿入された `data-controller` 値はこのデフォルト値の後に配置されます。

「」ブレード
<div {{ $attributes->merge(['data-controller' => $attributes->prepends('profile-controller')]) }}>
     {{ $スロット }}
</div>
「」

<a name="フィルタリング属性"></a>
#### 属性の取得とフィルタリング

`filter` メソッドを使用して属性をフィルタリングできます。 このメソッドは、属性バッグに属性を保持したい場合に `true` を返すクロージャを受け入れます。

「」ブレード
{{ $attributes->filter(fn (string $value, string $key) => $key == 'foo') }}
「」

便宜上、「whereStartsWith」メソッドを使用して、キーが指定された文字列で始まるすべての属性を取得できます。

「」ブレード
{{ $attributes->whereStartsWith('wire:model') }}
「」

逆に、`whereDoesntStartWith` メソッドを使用して、キーが指定された文字列で始まるすべての属性を除外することもできます。

「」ブレード
{{ $attributes->whereDoesntStartWith('wire:model') }}
「」

`first` メソッドを使用すると、指定された属性バッグ内の最初の属性をレンダリングできます。

「」ブレード
{{ $attributes->whereStartsWith('wire:model')->first() }}
「」

コンポーネントに属性が存在するかどうかを確認したい場合は、`has` メソッドを使用できます。 このメソッドは、属性名を唯一の引数として受け入れ、属性が存在するかどうかを示すブール値を返します。

「」ブレード
@if ($attributes->has('class'))
     <div>クラス属性が存在します</div>
@endif
「」

`get` メソッドを使用して特定の属性の値を取得できます。

「」ブレード
{{ $attributes->get('class') }}
「」

<a name="reserved-keywords"></a>
### 予約済みキーワード

デフォルトでは、一部のキーワードはコンポーネントをレンダリングするために Blade の内部使用のために予約されています。 次のキーワードは、コンポーネント内のパブリック プロパティまたはメソッド名として定義できません。

<div class="content-list" markdown="1">

- 「データ」
- 「レンダリング」
- `resolveView`
- ` shouldRender `
- 「見る」
- `withAttributes`
- `withName`

</div>

<a name="スロット"></a>
### スロット

多くの場合、追加のコンテンツを「スロット」経由でコンポーネントに渡す必要があります。 コンポーネント スロットは、`$slot` 変数をエコーすることによってレンダリングされます。 この概念を詳しく調べるために、「alert」コンポーネントに次のマークアップがあると想像してみましょう。

「」ブレード
<!-- /resources/views/components/alert.blade.php -->

<div class="警告アラート-危険">
     {{ $スロット }}
</div>
「」

コンポーネントにコンテンツを注入することで、コンテンツを「スロット」に渡すことができます。

「」ブレード
<x-アラート>
     <strong>おっと！</strong> 何か問題が発生しました。
</x-アラート>
「」

場合によっては、コンポーネントがコンポーネント内の異なる場所に複数の異なるスロットをレンダリングする必要がある場合があります。 「タイトル」スロットを挿入できるようにアラート コンポーネントを変更しましょう。

「」ブレード
<!-- /resources/views/components/alert.blade.php -->

<span class="alert-title">{{ $title }}</span>

<div class="警告アラート-危険">
     {{ $スロット }}
</div>
「」

`x-slot` タグを使用して、名前付きスロットの内容を定義できます。 明示的な `x-slot` タグ内にないコンテンツは、`$slot` 変数のコンポーネントに渡されます。

```xml
<x-アラート>
     <x-スロット:タイトル>
         サーバーエラー
     </x-スロット>

     <strong>おっと！</strong> 何か問題が発生しました。
</x-アラート>
「」

<a name="スコープ付きスロット"></a>
#### スコープ付きスロット

Vue などの JavaScript フレームワークを使用したことがある場合は、スロット内のコンポーネントからデータまたはメソッドにアクセスできる「スコープ スロット」に精通しているかもしれません。 コンポーネント上でパブリックメソッドまたはプロパティを定義し、`$component` 変数を介してスロット内のコンポーネントにアクセスすることで、Laravel でも同様の動作を実現できます。 この例では、「x-alert」コンポーネントのコンポーネント クラスにパブリックの「formatAlert」メソッドが定義されていると仮定します。

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
     <x-slot:Heading class="font-bold">
         見出し
     </x-スロット>

     コンテンツ

     <x-slot:footer class="text-sm">
         フッター
     </x-スロット>
</x-カード>
「」

スロット属性を操作するには、スロットの変数の「attributes」プロパティにアクセスします。 属性を操作する方法の詳細については、[コンポーネント属性](#component-attributes) に関するドキュメントを参照してください。

「」ブレード
@props([
     '見出し'、
     「フッター」、
])

<div {{ $attributes->class(['border']) }}>
     <h1 {{ $Heading->attributes->class(['text-lg']) }}>
         {{ $見出し }}
     </h1>

     {{ $スロット }}

     <フッター {{ $footer->属性->クラス(['text-gray-700']) }}>
         {{ $フッター }}
     </フッター>
</div>
「」

<a name="inline-component-views"></a>
### インラインコンポーネントビュー

非常に小さなコンポーネントの場合、コンポーネント クラスとコンポーネントのビュー テンプレートの両方を管理するのが面倒に感じる場合があります。 このため、コンポーネントのマークアップを「render」メソッドから直接返すことができます。

     /**
      * コンポーネントを表すビュー/コンテンツを取得します。
      */
     パブリック関数 render(): 文字列
     {
         <<<'ブレード' を返す
             <div class="警告アラート-危険">
                 {{ $スロット }}
             </div>
         刃;
     }

<a name="generate-inline-view-components"></a>
#### インライン ビュー コンポーネントの生成

インライン ビューをレンダリングするコンポーネントを作成するには、`make:component` コマンドを実行するときに `inline` オプションを使用できます。

```シェル
php 職人 make:component アラート --inline
「」

<a name="dynamic-components"></a>
### 動的コンポーネント

コンポーネントをレンダリングする必要があるが、実行時までどのコンポーネントをレンダリングすべきかわからない場合があります。 この状況では、Laravel の組み込み `dynamic-component` コンポーネントを使用して、実行時の値または変数に基づいてコンポーネントをレンダリングできます。

「」ブレード
<x-dynamic-component :component="$componentName" class="mt-4" />
「」

<a name="手動登録コンポーネント"></a>
### コンポーネントを手動で登録する

> **警告**
> コンポーネントの手動登録に関する次のドキュメントは、主にビューコンポーネントを含む Laravel パッケージを作成している人に適用されます。 パッケージを作成していない場合、コンポーネントのドキュメントのこの部分は関係ない可能性があります。

独自のアプリケーションのコンポーネントを作成する場合、コンポーネントは `app/View/Components` ディレクトリおよび `resources/views/components` ディレクトリ内で自動的に検出されます。

ただし、Blade コンポーネントを利用するパッケージを構築する場合、またはコンポーネントを従来とは異なるディレクトリに配置する場合は、Laravel がコンポーネントの場所を認識できるように、コンポーネント クラスとその HTML タグのエイリアスを手動で登録する必要があります。 通常、コンポーネントはパッケージのサービスプロバイダーの「boot」メソッドに登録する必要があります。

     Illuminate\Support\Facades\Blade を使用します。
     VendorPackage\View\Components\AlertComponent を使用します。

     /**
      * パッケージのサービスをブートストラップします。
      */
     パブリック関数 boot(): void
     {
         Blade::component('パッケージアラート', AlertComponent::class);
     }

コンポーネントが登録されると、そのタグ エイリアスを使用してレンダリングできます。

```blade
<x-package-alert/>
```

#### パッケージコンポーネントの自動ロード

あるいは、慣例に従って `componentNamespace` メソッドを使用してコンポーネント クラスを自動ロードすることもできます。 たとえば、「Nightshade」パッケージには、「Package\Views\Components」名前空間内に存在する「Calendar」コンポーネントと「ColorPicker」コンポーネントが含まれる場合があります。

     Illuminate\Support\Facades\Blade を使用します。

     /**
      * パッケージのサービスをブートストラップします。
      */
     パブリック関数 boot(): void
     {
         Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
     }

これにより、ベンダー名前空間で「package-name::」構文を使用してパッケージ コンポーネントを使用できるようになります。

「」ブレード
<x-nightshade::calendar />
<x-nightshade::color-picker />
「」

Blade は、コンポーネント名をパスカル文字に変換することで、このコンポーネントにリンクされているクラスを自動的に検出します。 サブディレクトリは、「ドット」表記を使用してサポートされています。

<a name="anonymous-components"></a>
## 匿名コンポーネント

インライン コンポーネントと同様に、匿名コンポーネントは、単一のファイルを介してコンポーネントを管理するメカニズムを提供します。 ただし、匿名コンポーネントは単一のビュー ファイルを使用し、関連するクラスを持ちません。 匿名コンポーネントを定義するには、「resources/views/components」ディレクトリ内に Blade テンプレートを配置するだけです。 たとえば、`resources/views/components/alert.blade.php` でコンポーネントを定義したと仮定すると、次のように単純にレンダリングできます。

「」ブレード
<x-アラート/>
「」

「.」文字を使用して、コンポーネントが「components」ディレクトリのさらに深くネストされているかどうかを示すことができます。 たとえば、コンポーネントが `resources/views/components/inputs/button.blade.php` で定義されていると仮定すると、次のようにレンダリングできます。

「」ブレード
<x-inputs.button/>
「」

<a name="anonymous-index-components"></a>
### 匿名インデックスコンポーネント

コンポーネントが多数の Blade テンプレートで構成されている場合、特定のコンポーネントのテンプレートを 1 つのディレクトリ内にグループ化したい場合があります。 たとえば、次のディレクトリ構造を持つ「accordion」コンポーネントを想像してください。

「なし」
/resources/views/components/accordion.blade.php
/resources/views/components/accordion/item.blade.php
「」

このディレクトリ構造により、アコーディオン コンポーネントとその項目を次のようにレンダリングできます。

「」ブレード
<x-アコーディオン>
     <x-accordion.item>
         ...
     </x-accordion.item>
</x-アコーディオン>
「」

ただし、「x-accordion」を介してアコーディオンコンポーネントをレンダリングするには、「index」アコーディオンコンポーネントテンプレートを「accordion」ディレクトリ内にネストするのではなく、「resources/views/components」ディレクトリに配置する必要がありました。 その他のアコーディオン関連のテンプレート。

ありがたいことに、Blade ではコンポーネントのテンプレート ディレクトリ内に「index.blade.php」ファイルを配置できます。 コンポーネントに「index.blade.php」テンプレートが存在する場合、それはコンポーネントの「ルート」ノードとしてレンダリングされます。 したがって、上記の例で示した同じ Blade 構文を引き続き使用できます。 ただし、ディレクトリ構造は次のように調整します。

「なし」
/resources/views/components/accordion/index.blade.php
/resources/views/components/accordion/item.blade.php
「」

<a name="データプロパティ属性"></a>
### データのプロパティ/属性

匿名コンポーネントには関連付けられたクラスがないため、どのデータを変数としてコンポーネントに渡す必要があるのか、どの属性をコンポーネントの [属性バッグ](#component-attributes) に配置する必要があるのかをどのように区別すればよいのか疑問に思うかもしれません。

コンポーネントの Blade テンプレートの先頭にある `@props` ディレクティブを使用して、どの属性をデータ変数と見なすかを指定できます。 コンポーネントの他のすべての属性は、コンポーネントの属性バッグを介して利用可能になります。 データ変数にデフォルト値を与えたい場合は、変数の名前を配列キーとして指定し、デフォルト値を配列値として指定できます。

「」ブレード
<!-- /resources/views/components/alert.blade.php -->

@props(['タイプ' => '情報', 'メッセージ'])

<div {{ $attributes->merge(['class' => 'alert アラート-'.$type]) }}>
     {{ $メッセージ }}
</div>
「」

上記のコンポーネント定義を考慮すると、次のようにコンポーネントをレンダリングできます。

```blade
<x-alert type="error" :message="$message" class="mb-4"/>
```

<a name="accessing-parent-data"></a>
### 親データへのアクセス

場合によっては、子コンポーネント内の親コンポーネントからデータにアクセスしたい場合があります。 このような場合、`@aware` ディレクティブを使用できます。 たとえば、親 `<x-menu>` と子 `<x-menu.item>` で構成される複雑なメニュー コンポーネントを構築していると想像してください。

「」ブレード
<x-メニュー カラー="パープル">
     <x-menu.item>...</x-menu.item>
     <x-menu.item>...</x-menu.item>
</x-メニュー>
「」

`<x-menu>` コンポーネントには次のような実装が含まれる場合があります。

「」ブレード
<!-- /resources/views/components/menu/index.blade.php -->

@props(['カラー' => 'グレー'])

<ul {{ $attributes->merge(['class' => 'bg-'.$color.'-200']) }}>
     {{ $スロット }}
</ul>
「」

`color` プロパティは親 (`<x-menu>`) にのみ渡されたため、`<x-menu.item>` 内では使用できません。 ただし、`@aware` ディレクティブを使用すると、`<x-menu.item>` 内でも使用できるようになります。

「」ブレード
<!-- /resources/views/components/menu/item.blade.php -->

@aware(['カラー' => 'グレー'])

<li {{ $attributes->merge(['class' => 'text-'.$color.'-800']) }}>
     {{ $スロット }}
</li>
「」

> **警告**
> `@aware` ディレクティブは、HTML 属性を介して親コンポーネントに明示的に渡されていない親データにはアクセスできません。 親コンポーネントに明示的に渡されないデフォルトの `@props` 値には、`@aware` ディレクティブではアクセスできません。

<a name="匿名コンポーネントのパス"></a>
### 匿名コンポーネントのパス

前に説明したように、匿名コンポーネントは通常、「resources/views/components」ディレクトリ内に Blade テンプレートを配置することによって定義されます。 ただし、デフォルトのパスに加えて、他の匿名コンポーネントのパスを Laravel に登録したい場合もあります。

`anonymousComponentPath` メソッドは、匿名コンポーネントの場所への「パス」を最初の引数として受け入れ、コンポーネントを配置するオプションの「名前空間」を 2 番目の引数として受け入れます。 通常、このメソッドは、アプリケーションの [サービス プロバイダー](/docs/{{version}}/providers) の 1 つの `boot` メソッドから呼び出す必要があります。

     /**
      * あらゆるアプリケーション サービスをブートストラップします。
      */
     パブリック関数 boot(): void
     {
         Blade::anonymousComponentPath(__DIR__.'/../components');
     }

上記の例のように、コンポーネント パスがプレフィックスを指定せずに登録されている場合、ブレード コンポーネントでも対応するプレフィックスなしでレンダリングされる可能性があります。 たとえば、上記で登録したパスに `panel.blade.php` コンポーネントが存在する場合、次のようにレンダリングされます。

「」ブレード
<x パネル />
「」

プレフィックス「namespaces」は、「anonymousComponentPath」メソッドの 2 番目の引数として指定できます。

     Blade::anonymousComponentPath(__DIR__.'/../components', 'ダッシュボード');

プレフィックスが指定されている場合、コンポーネントがレンダリングされるときに、コンポーネントの名前空間にコンポーネント名をプレフィックスとして付けることによって、その「名前空間」内のコンポーネントがレンダリングされることがあります。

「」ブレード
<x-ダッシュボード::パネル />
「」

<a name="建物レイアウト"></a>
## 建物のレイアウト

<a name="layouts-using-components"></a>
### コンポーネントを使用したレイアウト

ほとんどの Web アプリケーションは、さまざまなページにわたって同じ一般的なレイアウトを維持します。 作成するすべてのビューでレイアウト HTML 全体を繰り返す必要がある場合、アプリケーションを保守するのは非常に面倒で困難になります。 ありがたいことに、このレイアウトを単一の [Blade コンポーネント](#components) として定義し、アプリケーション全体で使用すると便利です。

<a name="レイアウト コンポーネントの定義"></a>
#### レイアウト コンポーネントの定義

たとえば、「todo」リスト アプリケーションを構築していると想像してください。 次のような `layout` コンポーネントを定義するとします。

「」ブレード
<!-- リソース/ビュー/コンポーネント/layout.blade.php -->

<html>
     <頭>
         <タイトル>{{ $title ?? 'Todo マネージャー' }}</title>
     </head>
     <本文>
         <h1>Todos</h1>
         <hr/>
         {{ $スロット }}
     </body>
</html>
「」

<a name="レイアウト コンポーネントの適用"></a>
#### レイアウト コンポーネントの適用

「レイアウト」コンポーネントが定義されたら、そのコンポーネントを利用するブレード ビューを作成できます。 この例では、タスク リストを表示する単純なビューを定義します。

「」ブレード
<!-- リソース/ビュー/tasks.blade.php -->

<x-レイアウト>
     @foreach ($tasks を $task として)
         {{ $タスク }}
     @endforeach
</x-レイアウト>
「」

コンポーネントに挿入されるコンテンツは、`layout` コンポーネント内のデフォルトの `$slot` 変数に提供されることに注意してください。 お気づきかもしれませんが、`layout` は、`$title` スロットが提供されている場合はそれも尊重します。 それ以外の場合は、デフォルトのタイトルが表示されます。 [コンポーネントのドキュメント](#components) で説明されている標準スロット構文を使用して、タスク リスト ビューからカスタム タイトルを挿入できます。

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

レイアウト ビューとタスク リスト ビューを定義したので、あとはルートから「タスク」ビューを返すだけです。

     App\Models\Task を使用します。

     Route::get('/tasks', function () {
         return view('タスク', ['タスク' => Task::all()]);
     });

<a name="layouts-using-template-inheritance"></a>
### テンプレートの継承を使用したレイアウト

<a name="レイアウトの定義"></a>
#### レイアウトの定義

レイアウトは「テンプレートの継承」によって作成することもできます。 これは、[コンポーネント](#components) が導入される前は、アプリケーションを構築する主な方法でした。

まず、簡単な例を見てみましょう。 まず、ページ レイアウトを検討します。 ほとんどの Web アプリケーションはさまざまなページにわたって同じ一般的なレイアウトを維持するため、このレイアウトを単一のブレード ビューとして定義すると便利です。

「」ブレード
<!-- リソース/ビュー/レイアウト/app.blade.php -->

<html>
     <頭>
         <title>アプリ名 - @yield('title')</title>
     </head>
     <本文>
         @section('サイドバー')
             これはマスターサイドバーです。
         @見せる

         <div class="コンテナ">
             @yield('コンテンツ')
         </div>
     </body>
</html>
「」

ご覧のとおり、このファイルには典型的な HTML マークアップが含まれています。 ただし、「@section」ディレクティブと「@yield」ディレクティブに注意してください。 `@section` ディレクティブは、名前が示すとおり、コンテンツのセクションを定義します。一方、`@yield` ディレクティブは、指定されたセクションのコンテンツを表示するために使用されます。

アプリケーションのレイアウトを定義したので、そのレイアウトを継承する子ページを定義しましょう。

<a name="extending-a-layout"></a>
#### レイアウトの拡張

子ビューを定義するときは、「@extends」Blade ディレクティブを使用して、子ビューが「継承」するレイアウトを指定します。 Blade レイアウトを拡張するビューは、「@section」ディレクティブを使用してコンテンツをレイアウトのセクションに挿入できます。 上の例に見られるように、これらのセクションの内容は `@yield` を使用してレイアウトに表示されることに注意してください。

「」ブレード
<!-- resource/views/child.blade.php -->

@extends('layouts.app')

@section('タイトル', 'ページタイトル')

@section('サイドバー')
     @@親

     <p>これはマスター サイドバーに追加されます。</p>
@endsection

@section('コンテンツ')
     <p>これは私の本文の内容です。</p>
@endsection
「」

この例では、`sidebar` セクションが `@@parent` ディレクティブを利用して、コンテンツを (上書きではなく) レイアウトのサイドバーに追加しています。 `@@parent` ディレクティブは、ビューがレンダリングされるときにレイアウトのコンテンツに置き換えられます。

> **注意**
> 前の例とは異なり、この `sidebar` セクションは `@show` ではなく `@endsection` で終わります。 `@endsection` ディレクティブはセクションを定義するだけですが、`@show` はセクションを定義して **即座に生成**します。

`@yield` ディレクティブは、2 番目のパラメータとしてデフォルト値も受け入れます。 この値は、生成されるセクションが未定義の場合に表示されます。

「」ブレード
@yield('コンテンツ', 'デフォルトのコンテンツ')
「」

<a name="forms"></a>
## フォーム

<a name="csrf-field"></a>
### CSRFフィールド

アプリケーションで HTML フォームを定義するときは常に、[CSRF 保護](/docs/{{version}}/csrf) ミドルウェアがリクエストを検証できるように、フォームに非表示の CSRF トークン フィールドを含める必要があります。 `@csrf` Blade ディレクティブを使用してトークン フィールドを生成できます。

「」ブレード
<フォームメソッド="POST" アクション="/プロファイル">
     @csrf

     ...
</form>
「」

<a name="メソッドフィールド"></a>
### メソッドフィールド

HTML フォームでは `PUT`、`PATCH`、または `DELETE` リクエストを作成できないため、これらの HTTP 動詞を偽装するには、非表示の `_method` フィールドを追加する必要があります。 `@method` Blade ディレクティブを使用すると、このフィールドを作成できます。

```blade
<form action="/foo/bar" method="POST">
    @method('PUT')

    ...
</form>
```

<a name="validation-errors"></a>
### 検証エラー

`@error` ディレクティブを使用すると、特定の属性に [検証エラー メッセージ](/docs/{{version}}/validation#quick-displaying-the-validation-errors) が存在するかどうかをすばやく確認できます。 `@error` ディレクティブ内で、`$message` 変数をエコーしてエラー メッセージを表示できます。

「」ブレード
<!-- /resources/views/post/create.blade.php -->

<label for="title">投稿タイトル</label>

<input id="タイトル"
     type="テキスト"
     class="@error('title') は無効です @enderror">

@error('タイトル')
     <div class="alert warning-danger">{{ $message }}</div>
@enderror
「」

`@error` ディレクティブは "if" ステートメントにコンパイルされるため、属性にエラーがない場合は `@else` ディレクティブを使用してコンテンツをレンダリングできます。

「」ブレード
<!-- /resources/views/auth.blade.php -->

<label for="email">メールアドレス</label>

<input id="メールアドレス"
     type="電子メール"
     class="@error('email') は無効です @else は有効です @enderror">
「」

[特定のエラー バッグの名前](/docs/{{version}}/validation#named-error-bags) を `@error` ディレクティブの 2 番目のパラメーターとして渡して、複数のエラー バッグを含むページ上の検証エラー メッセージを取得できます。 フォーム:

「」ブレード
<!-- /resources/views/auth.blade.php -->

<label for="email">メールアドレス</label>

<input id="メールアドレス"
     type="電子メール"
     class="@error('email', 'login') は無効です @enderror">

@error('電子メール', 'ログイン')
     <div class="alert warning-danger">{{ $message }}</div>
@enderror
「」

<a name="スタック"></a>
## スタック

Blade を使用すると、別のビューまたはレイアウトの別の場所にレンダリングできる名前付きスタックにプッシュできます。 これは、子ビューで必要な JavaScript ライブラリを指定する場合に特に役立ちます。

「」ブレード
@push('スクリプト')
     <script src="/example.js"></script>
@endpush
「」

指定されたブール式が「true」と評価された場合にコンテンツを「@push」したい場合は、「@pushIf」ディレクティブを使用できます。

「」ブレード
@pushIf($ shouldPush, 'スクリプト')
     <script src="/example.js"></script>
@endPushIf
「」

必要に応じて何度でもスタックにプッシュできます。 完全なスタックの内容をレンダリングするには、スタックの名前を `@stack` ディレクティブに渡します。

「」ブレード
<頭>
     <!-- 先頭の内容 -->

     @stack('スクリプト')
</head>
「」

コンテンツをスタックの先頭に追加したい場合は、`@prepend` ディレクティブを使用する必要があります。

「」ブレード
@push('スクリプト')
     これは二番目になります...
@endpush

// 後で...

@prepend('スクリプト')
     これが最初になります...
@endprepend
「」

<a name="サービスインジェクション"></a>
## サービスインジェクション

`@inject` ディレクティブは、Laravel [サービスコンテナ](/docs/{{version}}/container) からサービスを取得するために使用できます。 `@inject` に渡される最初の引数はサービスが配置される変数の名前であり、2 番目の引数は解決したいサービスのクラス名またはインターフェイス名です。

```blade
@inject('metrics', 'App\Services\MetricsService')

<div>
    Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
</div>
```

<a name="rendering-inline-blade-templates"></a>
## インライン ブレード テンプレートのレンダリング

場合によっては、生の Blade テンプレート文字列を有効な HTML に変換する必要があるかもしれません。 これは、「Blade」ファサードによって提供される「render」メソッドを使用して実行できます。 「render」メソッドは、Blade テンプレート文字列と、テンプレートに提供するオプションのデータ配列を受け入れます。

```php
Illuminate\Support\Facades\Blade を使用します。

return Blade::render('こんにちは、{{ $name }}', ['name' => 'ジュリアン・バシール']);
「」

Laravel は、インライン Blade テンプレートを「storage/framework/views」ディレクトリに書き込むことによってレンダリングします。 Blade テンプレートのレンダリング後に Laravel にこれらの一時ファイルを削除させたい場合は、メソッドに `deleteCachedView` 引数を指定できます。

```php
return Blade::render(
     「こんにちは、{{ $name }}」、
     ['名前' => 'ジュリアン・バシール'],
     deleteCachedView: true
);
「」

<a name="rendering-blade-fragments"></a>
## ブレードの破片のレンダリング

[Turbo](https://turbo.hotwired.dev/) や [htmx](https://htmx.org/) などのフロントエンド フレームワークを使用する場合、Blade テンプレートの一部のみを返す必要がある場合があります。 HTTP 応答。 ブレードの「フラグメント」を使用すると、まさにそれが可能になります。 まず、Blade テンプレートの一部を `@fragment` および `@endfragment` ディレクティブ内に配置します。

「」ブレード
@fragment('ユーザーリスト')
     <ul>
         @foreach ($users として $user)
             <li>{{ $user->名前 }}</li>
         @endforeach
     </ul>
@endfragment
「」

次に、このテンプレートを利用するビューをレンダリングするときに、「fragment」メソッドを呼び出して、指定されたフラグメントのみが送信 HTTP 応答に含まれるように指定できます。

```php
return view('ダッシュボード', ['ユーザー' => $ユーザー])->fragment('ユーザーリスト');
「」

`fragmentIf` メソッドを使用すると、指定された条件に基づいて条件付きでビューのフラグメントを返すことができます。 それ以外の場合は、ビュー全体が返されます。

```php
return view('ダッシュボード', ['ユーザー' => $ユーザー])
     ->fragmentIf($request->hasHeader('HX-Request'), 'user-list');
「」

`fragments` メソッドと `fragmentsIf` メソッドを使用すると、応答で複数のビュー フラグメントを返すことができます。 フラグメントは連結されます。

```php
view('ダッシュボード', ['ユーザー' => $ユーザー])
     ->fragments(['ユーザーリスト', 'コメントリスト']);

view('ダッシュボード', ['ユーザー' => $ユーザー])
     ->フラグメントIf(
         $request->hasHeader('HX-Request'),
         ['ユーザーリスト', 'コメントリスト']
     );
「」

<a name="extending-blade"></a>
## 延長ブレード

Blade では、「directive」メソッドを使用して独自のカスタム ディレクティブを定義できます。 Blade コンパイラはカスタム ディレクティブを検出すると、ディレクティブに含まれる式を使用して提供されたコールバックを呼び出します。

次の例では、指定された `$var` をフォーマットする `@datetime($var)` ディレクティブを作成します。これは、`DateTime` のインスタンスである必要があります。

     <?php

     名前空間 App\Providers;

     Illuminate\Support\Facades\Blade を使用します。
     Illuminate\Support\ServiceProvider を使用します。

     クラス AppServiceProvider は ServiceProvider を拡張します
     {
         /**
          ※各種アプリケーションサービスを登録します。
          */
         パブリック関数 register(): void
         {
             // ...
         }

         /**
          * あらゆるアプリケーション サービスをブートストラップします。
          */
         パブリック関数 boot(): void
         {
             Blade::directive('datetime', function (string $expression) {
                 return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
             });
         }
     }

ご覧のとおり、ディレクティブに渡される式に「format」メソッドを連鎖させます。 したがって、この例では、このディレクティブによって生成される最終的な PHP は次のようになります。

     <?php echo ($var)->format('m/d/Y H:i'); ?>

> **警告**
> Blade ディレクティブのロジックを更新した後、キャッシュされた Blade ビューをすべて削除する必要があります。 キャッシュされたブレード ビューは、「view:clear」アーティザン コマンドを使用して削除できます。

<a name="custom-echo-handlers"></a>
### カスタム エコー ハンドラー

Blade を使用してオブジェクトを「エコー」しようとすると、オブジェクトの __toString メソッドが呼び出されます。 [`__toString`](https://www.php.net/manual/en/ language.oop5.magic.php#object.tostring) メソッドは、PHP の組み込み「マジック メソッド」の 1 つです。 ただし、対話しているクラスがサードパーティのライブラリに属している場合など、特定のクラスの __toString メソッドを制御できない場合があります。

このような場合、Blade では、その特定の種類のオブジェクトにカスタム エコー ハンドラーを登録できます。 これを実現するには、Blade の「stringable」メソッドを呼び出す必要があります。 `stringable` メソッドはクロージャを受け入れます。 このクロージャは、レンダリングを担当するオブジェクトのタイプをタイプヒントする必要があります。 通常、「stringable」メソッドは、アプリケーションの「AppServiceProvider」クラスの「boot」メソッド内で呼び出す必要があります。

     Illuminate\Support\Facades\Blade を使用します。
     Money\Money を使用します。

     /**
      * あらゆるアプリケーション サービスをブートストラップします。
      */
     パブリック関数 boot(): void
     {
         Blade::stringable(function (Money $money) {
             return $money->formatTo('en_GB');
         });
     }

カスタム エコー ハンドラーを定義したら、Blade テンプレート内のオブジェクトをエコーするだけです。

「」ブレード
コスト: {{ $money }}
「」

<a name="custom-if-statements"></a>
### カスタムの If ステートメント

カスタム ディレクティブのプログラミングは、単純なカスタム条件文を定義する場合、必要以上に複雑になる場合があります。 そのため、Blade は、クロージャを使用してカスタム条件ディレクティブを迅速に定義できる `Blade::if` メソッドを提供します。 たとえば、アプリケーションに設定されたデフォルトの「ディスク」をチェックするカスタム条件を定義してみましょう。 これは、`AppServiceProvider` の `boot` メソッドで行うことができます。

     Illuminate\Support\Facades\Blade を使用します。

     /**
      * あらゆるアプリケーション サービスをブートストラップします。
      */
     パブリック関数 boot(): void
     {
         Blade::if('ディスク', function (string $value) {
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
