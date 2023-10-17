# バリデーション

- [はじめに](#introduction)
- [クイックスタート](#validation-quickstart)
    - [ルート定義](#quick-defining-the-routes)
    - [コントローラ作成](#quick-creating-the-controller)
    - [バリデーションロジック](#quick-writing-the-validation-logic)
    - [バリデーションエラーの表示](#quick-displaying-the-validation-errors)
    - [フォームの再入力](#repoptaining-forms)
    - [オプションフィールドに関する注意](#a-note-on-optional-fields)
    - [バリデーションエラーのレスポンス形式](#validation-error-response-format)
- [フォームリクエストのバリデーション](#form-request-validation)
    - [フォームリクエストの作成](#creating-form-requests)
    - [フォームリクエストの許可](#authorizing-form-requests)
    - [エラーメッセージのカスタマイズ](#customizing-the-error-messages)
    - [バリデーションのための入力準備](#preparing-input-for-validation)
- [バリデータの手動生成](#manually-creating-validators)
    - [自動リダイレクト](#automatic-redirection)
    - [名前付きエラーバッグ](#named-error-bags)
    - [エラー メッセージのカスタマイズ](#manual-customizing-the-error-messages)
    - [バリデーション後のフック](#after-validation-hook)
- [バリデーション済み入力値の利用](#working-with-validated-input)
- [エラーメッセージの取り扱い](#working-with-error-messages)
    - [Specifying Custom Messages In Language Files](#specifying-custom-messages-in-language-files)
    - [Specifying Attributes In Language Files](#specifying-attribute-in-language-files)
    - [Specifying Values In Language Files](#specifying-values-in-language-files)
- [Available Validation Rules](#available-validation-rules)
- [Conditionally Adding Rules](#conditionally-adding-rules)
- [Validating Arrays](#validating-arrays)
    - [Validating Nested Array Input](#validating-nested-array-input)
    - [Error Message Indexes & Positions](#error-message-indexes-and-positions)
- [Validating Files](#validating-files)
- [Validating Passwords](#validating-passwords)
- [Custom Validation Rules](#custom-validation-rules)
    - [Using Rule Objects](#using-rule-objects)
    - [Using Closures](#using-closures)
    - [Implicit Rules](#implicit-rules)

<a name="introduction"></a>
## はじめに

Laravel は、アプリケーションの受信データを検証（バリデーション）するために、いくつかの異なるアプローチを提供します。すべての受信 HTTP リクエストで使用できる `validate` メソッドを使用するのが最も一般的です。また、他のバリデーションのアプローチについても説明します。

Laravel には、データに適用できる便利なバリデーションルールを幅広く保有しています。特定のデータベーステーブル内で値が一意であるかどうかをバリデーションする機能も提供します。Laravel のすべてのバリデーション機能を理解できるように、これらのバリデーションルールのそれぞれについて詳しく説明します。

<a name="validation-quickstart"></a>
## クイックスタート

Laravel の強力なバリデーション機能について学ぶために、フォームをバリデーションし、ユーザーにエラーメッセージを表示する完全な例を見てみましょう。この高レベルの概要を読むことで、Laravel を使用して受信リクエストデータをバリデーションする一般的な方法を理解できるようになります。

<a name="quick-defining-the-routes"></a>
### ルート定義

まず、`routes/web.php` ファイルに以下のルートが定義されているとします。

    use App\Http\Controllers\PostController;

    Route::get('/post/create', [PostController::class, 'create']);
    Route::post('/post', [PostController::class, 'store']);

`GET` ルートはユーザーが新しいブログ投稿を作成するフォームを表示し、`POST` ルートは新しいブログ投稿をデータベースに保存します。

<a name="quick-creating-the-controller"></a>
### コントローラの作成

次に、これらのルートへの受信リクエストを処理する単純なコントローラを見てみましょう。ここでは `store` メソッドを空のままにしておきます。

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;
    use Illuminate\View\View;

    class PostController extends Controller
    {
        /**
         * Show the form to create a new blog post.
         */
        public function create(): View
        {
            return view('post.create');
        }

        /**
         * Store a new blog post.
         */
        public function store(Request $request): RedirectResponse
        {
            // Validate and store the blog post...

            $post = /** ... */

            return to_route('post.show', ['post' => $post->id]);
        }
    }

<a name="quick-writing-the-validation-logic"></a>
### バリデーションロジック

これで、新しいブログ投稿をバリデーションするロジックを `store` メソッドに入力する準備が整いました。これを行うには、`Illuminate\Http\Request` オブジェクトによって提供される `validate` メソッドを使用します。バリデーションルールにパスすると、コードは通常どおりに実行され続けます。ただし、バリデーションが失敗した場合は、 `Illuminate\Validation\ValidationException` 例外がスローされ、適切なエラーレスポンスが自動的にユーザーに返されます。

従来の HTTP リクエスト処理中にバリデーションが失敗した場合、直前の URL へのリダイレクトレスポンスが生成されます。受信リクエストが XHR リクエストの場合、[バリデーションエラーメッセージを含む JSON レスポンス](#validation-error-response-format) が返されます。

 `validate` メソッドをより深く理解するために、 `store` メソッドに戻りましょう。

    /**
     * 新しいブログ投稿の保存
     */
    public function store(Request $request): RedirectResponse
    {
        $validated = $request->validate([
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ]);

        // The blog post is valid...

        return redirect('/posts');
    }

ご覧のとおり、バリデーションルールは `validate` メソッドに渡されます。心配しないでください。利用可能なバリデーションルールはすべて [文書化](#available-validation-rules) されています。 繰り返しますが、バリデーションが失敗した場合は、適切なレスポンスが自動的に生成されます。バリデーションにパスすると、コントローラは通常どおりに実行を続けます。

あるいは、バリデーションルールを区切りる役割りを持つ `|` 文字列の代わりに、配列としてバリデーションルールを指定することもできます。

    $validatedData = $request->validate([
        'title' => ['required', 'unique:posts', 'max:255'],
        'body' => ['required'],
    ]);

さらに、`validateWithBag` メソッドを使用してリクエストをバリデーションし、エラーメッセージを [名前付きエラーバッグ](#named-error-bags) 内に保存することもできます。

    $validatedData = $request->validateWithBag('post', [
        'title' => ['required', 'unique:posts', 'max:255'],
        'body' => ['required'],
    ]);

<a name="stopping-on-first-validation-failure"></a>
#### 最初のバリデーション失敗時に停止

最初のバリデーションが失敗した後、残りの属性に対するバリデーションルールの実行を停止したい場合があります。これを行うには、`bail` ルールを属性に割り当てます。

    $request->validate([
        'title' => 'bail|required|unique:posts|max:255',
        'body' => 'required',
    ]);

この例では、`title` 属性の `unique` ルールが失敗した場合、`max` ルールはチェックされません。ルールは割り当てられた順序でバリデーションされます。

<a name="a-note-on-nested-attributes"></a>
#### ネストした属性に注意

受信 HTTP リクエストに 「ネストされた」フィールドデータが含まれている場合は、「ドット」構文を使用してバリデーションルールでこれらのフィールドを指定できます。

    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'author.name' => 'required',
        'author.description' => 'required',
    ]);

フィールド名にピリオドが含まれている場合は、そのピリオドをバックスラッシュでエスケープすることで、これが「ドット」構文として解釈されるのを明示的に防ぐことができます。

    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'v1\.0' => 'required',
    ]);

<a name="quick-displaying-the-validation-errors"></a>
### バリデーションエラーの表示

受信リクエストフィールドが指定されたバリデーションルーをパスしない場合はどうなるでしょうか? 前述したように、Laravel はユーザーを直前の場所に自動的にリダイレクトします。さらに、すべてのバリデーションエラーと [リクエスト入力](/docs/{{version}}/requests#retrieving-old-input) は自動的に [セッションに一時保存](/docs/{{version}}/session#flash-data) されます。

`$errors` 変数は、`web` ミドルウェアグループが提供する `Illuminate\View\Middleware\ShareErrorsFromSession` ミドルウェアによってアプリケーションのすべてのビューで共有されます。このミドルウェアが適用されると、常にビューで `$errors` 変数が使用できるようになり、`$errors` 変数が常に定義され、安全かつ便利に使用できることでしょう。`$errors` 変数は `Illuminate\Support\MessageBag` のインスタンスになります。このオブジェクトの操作の詳細については、[ドキュメントを確認してください](#working-with-error-messages)。

この例では、バリデーションが失敗したときにユーザーはコントローラの `create` メソッドにリダイレクトされ、ビューにエラーメッセージを表示させられます。

```blade
<!-- /resources/views/post/create.blade.php -->

<h1>Create Post</h1>

@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif

<!-- Create Post Form -->
```

<a name="quick-customizing-the-error-messages"></a>
#### エラーメッセージのカスタマイズ

Laravel の組み込みバリデーションルールにはそれぞれエラーメッセージがあり、アプリケーションの `lang/en/validation.php` ファイルにあります。このファイル内に、各バリデーションルールの翻訳エントリがあります。アプリケーションのニーズに基づいて、これらのメッセージを自由に変更または修正できます。

さらに、このファイルを別の言語ディレクトリにコピーして、メッセージをアプリケーションの言語に翻訳することもできます。Laravel の多言語化の詳細については、完全な [多言語化ドキュメント](/docs/{{version}}/localization) をご覧ください。

> **Warning**
> デフォルトでは、Laravel アプリケーションのスケルトンには `lang` ディレクトリが含まれません。Laravel の言語ファイルをカスタマイズしたい場合は、`lang:publish` Artisan コマンドを使用して言語ファイルを公開できます。

<a name="quick-xhr-requests-and-validation"></a>
#### XHR リクエストとバリデーション

この例では、従来のフォームを使用してデータをアプリケーションに送信しました。ただし、多くのアプリケーションは、JavaScript を利用したフロントエンドから XHR リクエストを受信します。XHRリクエスト中に `validate` メソッドを使用すると、Laravelはリダイレクトレスポンスを生成しません。代わりに、Laravel は [すべてのバリデーションエラーを含む JSON レスポンス](#validation-error-response-format) を生成します。この JSON レスポンスは 422 HTTP ステータスコードとともに送信されます。

<a name="the-at-error-directive"></a>
#### `@error` ディレクティブ

`@error` [Blade](/docs/{{version}}/blade) ディレクティブを使用すると、特定の属性にバリデーションエラーメッセージが存在するかどうかを迅速に判断できます。`@error` ディレクティブ内で、`$message` 変数をエコーしてエラーメッセージを表示できます。

```blade
<!-- /resources/views/post/create.blade.php -->

<label for="title">Post Title</label>

<input id="title"
    type="text"
    name="title"
    class="@error('title') is-invalid @enderror">

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

[名前付きエラーバッグ](#named-error-bags) を使用している場合は、エラー バッグの名前を `@error` ディレクティブの第２引数に渡せます。

```blade
<input ... class="@error('title', 'post') is-invalid @enderror">
```

<a name="repopulating-forms"></a>
### フォームの再入力

Laravel がバリデーションエラーによりリダイレクトレスポンスを生成すると、フレームワークは自動的に [すべてのリクエストの入力をセッションに一時保存](/docs/{{version}}/session#flash-data) します。これは、直後のリクエストの際、一時保存した入力データにアクセスし、ユーザーが送信しようとしたフォームに再入力できるようにするためです。

直前のリクエストから一時保存された入力を取得するには、`Illuminate\Http\Request` のインスタンスで `old` メソッドを呼び出します。`old` メソッドは、直前に一時保存された入力データを [セッション](/docs/{{version}}/session) から取得します。

    $title = $request->old('title');

Laravel はグローバルな `old` ヘルパも提供します。[Blade テンプレート](/docs/{{version}}/blade) 内で直前の入力を表示している場合は、`old` ヘルパを使用してフォームに再入力する方法が便利です。指定されたフィールドに直前の入力が存在しない場合は、`null` が返されます。

```blade
<input type="text" name="title" value="{{ old('title') }}">
```

<a name="a-note-on-optional-fields"></a>
### オプションのフィールドに関する注意

デフォルトで Laravel にはアプリケーションのグローバルミドルウェアスタックに `TrimStrings` および `ConvertEmptyStringsToNull` ミドルウェアが含まれています。これらのミドルウェアは、スタック内の `App\Http\Kernel` クラスにリストされています。このため、バリデータに `null` 値を無効と判定させたくない場合は、「オプション」のリクエストフィールドを `nullable` としてマークする必要があります。例えば

    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
        'publish_at' => 'nullable|date',
    ]);

この例では、 `publish_at` フィールドが `null` または有効な日付表現のいずれかであることを指定しています。`nullable` 修飾子がルール定義に追加されていない場合、バリデータは `null` を無効な日付として判定します。

<a name="validation-error-response-format"></a>
### バリデーションエラーのレスポンス形式

アプリケーションが `Illuminate\Validation\ValidationException` 例外をスローし、受信 HTTP リクエストが JSON レスポンスを期待している場合、Laravel はエラーメッセージを自動的にフォーマットし、`422 Unprocessable Entity` HTTP レスポンスを返します。

以下、バリデーションエラーの JSON レスポンス形式の例を確認できます。ネストされたエラーのキーは「ドット」記法に平坦化されることに注意してください。

```json
{
    "message": "The team name must be a string. (and 4 more errors)",
    "errors": {
        "team_name": [
            "The team name must be a string.",
            "The team name must be at least 1 characters."
        ],
        "authorization.role": [
            "The selected authorization.role is invalid."
        ],
        "users.0.email": [
            "The users.0.email field is required."
        ],
        "users.2.email": [
            "The users.2.email must be a valid email address."
        ]
    }
}
```

<a name="form-request-validation"></a>
## フォームリクエストのバリデーション

<a name="creating-form-requests"></a>
### フォームリクエストの作成

より複雑なバリデーションシナリオの場合は、「フォームリクエスト」を作成すると良いでしょう。フォームリクエストは、独自のバリデーションおよび認可ロジックをカプセル化するカスタムリクエストクラスです。フォームリクエストクラスを作成するには、`make:request` Artisan CLI コマンドを使用します。

```shell
php artisan make:request StorePostRequest
```

生成されたフォームリクエストクラスは `app/Http/Requests` ディレクトリに配置されます。このディレクトリが存在しない場合、`make:request` コマンドを実行する際に作成されます。Laravel によって生成される各フォームリクエストには、`authorize` と `rules` という２つのメソッドがあります。

ご想像のとおり、`authorize` メソッドは、現在認証されているユーザーがリクエストで表されるアクションを実行できるかを判断し、`rules` メソッドはリクエストのデータに適用する必要があるバリデーションルールを返します。

    /**
     * リクエストに適用されるバリデーションルールを取得
     *
     * @return array<string, \Illuminate\Contracts\Validation\Rule|array|string>
     */
    public function rules(): array
    {
        return [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ];
    }

> **Note**
> `rules` メソッドの引数で必要な依存関係をタイプヒントで指定できます。これらは、Laravel [サービスコンテナ](/docs/{{version}}/container) を介して自動的に解決されます。

では、バリデーションルールはどのように評価されるのでしょうか？必要なのは、コントローラメソッドでリクエストをタイプヒントすることだけです。受信フォームリクエストはコントローラメソッドが呼び出される前にバリデーションされます。つまり、コントローラにバリデーションロジックを複雑に記述する必要はありません。

    /**
     * 新しいブログ投稿を保存
     */
    public function store(StorePostRequest $request): RedirectResponse
    {
        // 有効な受信リクエスト

        // バリデーション済み入力データを取得
        $validated = $request->validated();

        // バリデーションされた入力データの一部を取得
        $validated = $request->safe()->only(['name', 'email']);
        $validated = $request->safe()->except(['name', 'email']);

        // ブログ投稿を保存

        return redirect('/posts');
    }

バリデーションが失敗した場合は、ユーザーを直前のページに戻すリダイレクトレスポンスが生成されます。エラーもセッションにも一時保存されるので、表示できるようになります。リクエストが XHR リクエストの場合、[バリデーションエラーの JSON 表現](#validation-error-response-format) を含む 422 ステータスコードを HTTP レスポンスによってユーザーへと返却されます。

<a name="adding-after-hooks-to-form-requests"></a>
#### フォームリクエストへの After フックの追加

フォームリクエストに「after」バリデーションフックを追加したい場合は、`withValidator` メソッドを使用できます。 このメソッドは完全に構築されたバリデータを受け取るため、バリデーションルールが実際に評価される前にそのメソッドのいずれかを呼び出すことができます。

    use Illuminate\Validation\Validator;

    /**
     * バリデータインスタンスの設定
     */
    public function withValidator(Validator $validator): void
    {
        $validator->after(function (Validator $validator) {
            if ($this->somethingElseIsInvalid()) {
                $validator->errors()->add('field', 'Something is wrong with this field!');
            }
        });
    }


<a name="request-stopping-on-first-validation-rule-failure"></a>
#### バリデーション失敗時の停止処理

リクエストクラスに `stopOnFirstFailure` プロパティを追加すると、バリデーションエラーが発生した場合にすべての属性のバリデーションを停止するようバリデータに指示します。

    /**
     * ルールが失敗した際バリデータを停止するかどうか
     *
     * @var bool
     */
    protected $stopOnFirstFailure = true;

<a name="customizing-the-redirect-location"></a>
#### リダイレクト先のカスタマイズ

前述したように、フォームリクエストのバリデーションに失敗した場合、ユーザーを直前の場所に戻すリダイレクトレスポンスが生成されます。ただし、この動作は自由にカスタマイズできます。これを行うには、フォームリクエストで `$redirect` プロパティを定義します。

    /**
     * バリデーション失敗時にリダイレクトする URI
     *
     * @var string
     */
    protected $redirect = '/dashboard';

または、ユーザーを名前付きルートにリダイレクトしたい場合は、代わりに `$redirectRoute` プロパティを定義します。

    /**
     * バリデーション失敗時にリダイレクトするルート
     *
     * @var string
     */
    protected $redirectRoute = 'dashboard';

<a name="authorizing-form-requests"></a>
### フォームリクエストの許可

フォームリクエストクラスには `authorize` メソッドも含まれます。このメソッド内で、認証済みユーザーが実際に特定のリソースを更新する権限を持っているかどうかを判断できます。たとえば、ユーザーが更新しようとしているブログコメントを実際に所有しているかどうかを判断できます。おそらく、このメソッド内で [認可ゲートとポリシー](/docs/{{version}}/authorization) を操作することになります。

    use App\Models\Comment;

    /**
     * ユーザーがこのリクエストの権限を持っているか判断
     */
    public function authorize(): bool
    {
        $comment = Comment::find($this->route('comment'));

        return $comment && $this->user()->can('update', $comment);
    }

すべてのフォームリクエストは Laravel のベースリクエストクラスを拡張するため、`user` メソッドを使用して現在認証されているユーザーにアクセスできます。また、上の例の `route` メソッドの呼び出しにも注目してください。このメソッドを使用すると、呼び出されるルートで定義されている URI パラメータ (以下の例の `{comment}` パラメータなど) へのアクセスが許可されます。

    Route::post('/comment/{comment}');

したがって、アプリケーションが [ルートモデル結合](/docs/{{version}}/routing#route-model-binding) を利用している場合、解決済みモデルにリクエストのプロパティとしてアクセスすることで、コードをさらに簡潔にすることができます。

    return $this->user()->can('update', $this->comment);

`authorize` メソッドが `false` を返した場合、403 ステータスコードを含む HTTP レスポンスが自動的に返され、コントローラメソッドは実行されません。

アプリケーションの別の部分でリクエストの認可ロジックを処理する場合は、`authorize` メソッドから `true` を返してください。

    /**
     * ユーザーがこのリクエストを行う権限を持っているか判断
     */
    public function authorize(): bool
    {
        return true;
    }

> **Note**
> `authorize` メソッドの引数で必要な依存関係をタイプヒントで指定できます。これらは、Laravelの [サービスコンテナ](/docs/{{version}}/container) を介して自動的に依存性解決されます。

<a name="customizing-the-error-messages"></a>
### エラーメッセージのカスタマイズ

`messages` メソッドをオーバーライドすることで、フォームリクエストで使用されるエラーメッセージをカスタマイズできます。このメソッドは、「属性とルールのペア」と「それに対応するエラーメッセージ」の配列を返す必要があります。

    /**
     * 定義済みのバリデーションルールのエラーメッセージを取得
     *
     * @return array<string, string>
     */
    public function messages(): array
    {
        return [
            'title.required' => 'A title is required',
            'body.required' => 'A message is required',
        ];
    }

<a name="customizing-the-validation-attributes"></a>
#### バリデーション属性のカスタマイズ

Laravel の組み込みバリデーションルールのエラーメッセージの多くには、`:attribute` プレースホルダが含まれています。バリデーションメッセージの `:attribute` プレースホルダをカスタム属性名に置き換えたい場合は、`attributes` メソッドをオーバーライドしてカスタム名を指定できます。このメソッドは、属性と名前のペアの配列を返す必要があります。

    /**
     * バリデーションエラーのカスタム属性を取得
     *
     * @return array<string, string>
     */
    public function attributes(): array
    {
        return [
            'email' => 'email address',
        ];
    }

<a name="preparing-input-for-validation"></a>
### バリデーションのための入力準備

バリデーションルールを適用する前にリクエストからのデータを準備、またはサニタイズする必要がある場合は、`prepareForValidation` メソッドを使用します。

    use Illuminate\Support\Str;

    /**
     * バリデーションのためのデータを準備
     */
    protected function prepareForValidation(): void
    {
        $this->merge([
            'slug' => Str::slug($this->slug),
        ]);
    }

同様に、バリデーションの完了後にリクエストデータをノーマライズする必要がある場合は、`passedValidation` メソッドを使用します。

    use Illuminate\Support\Str;

    /**
     * バリデーション完了後の処理
     */
    protected function passedValidation(): void
    {
        $this->replace(['name' => 'Taylor']);
    }

<a name="manually-creating-validators"></a>
## バリデータの手動生成

リクエストで `validate` メソッドを使用したくない場合は、`Validator` [ファサード](/docs/{{version}}/facades) を使用して、バリデータインスタンスを手動で作成します。このファサードの `make` メソッドは、新しいバリデータインスタンスを生成します。

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Validator;

    class PostController extends Controller
    {
        /**
         * 新しいブログ投稿を保存
         */
        public function store(Request $request): RedirectResponse
        {
            $validator = Validator::make($request->all(), [
                'title' => 'required|unique:posts|max:255',
                'body' => 'required',
            ]);

            if ($validator->fails()) {
                return redirect('post/create')
                            ->withErrors($validator)
                            ->withInput();
            }

            // バリデーションされた入力を取得
            $validated = $validator->validated();

            // バリデーションされた入力の一部を取得
            $validated = $validator->safe()->only(['name', 'email']);
            $validated = $validator->safe()->except(['name', 'email']);

            // ブログ投稿を保存

            return redirect('/posts');
        }
    }

`make` メソッドの第１引数は、バリデーション対象のデータです。第２引数は、データに適用する必要があるバリデーションルールの配列です。

リクエストのバリデーションが失敗したかどうかを判断した後、`withErrors` メソッドを使用してエラーメッセージをセッションに一時保存できます。この方法を使用すると、リダイレクト後に `$errors` 変数がビューと自動的に共有されるため、エラーメッセージを簡単にユーザーへと表示できるようになります。`withErrors` メソッドは、バリデータ、`MessageBag`、または PHP の `array` を引数に取ります。

#### 最初のバリデーション失敗時に停止

`stopOnFirstFailure` メソッドは、バリデーションエラーが１回発生すると、すべての属性のバリデーションを停止する必要があることをバリデータに指示します。

    if ($validator->stopOnFirstFailure()->fails()) {
        // ...
    }

<a name="automatic-redirection"></a>
### 自動リダイレクト

バリデータインスタンスを手動で作成する際に、HTTP リクエストの `validate` メソッドによって提供される自動リダイレクトを利用したい場合は、既存のバリデータインスタンスで `validate` メソッドを呼び出すことができます。バリデーションが失敗した場合、ユーザーは自動的にリダイレクトされます。XHR リクエストの場合は、[JSON レスポンスが返されます。](#validation-error-response-format)

    Validator::make($request->all(), [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ])->validate();

`validateWithBag` メソッドを使用して、バリデーションが失敗した場合は、エラーメッセージを [名前付きエラーバッグ](#named-error-bags) に保存できます。

    Validator::make($request->all(), [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ])->validateWithBag('post');

<a name="named-error-bags"></a>
### 名前付きエラーバッグ

1 つのページに複数のフォームがある場合は、バリデーションエラーを含む `MessageBag` に名前を付けると、特定のフォームのエラーメッセージを取得できるようになります。これを実現するには、第２引数に名前を指定して `withErrors` に渡します。

    return redirect('register')->withErrors($validator, 'login');

その後、`$errors` 変数から名前付きの `MessageBag` インスタンスにアクセスできます。

```blade
{{ $errors->login->first('email') }}
```

<a name="manual-customizing-the-error-messages"></a>
### エラーメッセージのカスタマイズ

必要に応じて、Laravel が提供するデフォルトのエラーメッセージの代わりに、バリデータインスタンスが使用するカスタムエラーメッセージを指定できます。カスタムメッセージを指定するにはいくつかの方法があります。まず、カスタムメッセージを第３引数に指定して `Validator::make` メソッドに渡します。

    $validator = Validator::make($input, $rules, $messages = [
        'required' => 'The :attribute field is required.',
    ]);

この例では、`:attribute` プレースホルダはバリデーション中のフィールドの実際の名前に置き換えられます。バリデーションメッセージでは他のプレースホルダを利用することもできます。 例えば

    $messages = [
        'same' => 'The :attribute and :other must match.',
        'size' => 'The :attribute must be exactly :size.',
        'between' => 'The :attribute value :input is not between :min - :max.',
        'in' => 'The :attribute must be one of the following types: :values',
    ];

<a name="specifying-a-custom-message-for-a-given-attribute"></a>
#### 特定の属性にカスタムメッセージの指定

場合によっては、特定の属性に対してのみカスタムエラーメッセージを指定したい場合があります。「ドット」表記を使用してこれを行うことができます。最初に属性の名前を指定し、次にルールを指定します。

    $messages = [
        'email.required' => 'We need to know your email address!',
    ];

<a name="specifying-custom-attribute-values"></a>
#### カスタム属性値の指定

Laravel の組み込みエラーメッセージの多くには、バリデーション中のフィールドまたは属性の名前に置き換えられる `:attribute` プレースホルダが含まれています。特定のフィールドのこれらのプレースホルダを置換するために使用される値をカスタマイズするには、カスタム属性の配列を第４引数として `Validator::make` メソッドに渡します。

    $validator = Validator::make($input, $rules, $messages, [
        'email' => 'email address',
    ]);

<a name="after-validation-hook"></a>
### After Validation Hook

You may also attach callbacks to be run after validation is completed. This allows you to easily perform further validation and even add more error messages to the message collection. To get started, call the `after` method on a validator instance:

    use Illuminate\Support\Facades;
    use Illuminate\Validation\Validator;

    $validator = Facades\Validator::make(/* ... */);

    $validator->after(function (Validator $validator) {
        if ($this->somethingElseIsInvalid()) {
            $validator->errors()->add(
                'field', 'Something is wrong with this field!'
            );
        }
    });

    if ($validator->fails()) {
        // ...
    }

<a name="working-with-validated-input"></a>
## バリデーション済み入力値の利用

フォームリクエストまたは手動で作成したバリデータインスタンスを使用して受信リクエストデータをバリデーションした後、実際にバリデーション済みの受信リクエストデータを取得したい場合があります。これはいくつかの方法で実現できます。まず、フォームリクエストまたはバリデータインスタンスで `validated` メソッドを呼び出す方法です。このメソッドは、バリデーション済みデータの配列を返します。

    $validated = $request->validated();

    $validated = $validator->validated();

または、フォームリクエストやバリデータのインスタンスで `safe` メソッドを呼び出すこともできます。このメソッドは、 `Illuminate\Support\ValidatedInput` のインスタンスを返します。このオブジェクトは、バリデーション済みデータのサブセットや配列全体を取得するための `only`、`except`、`all` メソッドを用意しています。

    $validated = $request->safe()->only(['name', 'email']);

    $validated = $request->safe()->except(['name', 'email']);

    $validated = $request->safe()->all();

さらに、`Illuminate\Support\ValidatedInput` インスタンスは配列のようにループ処理してアクセスすることも可能です。

    // バリデーション済みデータをループ処理
    foreach ($request->safe() as $key => $value) {
        // ...
    }

    // バリデーション済みデータを配列としてアクセス
    $validated = $request->safe();

    $email = $validated['email'];

バリデーション済みデータにフィールドを追加したい場合は、`merge` メソッドを呼び出します。

    $validated = $request->safe()->merge(['name' => 'Taylor Otwell']);

バリデーションされたデータを [コレクション](/docs/{{version}}/collections) インスタンスとして取得したい場合は、`collect` メソッドを呼び出します。

    $collection = $request->safe()->collect();

<a name="working-with-error-messages"></a>
## エラーメッセージの取り扱い

`Validator` インスタンスで `errors` メソッドを呼び出した後、`Illuminate\Support\MessageBag` インスタンスを受け取ります。このインスタンスには、エラーメッセージを処理するためのさまざまな便利なメソッドが含まれています。すべてのビューで自動的に利用可能になる `$errors` 変数も、`MessageBag` クラスのインスタンスです。

<a name="retrieving-the-first-error-message-for-a-field"></a>
#### 指定フィールドの最初のエラーメッセージ取得

指定したフィールドの最初のエラーメッセージを取得するには、`first` メソッドを使用します。

    $errors = $validator->errors();

    echo $errors->first('email');

<a name="retrieving-all-error-messages-for-a-field"></a>
#### 指定フィールドの全エラーメッセージの取得

指定したフィールドのすべてのメッセージの配列を取得する必要がある場合は、`get` メソッドを使用します。

    foreach ($errors->get('email') as $message) {
        // ...
    }

配列形式のフィールドをバリデーションする場合は、`*` 文字を使用して配列要素ごとにすべてのメッセージを取得できます。

    foreach ($errors->get('attachments.*') as $message) {
        // ...
    }

<a name="retrieving-all-error-messages-for-all-fields"></a>
#### 全フィールドの全エラーメッセージを取得

全フィールドの全メッセージの配列を取得するには、`all` メソッドを使用します。

    foreach ($errors->all() as $message) {
        // ...
    }

<a name="determining-if-messages-exist-for-a-field"></a>
#### 指定フィールドのメッセージ存在確認

`has` メソッドは、指定のフィールドにエラーメッセージが存在するかどうかを判断するために使用します。

    if ($errors->has('email')) {
        // ...
    }

<a name="specifying-custom-messages-in-language-files"></a>
### 言語ファイルでのカスタムメッセージの指定

Laravel の組み込みバリデーションルールにはそれぞれエラーメッセージがあり、アプリケーションの `lang/en/validation.php` ファイルにあります。このファイル内に、各バリデーションルールの翻訳エントリがあります。アプリケーションのニーズに基づいて、これらのメッセージを自由に変更または修正できます。

さらに、このファイルを別の言語ディレクトリにコピーして、メッセージをアプリケーションの言語に翻訳することもできます。 Laravel の多言語化の詳細については、完全な [多言語化ドキュメント](/docs/{{version}}/localization) をご覧ください。

> **Warning**
> デフォルトでは、Laravel アプリケーションのスケルトンには `lang` ディレクトリが含まれません。Laravel の言語ファイルをカスタマイズしたい場合は、`lang:publish` Artisan コマンドを使用して言語ファイルを公開できます。

<a name="custom-messages-for-specific-attributes"></a>
#### 特定の属性のカスタムメッセージ

アプリケーションのバリデーション言語ファイルにて、指定した属性とルールの組み合わせに使用されるエラーメッセージをカスタマイズできます。これを行うには、アプリケーションの `lang/xx/validation.php` 言語ファイルの `custom` 配列にカスタマイズしたメッセージを追加します。

    'custom' => [
        'email' => [
            'required' => 'We need to know your email address!',
            'max' => 'Your email address is too long!'
        ],
    ],

<a name="specifying-attribute-in-language-files"></a>
### 言語ファイルでの属性の指定

Laravel の組み込みエラーメッセージの多くには、バリデーション中のフィールドまたは属性の名前に置き換えられる `:attribute` プレースホルダが含まれています。バリデーションメッセージの `:attribute` 部分をカスタム値に置き換えたい場合は、`lang/xx/validation.php` 言語ファイルの `attributes` 配列でカスタム属性名を指定できます。

    'attributes' => [
        'email' => 'email address',
    ],

> **Warning**
> デフォルトでは、Laravel アプリケーションのスケルトンには `lang` ディレクトリが含まれません。 Laravel の言語ファイルをカスタマイズしたい場合は、 `lang:publish` Artisan コマンドを使用して言語ファイルを公開できます。

<a name="specifying-values-in-language-files"></a>
### 言語ファイルでの値の指定

Laravel の組み込みバリデーションルールのエラーメッセージの一部には、リクエスト属性の現在の値に置き換えられる `:value` プレースホルダが含まれています。ただし、場合によっては、バリデーションメッセージの `:value` 部分を値のカスタム表現に置き換えたい場合があります。たとえば、 `payment_type` の値が `cc` の場合にクレジットカード番号が必要であることを指定する次のルールを考えてみましょう。

    Validator::make($request->all(), [
        'credit_card_number' => 'required_if:payment_type,cc'
    ]);

このバリデーションルールが失敗すると、次のエラーメッセージが生成されます。

```none
The credit card number field is required when payment type is cc.（訳：支払いタイプが cc の場合、クレジットカード番号フィールドは必須となります。）
```

支払いタイプの値として `cc` を表示する代わりに、`lang/xx/validation.php` 言語ファイル内に `values` 配列を定義することで、よりユーザーフレンドリーな値表現を指定できます。

    'values' => [
        'payment_type' => [
            'cc' => 'credit card'
        ],
    ],

> **Warning**
> デフォルトでは、Laravel アプリケーションのスケルトンには `lang` ディレクトリが含まれません。Laravel の言語ファイルをカスタマイズしたい場合は、`lang:publish` Artisan コマンドを使用して言語ファイルを公開できます。

この値を定義すると、バリデーションルールによって次のエラーメッセージが生成されます。

```none
The credit card number field is required when payment type is credit card.（訳：支払いタイプがクレジットカードの場合、クレジットカード番号フィールドは必須です。）
```

<a name="available-validation-rules"></a>
## 利用可能なバリデーションルール

以下は、利用可能なすべてのバリデーションルールとその機能の一覧です。

<style>
    .collection-method-list > p {
        columns: 10.8em 3; -moz-columns: 10.8em 3; -webkit-columns: 10.8em 3;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
</style>

<div class="collection-method-list" markdown="1">

[Accepted](#rule-accepted)
[Accepted If](#rule-accepted-if)
[Active URL](#rule-active-url)
[After (Date)](#rule-after)
[After Or Equal (Date)](#rule-after-or-equal)
[Alpha](#rule-alpha)
[Alpha Dash](#rule-alpha-dash)
[Alpha Numeric](#rule-alpha-num)
[Array](#rule-array)
[Ascii](#rule-ascii)
[Bail](#rule-bail)
[Before (Date)](#rule-before)
[Before Or Equal (Date)](#rule-before-or-equal)
[Between](#rule-between)
[Boolean](#rule-boolean)
[Confirmed](#rule-confirmed)
[Current Password](#rule-current-password)
[Date](#rule-date)
[Date Equals](#rule-date-equals)
[Date Format](#rule-date-format)
[Decimal](#rule-decimal)
[Declined](#rule-declined)
[Declined If](#rule-declined-if)
[Different](#rule-different)
[Digits](#rule-digits)
[Digits Between](#rule-digits-between)
[Dimensions (Image Files)](#rule-dimensions)
[Distinct](#rule-distinct)
[Doesnt Start With](#rule-doesnt-start-with)
[Doesnt End With](#rule-doesnt-end-with)
[Email](#rule-email)
[Ends With](#rule-ends-with)
[Enum](#rule-enum)
[Exclude](#rule-exclude)
[Exclude If](#rule-exclude-if)
[Exclude Unless](#rule-exclude-unless)
[Exclude With](#rule-exclude-with)
[Exclude Without](#rule-exclude-without)
[Exists (Database)](#rule-exists)
[File](#rule-file)
[Filled](#rule-filled)
[Greater Than](#rule-gt)
[Greater Than Or Equal](#rule-gte)
[Image (File)](#rule-image)
[In](#rule-in)
[In Array](#rule-in-array)
[Integer](#rule-integer)
[IP Address](#rule-ip)
[JSON](#rule-json)
[Less Than](#rule-lt)
[Less Than Or Equal](#rule-lte)
[Lowercase](#rule-lowercase)
[MAC Address](#rule-mac)
[Max](#rule-max)
[Max Digits](#rule-max-digits)
[MIME Types](#rule-mimetypes)
[MIME Type By File Extension](#rule-mimes)
[Min](#rule-min)
[Min Digits](#rule-min-digits)
[Missing](#rule-missing)
[Missing If](#rule-missing-if)
[Missing Unless](#rule-missing-unless)
[Missing With](#rule-missing-with)
[Missing With All](#rule-missing-with-all)
[Multiple Of](#rule-multiple-of)
[Not In](#rule-not-in)
[Not Regex](#rule-not-regex)
[Nullable](#rule-nullable)
[Numeric](#rule-numeric)
[Password](#rule-password)
[Present](#rule-present)
[Prohibited](#rule-prohibited)
[Prohibited If](#rule-prohibited-if)
[Prohibited Unless](#rule-prohibited-unless)
[Prohibits](#rule-prohibits)
[Regular Expression](#rule-regex)
[Required](#rule-required)
[Required If](#rule-required-if)
[Required Unless](#rule-required-unless)
[Required With](#rule-required-with)
[Required With All](#rule-required-with-all)
[Required Without](#rule-required-without)
[Required Without All](#rule-required-without-all)
[Required Array Keys](#rule-required-array-keys)
[Same](#rule-same)
[Size](#rule-size)
[Sometimes](#validating-when-present)
[Starts With](#rule-starts-with)
[String](#rule-string)
[Timezone](#rule-timezone)
[Unique (Database)](#rule-unique)
[Uppercase](#rule-uppercase)
[URL](#rule-url)
[ULID](#rule-ulid)
[UUID](#rule-uuid)

</div>

<a name="rule-accepted"></a>
#### accepted

バリデーション対象のフィールドは、`"yes"`、`"on"`、`1`、`true` でなければなりません。これは、「利用規約」への同意または同様のフィールドをバリデーションするのに役立ちます。

<a name="rule-accepted-if"></a>
#### accepted_if:他のフィールド,値,...

「他のフィールド」が指定した「値」と等しい場合、このフィールドが `"yes"`、`"on"`、`1`、`true`であることをバリデーション処理します。これは「利用規約」への同意または同様のフィールドをバリデーションするのに役立ちます。

<a name="rule-active-url"></a>
#### active_url

バリデーション対象のフィールドには、`dns_get_record` PHP 関数に従って有効な A または AAAA レコードが必要です。指定した URL のホスト名は、`dns_get_record` に渡される前に、`parse_url` PHP 関数を使用して抽出されます。

<a name="rule-after"></a>
#### after:_日付_

バリデーション対象のフィールドは、指定された日付以降の値であるかをバリデーション処理します。日付は有効な `DateTime` インスタンスに変換するために `strtotime` PHP 関数に渡されます。

    'start_date' => 'required|date|after:tomorrow'

`strtotime` によって評価される日付文字列を渡す代わりに、日付と比較する他のフィールドを指定できます。

    'finish_date' => 'required|date|after:start_date'

<a name="rule-after-or-equal"></a>
#### after\_or\_equal:_日付_

バリデーション対象のフィールドは、指定した日付以降の値であるかをバリデーション処理します。詳細については、[after](#rule-after) ルールを参照してください。

<a name="rule-alpha"></a>
#### alpha

バリデーション対象のフィールドは、[`\p{L}`](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AL%3A%5D&g=&i=) と [`\p{M}`](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AM%3A%5D&g=&i=) に含まれる Unicode のアルファベット文字のみであるかをバリデーション処理します。

このバリデーションルールを ASCII 範囲 (`a-z` と `A-Z`) の文字に制限するには、バリデーションルールに `ascii` オプションを指定します。

```php
'username' => 'alpha:ascii',
```

<a name="rule-alpha-dash"></a>
#### alpha_dash

バリデーション対象のフィールドは、[`\p{L}`](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AL%3A%5D&g=&i=) と [`\p{M}`](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AM%3A%5D&g=&i=) に含まれる Unicode の英数字、ASCII ダッシュ (`-`) および ASCII アンダースコア (`_`) であるかをバリデーション処理します。

このバリデーションルールを ASCII 範囲 (`a-z` と `A-Z`) の文字に制限するには、バリデーションルールに `ascii` オプションを指定します。

```php
'username' => 'alpha_dash:ascii',
```

<a name="rule-alpha-num"></a>
#### alpha_num

バリデーション対象のフィールドは、[`\p{L}`](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AL%3A%5D&g=&i=) と [`\p{M}`](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AM%3A%5D&g=&i=) に含まれる Unicode の英数字であるかをバリデーション処理します。

このバリデーションルールを ASCII 範囲 (`a-z` および `A-Z`) の文字に制限するには、バリデーションルールに `ascii` オプションを指定します。

```php
'username' => 'alpha_num:ascii',
```

<a name="rule-array"></a>
#### array

バリデーション対象のフィールドは PHP の `配列` であるかをバリデーション処理します。

追加の値を `array` ルールに指定する場合、入力配列内の各キーがルールに指定される値のリスト内に存在する必要があります。次の例では、入力配列の `admin` キーは、`array` ルールに指定した値のリストに含まれていないため無効となります。

    use Illuminate\Support\Facades\Validator;

    $input = [
        'user' => [
            'name' => 'Taylor Otwell',
            'username' => 'taylorotwell',
            'admin' => true,
        ],
    ];

    Validator::make($input, [
        'user' => 'array:name,username',
    ]);

一般的に、配列内に存在できる配列キーを常に指定する必要があります。

<a name="rule-ascii"></a>
#### ascii

バリデーション対象のフィールドが全て 7 ビット ASCII 文字であるかをバリデーション処理します。

<a name="rule-bail"></a>
#### bail

バリデーションが最初に失敗した時点で、残りのバリデーションルールの実行を停止します。

 `bail` ルールは、バリデーションが失敗が発生した場合にのみ特定フィールドのバリデーションのみを停止しますが、`stopOnFirstFailure` メソッドは、ひとつのバリデーションの失敗が発生するとすべての属性のバリデーションを停止する必要があることをバリデータに通知します。

    if ($validator->stopOnFirstFailure()->fails()) {
        // ...
    }

<a name="rule-before"></a>
#### before:_日付_

バリデーション対象のフィールドは、指定された日付より前の値である必要があります。日付は有効な `DateTime` インスタンスに変換するために PHP の `strtotime` 関数に渡されます。さらに、[`after`](#rule-after) ルールと同様に、バリデーション中の別のフィールドの名前を `date` の値として指定することもできます。

<a name="rule-before-or-equal"></a>
#### before\_or\_equal:_日付_

バリデーション対象のフィールドは、指定された日付以前の値である必要があります。日付は有効な `DateTime` インスタンスに変換するために PHP の `strtotime` 関数に渡されます。さらに、[`after`](#rule-after) ルールと同様に、バリデーション中の別のフィールドの名前を `date` の値として指定することもできます。

<a name="rule-between"></a>
#### between:_最大値_,_最小値_

バリデーション中のフィールドのサイズは、指定された _最小値_ と _最大値_ (両端を含む) の間である必要があります。文字列、数値、配列、ファイルは、[`size`](#rule-size) ルールと同じ方法で評価されます。

<a name="rule-boolean"></a>
#### boolean

バリデーション中のフィールドは、論理値として有効である必要があります。受け入れられる入力は、`true`、`false`、`1`、`0`、`"1"`、`"0"` です。

<a name="rule-confirmed"></a>
#### confirmed

バリデーション中のフィールドは、`{field}_confirmation` フィールドと一致する必要があります。たとえば、バリデーション対象のフィールドが `password` の場合、 `password_confirmation` フィールドが入力に存在し、値も一致している必要があります。

<a name="rule-current-password"></a>
#### current_password

バリデーション中のフィールドは、認証されたユーザーのパスワードと一致する必要があります。ルールの最初のパラメータで、[認証ガード](/docs/{{version}}/authentication) を指定できます。

    'password' => 'current_password:api'

<a name="rule-date"></a>
#### date

バリデーション対象のフィールドは、PHP 関数の `strtotime` に従った有効な非相対日付である必要があります。

<a name="rule-date-equals"></a>
#### date_equals:_日付_

バリデーション対象のフィールドは、指定した日付と一致する必要があります。日付は、有効な `DateTime` インスタンスに変換するため、PHP の `strtotime` 関数に渡されます。

<a name="rule-date-format"></a>
#### date_format:_フォーマット_,...

バリデーション中のフィールドは、指定した_フォーマット_のいずれかに一致する必要があります。フィールドをバリデーションするときは、両方ではなく、`date` または `date_format` の **どちらか** を使用する必要があります。このバリデーションルールは、PHP の [DateTime](https://www.php.net/manual/en/class.datetime.php) クラスでサポートされるすべての形式をサポートします。

<a name="rule-decimal"></a>
#### decimal:_最小値_,_最大値_

バリデーション対象のフィールドは、指定された小数点以下の桁数を含む必要があります。

    // Must have exactly two decimal places (9.99)...
    'price' => 'decimal:2'

    // Must have between 2 and 4 decimal places...
    'price' => 'decimal:2,4'

<a name="rule-declined"></a>
#### declined

バリデーションされるフィールドは、`"no"`、`"off"`、`0`、`false` である必要があります。

<a name="rule-declined-if"></a>
#### declined_if:他のフィールド,値,...

バリデーション中の別のフィールドが指定された値と等しい場合、バリデーション中のフィールドは `"no"`、`"off"`、`0`、`false` である必要があります。

<a name="rule-different"></a>
#### different:_フィールド_

バリデーション中のフィールドは、`different` で指定したフィールドとは異なる値を持つ必要があります。

<a name="rule-digits"></a>
#### digits:_値_

バリデーション中のフィールドは、整数であり `digits` で指定した値の桁数である必要があります。

<a name="rule-digits-between"></a>
#### digits_between:_min_,_max_

バリデーション中のフィールドは、整数かつ指定した最小値と最大値の桁数の間である必要があります。

<a name="rule-dimensions"></a>
#### dimensions

バリデーション中のファイルは、ルールのパラメータで指定されたサイズの制約を満たす画像である必要があります。

    'avatar' => 'dimensions:min_width=100,min_height=200'

使用可能な制約パラメータは、_min\_width_、_max\_width_、_min\_height_、_max\_height_、_width_、_height_、_ratio_ です。

_ratio_ 制約は、幅を高さで割ったものとして表す必要があります。これは、`3/2` のような分数または `1.5` のような浮動小数点数で指定できます。

    'avatar' => 'dimensions:ratio=3/2'

このルールには複数の引数が必要なため、 `Rule::dimensions` メソッドを使用してルールをスムーズに構築してください。

    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'avatar' => [
            'required',
            Rule::dimensions()->maxWidth(1000)->maxHeight(500)->ratio(3 / 2),
        ],
    ]);

<a name="rule-distinct"></a>
#### distinct

配列をバリデーションする場合、バリデーション対象のフィールドに重複する値があってはなりません。

    'foo.*.id' => 'distinct'

distinct は、デフォルトで緩い変数比較を使用します。厳密な比較を使用するには、バリデーションルール定義に `strict` パラメータを追加します。

    'foo.*.id' => 'distinct:strict'

バリデーションルールの引数に `ignore_case` を追加して、大文字と小文字の違いを無視するルールを加えられます。

    'foo.*.id' => 'distinct:ignore_case'

<a name="rule-doesnt-start-with"></a>
#### doesnt_start_with:_foo_,_bar_,...

バリデーション中のフィールドは、指定された値のいずれかで始まってはなりません。

<a name="rule-doesnt-end-with"></a>
#### doesnt_end_with:_foo_,_bar_,...

バリデーション中のフィールドは、指定された値のいずれかで終わってはいけません。

<a name="rule-email"></a>
#### email

バリデーション中のフィールドは、電子メールアドレスのフォーマットである必要があります。このバリデーションルールは、電子メールアドレスをバリデーションするために [`egulias/email-validator`](https://github.com/egulias/EmailValidator) パッケージを利用します。デフォルトでは、 `RFCValidation` バリデータが適用されますが、他のバリデーションスタイルも適用できます。

    'email' => 'email:rfc,dns'

上記の例では、`RFCValidation` と `DNSCheckValidation` バリデーションを適用します。適用できるバリデーションスタイルの完全なリストは次のとおりです。

<div class="content-list" markdown="1">

- `rfc`: `RFCValidation`
- `strict`: `NoRFCWarningsValidation`
- `dns`: `DNSCheckValidation`
- `spoof`: `SpoofCheckValidation`
- `filter`: `FilterEmailValidation`
- `filter_unicode`: `FilterEmailValidation::unicode()`

</div>

PHP の `filter_var` 関数を使用する `filter` バリデータは Laravel に同梱されており、Laravel バージョン 5.8 より前の Laravel のデフォルトの電子メールバリデーション動作でした。

> **Warning**
> `dns` と `spoof` バリデータには、PHP の `intl` 拡張が必要です。

<a name="rule-ends-with"></a>
#### ends_with:_foo_,_bar_,...

バリデーション中のフィールドは、指定された値のいずれかで終わる必要があります。

<a name="rule-enum"></a>
#### enum

 `Enum` ルールは、バリデーション対象のフィールドに有効な列挙値が含まれているかどうかをバリデーションするクラスベースのルールです。`Enum` ルールは、コンストラクタ引数として列挙型の名前を受け取ります。

    use App\Enums\ServerStatus;
    use Illuminate\Validation\Rules\Enum;

    $request->validate([
        'status' => [new Enum(ServerStatus::class)],
    ]);

<a name="rule-exclude"></a>
#### exclude

バリデーション中のフィールドは、`validate` と `validated` メソッドによって返されるリクエストデータから除外されます。

<a name="rule-exclude-if"></a>
#### exclude_if:_他のフィールド_,_値_

「_他のフィールド_」が「_値_」と等しい場合、バリデーション中のフィールドは `validate` と `validated` メソッドによって返されるリクエストデータから除外されます。

複雑な条件付き除外ロジックが必要な場合は、`Rule::excludeIf` メソッドを利用できます。このメソッドは論理値またはクロージャを引数に受け取ります。クロージャが指定された場合、クロージャはバリデーション中のフィールドを除外する必要があるかどうかを示すために `true` または `false` を返す必要があります。

    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rule;

    Validator::make($request->all(), [
        'role_id' => Rule::excludeIf($request->user()->is_admin),
    ]);

    Validator::make($request->all(), [
        'role_id' => Rule::excludeIf(fn () => $request->user()->is_admin),
    ]);

<a name="rule-exclude-unless"></a>
#### exclude_unless:_他のフィールド_,_値_

「_他のフィールド_ 」が「_値_」と等しくない場合、バリデーション中のフィールドは `validate` および `validated` メソッドによって返されるリクエストデータから除外されます。「_値_」が `null` (`exclude_unless:name,null`) の場合、比較フィールドが `null` であるか、比較フィールドがリクエストデータに含まれていない場合を除き、バリデーション対象のフィールドは除外されます。

<a name="rule-exclude-with"></a>
#### exclude_with:_他のフィールド_

「_他のフィールド_」が存在する場合、バリデーション中のフィールドは、`validate` および `validated` メソッドによって返されるリクエストデータから除外されます。

<a name="rule-exclude-without"></a>
#### exclude_without:_他のフィールド_

「他のフィールド」が存在しない場合、バリデーション中のフィールドは、`validate` と `validated` メソッドによって返されるリクエストデータから除外されます。

<a name="rule-exists"></a>
#### exists:_テーブル_,_カラム_

バリデーション中のフィールドは、指定したデータベーステーブルに存在する必要があります。

<a name="basic-usage-of-exists-rule"></a>
#### exists ルールの基本的な使用法

    'state' => 'exists:states'

`column` オプションが指定されていない場合は、フィールド名が使用されます。上記の場合、`states` データベーステーブルに、リクエストの `state` 属性値と一致する `state` カラム値を持つレコードが含まれていることをバリデーションします。

<a name="specifying-a-custom-column-name"></a>
#### カスタムカラム名の指定

データベーステーブル名の後に配置することで、バリデーションルールで使用するデータベースのカラム名を明示的に指定できます。

    'state' => 'exists:states,abbreviation'

場合によっては、 `exists` クエリに使用する特定のデータベース接続を指定する必要がある場合があります。これを行うには、テーブル名の前に接続名を追加します。

    'email' => 'exists:connection.staff,email'

テーブル名を直接指定する代わりに、テーブル名の決定に使用する Eloquent モデルを指定することもできます。

    'user_id' => 'exists:App\Models\User,id'

バリデーションルールによって実行されるクエリをカスタマイズしたい場合は、`Rule` クラスを使用してルールをスムーズに定義できます。以下の例では、バリデーションルールを `|` 文字で区切るのではなく、配列として指定します。

    use Illuminate\Database\Query\Builder;
    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'email' => [
            'required',
            Rule::exists('staff')->where(function (Builder $query) {
                return $query->where('account_id', 1);
            }),
        ],
    ]);

`exists` メソッドの第２引数にカラム名を指定することで、`Rule::exists` メソッドによって生成される `exists` ルールで使用するデータベースカラム名を明示的に指定できます。

    'state' => Rule::exists('states', 'abbreviation'),

<a name="rule-file"></a>
#### file

バリデーション中のフィールドは、正常にアップロードされたファイルである必要があります。

<a name="rule-filled"></a>
#### filled

バリデーション中のフィールドが存在する場合、空であってはなりません。

<a name="rule-gt"></a>
#### gt:_フィールド_

バリデーション対象のフィールドは、指定した「フィールド」の値より大きい値である必要があります。２つのフィールドは同じタイプである必要があります。文字列、数値、配列、ファイルは、[`size`](#rule-size) ルールと同じ規則を使用して評価されます。

<a name="rule-gte"></a>
#### gte:_フィールド_

バリデーション対象のフィールドは、指定した「フィールド」の値以上である必要があります。２つのフィールドは同じタイプである必要があります。文字列、数値、配列、ファイルは、[`size`](#rule-size) ルールと同じ規則を使用して評価されます。

<a name="rule-image"></a>
#### image

バリデーションされるファイルは画像 (jpg、jpeg、png、bmp、gif、svg、webp) である必要があります。

<a name="rule-in"></a>
#### in:_foo_,_bar_,...

バリデーション中のフィールドは、指定した値のリストに含まれている必要があります。このルールでは配列を `implode` する必要があることが多いため、`Rule::in` メソッドを使用することでルールをスムーズに構築できます。

    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'zones' => [
            'required',
            Rule::in(['first-zone', 'second-zone']),
        ],
    ]);

`in` ルールを `array` ルールと組み合わせる場合、入力配列内の各値は、 `in` ルールに指定した値のリスト内に存在する必要があります。以下の例では、入力配列内の `LAS` 空港コードは、`in` ルールに指定された空港のリストに含まれていないため無効です。

    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rule;

    $input = [
        'airports' => ['NYC', 'LAS'],
    ];

    Validator::make($input, [
        'airports' => [
            'required',
            'array',
        ],
        'airports.*' => Rule::in(['NYC', 'LIT']),
    ]);

<a name="rule-in-array"></a>
#### in_array:_他のフィールド_.*

バリデーション中のフィールドは、「他のフィールド」の値のいずれかである必要があります。

<a name="rule-integer"></a>
#### integer

バリデーション中のフィールドは整数である必要があります。

> **Warning**  
> このバリデーションルールは、入力が「整数」変数タイプであることをバリデーションするのではなく、入力が PHP の `FILTER_VALIDATE_INT` ルールで受け入れられるタイプであることのみをバリデーションします。入力が数値であることをバリデーションする必要がある場合は、このルールを [`numeric` バリデーションルール](#rule-numeric) と組み合わせて使用してください。

<a name="rule-ip"></a>
#### ip

バリデーション対象のフィールドは IP アドレスの形式である必要があります。

<a name="ipv4"></a>
#### ipv4

バリデーション対象のフィールドは IPv4 アドレスの形式である必要があります。

<a name="ipv6"></a>
#### ipv6

バリデーション対象のフィールドは IPv6 アドレスの形式である必要があります。

<a name="rule-json"></a>
#### json

バリデーション対象のフィールドは有効な JSON 文字列である必要があります。

<a name="rule-lt"></a>
#### lt:_フィールド_

バリデーション対象のフィールドは、指定した「フィールド」の値より小さくなければなりません。２つのフィールドは同じタイプである必要があります。文字列、数値、配列、ファイルは、[`size`](#rule-size) ルールと同じ規則を使用して評価されます。

<a name="rule-lte"></a>
#### lte:_フィールド_

バリデーション対象のフィールドは、指定した「 _フィールド_ 」の値以下である必要があります。２つのフィールドは同じタイプである必要があります。文字列、数値、配列、ファイルは、[`size`](#rule-size) ルールと同じ規則を使用して評価されます。

<a name="rule-lowercase"></a>
#### lowercase

バリデーション中のフィールドは小文字である必要があります。

<a name="rule-mac"></a>
#### mac_address

バリデーション対象のフィールドは MAC アドレスである必要があります。

<a name="rule-max"></a>
#### max:_値_

バリデーション対象のフィールドは、最大値として指定した「 _値_ 」以下である必要があります。文字列、数値、配列、ファイルは、[`size`](#rule-size) ルールと同じ方法で評価されます。

<a name="rule-max-digits"></a>
#### max_digits:_値_

バリデーションされる整数の桁数は、最大値として指定した「 _値_ 」以下の桁数である必要があります。

<a name="rule-mimetypes"></a>
#### mimetypes:_text/plain_,...

バリデーション中のファイルは、指定された MIME タイプのいずれかに一致する必要があります。

    'video' => 'mimetypes:video/avi,video/mpeg,video/quicktime'

アップロードされたファイルの MIME タイプを判別するために、ファイルの内容が読み取られ、フレームワークは MIME タイプを推測しようとしますが、これはクライアントが提供した MIME タイプとは異なる場合があります。

<a name="rule-mimes"></a>
#### mimes:_foo_,_bar_,...

バリデーション中のファイルは、指定した拡張子のいずれかに対応する MIME タイプを持っている必要があります。

<a name="basic-usage-of-mime-rule"></a>
#### MIME ルールの基本的な使用法

    'photo' => 'mimes:jpg,bmp,png'

拡張子の指定のみでも良いですが、このルールは、実際にファイルの内容を読み取り、その MIME タイプを推測することによってファイルの MIME タイプをバリデーションします。 MIME タイプとそれに対応する拡張子の完全なリストは、次の場所にあります。

[https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types](https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types)

<a name="rule-min"></a>
#### min:_値_

バリデーション中のフィールドには、最小値として指定した「値」以上である必要があります。文字列、数値、配列、ファイルは、[`size`](#rule-size) ルールと同じ方法で評価されます。

<a name="rule-min-digits"></a>
#### min_digits:_値_

バリデーション中のフィールドの整数は、最小値として指定した「値」以上の桁数を持つ必要があります。

<a name="rule-multiple-of"></a>
#### multiple_of:_値_

バリデーション中のフィールドは、指定した「値」の倍数である必要があります。

<a name="rule-missing"></a>
#### missing

バリデーション中のフィールドは、入力データに存在してはなりません。

 <a name="rule-missing-if"></a>
 #### missing_if:_他のフィールド_,_値_,...

指定した「他のフィールド」が、指定した「値」のいずれかと等しい場合、バリデーション中のフィールドは存在してはなりません。

 <a name="rule-missing-unless"></a>
 #### missing_unless:_他のフィールド_,_値_

指定した「他のフィールド」が、指定した「値」と等しくない場合、バリデーション中のフィールドは存在してはなりません。

 <a name="rule-missing-with"></a>
 #### missing_with:_foo_,_bar_,...

指定したフィールドのいずれか１つでも存在する場合、バリデーション中のフィールドは存在してはなりません。

 <a name="rule-missing-with-all"></a>
 #### missing_with_all:_foo_,_bar_,...

指定したフィールドがすべて存在する場合にのみ、バリデーション中のフィールドは存在してはなりません。

<a name="rule-not-in"></a>
#### not_in:_foo_,_bar_,...

バリデーション中のフィールドは、指定した値のリストに含めてはなりません。`Rule::notIn` メソッドを使用すると、ルールをスムーズに構築できます。

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'toppings' => [
            'required',
            Rule::notIn(['sprinkles', 'cherries']),
        ],
    ]);

<a name="rule-not-regex"></a>
#### not_regex:_正規表現_

バリデーション中のフィールドは、指定した正規表現と一致しては **なりません**。

内部的に、このルールは PHP の `preg_match` 関数を使用しています。指定したパターンは、有効な区切り文字も含め `preg_match` で必要とされるフォーマットと同じように従う必要があります。たとえば、`'email' => 'not_regex:/^.+$/i'` のように記述します。

> **Warning**
> `regex` / `not_regex` パターンを使用する場合、特に正規表現に `|` 文字が含まれている場合は、`|` 区切り文字を使用する代わりに配列を使用してバリデーションルールを指定する必要がある場合があります。

<a name="rule-nullable"></a>
#### nullable

バリデーション中のフィールドは `null` である可能性を許容します。

<a name="rule-numeric"></a>
#### numeric

バリデーション中のフィールドは [数値](https://www.php.net/manual/en/function.is-numeric.php) である必要があります。

<a name="rule-password"></a>
#### password

バリデーション中のフィールドは、認証済みユーザーのパスワードと一致する必要があります。

> **Warning**
> このルールは、Laravel 9 で削除する目的で `current_password` に名前変更されました。代わりに [current_password](#rule-current-password) ルールを使用してください。

<a name="rule-present"></a>
#### present

バリデーション中のフィールドは、入力データに存在する必要があります。

<a name="rule-prohibited"></a>
#### prohibited

バリデーション中のフィールドは、存在しないか、または空である必要があります。以下の基準のいずれかを満たしている場合、フィールドは `空` です。

<div class="content-list" markdown="1">

- 値が `null` である
- 値が空の文字列である
- 値が空の配列、または空の `Countable` オブジェクトである
- 値がパスのないアップロード済みファイルである

</div>

<a name="rule-prohibited-if"></a>
#### prohibited_if:_他のフィールド_,_値_,...

指定した「他のフィールド」が指定した「値」のいずれかと等しい場合、バリデーション中のフィールドは存在しないか、空である必要があります。以下の基準のいずれかを満たしている場合、そのフィールドは `空` と判定されます。

<div class="content-list" markdown="1">

- 値が `null` である
- 値が空の文字列である
- 値が空の配列、または空の `Countable` オブジェクトである
- 値がパスのないアップロード済みファイルである

</div>

複雑な条件付き禁止ロジックが必要な場合は、`Rule::prohibitedIf` メソッドを利用してください。このメソッドは論理値またはクロージャを引数に取ります。クロージャが指定された場合、そのクロージャはバリデーション中のフィールドを禁止する必要があるかどうかを示すために `true` または `false` を返す必要があります。

    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rule;

    Validator::make($request->all(), [
        'role_id' => Rule::prohibitedIf($request->user()->is_admin),
    ]);

    Validator::make($request->all(), [
        'role_id' => Rule::prohibitedIf(fn () => $request->user()->is_admin),
    ]);

<a name="rule-prohibited-unless"></a>
#### prohibited_unless:_他のフィールド_,_値_,...

指定した「他のフィールド」が指定した「値」のいずれかと等しい場合を除き、バリデーション中のフィールドが、存在しないか空である必要があります。以下の基準のいずれかを満たしている場合、そのフィールドは `空` と判定されます。

<div class="content-list" markdown="1">

- 値は `null` である
- 値は空の文字列である
- 値が空の配列または空の `Countable` オブジェクトである
- 値はパスのないアップロード済みファイルである

</div>

<a name="rule-prohibits"></a>
#### prohibits:_他のフィールド_,...

バリデーション中のフィールドが、存在しないか空でない場合、指定した「他のフィールド」のすべてのフィールドが存在しないか空である必要があります。以下の基準のいずれかを満たしている場合、そのフィールドは `空` と判定されます。

<div class="content-list" markdown="1">

- 値は `null` である
- 値は空の文字列である
- 値が空の配列または空の `Countable` オブジェクトである
- 値はパスのないアップロード済みファイルである

</div>

<a name="rule-regex"></a>
#### regex:_正規表現_

バリデーション中のフィールドは、指定した正規表現と一致する必要があります。

内部的に、このルールは PHP の `preg_match` 関数を使用しています。指定したパターンは、有効な区切り文字も含め `preg_match` で必要とされるフォーマットと同じように従う必要があります。たとえば、`'email' => 'not_regex:/^.+$/i'` のように記述します。

> **Warning**
> `regex` / `not_regex` パターンを使用する場合、特に正規表現に `|` 文字が含まれている場合は、`|` 区切り文字を使用する代わりに配列を使用してバリデーションルールを指定する必要がある場合があります。

<a name="rule-required"></a>
#### required

バリデーション中のフィールドは、入力データに存在する必要があり、かつ空であってはなりません。以下の基準のいずれかを満たしている場合、そのフィールドは `空` と判定されます。

<div class="content-list" markdown="1">

- 値は `null` である
- 値は空の文字列である
- 値が空の配列または空の `Countable` オブジェクトである
- 値はパスのないアップロード済みファイルである

</div>

<a name="rule-required-if"></a>
#### required_if:_他のフィールド_,_値_,...

指定した「他のフィールド」が指定した「値」のいずれかと等しい場合、バリデーション中のフィールドは存在する必要があり、かつ空であってはなりません。

`required_if` ルールのより複雑な条件を構築したい場合は、`Rule::requiredIf` メソッドを使用できます。このメソッドは論理値またはクロージャを受け取ります。クロージャが渡されると、クロージャはバリデーション中のフィールドが必要かどうかを示すために `true` または `false` を返す必要があります。

    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rule;

    Validator::make($request->all(), [
        'role_id' => Rule::requiredIf($request->user()->is_admin),
    ]);

    Validator::make($request->all(), [
        'role_id' => Rule::requiredIf(fn () => $request->user()->is_admin),
    ]);

<a name="rule-required-unless"></a>
#### required_unless:_他のフィールド_,_値_,...

指定した「他のフィールド」が指定した「値」のいずれにも一致していない場合、バリデーション中のフィールドは存在するし、空でない必要があります。これは、指定した「値」が `null` でない限り、リクエストデータに指定した「他のフィールド」が存在する必要があることを意味します。指定した「 値」が `null` (`required_unless:name,null`) の場合、比較フィールドが `null` であるか、比較フィールドがリクエスト データに存在しない場合を除き、バリデーション対象のフィールドは必須になります。

<a name="rule-required-with"></a>
#### required_with:_foo_,_bar_,...

指定した他のフィールドのいずれかが存在する場合、バリデーション中のこのフィールドが存在し、空でない必要があります。

<a name="rule-required-with-all"></a>
#### required_with_all:_foo_,_bar_,...

指定した他のフィールドがすべて存在する場合、バリデーション中のこのフィールドが存在し、空でない必要があります。

<a name="rule-required-without"></a>
#### required_without:_foo_,_bar_,...

指定した他のフィールドのいずれかが空であるか存在しない場合にのみ、バリデーション中のフィールドは空ではなく存在する必要があります。

<a name="rule-required-without-all"></a>
#### required_without_all:_foo_,_bar_,...

指定したフィールドの全てが空であるか存在しない場合にのみ、バリデーション対象のフィールドは空ではなく存在する必要があります。

<a name="rule-required-array-keys"></a>
#### required_array_keys:_foo_,_bar_,...

バリデーション中のフィールドは、配列である必要があり、少なくとも指定されたキーが含まれている必要があります。

<a name="rule-same"></a>
#### same:_フィールド_

指定した「フィールド」は、バリデーション中のフィールドと同じ値で一致する必要があります。

<a name="rule-size"></a>
#### size:_値_

バリデーション中のフィールドのサイズは、指定した「値」と一致する必要があります。文字列データの場合、指定した「値」は文字数に対応します。数値データの場合、「値」は整数値 (属性には `numeric` または `integer` ルールも必要です) となります。配列の場合、「値」は配列の `count` に対応します。ファイルの場合、「値」はキロバイト単位のファイルサイズに対応します。いくつかの例を見てみましょう。

    // 文字列の長さがちょうど 12 文字であるか
    'title' => 'size:12';

    // 指定された整数が 10 であるか
    'seats' => 'integer|size:10';

    // 配列はちょうど 5 つの要素があるか
    'tags' => 'array|size:5';

    // アップロードされたファイルがちょうど 512 キロバイトであるか
    'image' => 'file|size:512';

<a name="rule-starts-with"></a>
#### starts_with:_foo_,_bar_,...

バリデーション中のフィールドは、指定された値のいずれかで始まる必要があります。

<a name="rule-string"></a>
#### string

バリデーション中のフィールドは文字列である必要があります。フィールドが `null` であることも許可したい場合は、フィールドに `nullable` ルールを指定してください。

<a name="rule-timezone"></a>
#### timezone

The field under validation must be a valid timezone identifier according to the `DateTimeZone::listIdentifiers` method.

The arguments [accepted by the `DateTimeZone::listIdentifiers` method](https://www.php.net/manual/en/datetimezone.listidentifiers.php) may also be provided to this validation rule:

    'timezone' => 'required|timezone:all';

    'timezone' => 'required|timezone:Africa';

    'timezone' => 'required|timezone:per_country,US';

<a name="rule-unique"></a>
#### unique:_テーブル_,_カラム_

バリデーション中のフィールドは、指定したデータベーステーブル内に存在してはなりません。

**カスタムテーブル / カラム名の指定**

テーブル名を直接指定する代わりに、テーブル名の決定に使用する Eloquent モデルを指定することもできます。

    'email' => 'unique:App\Models\User,email_address'

The `column` option may be used to specify the field's corresponding database column. If the `column` option is not specified, the name of the field under validation will be used.

    'email' => 'unique:users,email_address'

**Specifying A Custom Database Connection**

Occasionally, you may need to set a custom connection for database queries made by the Validator. To accomplish this, you may prepend the connection name to the table name:

    'email' => 'unique:connection.users,email_address'

**Forcing A Unique Rule To Ignore A Given ID:**

Sometimes, you may wish to ignore a given ID during unique validation. For example, consider an "update profile" screen that includes the user's name, email address, and location. You will probably want to verify that the email address is unique. However, if the user only changes the name field and not the email field, you do not want a validation error to be thrown because the user is already the owner of the email address in question.

To instruct the validator to ignore the user's ID, we'll use the `Rule` class to fluently define the rule. In this example, we'll also specify the validation rules as an array instead of using the `|` character to delimit the rules:

    use Illuminate\Database\Eloquent\Builder;
    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'email' => [
            'required',
            Rule::unique('users')->ignore($user->id),
        ],
    ]);

> **Warning**  
> You should never pass any user controlled request input into the `ignore` method. Instead, you should only pass a system generated unique ID such as an auto-incrementing ID or UUID from an Eloquent model instance. Otherwise, your application will be vulnerable to an SQL injection attack.

Instead of passing the model key's value to the `ignore` method, you may also pass the entire model instance. Laravel will automatically extract the key from the model:

    Rule::unique('users')->ignore($user)

If your table uses a primary key column name other than `id`, you may specify the name of the column when calling the `ignore` method:

    Rule::unique('users')->ignore($user->id, 'user_id')

By default, the `unique` rule will check the uniqueness of the column matching the name of the attribute being validated. However, you may pass a different column name as the second argument to the `unique` method:

    Rule::unique('users', 'email_address')->ignore($user->id)

**Adding Additional Where Clauses:**

You may specify additional query conditions by customizing the query using the `where` method. For example, let's add a query condition that scopes the query to only search records that have an `account_id` column value of `1`:

    'email' => Rule::unique('users')->where(fn (Builder $query) => $query->where('account_id', 1))

<a name="rule-uppercase"></a>
#### uppercase

The field under validation must be uppercase.

<a name="rule-url"></a>
#### url

The field under validation must be a valid URL.

<a name="rule-ulid"></a>
#### ulid

The field under validation must be a valid [Universally Unique Lexicographically Sortable Identifier](https://github.com/ulid/spec) (ULID).

<a name="rule-uuid"></a>
#### uuid

The field under validation must be a valid RFC 4122 (version 1, 3, 4, or 5) universally unique identifier (UUID).

<a name="conditionally-adding-rules"></a>
## Conditionally Adding Rules

<a name="skipping-validation-when-fields-have-certain-values"></a>
#### Skipping Validation When Fields Have Certain Values

You may occasionally wish to not validate a given field if another field has a given value. You may accomplish this using the `exclude_if` validation rule. In this example, the `appointment_date` and `doctor_name` fields will not be validated if the `has_appointment` field has a value of `false`:

    use Illuminate\Support\Facades\Validator;

    $validator = Validator::make($data, [
        'has_appointment' => 'required|boolean',
        'appointment_date' => 'exclude_if:has_appointment,false|required|date',
        'doctor_name' => 'exclude_if:has_appointment,false|required|string',
    ]);

Alternatively, you may use the `exclude_unless` rule to not validate a given field unless another field has a given value:

    $validator = Validator::make($data, [
        'has_appointment' => 'required|boolean',
        'appointment_date' => 'exclude_unless:has_appointment,true|required|date',
        'doctor_name' => 'exclude_unless:has_appointment,true|required|string',
    ]);

<a name="validating-when-present"></a>
#### Validating When Present

In some situations, you may wish to run validation checks against a field **only** if that field is present in the data being validated. To quickly accomplish this, add the `sometimes` rule to your rule list:

    $v = Validator::make($data, [
        'email' => 'sometimes|required|email',
    ]);

In the example above, the `email` field will only be validated if it is present in the `$data` array.

> **Note**  
> If you are attempting to validate a field that should always be present but may be empty, check out [this note on optional fields](#a-note-on-optional-fields).

<a name="complex-conditional-validation"></a>
#### Complex Conditional Validation

Sometimes you may wish to add validation rules based on more complex conditional logic. For example, you may wish to require a given field only if another field has a greater value than 100. Or, you may need two fields to have a given value only when another field is present. Adding these validation rules doesn't have to be a pain. First, create a `Validator` instance with your _static rules_ that never change:

    use Illuminate\Support\Facades\Validator;

    $validator = Validator::make($request->all(), [
        'email' => 'required|email',
        'games' => 'required|numeric',
    ]);

Let's assume our web application is for game collectors. If a game collector registers with our application and they own more than 100 games, we want them to explain why they own so many games. For example, perhaps they run a game resale shop, or maybe they just enjoy collecting games. To conditionally add this requirement, we can use the `sometimes` method on the `Validator` instance.

    use Illuminate\Support\Fluent;

    $validator->sometimes('reason', 'required|max:500', function (Fluent $input) {
        return $input->games >= 100;
    });

The first argument passed to the `sometimes` method is the name of the field we are conditionally validating. The second argument is a list of the rules we want to add. If the closure passed as the third argument returns `true`, the rules will be added. This method makes it a breeze to build complex conditional validations. You may even add conditional validations for several fields at once:

    $validator->sometimes(['reason', 'cost'], 'required', function (Fluent $input) {
        return $input->games >= 100;
    });

> **Note**  
> The `$input` parameter passed to your closure will be an instance of `Illuminate\Support\Fluent` and may be used to access your input and files under validation.

<a name="complex-conditional-array-validation"></a>
#### Complex Conditional Array Validation

Sometimes you may want to validate a field based on another field in the same nested array whose index you do not know. In these situations, you may allow your closure to receive a second argument which will be the current individual item in the array being validated:

    $input = [
        'channels' => [
            [
                'type' => 'email',
                'address' => 'abigail@example.com',
            ],
            [
                'type' => 'url',
                'address' => 'https://example.com',
            ],
        ],
    ];

    $validator->sometimes('channels.*.address', 'email', function (Fluent $input, Fluent $item) {
        return $item->type === 'email';
    });

    $validator->sometimes('channels.*.address', 'url', function (Fluent $input, Fluent $item) {
        return $item->type !== 'email';
    });

Like the `$input` parameter passed to the closure, the `$item` parameter is an instance of `Illuminate\Support\Fluent` when the attribute data is an array; otherwise, it is a string.

<a name="validating-arrays"></a>
## Validating Arrays

As discussed in the [`array` validation rule documentation](#rule-array), the `array` rule accepts a list of allowed array keys. If any additional keys are present within the array, validation will fail:

    use Illuminate\Support\Facades\Validator;

    $input = [
        'user' => [
            'name' => 'Taylor Otwell',
            'username' => 'taylorotwell',
            'admin' => true,
        ],
    ];

    Validator::make($input, [
        'user' => 'array:username,locale',
    ]);

In general, you should always specify the array keys that are allowed to be present within your array. Otherwise, the validator's `validate` and `validated` methods will return all of the validated data, including the array and all of its keys, even if those keys were not validated by other nested array validation rules.

<a name="validating-nested-array-input"></a>
### Validating Nested Array Input

Validating nested array based form input fields doesn't have to be a pain. You may use "dot notation" to validate attributes within an array. For example, if the incoming HTTP request contains a `photos[profile]` field, you may validate it like so:

    use Illuminate\Support\Facades\Validator;

    $validator = Validator::make($request->all(), [
        'photos.profile' => 'required|image',
    ]);

You may also validate each element of an array. For example, to validate that each email in a given array input field is unique, you may do the following:

    $validator = Validator::make($request->all(), [
        'person.*.email' => 'email|unique:users',
        'person.*.first_name' => 'required_with:person.*.last_name',
    ]);

Likewise, you may use the `*` character when specifying [custom validation messages in your language files](#custom-messages-for-specific-attributes), making it a breeze to use a single validation message for array based fields:

    'custom' => [
        'person.*.email' => [
            'unique' => 'Each person must have a unique email address',
        ]
    ],

<a name="accessing-nested-array-data"></a>
#### Accessing Nested Array Data

Sometimes you may need to access the value for a given nested array element when assigning validation rules to the attribute. You may accomplish this using the `Rule::forEach` method. The `forEach` method accepts a closure that will be invoked for each iteration of the array attribute under validation and will receive the attribute's value and explicit, fully-expanded attribute name. The closure should return an array of rules to assign to the array element:

    use App\Rules\HasPermission;
    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rule;

    $validator = Validator::make($request->all(), [
        'companies.*.id' => Rule::forEach(function (string|null $value, string $attribute) {
            return [
                Rule::exists(Company::class, 'id'),
                new HasPermission('manage-company', $value),
            ];
        }),
    ]);

<a name="error-message-indexes-and-positions"></a>
### Error Message Indexes & Positions

When validating arrays, you may want to reference the index or position of a particular item that failed validation within the error message displayed by your application. To accomplish this, you may include the `:index` (starts from `0`) and `:position` (starts from `1`) placeholders within your [custom validation message](#manual-customizing-the-error-messages):

    use Illuminate\Support\Facades\Validator;

    $input = [
        'photos' => [
            [
                'name' => 'BeachVacation.jpg',
                'description' => 'A photo of my beach vacation!',
            ],
            [
                'name' => 'GrandCanyon.jpg',
                'description' => '',
            ],
        ],
    ];

    Validator::validate($input, [
        'photos.*.description' => 'required',
    ], [
        'photos.*.description.required' => 'Please describe photo #:position.',
    ]);

Given the example above, validation will fail and the user will be presented with the following error of _"Please describe photo #2."_

<a name="validating-files"></a>
## Validating Files

Laravel provides a variety of validation rules that may be used to validate uploaded files, such as `mimes`, `image`, `min`, and `max`. While you are free to specify these rules individually when validating files, Laravel also offers a fluent file validation rule builder that you may find convenient:

    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rules\File;

    Validator::validate($input, [
        'attachment' => [
            'required',
            File::types(['mp3', 'wav'])
                ->min(1024)
                ->max(12 * 1024),
        ],
    ]);

If your application accepts images uploaded by your users, you may use the `File` rule's `image` constructor method to indicate that the uploaded file should be an image. In addition, the `dimensions` rule may be used to limit the dimensions of the image:

    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rules\File;

    Validator::validate($input, [
        'photo' => [
            'required',
            File::image()
                ->min(1024)
                ->max(12 * 1024)
                ->dimensions(Rule::dimensions()->maxWidth(1000)->maxHeight(500)),
        ],
    ]);

> **Note**  
> More information regarding validating image dimensions may be found in the [dimension rule documentation](#rule-dimensions).

<a name="validating-files-file-types"></a>
#### File Types

Even though you only need to specify the extensions when invoking the `types` method, this method actually validates the MIME type of the file by reading the file's contents and guessing its MIME type. A full listing of MIME types and their corresponding extensions may be found at the following location:

[https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types](https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types)

<a name="validating-passwords"></a>
## Validating Passwords

To ensure that passwords have an adequate level of complexity, you may use Laravel's `Password` rule object:

    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rules\Password;

    $validator = Validator::make($request->all(), [
        'password' => ['required', 'confirmed', Password::min(8)],
    ]);

The `Password` rule object allows you to easily customize the password complexity requirements for your application, such as specifying that passwords require at least one letter, number, symbol, or characters with mixed casing:

    // Require at least 8 characters...
    Password::min(8)

    // Require at least one letter...
    Password::min(8)->letters()

    // Require at least one uppercase and one lowercase letter...
    Password::min(8)->mixedCase()

    // Require at least one number...
    Password::min(8)->numbers()

    // Require at least one symbol...
    Password::min(8)->symbols()

In addition, you may ensure that a password has not been compromised in a public password data breach leak using the `uncompromised` method:

    Password::min(8)->uncompromised()

Internally, the `Password` rule object uses the [k-Anonymity](https://en.wikipedia.org/wiki/K-anonymity) model to determine if a password has been leaked via the [haveibeenpwned.com](https://haveibeenpwned.com) service without sacrificing the user's privacy or security.

By default, if a password appears at least once in a data leak, it will be considered compromised. You can customize this threshold using the first argument of the `uncompromised` method:

    // Ensure the password appears less than 3 times in the same data leak...
    Password::min(8)->uncompromised(3);

Of course, you may chain all the methods in the examples above:

    Password::min(8)
        ->letters()
        ->mixedCase()
        ->numbers()
        ->symbols()
        ->uncompromised()

<a name="defining-default-password-rules"></a>
#### Defining Default Password Rules

You may find it convenient to specify the default validation rules for passwords in a single location of your application. You can easily accomplish this using the `Password::defaults` method, which accepts a closure. The closure given to the `defaults` method should return the default configuration of the Password rule. Typically, the `defaults` rule should be called within the `boot` method of one of your application's service providers:

```php
use Illuminate\Validation\Rules\Password;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Password::defaults(function () {
        $rule = Password::min(8);

        return $this->app->isProduction()
                    ? $rule->mixedCase()->uncompromised()
                    : $rule;
    });
}
```

Then, when you would like to apply the default rules to a particular password undergoing validation, you may invoke the `defaults` method with no arguments:

    'password' => ['required', Password::defaults()],

Occasionally, you may want to attach additional validation rules to your default password validation rules. You may use the `rules` method to accomplish this:

    use App\Rules\ZxcvbnRule;

    Password::defaults(function () {
        $rule = Password::min(8)->rules([new ZxcvbnRule]);

        // ...
    });

<a name="custom-validation-rules"></a>
## Custom Validation Rules

<a name="using-rule-objects"></a>
### Using Rule Objects

Laravel provides a variety of helpful validation rules; however, you may wish to specify some of your own. One method of registering custom validation rules is using rule objects. To generate a new rule object, you may use the `make:rule` Artisan command. Let's use this command to generate a rule that verifies a string is uppercase. Laravel will place the new rule in the `app/Rules` directory. If this directory does not exist, Laravel will create it when you execute the Artisan command to create your rule:

```shell
php artisan make:rule Uppercase
```

Once the rule has been created, we are ready to define its behavior. A rule object contains a single method: `validate`. This method receives the attribute name, its value, and a callback that should be invoked on failure with the validation error message:

    <?php

    namespace App\Rules;

    use Closure;
    use Illuminate\Contracts\Validation\ValidationRule;

    class Uppercase implements ValidationRule
    {
        /**
         * Run the validation rule.
         */
        public function validate(string $attribute, mixed $value, Closure $fail): void
        {
            if (strtoupper($value) !== $value) {
                $fail('The :attribute must be uppercase.');
            }
        }
    }

Once the rule has been defined, you may attach it to a validator by passing an instance of the rule object with your other validation rules:

    use App\Rules\Uppercase;

    $request->validate([
        'name' => ['required', 'string', new Uppercase],
    ]);

#### Translating Validation Messages

Instead of providing a literal error message to the `$fail` closure, you may also provide a [translation string key](/docs/{{version}}/localization) and instruct Laravel to translate the error message:

    if (strtoupper($value) !== $value) {
        $fail('validation.uppercase')->translate();
    }

If necessary, you may provide placeholder replacements and the preferred language as the first and second arguments to the `translate` method:

    $fail('validation.location')->translate([
        'value' => $this->value,
    ], 'fr')

#### Accessing Additional Data

If your custom validation rule class needs to access all of the other data undergoing validation, your rule class may implement the `Illuminate\Contracts\Validation\DataAwareRule` interface. This interface requires your class to define a `setData` method. This method will automatically be invoked by Laravel (before validation proceeds) with all of the data under validation:

    <?php

    namespace App\Rules;

    use Illuminate\Contracts\Validation\DataAwareRule;
    use Illuminate\Contracts\Validation\ValidationRule;

    class Uppercase implements DataAwareRule, ValidationRule
    {
        /**
         * All of the data under validation.
         *
         * @var array<string, mixed>
         */
        protected $data = [];

        // ...

        /**
         * Set the data under validation.
         *
         * @param  array<string, mixed>  $data
         */
        public function setData(array $data): static
        {
            $this->data = $data;

            return $this;
        }
    }

Or, if your validation rule requires access to the validator instance performing the validation, you may implement the `ValidatorAwareRule` interface:

    <?php

    namespace App\Rules;

    use Illuminate\Contracts\Validation\ValidationRule;
    use Illuminate\Contracts\Validation\ValidatorAwareRule;
    use Illuminate\Validation\Validator;

    class Uppercase implements ValidationRule, ValidatorAwareRule
    {
        /**
         * The validator instance.
         *
         * @var \Illuminate\Validation\Validator
         */
        protected $validator;

        // ...

        /**
         * Set the current validator.
         */
        public function setValidator(Validator $validator): static
        {
            $this->validator = $validator;

            return $this;
        }
    }

<a name="using-closures"></a>
### Using Closures

If you only need the functionality of a custom rule once throughout your application, you may use a closure instead of a rule object. The closure receives the attribute's name, the attribute's value, and a `$fail` callback that should be called if validation fails:

    use Illuminate\Support\Facades\Validator;
    use Closure;

    $validator = Validator::make($request->all(), [
        'title' => [
            'required',
            'max:255',
            function (string $attribute, mixed $value, Closure $fail) {
                if ($value === 'foo') {
                    $fail("The {$attribute} is invalid.");
                }
            },
        ],
    ]);

<a name="implicit-rules"></a>
### Implicit Rules

By default, when an attribute being validated is not present or contains an empty string, normal validation rules, including custom rules, are not run. For example, the [`unique`](#rule-unique) rule will not be run against an empty string:

    use Illuminate\Support\Facades\Validator;

    $rules = ['name' => 'unique:users,name'];

    $input = ['name' => ''];

    Validator::make($input, $rules)->passes(); // true

For a custom rule to run even when an attribute is empty, the rule must imply that the attribute is required. To quickly generate a new implicit rule object, you may use the `make:rule` Artisan command with the `--implicit` option:

```shell
php artisan make:rule Uppercase --implicit
```

> **Warning**  
> An "implicit" rule only _implies_ that the attribute is required. Whether it actually invalidates a missing or empty attribute is up to you.
