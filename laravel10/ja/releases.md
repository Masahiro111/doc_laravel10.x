# リリースノート

- [バージョニング規約](#versioning-scheme)
- [サポートポリシー](#support-policy)
- [Laravel 10](#laravel-10)

<a name="versioning-scheme"></a>
## バージョニング規約

Laravel 及びその他のファーストパーティパッケージは、[セマンティック バージョニング](https://semver.org) に従います。メジャーリリースは毎年（ Q1  頃 ）リリースされますが、マイナーおよびパッチリリースは毎週リリースされる場合があります。マイナーおよびパッチリリースには、**決して** 互換性のない変更は含まれません。

アプリケーションまたはパッケージから Laravel フレームワークまたはそのコンポーネントを参照する場合、Laravel のメジャーリリースには重大な変更が含まれているため、常に `^10.0` などのバージョン制約を使用する必要があります。 ただし、1 日以内に新しいメジャーリリースに更新できるよう常に努めています。

<a name="named-arguments"></a>
#### 名前付き引数

[名前付き引数](https://www.php.net/manual/en/functions.arguments.php#functions.named-arguments) は、Laravel の後方互換性ガイドラインには含まれていません。 Laravel のコードベースを改善するために、必要に応じて関数の引数の名前を変更することがあります。 そのため、Laravel のメソッドを呼び出すときに名前付き引数を使用する場合は、将来的にパラメーター名が変更される可能性があることを理解しておく必要があります。

<a name="support-policy"></a>
## サポートポリシー

Laravel のすべてのリリースについて、バグ修正は 18 ヶ月間提供され、セキュリティ修正は 2 年間提供されます。Lumen を含むすべての追加ライブラリについては、最新のメジャーリリースのみがバグ修正を受けます。また、[Laravelがサポートするデータベースバージョン](/laravel10_ja/database#introduction) についても確認してください。

<div class="overflow-auto">

| Version | PHP (*) | リリース日 | バグ修正のサポート終了日 | セキュリティ修正のサポート終了日 |
| --- | --- | --- | --- | --- |
| 8 | 7.3 - 8.1 | 2020年9月8日 | 2022年7月26日 | 2023年1月24日 |
| 9 | 8.0 - 8.2 | 2022年2月8日 | 2023年8月8日 | 2024年2月6日 |
| 10 | 8.1 - 8.2 | 2023年2月14日 | 2024年8月6日 | 2025年2月4日 |
| 11 | 8.2 | 2024年第1四半期 | 2025年8月5日 | 2026年2月3日 |

</div>

<!-- <div class="version-colors">
     <div class="エンドオブライフ">
         <div class="color-box"></div>
         <div>生産終了</div>
     </div>
     <div class="security-fixes">
         <div class="color-box"></div>
         <div>セキュリティ修正のみ</div>
     </div>
</div> -->

(*) 対応している PHP のバージョン

<a name="laravel-10"></a>
## Laravel 10

ご存知かもしれませんが、Laravel 8 のリリースをもって Laravel は年次リリースに移行しました。以前は、メジャーバージョンが 6 ヶ月ごとにリリースされていました。この移行は、コミュニティのメンテナンス負担を軽減し、開発チームが破壊的変更を導入せずに素晴らしい新機能を提供することに挑戦するためのものです。そのため、Laravel 9 には後方互換性を損なわないさまざまな堅牢な機能が提供されています。

したがって、現行リリースで素晴らしい新機能を提供するというこのコミットメントは、将来の「メジャー」リリースが、アップストリームの依存関係のアップグレードなどの「メンテナンス」タスクに主に使用されることにつながる可能性があります。これはリリースノートでも確認できます。

Laravel 10 は、Laravel 9.x での改善を続けて、すべてのアプリケーションスケルトンメソッドと、フレームワーク全体でクラスを生成するために使用されるすべてのスタブファイルに引数と戻り値の型を導入しています。さらに、外部プロセスの開始および操作のための新しい、開発者に優しい抽象化レイヤーが導入されました。さらに、Laravel Pennant が導入され、アプリケーションの「機能フラグ」を管理するための素晴らしいアプローチが提供されています。

<a name="php-8"></a>
### PHP 8.1

Laravel 10.x には、最低でも PHP バージョン 8.1 が必要です。

<a name="types"></a>
### 型

_アプリケーションスケルトンとスタブの型ヒントは、[Nuno Maduro](https://github.com/nunomaduro)_ によって寄稿されました。

Laravel は当初、当時利用可能だった PHP のすべての型ヒント機能を使用していました。しかし、その後の年に PHP にはさらに多くの新機能が追加されました。これには、追加のプリミティブ型ヒント、戻り値の型、およびユニオン型が含まれます。

Laravel 10.x では、アプリケーションスケルトンとフレームワークで使用されるすべてのスタブが徹底的に更新され、すべてのメソッドシグネチャに引数と戻り値の型が導入されます。さらに、余分な「ドックブロック」型ヒント情報が削除されました。

この変更は、既存のアプリケーションと完全に後方互換性があります。したがって、これらの型ヒントがない既存のアプリケーションは引き続き正常に機能します。

<a name="laravel-pennant"></a>
### Laravel Pennant

_Laravel Pennant は [Tim MacDonald](https://github.com/timacdonald)_ によって開発されました。

新しいファーストパーティパッケージである Laravel Pennant がリリースされました。Laravel Pennant は、アプリケーションの機能フラグを管理するための軽量で効率的なアプローチを提供します。Pennant には、メモリ内の `array` ドライバと永続的な機能ストレージ用の `database` ドライバが標準で含まれています。

機能は、`Feature::define` メソッドを使用して簡単に定義できます：

```php
use Laravel\Pennant\Feature;
use Illuminate\Support\Lottery;

Feature::define('new-onboarding-flow', function () {
    return Lottery::odds(1, 10);
});
```

機能が定義されると、現在のユーザーが指定された機能にアクセスできるかどうかを簡単に判断できます。

```php
if (Feature::active('new-onboarding-flow')) {
    // ...
}
```

もちろん、便宜上、Blade ディレクティブも利用できます。

```blade
@feature('new-onboarding-flow')
    <div>
        <!-- ... -->
    </div>
@endfeature
```

Pennant は、より高度なさまざまな機能と API を提供します。 詳細については、[包括的なペナントのドキュメント](/laravel10_ja/pennant) を参照してください。

<a name="process"></a>
### プロセスのやり取り

_プロセス抽象化レイヤーは、[Nuno Maduro](https://github.com/nunomaduro) と [Taylor Otwell](https://github.com/taylorotwell)_ によって寄稿されました。

Laravel 10.x では、新しい `Process` ファサードを介して外部プロセスの開始とやり取りを行う美しい抽象化レイヤーが導入されています。

```php
use Illuminate\Support\Facades\Process;

$result = Process::run('ls -la');

return $result->output();
```

プロセスはプールで開始することもできるため、並行プロセスの実行と管理が容易になります。

```php
use Illuminate\Process\Pool;
use Illuminate\Support\Facades\Process;

[$first, $second, $third] = Process::concurrently(function (Pool $pool) {
    $pool->command('cat first.txt');
    $pool->command('cat second.txt');
    $pool->command('cat third.txt');
});

return $first->output();
```

さらに、簡単なテストのためにプロセスを偽造することもできます。

```php
Process::fake();

// ...

Process::assertRan('ls -la');
```

プロセスとのやり取りの詳細については、[包括的なプロセス ドキュメント](/laravel10_ja/processes) を参照してください。

<a name="test-profiling"></a>
### テストプロファイリング

_テスト プロファイリングは [Nuno Maduro](https://github.com/nunomaduro)_ によって提供されました。

Artisan `test` コマンドに新しい `--profile` オプションが追加されました。これにより、アプリケーションの最も遅いテストを簡単に特定できます。

```shell
php artisan test --profile
```

便宜上、最も遅いテストは CLI 出力内に直接表示されます。

<p align="center">
     <img width="100%" src="https://user-images.githubusercontent.com/5457236/217328439-d8d983ec-d0fc-4cde-93d9-ae5bccf5df14.png"/>
</p>

<a name="pest-scaffolding"></a>
### Pest スキャフォールディング

新しい Laravel プロジェクトは、デフォルトで Pest テストスキャフォールディングを使用して作成することができます。この機能を利用するには、Laravel インストーラーで新しいアプリケーションを作成する際に `--pest` フラグを指定してください。

```shell
laravel new example-application --pest
```

<a name="generator-cli-prompts"></a>
### ジェネレーター CLI プロンプト

_ジェネレーター CLI プロンプトは [Jess Archer](https://github.com/jessarcher) によって寄稿されました。_

フレームワークの開発者エクスペリエンスを向上させるために、Laravel の組み込みの `make` コマンドはすべて入力を必要としなくなりました。 コマンドが入力なしで呼び出された場合、必要な引数を求めるプロンプトが表示されます。

```shell
php artisan make:controller
```

<a name="horizon-telescope-facelift"></a>
### Horizon / Telescope の改善

[Horizon](/laravel10_ja/horizon) と [Telescope](/laravel10_ja/telescope) は、改善されたタイポグラフィ、間隔、およびデザインを含む新鮮でモダンな外観で更新されました。

<img src="https://laravel.com/img/docs/horizon-example.png">
