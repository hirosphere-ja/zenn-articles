---
title: "Laravelでバリデーションを実装してみた"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["PHP","Laravel"]
published: true
published_at: 2021-07-04 08:00
---
:::message
この記事は2021年7月4日に個人ブログへ投稿した記事の転載です。
:::
# バリデーションとは
バリデーションとは入力時に必要な値が入力されているかチェックする機能のことです。
バリデーションを設定することで入力エラーを未然に防ぎ、必要な値の入力を促します。
今回はlaravel公式サイトの日本語訳サイトのドキュメントを参考にしてバリデーションを実装してみました。
https://readouble.com/laravel/6.x/ja/validation.html

# バリデーションエラーを表示
バリデーションエラーを表示できるようにviewファイルに以下のコードを追加しました。
```php
@if($errors->any())
  <div class="alert alert-danger">
    <ul>
      @foreach($errors->all() as $error)
        <li>{{$error}}</li>
      @endforeach
    </ul>
  </div>
@endif
```

# バリ<!-- 無視 -->データの生成
バリ<!-- 無視 -->データとはバリデーションを行う機能のことです。
バリ<!-- 無視 -->データの生成するにはバリ<!-- 無視 -->データを使うコントローラーにValidatorファサードを使えるようにコードを追加する必要があります。
今回はUserController.phpに追加します。
こういうコードって忘れがちなんで気をつけていただければと思います。
```diff php
  namespace App\Http\Controllers;

  use App\Http\Controllers\Controller;
  use Illuminate\Http\Request;
+ use Illuminate\Support\Facades\Validator; //この行を追加
```

次にUseControllerクラスの`store`関数に次のコードを追加します。
```php
class UserController extends Controller {
  public function store(Request $request) {
    $validator = Validator::make($request->all(), [
      'title' => 'unique:users',
    ]);
    if ($validator->fails()) {
      return redirect('users/create')
        ->withErrors($validator)
        ->withInput();
    }
  }
}
```
`make`メソッドの第1引数の`$request->all()`はバリデーションを行うデータでリクエストのすべてを取得しています。
第2引数はそのデータに適用するバリデーションのルールです。
`title`に適用するルールは`unique:users`でusersテーブルに使われている`title`は重複不可と言う意味です。
次にバリデーションでエラーが生じた場合のコードがif文から書いてありますが、私もよくわからないのでこういうものだということにしておきます。

さきほどのコードで出てきたバリ<!-- 無視 -->データは3つに引数があります。
```php
$validator = Validator::make($input, $rules, $messages);
```
第1引数と第2引数は上記で説明したように最後の第3引数はエラーメッセージのカスタムメッセージを渡します。
最初に出てきたエラーメッセージ”error message”を日本語にしてみたいと思います。
以下のようなコードでカスタムエラーメッセージを指定できます。
```php
$messages = [
  'title.unique' => 'titleが重複しています。',
];
```

# バリ<!-- 無視 -->データのまとめ
```php
//第1引数：バリデーションを行うデータ
$input = $request->all();
//第2引数：そのデータの適用するルール
$rules = ['title' => 'unique:users'];
//第3引数：カスタムエラーメッセージを指定する
$messages = ['title.unique' => 'titleが重複しています。'];
$validator = Validator::make($input, $rules, $messages);
```
上記のように3つの引数をメソッドの外で指定します。
うすると複雑になってもメンテナンスしやすいんじゃないでしょうか？

わかりやすくしたつもりなので皆様の参考になればと思います。
