# 貢献ガイド

- [バグ報告](#bug-reports)
- [サポートに関する質問](#support-questions)
- [コア開発ディスカッション](#core-development-discussion)
- [どのブランチ?](#which-branch)
- [コンパイル済みアセット](#compiled-assets)
- [セキュリティの脆弱性](#security-vulnerabilities)
- [コーディングスタイル](#coding-style)
    - [PHPDoc](#phpdoc)
    - [StyleCI](#styleci)
- [行動規範](#code-of-conduct)

<a name="bug-reports"></a>
## バグ報告

積極的な協力を促すために、Laravel はプルリクエストを強く推奨しており、バグ報告だけでなく、プルリクエストも推奨しています。新機能のテストがすべて合格している場合に限り、"ready for review（レビューの準備ができました）"（ドラフト状態でない）とマークされたプルリクエストがレビューされます。"draft（下書き）" 状態のまま放置された、活動のないプルリクエストは数日後にクローズされます。

ただし、バグ報告を行う場合、問題のタイトルと明確な説明が含まれるべきです。また、可能な限り関連する情報と、問題を示すコードサンプルも含めてください。バグ報告の目的は、自分自身や他の人がバグを再現し、修正を行いやすくすることです。

バグ報告は、同じ問題を抱える他の人があなたと協力して問題を解決できることを期待して作成されます。バグ報告が自動的に活動を見ることや、他の人がすぐに修正に取り組むことを期待しないでください。バグ報告を作成することで、自分自身や他の人が問題を修正するための道を開くことができます。協力したい場合は、[問題トラッカーに記載されているバグ] (https://github.com/issues?q=is%3Aopen+is%3Aissue+label%3Abug+user%3Alaravel) を修正することで助けることができます。Laravel のすべての問題を表示するには、GitHub で認証されている必要があります。

Laravel の使用中に不適切な DocBlock、PHPStan、または IDE の警告に気付いた場合は、GitHub Issue を作成しないでください。 代わりに、プルリクエストを提出して問題を解決してください。

Laravel のソース コードは GitHub で管理されており、Laravel プロジェクトごとにリポジトリがあります。

<div class="content-list" markdown="1">

- [Laravel Application](https://github.com/laravel/laravel)
- [Laravel Art](https://github.com/laravel/art)
- [Laravel Documentation](https://github.com/laravel/docs)
- [Laravel Dusk](https://github.com/laravel/dusk)
- [Laravel Cashier Stripe](https://github.com/laravel/cashier)
- [Laravel Cashier Paddle](https://github.com/laravel/cashier-paddle)
- [Laravel Echo](https://github.com/laravel/echo)
- [Laravel Envoy](https://github.com/laravel/envoy)
- [Laravel Framework](https://github.com/laravel/framework)
- [Laravel Homestead](https://github.com/laravel/homestead)
- [Laravel Homestead Build Scripts](https://github.com/laravel/settler)
- [Laravel Horizon](https://github.com/laravel/horizon)
- [Laravel Jetstream](https://github.com/laravel/jetstream)
- [Laravel Passport](https://github.com/laravel/passport)
- [Laravel Pennant](https://github.com/laravel/pennant)
- [Laravel Pint](https://github.com/laravel/pint)
- [Laravel Sail](https://github.com/laravel/sail)
- [Laravel Sanctum](https://github.com/laravel/sanctum)
- [Laravel Scout](https://github.com/laravel/scout)
- [Laravel Socialite](https://github.com/laravel/socialite)
- [Laravel Telescope](https://github.com/laravel/telescope)
- [Laravel Website](https://github.com/laravel/laravel.com-next)

</div>

<a name="support-questions"></a>
## サポートに関する質問

Laravel の GitHub Issue トラッカーは、Laravel のヘルプやサポートを提供することを意図したものではありません。 代わりに、次のいずれかのチャネルを使用してください。

<div class="content-list" markdown="1">

- [GitHub Discussions](https://github.com/laravel/framework/discussions)
- [Laracasts Forums](https://laracasts.com/discuss)
- [Laravel.io Forums](https://laravel.io/forum)
- [StackOverflow](https://stackoverflow.com/questions/tagged/laravel)
- [Discord](https://discord.gg/laravel)
- [Larachat](https://larachat.co)
- [IRC](https://web.libera.chat/?nick=artisan&channels=#laravel)

</div>

<a name="core-development-discussion"></a>
## コア開発のディスカッション

Laravel フレームワークリポジトリの [GitHub ディスカッション ボード](https://github.com/laravel/framework/discussions) で、新機能の提案や既存のLaravelの挙動の改善を提案できます。 新機能を提案する場合は、その機能を実装するために必要なコードの一部を実装する意欲があることが望ましいです。

バグ、新機能、既存機能の実装に関する非公式なディスカッションは、[Laravel Discord サーバー](https://discord.gg/laravel) の `#internals` チャネルで行われます。 Laravel のメンテナーである Taylor Otwell は通常、平日の午前 8 時から午後 5 時 (UTC-06:00 またはアメリカ/シカゴ) に通常チャンネルに参加し、他の時間帯には時折参加しています。

<a name="which-branch"></a>
## どのブランチ？

**すべて**のバグ修正は、バグ修正をサポートする最新バージョン (現在は `10.x`) に送信する必要があります。 バグ修正は、今後のリリースでのみ存在する機能を修正する場合を除いて、`master` ブランチに送信するべきではありません。

現在リリースと **完全な下位互換性** がある **マイナー** 機能は、最新の安定版ブランチ (現在は `10.x`) に送信できます。

**メジャー** な新機能または重大な変更を伴う機能は、次のリリースを含む `master` ブランチに常に送信する必要があります。

<a name="compiled-assets"></a>
## コンパイル済みアセット

`laravel/laravel` リポジトリの `resources/css` や `resources/js` など、コンパイルされたファイルに影響を与える変更を提出する場合は、コンパイル済みのファイルをコミットしないでください。そのサイズが大きいため、実際にはメンテナーがレビューすることができません。これは、Laravel に悪意のあるコードを注入する方法として悪用される可能性があります。これを防御的に防ぐために、すべてのコンパイル済みファイルは Laravel のメンテナーによって生成され、コミットされます。

<a name="security-vulnerabilities"></a>
## セキュリティの脆弱性

Laravel 内にセキュリティの脆弱性を発見した場合は、Taylor Otwell (<a href="mailto:taylor@laravel.com">taylor@laravel.com</a>) にメールを送信してください。 すべてのセキュリティの脆弱性は迅速に対処されます。

<a name="coding-style"></a>
## コーディングスタイル

Laravel は [PSR-2](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md) コーディング標準と [PSR- 4](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md) オートローディング標準に従います。

<a name="phpdoc"></a>
### PHPDoc

以下は、有効な Laravel ドキュメント ブロックの例です。 `@param` 属性の後には2つのスペースがあり、引数の型、さらに 2つのスペース、最後に変数名が続きます。

    /**
     * Register a binding with the container.
     *
     * @param  string|array  $abstract
     * @param  \Closure|string|null  $concrete
     * @param  bool  $shared
     * @return void
     *
     * @throws \Exception
     */
    public function bind($abstract, $concrete = null, $shared = false)
    {
        // ...
    }

@paramまたは@return属性がネイティブ型の使用により冗長である場合は、削除することができます。

    /**
     * Execute the job.
     */
    public function handle(AudioProcessor $processor): void
    {
        //
    }

ただし、ネイティブ型が一般的である場合は、`@param` または `@return` 属性を使用して一般的な型を指定してください。

    /**
     * Get the attachments for the message.
     *
     * @return array<int, \Illuminate\Mail\Mailables\Attachment>
     */
    public function attachments(): array
    {
        return [
            Attachment::fromStorage('/path/to/file'),
        ];
    }

<a name="styleci"></a>
### StyleCI

コードスタイルが完璧でなくても心配しないでください。 [StyleCI](https://styleci.io/) は、プルリクエストがマージされた後、スタイルの修正を自動的に Laravel リポジトリにマージします。これにより、貢献の内容に焦点を当てることができ、コードスタイルは問題になりません。

<a name="行動規範"></a>
＃＃ 行動規範

Laravelの行動規範は、Rubyの行動規範を基にしています。行動規範に違反する行為がある場合は、Taylor Otwell（taylor@laravel.com）に報告してください。

<div class="content-list" markdown="1">

- 参加者は、反対意見に寛容であること。
- 参加者は、言語と行動が個人攻撃や軽蔑的な個人的発言を含まないようにする必要があります。
- 他者の言葉や行動を解釈するとき、参加者は常に善意を持って解釈する必要があります。
・ハラスメントと判断される行為は一切認めません。

</div>
