# アップグレードガイド

- [9.x から 10.0 へのアップグレード](#upgrade-10.0)

<a name="high-impact-changes"></a>
## 影響度の高い変更点

<div class="content-list" markdown="1">

- [依存関係の更新](#updating-dependencies)
- [最小安定性の更新](#updating-minimum-stability)

</div>

<a name="medium-impact-changes"></a>
## 影響が中程度の変更点

<div class="content-list" markdown="1">

- [データベース構文](#database-expressions)
- [モデルの "Dates" プロパティ](#model-dates-property)
- [Monolog 3](#monolog-3)
- [Redis キャッシュタグ](#redis-cache-tags)
- [サービスのモック](#service-mocking)
- [言語ディレクトリ](#language-directory)

</div>

<a name="low-impact-changes"></a>
## 影響度の低い変更点

<div class="content-list" markdown="1">

- [クロージャバリデーションルールメッセージ](#closure-validation-rule-messages)
- [パブリックパス結合](#public-path-binding)
- [クエリ例外のコンストラクタ](#query-exception-constructor)
- [レート制限の戻り値](#rate-limiter-return-values)
- [リレーションの `getBaseQuery` メソッド](#relation-getbasequery-method)
- [`Redirect::home` メソッド](#redirect-home)
- [`Bus::dispatchNow` メソッド](#dispatch-now)
- [`registerPolicies` メソッド](#register-policies)
- [ULID カラム](#ulid-columns)

</div>

<a name="upgrade-10.0"></a>
## 9.x から 10.0 へのアップグレード

<a name="estimated-upgrade-time-??-minutes"></a>
#### 推定アップグレード時間: 10 分

> **Note**  
> 重大な変更をすべて文書化するよう努めています。 これらの重大な変更の一部はフレームワークの一部分にあるため、これらの変更の一部のみが実際にアプリケーションに影響を与える可能性があります。 時間を節約したいですか？ [Laravel Shift](https://laravelshift.com/) を使用して、アプリケーションのアップグレードを自動化できます。

<a name="updating-dependencies"></a>
### 依存関係の更新

**影響の可能性: 高**

#### PHP 8.1.0 が必要

Laravel には、PHP 8.1.0 以降が必要になりました。

#### Composer 2.2.0 が必要

Laravel には [Composer](https://getcomposer.org) 2.2.0 以降が必要になりました。

#### Composer の依存関係

アプリケーションの `composer.json` ファイルで次の依存関係を更新する必要があります。

<div class="content-list" markdown="1">

- `laravel/framework` を `^10.0` に更新
- `laravel/sanctum` を `^3.2` に更新
- `doctrine/dbal` を `^3.0` に更新
- `spatie/laravel-ignition` を `^2.0` に更新
- `laravel/passport` を `^11.0` に更新 ([アップグレードガイド](https://github.com/laravel/passport/blob/11.x/UPGRADE.md))

</div>

Sanctum 3.x へのアップグレードを 2.x リリースシリーズから行う場合は、[Sanctum アップグレード ガイド](https://github.com/laravel/sanctum/blob/3.x/UPGRADE.md)を参照してください。

さらに、[PHPUnit 10](https://phpunit.de/announcements/phpunit-10.html) を使用したい場合は、アプリケーションの `phpunit.xml` 設定ファイルの `<coverage>` セクションから `processUncoveredFiles` 属性を削除してください。次に、アプリケーションの `composer.json` ファイルで以下の依存関係を更新してください。

<div class="content-list" markdown="1">

- `nunomaduro/collision` を `^7.0` に更新
- `phpunit/phpunit` を `^10.0` に更新

</div>

最後に、アプリケーションで使用されている他のサードパーティパッケージを調べ、Laravel 10 のサポートに適切なバージョンを使用していることを確認してください。

<a name="updating-minimum-stability"></a>
#### 最小安定性

アプリケーションの `composer.json` ファイルで `minimum-stability` 設定を `stable` に更新する必要があります。また、`minimum-stability` のデフォルト値が `stable` であるため、アプリケーションの `composer.json` ファイルからこの設定を削除することもできます。

```json
"minimum-stability": "stable",
```

###  アプリケーション

<a name="public-path-binding"></a>
#### パブリックパス結合

**影響の可能性: 低**

アプリケーションがコンテナに `path.public` を結合することで "パブリックパス" をカスタマイズしている場合、代わりに Illuminate\Foundation\Application オブジェクトが提供する usePublicPath メソッドを呼び出すようにコードを更新する必要があります。

```php
app()->usePublicPath(__DIR__.'/public');
```

### 認可

<a name="register-policies"></a>
### `registerPolicies` メソッド

**影響の可能性: 低**

`AuthServiceProvider` の `registerPolicies` メソッドは、フレームワークによって自動的に呼び出されるようになりました。 したがって、アプリケーションの `AuthServiceProvider` の `boot` メソッドからこのメソッドへの呼び出しを削除できます。

### キャッシュ

<a name="redis-cache-tags"></a>
#### Redis キャッシュタグ

**影響の可能性: 中**

Redis [キャッシュ タグ](/docs/{{version}}/cache#cache-tags) のサポートが書き直され、パフォーマンスとストレージ効率が向上しました。 Laravel の以前のリリースでは、アプリケーションのキャッシュドライバとして Redis を使用すると、古いキャッシュタグがキャッシュに蓄積されていました。

ただし、古いキャッシュ タグ エントリを適切に削除するには、Laravel の新しい `cache:prune-stale-tags` Artisan コマンドをアプリケーションの `App\Console\Kernel` で [スケジュール](/docs/{{version}}/scheduling) する必要があります。 クラス：

    $schedule->command('cache:prune-stale-tags')->hourly();

### データベース

<a name="database-expressions"></a>
#### データベース構文

**影響の可能性: 中**

データベースの「構文」（通常は `DB::raw` を介して生成される）は、`Laravel 10.x` で将来的に追加機能を提供するために再構築されました。特に、文法の生の文字列値は、現在、構文の `getValue(Grammar $grammar)` メソッドを介して取得する必要があります。 `(string)` を使って構文を文字列にキャストすることは、すでにサポートされていません。

通常、これはエンドユーザーアプリケーションに影響しません。ただし、アプリケーションがデータベース構文を `(string)` を使って文字列にキャストしたり、構文の `__toString` メソッドを直接呼び出したりしている場合は、代わりに `getValue` メソッドを呼び出すようにコードを更新する必要があります。

```php
use Illuminate\Support\Facades\DB;

$expression = DB::raw('select 1');

$string = $expression->getValue(DB::connection()->getQueryGrammar());
```

<a name="query-exception-constructor"></a>
#### クエリ例外コンストラクタ

**影響の可能性: 非常に低い**

`Illuminate\Database\QueryException` コンストラクタは、文字列の接続名を最初の引数として受け入れるようになりました。 アプリケーションがこの例外を手動でスローしている場合は、それに応じてコードを調整する必要があります。

<a name="ulid-columns"></a>
#### ULID カラム

**影響の可能性: 低**

マイグレーションが引数なしで `ulid` メソッドを呼び出すと、カラムは `ulid` という名前になります。 以前のリリースの Laravel では、このメソッドを引数なしで呼び出すと、誤って `uuid` という名前の列が作成されました。

     $table->ulid();

`ulid` メソッドを呼び出すときに列名を明示的に指定するには、列名をメソッドに渡します。

     $table->ulid('ulid');

### Eloquent

<a name="model-dates-property"></a>
#### モデル "Dates" プロパティ

**影響の可能性: 中**

Eloquent モデルの非推奨の `$dates` プロパティが削除されました。 代わりに `$casts` プロパティを使用する必要があります。

```php
protected $casts = [
    'deployed_at' => 'datetime',
];
```

<a name="relation-getbasequery-method"></a>
#### リレーション `getBaseQuery` メソッド

**影響の可能性: 非常に低い**

`Illuminate\Database\Eloquent\Relations\Relation` クラスの `getBaseQuery` メソッドの名前が `toBase` に変更されました。

### ローカリゼーション

<a name="language-directory"></a>
#### 言語ディレクトリ

**影響の可能性: なし**

既存のアプリケーションとは関係ありませんが、Laravel アプリケーション スケルトンにはデフォルトで `lang` ディレクトリが含まれなくなりました。 代わりに、新しい Laravel アプリケーションを作成するときは、`lang:publish` Artisan コマンドを使用して公開できます。

```shell
php artisan lang:publish
```

### ログ

<a name="monolog-3"></a>
#### Monolog 3

**影響の可能性: 中**

Laravel の Monolog 依存関係が Monolog 3.x に更新されました。 アプリケーション内で Monolog を直接操作している場合は、Monolog の [アップグレード ガイド](https://github.com/Seldaek/monolog/blob/main/UPGRADE.md) を確認する必要があります。

BugSnag や Rollbar などのサードパーティのログサービスを使用している場合、これらのサードパーティのパッケージを Monolog 3.x および Laravel 10.x をサポートするバージョンにアップグレードする必要がある場合があります。

### キュー

<a name="dispatch-now"></a>
#### `Bus::dispatchNow` メソッド

**影響の可能性: 低**

非推奨の `Bus::dispatchNow` および `dispatch_now` メソッドは削除されました。 代わりに、アプリケーションはそれぞれ `Bus::dispatchSync` メソッドと `dispatch_sync` メソッドを使用する必要があります。

### ルーティング

<a name="middleware-aliases"></a>
#### ミドルウェアのエイリアス

**影響の可能性: オプション**

新しい Laravel アプリケーションでは、`App\Http\Kernel` クラスの `$routeMiddleware` プロパティが、その目的をより明確に反映するために `$middlewareAliases` に名前が変更されました。 既存のアプリケーションでこのプロパティの名前を変更してもかまいませんが、必須ではありません。

<a name="rate-limiter-return-values"></a>
#### レート制限の戻り値

**影響の可能性: 低**

`RateLimiter::attempt` メソッドを呼び出すと、提供されたクロージャによって返される値がメソッドによって返されるようになりました。 何も返されないか `null` が返された場合、`attempt` メソッドは `true` を返します。

```php
$value = RateLimiter::attempt('key', 10, fn () => ['example'], 1);

$value; // ['example']
```

<a name="redirect-home"></a>
#### `Redirect::home` メソッド

**影響の可能性: 非常に低い**

非推奨の `Redirect::home` メソッドは削除されました。 代わりに、アプリケーションは明示的に名前が付けられたルートにリダイレクトする必要があります。

```php
return Redirect::route('home');
```

### テスト

<a name="service-mocking"></a>
#### サービスのモック化

**影響の可能性: 中**

フレームワークから非推奨の `MocksApplicationServices` トレイトが削除されました。このトレイトは、`expectsEvents`、`expectsJobs`、および `expectsNotifications` などのテストメソッドを提供していました。

これらのメソッドを使用しているアプリケーションの場合、それぞれ `Event::fake`、`Bus::fake`、および `Notification::fake` に移行することをお勧めします。フェイクを使ったモック化については、該当するコンポーネントのドキュメントを参照してください。

### バリデーション

<a name="closure-validation-rule-messages"></a>
#### クロージャバリデーションルールのメッセージ

**影響の可能性: 非常に低い**

クロージャベースのカスタムバリデーションルールを記述する場合、`$fail` コールバックを複数回呼び出すと、前のメッセージを上書きする代わりに、メッセージを配列に追加するようになりました。 通常、これによってアプリケーションに影響はありません。

さらに、`$fail` コールバックがオブジェクトを返すようになりました。 以前にバリデーションクロージャの戻り値の型ヒントを使用していた場合、型ヒントを更新する必要があります。

```php
public function rules()
{
    'name' => [
        function ($attribute, $value, $fail) {
            $fail('validation.translation.key')->translate();
        },
    ],
}
```

<a name="その他"></a>
### その他

`laravel/laravel` [GitHub リポジトリ](https://github.com/laravel/laravel) の変更も参照することをお勧めします。これらの変更の多くは必須ではありませんが、これらのファイルをアプリケーションと同期させたい場合があります。このアップグレードガイドでいくつかの変更が取り上げられますが、設定ファイルやコメントの変更などは取り上げられません。

[GitHub の比較ツール](https://github.com/laravel/laravel/compare/9.x...10.x) を使って簡単に変更を確認し、重要な更新を選択できます。ただし、GitHub の比較ツールで表示される変更の多くは、PHP のネイティブタイプの採用によるものです。これらの変更は後方互換性があり、Laravel 10 への移行中にそれらを採用することは任意です。
