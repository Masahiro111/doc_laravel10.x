# アーティザンコンソール

- [はじめに](#introduction)
    - [Tinker (REPL)](#tinker)
- [コマンドの書き込み](#writing-commands)
    - [コマンドの生成](#generating-commands)
    - [コマンド構造](#command-structure)
    - [クロージャコマンド](#closure-commands)
    - [分離可能なコマンド](#isolatable-commands)
- [入力期待値の定義](#defining-input-expectations)
    - [引数](#arguments)
    - [オプション](#options)
    - [入力配列](#input-arrays)
    - [入力の説明](#input-descriptions)
    - [欠落入力のプロンプト](#prompting-for-missing-input)
- [コマンド I/O](#command-io)
    - [入力の取得](#retrieving-input)
    - [入力のプロンプト](#prompting-for-input)
    - [書き込み出力](#writing-output)
- [コマンドの登録](#registering-commands)
- [プログラムで実行するコマンド](#programmatically-executing-commands)
    - [他のコマンドからコマンドを呼び出す](#calling-commands-from-other-commands)
- [シグナル処理](#signal-handling)
- [スタブカスタマイズ](#stub-customization)
- [イベント](#events)

<a name="introduction"></a>
## はじめに

Artisan は、Laravel に含まれるコマンドライン インターフェイスです。Artisan はアプリケーションのルートに「artisan」スクリプトとして存在し、アプリケーションの構築を支援する便利なコマンドを多数提供します。使用可能なすべての Artisan コマンドのリストを表示するには、「list」コマンドを使用します。

```shell
php artisan list
```

すべてのコマンドには、コマンドで使用可能な引数とオプションを表示および説明する「ヘルプ」画面も含まれています。ヘルプ画面を表示するには、コマンド名の前に「help」を付けます。

```shell
php artisan help migrate
```

<a name="laravel-sail"></a>
#### Laravel セイル

ローカル開発環境として [Laravel Sail](/docs/{{version}}/sail) を使用している場合は、必ず「sail」コマンドラインを使用して Artisan コマンドを呼び出してください。Sail は、アプリケーションの Docker コンテナ内で Artisan コマンドを実行します。

```shell
./vendor/bin/sail artisan list
```

<a name="tinker"></a>
### Tinker (REPL)

Laravel Tinker は、[PsySH](https://github.com/bobthecow/psysh) パッケージを利用した、Laravel フレームワーク用の強力な REPL です。

<a name="installation"></a>
#### インストール

すべての Laravel アプリケーションにはデフォルトで Tinker が含まれています。ただし、以前にアプリケーションから Tinker を削除した場合は、Composer を使用して Tinker をインストールできます。

```shell
composer require laravel/tinker
```

> **Note**
> Laravel アプリケーションと対話するためのグラフィカル UI をお探しですか? 【ティンカーウェル】(https://tinkerwell.app)をチェックしてみてください！

<a name="usage"></a>
#### 使い方

Tinker を使用すると、Eloquent モデル、ジョブ、イベントなどを含む Laravel アプリケーション全体をコマンドラインで操作できます。Tinker 環境に入るには、「tinker」 Artisan コマンドを実行します。

```シェル
php 職人のいじくり回し
```

`vendor:publish` コマンドを使用して、Tinker の設定ファイルを公開できます。

```シェル
php 職人ベンダー:publish --provider="Laravel\Tinker\TinkerServiceProvider"
```

> **警告**  
> `Dispatchable` クラスの `dispatch` ヘルパー関数と `dispatch` メソッドは、ガベージ コレクションに依存してジョブをキューに配置します。したがって、tinker を使用する場合は、ジョブをディスパッチするために `Bus::dispatch` または `Queue::push` を使用する必要があります。

<a name="コマンド許可リスト"></a>
#### コマンド許可リスト

Tinker は、「許可」リストを利用して、シェル内でどの Artisan コマンドの実行を許可するかを決定します。デフォルトでは、`clear-compiled`、`down`、`env`、`inspire`、`migrate`、`optimize`、および `up` コマンドを実行できます。さらに多くのコマンドを許可したい場合は、`tinker.php` 設定ファイルの `commands` 配列にコマンドを追加できます。

    'コマンド' => [
        // App\Console\Commands\ExampleCommand::class,
    ],

<a name="クラスをエイリアス化すべきではない"></a>
#### エイリアスを付けるべきではないクラス

通常、Tinker でクラスを操作すると、Tinker は自動的にクラスのエイリアスを作成します。ただし、クラスによっては別名を付けたくない場合もあります。これを行うには、`tinker.php` 設定ファイルの `dont_alias` 配列にクラスをリストします。

    'dont_alias' => [
        アプリ\モデル\ユーザー::クラス、
    ],

<a name="書き込みコマンド"></a>
## コマンドの書き込み

Artisan で提供されるコマンドに加えて、独自のカスタム コマンドを作成できます。コマンドは通常、「app/Console/Commands」ディレクトリに保存されます。ただし、Composer によってコマンドをロードできる限り、独自の保存場所を自由に選択できます。

<a name="コマンドの生成"></a>
### コマンドの生成

新しいコマンドを作成するには、「make:command」アーティザン コマンドを使用できます。このコマンドは、`app/Console/Commands` ディレクトリに新しいコマンド クラスを作成します。このディレクトリがアプリケーションに存在しなくても心配する必要はありません。このディレクトリは、「make:command」Artisan コマンドを初めて実行するときに作成されます。

```シェル
php 職人 make:command SendEmails
```

<a name="コマンド構造"></a>
### コマンド構造

コマンドを生成した後、クラスの「signature」プロパティと「description」プロパティに適切な値を定義する必要があります。これらのプロパティは、コマンドを「リスト」画面に表示するときに使用されます。`signature` プロパティを使用すると、[コマンドの入力期待値](#defining-input-expectations) を定義することもできます。コマンドが実行されると、`handle` メソッドが呼び出されます。コマンド ロジックをこのメソッドに配置できます。

コマンドの例を見てみましょう。コマンドの `handle` メソッドを介して、必要な依存関係をリクエストできることに注意してください。Laravel [サービスコンテナ](/docs/{{version}}/container) は、このメソッドのシグネチャでタイプヒントされているすべての依存関係を自動的に挿入します。

    <?php

    名前空間 App\Console\Commands;

    App\Models\User を使用します。
    App\Support\DripEmailer を使用します。
    Illuminate\Console\Command を使用します。

    class SendEmails extends コマンド
    {
        /**
         * コンソール コマンドの名前と署名。
         *
         * @var 文字列
         */
        protected $signature = 'mail:send {user}';

        /**
         ※コンソールコマンドの説明です。
         *
         * @var 文字列
         */
        protected $description = 'ユーザーにマーケティング電子メールを送信';

        /**
         ※コンソールコマンドを実行します。
         */
        パブリック関数ハンドル(DripEmailer $drip): void
        {
            $drip->send(User::find($this->argument('user')));
        }
    }

> **注意**  
> コードをより再利用するには、コンソール コマンドを軽量にし、アプリケーション サービスに任せてタスクを実行することをお勧めします。上の例では、電子メールの送信という「重労働」を行うためにサービス クラスを挿入していることに注意してください。

<a name="closure-commands"></a>
### 終了コマンド

クロージャ ベースのコマンドは、コンソール コマンドをクラスとして定義する代替手段を提供します。ルート クロージャがコントローラの代替であるのと同じように、コマンド クロージャはコマンド クラスの代替であると考えてください。`app/Console/Kernel.php` ファイルの `commands` メソッド内で、Laravel は `routes/console.php` ファイルを読み込みます。

    /**
     * アプリケーションのクロージャベースのコマンドを登録します。
     */
    保護された関数コマンド(): void
    {
        requirebase_path('routes/console.php');
    }

このファイルは HTTP ルートを定義しませんが、アプリケーションへのコンソール ベースのエントリ ポイント (ルート) を定義します。このファイル内で、`Artisan::command` メソッドを使用して、クロージャ ベースのコンソール コマンドをすべて定義できます。`command` メソッドは 2 つの引数を受け入れます: [コマンド シグネチャ](#defining-input-expectations) と、コマンドの引数とオプションを受け取るクロージャです。

    Artisan::command('mail:send {user}', function (string $user) {
        $this->info("メールの送信先: {$user}!");
    });

クロージャは基礎となるコマンド インスタンスにバインドされているため、通常は完全なコマンド クラスでアクセスできるすべてのヘルパー メソッドに完全にアクセスできます。

<a name="type-hinting-dependency"></a>
#### タイプヒンティングの依存関係

コマンドの引数とオプションを受け取ることに加えて、コマンド クロージャは、[サービス コンテナ](/docs/{{version}}/container) から解決したい追加の依存関係をタイプヒントで指定することもできます。

    App\Models\User を使用します。
    App\Support\DripEmailer を使用します。

    Artisan::command('mail:send {user}', function (DripEmailer $drip, string $user) {
        $drip->send(ユーザー::find($user));
    });

<a name="closure-command-descriptions"></a>
#### クロージャコマンドの説明

クロージャベースのコマンドを定義する場合、「目的」メソッドを使用してコマンドに説明を追加できます。この説明は、`phpArtisanList`または`phpArtisanhelp`コマンドを実行すると表示されます。

    Artisan::command('mail:send {user}', function (string $user) {
        // ...
    })->Purpose('ユーザーにマーケティング電子メールを送信');

<a name="isolatable-commands"></a>
### 分離可能なコマンド

> **警告**
> この機能を利用するには、アプリケーションは `memcached`、`redis`、`dynamodb`、`database`、`file`、または `array` キャッシュ ドライバーをアプリケーションのデフォルト キャッシュ ドライバーとして使用する必要があります。さらに、すべてのサーバーが同じ中央キャッシュ サーバーと通信している必要があります。

場合によっては、コマンドのインスタンスを一度に 1 つだけ実行できるようにしたい場合があります。これを実現するには、コマンド クラスに `Illuminate\Contracts\Console\Isolatable` インターフェイスを実装します。

    <?php

    名前空間 App\Console\Commands;

    Illuminate\Console\Command を使用します。
    Illuminate\Contracts\Console\Isolatable を使用します。

    class SendEmails extends コマンドは Isolatable を実装します
    {
        // ...
    }

コマンドが「分離可能」としてマークされている場合、Laravel は自動的にコマンドに「--isoulated」オプションを追加します。そのオプションを指定してコマンドが呼び出されると、Laravel はそのコマンドの他のインスタンスがすでに実行されていないことを確認します。Laravel は、アプリケーションのデフォルトのキャッシュドライバーを使用してアトミックロックの取得を試みることによってこれを実現します。コマンドの他のインスタンスが実行中の場合、コマンドは実行されません。ただし、コマンドは引き続き正常終了ステータス コードで終了します。

```シェル
php 職人メール:send 1 --isoulated
```

コマンドが実行できない場合に返される終了ステータス コードを指定したい場合は、「isoulated」オプションを使用して目的のステータス コードを指定できます。

```シェル
php 職人メール:send 1 --isorated=12
```

<a name="ロックid"></a>
#### ロック ID

デフォルトでは、Laravel はコマンド名を使用して、アプリケーションのキャッシュ内のアトミック ロックを取得するために使用される文字列キーを生成します。ただし、Artisan コマンド クラスで `isolatableId` メソッドを定義することでこのキーをカスタマイズでき、コマンドの引数またはオプションをキーに統合できます。

```php
/**
 * コマンドの分離可能な ID を取得します。
 */
パブリック関数 isolatableId(): 文字列
{
    return $this->argument('user');
}
```

<a name="ロック有効期限"></a>
#### ロックの有効期限

デフォルトでは、分離ロックはコマンドの終了後に期限切れになります。または、コマンドが中断されて完了できない場合、ロックは 1 時間後に期限切れになります。ただし、コマンドで `isolationLockExpiresAt` メソッドを定義することで、ロックの有効期限を調整できます。

```php
DateTimeInterface を使用します。
DateInterval を使用します。

/**
 * コマンドの分離ロックがいつ期限切れになるかを決定します。
 */
パブリック関数isolationLockExpiresAt(): DateTimeInterface|DateInterval
{
    return now()->addMinutes(5);
}
```

<a name="定義入力期待値"></a>
## 入力の期待値の定義

コンソール コマンドを作成するときは、引数またはオプションを通じてユーザーからの入力を収集するのが一般的です。Laravel では、コマンドの `signature` プロパティを使用して、ユーザーから期待する入力を定義するのが非常に便利です。「signature」プロパティを使用すると、コマンドの名前、引数、オプションを単一の表現力豊かなルートのような構文で定義できます。

<a name="arguments"></a>
### 引数

ユーザーが指定したすべての引数とオプションは中括弧で囲まれます。次の例では、コマンドは 1 つの必須引数 `user` を定義します。

    /**
     * コンソール コマンドの名前と署名。
     *
     * @var 文字列
     */
    protected $signature = 'mail:send {user}';

引数をオプションにしたり、引数のデフォルト値を定義したりすることもできます。

    // オプションの引数...
    'メール: {ユーザー?} を送信'

    // デフォルト値を持つオプションの引数...
    'mail:send {user=foo}'

<a name="オプション"></a>
### オプション

オプションは、引数と同様、ユーザー入力の別の形式です。コマンド ライン経由でオプションを指定する場合、オプションには 2 つのハイフン (「--」) が接頭辞として付けられます。オプションには、値を受け取るオプションと受け取らないオプションの 2 種類があります。値を受け取らないオプションは、ブール値の「スイッチ」として機能します。このタイプのオプションの例を見てみましょう。

    /**
     * コンソール コマンドの名前と署名。
     *
     * @var 文字列
     */
    protected $signature = 'mail:send {user} {--queue}';

この例では、Artisan コマンドを呼び出すときに「--queue」スイッチを指定できます。`--queue` スイッチが渡された場合、オプションの値は `true` になります。それ以外の場合、値は「false」になります。

```シェル
php 職人メール:send 1 --queue
```

<a name="オプション付き値"></a>
#### 値を含むオプション

次に、値を期待するオプションを見てみましょう。ユーザーがオプションの値を指定する必要がある場合は、オプション名の末尾に「=」記号を付ける必要があります。

    /**
     * コンソール コマンドの名前と署名。
     *
     * @var 文字列
     */
    protected $signature = 'mail:send {user} {--queue=}';

この例では、ユーザーは次のようにオプションの値を渡すことができます。コマンドを呼び出すときにオプションが指定されていない場合、その値は「null」になります。

```シェル
php 職人メール:send 1 --queue=default
```

オプション名の後にデフォルト値を指定することで、オプションにデフォルト値を割り当てることができます。ユーザーによってオプション値が渡されない場合は、デフォルト値が使用されます。

    'mail:send {user} {--queue=default}'

<a name="オプション-ショートカット"></a>
#### オプションのショートカット

オプションを定義するときにショートカットを割り当てるには、オプション名の前にショートカットを指定し、区切り文字として「|」文字を使用してショートカットと完全なオプション名を区切ります。

    'mail:send {user} {--Q|queue}'

ターミナルでコマンドを呼び出すときは、オプションのショートカットの前に 1 つのハイフンを付ける必要があり、オプションの値を指定するときに「=」文字を含めないでください。

```シェル
php 職人メール:send 1 -Qdefault
```

<a name="input-arrays"></a>
### 入力配列

複数の入力値を想定する引数またはオプションを定義したい場合は、「*」文字を使用できます。まず、そのような引数を指定する例を見てみましょう。

    'メール: {ユーザー*} を送信'

このメソッドを呼び出すとき、「user」引数をコマンドラインに順番に渡すことができます。たとえば、次のコマンドは、`user` の値を、値として `1` と `2` を含む配列に設定します。

```シェル
php 職人メール:送信 1 2
```

この `*` 文字をオプションの引数定義と組み合わせて、引数の 0 個以上のインスタンスを許可できます。

    'メール: {ユーザー?*} を送信'

<a name="オプション配列"></a>
#### オプション配列

複数の入力値を予期するオプションを定義する場合、コマンドに渡される各オプション値の先頭にオプション名を付ける必要があります。

    'メール:送信 {--id=*}'

このようなコマンドは、複数の `--id` 引数を渡すことによって呼び出すことができます。

```シェル
php 職人メール:send --id=1 --id=2
```

<a name="input-descriptions"></a>
### 入力の説明

コロンを使用して引数名と説明を区切ることにより、入力引数とオプションに説明を割り当てることができます。コマンドを定義するのに少し余裕が必要な場合は、自由に定義を複数行に分けて記述してください。

    /**
     * コンソール コマンドの名前と署名。
     *
     * @var 文字列
     */
    protected $signature = 'メール:送信
                            {user : ユーザーのID}
                            {--queue : ジョブをキューに入れるかどうか}';

<a name="不足入力のプロンプト"></a>
### 不足している入力のプロンプト

コマンドに必須の引数が含まれている場合、それらが指定されていないと、ユーザーはエラー メッセージを受け取ります。あるいは、「PromptsForMissingInput」インターフェースを実装することで、必要な引数が欠落している場合にユーザーに自動的にプロンプ​​トを表示するようにコマンドを構成することもできます。

    <?php

    名前空間 App\Console\Commands;

    Illuminate\Console\Command を使用します。
    Illuminate\Contracts\Console\PromptsForMissingInput を使用します。

    class SendEmails extends Command 実装 PromptsForMissingInput
    {
        /**
         * コンソール コマンドの名前と署名。
         *
         * @var 文字列
         */
        protected $signature = 'mail:send {user}';

        // ...
    }

Laravel がユーザーから必要な引数を収集する必要がある場合、引数の名前または説明を使用して質問をインテリジェントに表現することで、自動的にユーザーに引数を求めます。必要な引数を収集するために使用される質問をカスタマイズしたい場合は、引数名をキーとする質問の配列を返す `promptForMissingArgumentsUsing` メソッドを実装できます。

    /**
     * 返された質問を使用して、入力引数が欠落しているかどうかを確認するプロンプトを表示します。
     *
     * @return 配列
     */
    保護された関数promptForMissingArgumentsUsing()
    {
        戻る [
            'user' => 'どのユーザー ID がメールを受信する必要がありますか?',
        ];
    }

質問とプレースホルダーを含むタプルを使用して、プレースホルダー テキストを提供することもできます。

    戻る [
        'user' => ['どのユーザー ID がメールを受信しますか?', '例: 123'],
    ];

プロンプトを完全に制御したい場合は、ユーザーにプロンプ​​トを表示し、その回答を返すクロージャーを提供できます。

    App\Models\User を使用します。
    Laravel\Prompts\search 関数を使用します。

    // ...

    戻る [
        'ユーザー' => fn () => 検索(
            ラベル: 'ユーザーの検索:',
            プレースホルダー: 'テイラー・オトウェルなど',
            オプション: fn ($value) => strlen($value) > 0
                ? User::where('name', 'like', "%{$value}%")->pluck('name', 'id')
                : []
        ),
    ];

> **注意**  
包括的な [Laravel プロンプト](/docs/{{version}}/prompts) ドキュメントには、利用可能なプロンプトとその使用法に関する追加情報が含まれています。

ユーザーに [options](#options) の選択または入力を求めるプロンプトを表示したい場合は、コマンドの `handle` メソッドにプロンプ​​トを含めることができます。ただし、不足している引数についても自動的にプロンプ​​トが表示された場合にのみユーザーにプロンプ​​トを表示したい場合は、`afterPromptingForMissingArguments` メソッドを実装できます。

    Symfony\Component\Console\Input\InputInterface を使用します。
    Symfony\Component\Console\Output\OutputInterface を使用します。
    Laravel\Prompts\confirm 関数を使用します。

    // ...

    /**
     * ユーザーに引数の不足を求めるプロンプトが表示された後でアクションを実行します。
     *
     * @param \Symfony\Component\Console\Input\InputInterface $input
     * @param \Symfony\Component\Console\Output\OutputInterface $output
     * @return void
     */
    保護された関数 afterPromptingForMissingArguments(InputInterface $input, OutputInterface $output)
    {
        $input->setOption('キュー',confirm(
            ラベル: 'メールをキューに入れますか?',
            デフォルト: $this->option('queue')
        ));
    }

<a name="command-io"></a>
## コマンド I/O

<a name="入力の取得"></a>
### 入力の取得

コマンドの実行中に、コマンドで受け入れられる引数とオプションの値にアクセスする必要がある場合があります。これを行うには、「argument」メソッドと「option」メソッドを使用します。引数またはオプションが存在しない場合は、「null」が返されます。

    /**
     ※コンソールコマンドを実行します。
     */
    パブリック関数ハンドル(): void
    {
        $userId = $this->argument('user');
    }

すべての引数を「配列」として取得する必要がある場合は、「arguments」メソッドを呼び出します。

    $arguments = $this->arguments();

オプションは、「option」メソッドを使用して引数と同じくらい簡単に取得できます。すべてのオプションを配列として取得するには、「options」メソッドを呼び出します。

    // 特定のオプションを取得します...
    $queueName = $this->option('queue');

    // すべてのオプションを配列として取得します...
    $options = $this->options();

<a name="入力のプロンプト"></a>
### 入力を求めるプロンプト

> **注意**  
> [Laravel Prompts](/docs/{{version}}/prompts) は、プレースホルダーテキストや検証などのブラウザーのような機能を備えた、美しくユーザーフレンドリーなフォームをコマンドラインアプリケーションに追加するための PHP パッケージです。

出力を表示するだけでなく、コマンドの実行中にユーザーに入力を求めることもできます。「ask」メソッドは、指定された質問をユーザーに要求し、入力を受け入れてから、ユーザーの入力をコマンドに返します。

    /**
     ※コンソールコマンドを実行します。
     */
    パブリック関数ハンドル(): void
    {
        $name = $this->ask('あなたの名前は何ですか?');

        // ...
    }

「secret」メソッドは「ask」に似ていますが、ユーザーがコンソールに入力するときにユーザーの入力は表示されません。この方法は、パスワードなどの機密情報を要求する場合に役立ちます。

    $password = $this->secret('パスワードは何ですか?');

<a name="確認を求める"></a>
#### 確認を求める

ユーザーに簡単な「はいまたはいいえ」の確認を求める必要がある場合は、`confirm` メソッドを使用できます。デフォルトでは、このメソッドは「false」を返します。ただし、ユーザーがプロンプトに対して「y」または「yes」を入力すると、メソッドは「true」を返します。

    if ($this->confirm('続行しますか?')) {
        // ...
    }

必要に応じて、「confirm」メソッドの 2 番目の引数として「true」を渡すことで、確認プロンプトがデフォルトで「true」を返すように指定できます。

    if ($this->confirm('続行しますか?', true)) {
        // ...
    }

<a name="オートコンプリート"></a>
#### 自動補完

「anticipate」メソッドを使用すると、可能な選択肢を自動補完することができます。ユーザーは、オートコンプリートのヒントに関係なく、任意の回答を入力できます。

    $name = $this->anticipate('あなたの名前は何ですか?', ['テイラー', 'デイル']);

あるいは、クロージャを 2 番目の引数として `anticipate` メソッドに渡すこともできます。クロージャは、ユーザーが入力文字を入力するたびに呼び出されます。クロージャは、これまでのユーザーの入力を含む文字列パラメータを受け入れ、オートコンプリートのオプションの配列を返す必要があります。

    $name = $this->anticipate('住所は何ですか?', function (string $input) {
        // オートコンプリート オプションを返します...
    });

<a name="複数選択の質問"></a>
＃＃＃＃ 複数の選択肢の質問

質問するときにユーザーに事前定義された一連の選択肢を提供する必要がある場合は、`choice` メソッドを使用できます。オプションが選択されていない場合に、メソッドの 3 番目の引数としてインデックスを渡すことにより、デフォルト値の配列インデックスが返されるように設定できます。

    $name = $this->choice(
        'あなたの名前は何ですか？'、
        ['テイラー'、'デイル']、
        $defaultIndex
    );

さらに、`choice` メソッドは、有効な応答を選択する最大試行回数と複数の選択が許可されるかどうかを決定するためのオプションの 4 番目と 5 番目の引数を受け入れます。

    $name = $this->choice(
        'あなたの名前は何ですか？'、
        ['テイラー'、'デイル']、
        $defaultIndex、
        $maxAttempts = null、
        $allowMultipleSelections = false
    );

<a name="書き込み出力"></a>
### 出力の書き込み

出力をコンソールに送信するには、`line`、`info`、`comment`、`question`、`warn`、および `error` メソッドを使用できます。これらの各メソッドは、目的に応じて適切な ANSI カラーを使用します。たとえば、一般的な情報をユーザーに表示してみましょう。通常、「info」メソッドはコンソールに緑色のテキストとして表示されます。

    /**
     ※コンソールコマンドを実行します。
     */
    パブリック関数ハンドル(): void
    {
        // ...

        $this->info('コマンドは成功しました!');
    }

エラーメッセージを表示するには、`error` メソッドを使用します。通常、エラー メッセージ テキストは赤色で表示されます。

    $this->error('何か問題が発生しました!');

`line` メソッドを使用すると、色の付いていないプレーンなテキストを表示できます。

    $this->line('これを画面に表示します');

`newLine` メソッドを使用して空行を表示できます。

    // 空白行を 1 行書きます...
    $this->newLine();

    // 空白行を 3 行書きます...
    $this->newLine(3);

<a name="テーブル"></a>
#### テーブル

「table」メソッドを使用すると、複数の行/列のデータを簡単に正しくフォーマットできます。テーブルの列名とデータを指定するだけで、Laravel が自動的に実行します。
テーブルの適切な幅と高さを自動的に計算します。

    App\Models\User を使用します。

    $this->table(
        ['名前', 'メールアドレス'],
        ユーザー::all(['名前', '電子メール'])->toArray()
    );

<a name="progress-bars"></a>
#### プログレスバー

長時間実行されるタスクの場合は、タスクの完了度をユーザーに知らせる進行状況バーを表示すると便利です。`withProgressBar` メソッドを使用すると、Laravel は進行状況バーを表示し、指定された反復可能な値を超えて反復ごとに進行状況を進めます。

    App\Models\User を使用します。

    $users = $this->withProgressBar(User::all(), function (User $user) {
        $this->performTask($user);
    });

場合によっては、進行状況バーの進み方を手動で制御する必要がある場合があります。まず、プロセスが反復処理される合計ステップ数を定義します。次に、各項目を処理した後、進行状況バーを進めます。

    $users = App\Models\User::all();

    $bar = $this->output->createProgressBar(count($users));

    $bar->start();

    foreach ($users として $user) {
        $this->performTask($user);

        $bar->advance();
    }

    $bar->finish();

> **注意**  
> より高度なオプションについては、[Symfony Progress Bar コンポーネントのドキュメント](https://symfony.com/doc/current/components/console/helpers/progressbar.html) を確認してください。

<a name="コマンドの登録"></a>
## コマンドの登録

すべてのコンソール コマンドは、アプリケーションの「コンソール カーネル」であるアプリケーションの `App\Console\Kernel` クラス内に登録されます。このクラスの `commands` メソッド内に、カーネルの `load` メソッドの呼び出しが表示されます。「load」メソッドは「app/Console/Commands」ディレクトリをスキャンし、そこに含まれる各コマンドを Artisan に自動的に登録します。さらに、自由に「load」メソッドを追加呼び出しして、アーティザン コマンドの他のディレクトリをスキャンすることもできます。

    /**
     ※アプリケーションのコマンドを登録します。
     */
    保護された関数コマンド(): void
    {
        $this->load(__DIR__.'/コマンド');
        $this->load(__DIR__.'/../Domain/Orders/Commands');

        // ...
    }

必要に応じて、コマンドのクラス名を `App\Console\Kernel` クラス内の `$commands` プロパティに追加することで、コマンドを手動で登録できます。このプロパティがカーネルでまだ定義されていない場合は、手動で定義する必要があります。Artisan が起動すると、このプロパティにリストされているすべてのコマンドが [サービス コンテナ](/docs/{{version}}/container) によって解決され、Artisan に登録されます。

    protected $commands = [
        コマンド\SendEmails::class
    ];

<a name="プログラムによるコマンドの実行"></a>
## プログラムによるコマンドの実行

場合によっては、CLI の外部で Artisan コマンドを実行したい場合があります。たとえば、ルートまたはコントローラーからアーティザン コマンドを実行したい場合があります。これを実現するには、「Artisan」ファサードで「call」メソッドを使用できます。`call` メソッドは、コマンドのシグネチャ名またはクラス名のいずれかを最初の引数として受け入れ、コマンド パラメータの配列を 2 番目の引数として受け入れます。終了コードが返されます。

    Illuminate\Support\Facades\Artisan を使用します。

    Route::post('/user/{user}/mail', function (string $user) {
        $exitCode = Artisan::call('mail:send', [
            'user' => $user、'--queue' => 'default'
        ]);

        // ...
    });

あるいは、Artisan コマンド全体を文字列として `call` メソッドに渡すこともできます。

    Artisan::call('mail:send 1 --queue=default');

<a name="配列値の受け渡し"></a>
#### 配列値の受け渡し

コマンドが配列を受け入れるオプションを定義している場合は、そのオプションに値の配列を渡すことができます。

    Illuminate\Support\Facades\Artisan を使用します。

    Route::post('/mail', function () {
        $exitCode = Artisan::call('mail:send', [
            '--id' => [5, 13]
        ]);
    });

<a name="passing-boolean-values"></a>
#### ブール値を渡す

文字列値を受け入れないオプションの値を指定する必要がある場合 (たとえば、「移行:リフレッシュ」コマンドの「--force」フラグ)、その値として「true」または「false」を渡す必要があります。オプション：

    $exitCode = Artisan::call('移行:リフレッシュ', [
        '--force' => true、
    ]);

<a name="queueing-artisan-commands"></a>
#### アーティザン コマンドのキューイング

「Artisan」ファサードの「queue」メソッドを使用すると、Artisan コマンドをキューに入れて、[キュー ワーカー](/docs/{{version}}/queues) によってバックグラウンドで処理されるようにすることもできます。この方法を使用する前に、キューを設定し、キュー リスナーを実行していることを確認してください。

    Illuminate\Support\Facades\Artisan を使用します。

    Route::post('/user/{user}/mail', function (string $user) {
        Artisan::queue('mail:send', [
            'user' => $user、'--queue' => 'default'
        ]);

        // ...
    });

`onConnection` メソッドと `onQueue` メソッドを使用すると、Artisan コマンドをディスパッチする接続またはキューを指定できます。

    Artisan::queue('mail:send', [
        'ユーザー' => 1、'--キュー' => 'デフォルト'
    ])->onConnection('redis')->onQueue('commands');

<a name="その他のコマンドからのコマンドの呼び出し"></a>
### 他のコマンドからのコマンドの呼び出し

場合によっては、既存の Artisan コマンドから他のコマンドを呼び出したい場合があります。`call` メソッドを使用してこれを行うことができます。この「call」メソッドは、コマンド名とコマンド引数/オプションの配列を受け入れます。

    /**
     ※コンソールコマンドを実行します。
     */
    パブリック関数ハンドル(): void
    {
        $this->call('mail:send', [
            'ユーザー' => 1、'--キュー' => 'デフォルト'
        ]);

        // ...
    }

別のコンソール コマンドを呼び出してその出力をすべて抑制したい場合は、`callSilently` メソッドを使用できます。`callSilently` メソッドには、`call` メソッドと同じシグネチャがあります。

    $this->callSilently('mail:send', [
        'ユーザー' => 1、'--キュー' => 'デフォルト'
    ]);

<a name="信号処理"></a>
## 信号処理

ご存知かもしれませんが、オペレーティング システムでは、実行中のプロセスにシグナルを送信できます。たとえば、「SIGTERM」シグナルは、オペレーティング システムがプログラムに終了を要求する方法です。Artisan コンソール コマンドでシグナルをリッスンし、シグナルが発生したときにコードを実行したい場合は、「trap」メソッドを使用できます。

    /**
     ※コンソールコマンドを実行します。
     */
    パブリック関数ハンドル(): void
    {
        $this->trap(SIGTERM, fn () => $this-> shouldKeepRunning = false);

        while ($this-> shouldKeepRunning) {
            // ...
        }
    }

複数のシグナルを一度にリッスンするには、シグナルの配列を「trap」メソッドに提供します。

    $this->trap([SIGTERM, SIGQUIT], function (int $signal) {
        $this-> shouldKeepRunning = false;

        ダンプ($sign); // SIGTERM / SIGQUIT
    });

<a name="スタブカスタマイズ"></a>
## スタブのカスタマイズ

Artisan コンソールの「make」コマンドは、コントローラー、ジョブ、移行、テストなどのさまざまなクラスを作成するために使用されます。これらのクラスは、入力に基づいて値が設定される「スタブ」ファイルを使用して生成されます。ただし、Artisan によって生成されたファイルに小さな変更を加えたい場合があります。これを実現するには、`stub:publish` コマンドを使用して最も一般的なスタブをアプリケーションに公開し、カスタマイズできるようにします。

```シェル
php 職人スタブ:公開
```

公開されたスタブは、アプリケーションのルートの「stubs」ディレクトリ内に配置されます。これらのスタブに加えた変更は、Artisan の「make」コマンドを使用して対応するクラスを生成するときに反映されます。

<a name="イベント"></a>
## イベント

Artisan は、コマンドの実行時に 3 つのイベント (「Illuminate\Console\Events\ArtisanStarting」、「Illuminate\Console\Events\CommandStarting」、および「Illuminate\Console\Events\CommandFinished」) を送出します。ArtisanStarting イベントは、Artisan が実行を開始するとすぐに送出されます。次に、コマンドが実行される直前に `CommandStarting` イベントが送出されます。最後に、コマンドの実行が終了すると、「CommandFinished」イベントが送出されます。