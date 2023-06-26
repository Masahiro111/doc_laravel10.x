# CSRF 保護

- [はじめに](#csrf-introduction)
- [CSRF リクエストの防止](#preventing-csrf-requests)
     - [除外 URI](#csrf-excluding-uris)
- [X-CSRF-Token](#csrf-x-csrf-token)
- [X-XSRF-Token](#csrf-x-xsrf-token)

<a name="csrf-introduction"></a>
## はじめに

クロスサイトリクエストフォージェリは、認証されたユーザーに代わって未承認のコマンドが実行される、一種の悪意のある攻撃です。ありがたいことに、Laravel を使用すると、[クロスサイトリクエストフォージェリ](https://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF) 攻撃からアプリケーションを簡単に保護できます。

<a name="csrf-explanation"></a>
#### 脆弱性の説明

クロスサイトリクエストフォージェリに詳しくない方のために、この脆弱性が悪用される例について説明します。アプリケーションに、認証されたユーザーの電子メールアドレスを変更するための `POST` リクエストを受け入れる `/user/email` ルートがあると想像してください。今回のルートでは、ユーザーが登録したいと思っている電子メールアドレスが `email` 入力フィールドに含まれていることを想定しています。

CSRF 保護がないと、悪意のある Web サイトは、アプリケーションの `/user/email` ルートを指す HTML フォームを作成し、悪意のあるユーザー自身の電子メールアドレスを送信する可能性があります。

```blade
<form action="https://your-application.com/user/email" method="POST">
    <input type="email" value="malicious-email@example.com">
</form>

<script>
    document.forms[0].submit();
</script>
```

  悪意のある Web サイトがページのロード時に自動的にフォームを送信する場合、悪意のあるユーザーは、アプリケーションの疑いを持たないユーザーを誘導して Web サイトにアクセスさせるだけでよく、そのユーザーの電子メールアドレスはアプリケーション内で変更されます。

  この脆弱性を防ぐには、悪意のあるアプリケーションがアクセスできないシークレットセッション値について、受信するすべての `POST`、`PUT`、`PATCH`、`DELETE` リクエストを確認する必要があります。

<a name="preventing-csrf-requests"></a>
## CSRF リクエストの防止

Laravelは、アプリケーションによって管理されるアクティブな [ユーザーセッション](/docs/{{version}}/session) ごとにCSRF「トークン」を自動的に生成します。このトークンは、認証済みユーザーが実際にアプリケーションにリクエストを行っている本人であることを確認するために使用されます。このトークンはユーザーのセッションに保存され、セッションが再生成されるたびに変更されるため、悪意のあるアプリケーションはアクセスできません。

現在のセッションの CSRF トークンには、リクエストのセッションまたは `csrf_token` ヘルパ関数を介してアクセスできます。

    use Illuminate\Http\Request;

    Route::get('/token', function (Request $request) {
        $token = $request->session()->token();

        $token = csrf_token();

        // ...
    });

アプリケーションで `POST`、`PUT`、`PATCH`、または `DELETE` HTML フォームを定義するときは常に、CSRF 保護ミドルウェアがリクエストを検証できるように、フォームに非表示の CSRF `_token` フィールドを含める必要があります。便宜上、`@csrf` Blade ディレクティブを使用して、非表示のトークン入力フィールドを生成できます。

```blade
<form method="POST" action="/profile">
    @csrf

    <!-- Equivalent to... -->
    <input type="hidden" name="_token" value="{{ csrf_token() }}" />
</form>
```

デフォルトで `web` ミドルウェアグループには、`App\Http\Middleware\VerifyCsrfToken` [ミドルウェア](/docs/{{version}}/middleware) が含まれており、リクエスト入力内のトークンが一致するかどうかを自動的に検証します。セッションに保存されたトークン。 これら 2 つのトークンが一致すると、認証されたユーザーがリクエストを開始したユーザーであることがわかります。

<a name="csrf-tokens-and-spas"></a>
### CSRF トークンと SPA

Laravel を API バックエンドとして利用する SPA を構築している場合は、API での認証と CSRF 脆弱性からの保護に関する情報について、[Laravel Sanctum ドキュメント](/docs/{{version}}/sanctum) を参照してください。

<a name="csrf-excluding-uris"></a>
### CSRF 保護から除外する URI

CSRF保護からURIのセットを除外したい場合があります。たとえば、[Stripe](https://stripe.com) を使用して支払いを処理し、同社の Webhook システムを利用している場合、Stripe の Webhook ハンドラルートを CSRF 保護から除外する必要があります。Stripe はあなたのルートに送信する CSRF トークンを把握できないためです。

通常、この種のルートは、`App\Providers\RouteServiceProvider` が `routes/web.php` ファイル内のすべてのルートに適用する `web` ミドルウェアグループの外側に配置する必要があります。ただし、ルートの URI を `VerifyCsrfToken` ミドルウェアの `$except` プロパティに追加することで、ルートを除外することもできます。

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as Middleware;

    class VerifyCsrfToken extends Middleware
    {
        /**
         * The URIs that should be excluded from CSRF verification.
         *
         * @var array
         */
        protected $except = [
            'stripe/*',
            'http://example.com/foo/bar',
            'http://example.com/foo/*',
        ];
    }

> **Note**  
> 便宜上、 [テストの実行](/docs/{{version}}/testing) 時に CSRF ミドルウェアはすべてのルートに対して自動的に無効になります。

<a name="csrf-x-csrf-token"></a>
## X-CSRF-TOKEN

`App\Http\Middleware\VerifyCsrfToken` ミドルウェアは、POST パラメータとして CSRF トークンをチェックするだけでなく、`X-CSRF-TOKEN` リクエスト ヘッダもチェックします。 たとえば、HTML の `meta` タグにトークンを含めることができます。

```blade
<meta name="csrf-token" content="{{ csrf_token() }}">
```

次に、jQuery などのライブラリに、すべてのリクエストヘッダーにトークンを自動的に追加するように指示できます。これにより、レガシー JavaScript テクノロジを使用して、AJAX ベースのアプリケーションにシンプルで便利な CSRF 保護が提供されます。

```js
$.ajaxSetup({
    headers: {
        'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
    }
});
```

<a name="csrf-x-xsrf-token"></a>
## X-XSRF-TOKEN

Laravel は、フレームワークによって生成された各レスポンスに含まれる暗号化された `XSRF-TOKEN` クッキーに現在の CSRF トークンを保存します。クッキー値を使用して `X-XSRF-TOKEN` リクエストヘッダを設定できます。

Angular や Axios などの一部の JavaScript フレームワークやライブラリでは、同じオリジンのリクエストの `X-XSRF-TOKEN` ヘッダにその値が自動的に配置されるため、このクッキーは主に開発者の利便性を目的として送信されます。

> **Note**  
> デフォルトでは、`resources/js/bootstrap.js` ファイルには、`X-XSRF-TOKEN` ヘッダーを自動的に送信する Axios HTTP ライブラリが含まれています。
