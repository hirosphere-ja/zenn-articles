---
title: "LaravelでVueを使う時のリレーション"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["PHP","Laravel","Vue"]
published: true
published_at: 2021-07-22 08:00
---
:::message
この記事は2021年7月に個人ブログへ投稿した複数の記事をまとめたものです。
:::
LaravelでVueを使う時のリレーションについて説明します。
:::message
当サイトでは米国株を題材に作成しています。
:::

# マイグレーションファイルを作成
以下のコマンドを入力してマイグレーションファイルを作成します。
```php
php artisan make:migration create_usstocks_table
php artisan make:migration create_markets_table
```
`php artisan make:migration`がコマンド名で`create_usstocks_table`がファイル名となります。
ちなみにファイル名が`usstocks`ではなくて、`create_usstocks_table`なのか？
Laravelのプロジェクトを作成するとデフォルトでマイグレーションファイルがあり、ファイル名の命名規則が`create_（テーブル名・複数形）_table`となっています。
それにしたがって作成します。

# テーブルのカラムを設定
作成した各マイグレーションファイルにテーブルのカラムを設定します。

## usstocksテーブル
usstocksテーブルには以下の3つのカラムを設定します
`ticker`：ティッカー（米国株の個々の銘柄を識別するためにつけられた記号）
`name`：銘柄名
`market_id`：市場ID
```php:create_usstocks_table.php
<?php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;
class CreateUsstocksTable extends Migration {
  public function up() {
    Schema::create('usstocks', function (Blueprint $table) {
      $table->string('ticker')->unique()->primary();
      $table->string('name');
      $table->integer('market_id');
    });
  }
  public function down() {
    Schema::dropIfExists('usstocks');
  }
}
```
7行目の`Schema::create`内に必要なカラムを設定します。
8行目にはテーブルの主キーとなる`ticker`をカラム名とします。
通常、主キーは整数を使いますが今回は文字列を主キーとします。
`string`は文字列、`unique`は重複しないこと、`primary`は主キーを指定しています。
9行目は`name`をカラム名として`string`で文字列を指定します。
10行目はリレーションで使う`market_id`をカラム名として`integer`は整数を指定しています。

## marketsテーブル
marketsテーブルには以下の2つのカラムを設定します
`market_id`：市場ID（1・2）
`market`：市場名（NYSE・NASDAQ）
以下にレコードを設定します。
1：NYSE
2：NASDAQ
```php:create_markets_table.php
<?php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;
class CreateUsstocksTable extends Migration {
  public function up() {
    Schema::create('markets', function (Blueprint $table) {
      $table->bigIncrements('market_id');
      $table->string('market')->unique();
    });
  }
  public function down() {
    Schema::dropIfExists('markets');
  }
}
```
8行目にテーブルの主キーとなる`market_id`をカラム名として`bigIncrements`は主キーを指定しています。
9行目の`market`をカラム名として`string`で文字列を指定します。

# Modelの作成
## Modelの作成
まずはusstocksテーブルとmarketsテーブルに対応したModelファイルを作成します。
usstocksテーブルに対応したModel名を`Usstock`として、marketsテーブルに対応したModel名を`Market`とします。

`php artisan make:model`がコマンド名で`Usstocks`がファイル名となります。

```
php artisan make:model Usstock
php artisan make:model Market
```

Laravelのプロジェクトを作成するとデフォルトでUserという名前のModelファイルがありファイル名は大文字で始まっています。
それにしたがって作成します。

## Modelの設定
リレーションで`usstock`から`market`へアクセスできるようにする設定します。
```php:Usstock.php
<?php
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Usstock extends Model {
  // 文字列でプライマリーキーを使用するために必要な設定
  protected $keyType = 'string';
  protected $primaryKey = 'ticker';

  public function market(){
    return $this->belongsTo(Market::class,'market_id','market_id');
  }
}
```
12行目から`market`関数を作成して`belongsTo`メソッドを使ってリレーションの設定を記述します。
13行目の`belongsTo`メソッドの第1引数は`Market`クラスを指定します。
第2引数には`usstocks`テーブルの`market_id`を指定します。
第3引数には親となる`markets`テーブルの`market_id`を指定します。

他にも`hasOne`や`hasMany`などメソッドがありますので以下の日本語リファレンスを参照してください。

https://readouble.com/laravel/8.x/ja/eloquent-relationships.html

9行目と10行目には文字列を主キーを使用する場合には必要になります。
こちらを参考にしてみてください。

https://zenn.dev/zenn/articles/2021-07-04-laravel-string-primarykey

# Resourceの作成
## Resourceの作成
まずにusstocksテーブルに対応したResourceファイルを作成します。
Vueでデータを受け取るにはJSON形式になっていないと受け取ることができません。
ResourceファイルではそのデータをJSON形式に変換する設定を行います。
usstocksテーブルに対応したResource名を`UsstockResource`とします。
marketsテーブルに対応したResourceは必要ないので作成しません。

`php artisan make:resource`がコマンド名で`UsstockResource`がファイル名となります。
```
php artisan make:resource UsstockResource
```
## Resourceの設定
```php
<?php
namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\JsonResource;

class UsstockResource extends JsonResource
{
  public function toArray($request) {
    return [
      "ticker" => $this->ticker,
      "name" => $this->name,
      "market" => $this->market,
    ];
  }
}
```

# Controllerの作成
## Controllerの作成
usstocksテーブルに対応したControllerファイルを作成します。
usstocksテーブルに対応したController名を`UsstockController`とします。
marketsテーブルに対応したControllerは必要ないので作成しません。

`php artisan make:controller`がコマンド名で`UsstockController`がファイル名となります
```
php artisan make:controller UsstockController
```

## リレーション前のControllerの設定
リレーションを理解するには順序立てて見るとわかりやすいのでリレーションする前の一覧表示を設定します。
11行目ではusstocksテーブルの全データを取得するためallメソッドを使用して、引数`$usstocks`に代入しています。
12行目では引数`$usstocks`を`UsstockResource`の`collection`メソッドに渡してJSON形式で出力します。
```php
<?php
namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Http\Resources\UsstockResource;
use Illuminate\Http\Request;
use App\Models\Usstock;

class UsstockController extends Controller {
  public function index() {
    $usstocks = Usstock::all();
    return UsstockResource::collection($usstocks);
  }
}
```
リレーションせずに一覧表示したJSON形式ではmarket_idのみが表示されています。
```json
"data":[
  {
    "ticker": "AAPL",
    "name": "アップル",
    "market_id": 2,
  },
]
```

## リレーション後のControllerの設定
次はリレーション後の一覧表示を設定します。
11行目をusstocksテーブルにmarketsテーブルのデータをリレーションするため`with`メソッドに前回Usstockモデルで作成した`market`関数を渡して`get`で取得して引数`$usstocks`に代入しています。
```php
<?php
namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Http\Resources\UsstockResource;
use Illuminate\Http\Request;
use App\Models\Usstock;

class UsstockController extends Controller {
  public function index() {
    $usstocks = Usstock::with('market')->get();
    return UsstockResource::collection($usstocks);
  }
}
```
リレーション後の一覧表示したJSON形式ではmarketオブジェクト内にmarket_idとmarketが表示されています。
```json
"data": [
  {
    "ticker": "AAPL",
    "name": "アップル",
    "market": {
      "market_id": 2,
      "market": "NASDAQ"
    },
  },
]
```

以上で取得したJSONデータを渡してVueで使用できるようになりました。
migrationからModelやControllerまで作成してリレーションの設定までできればある程度のWebアプリであれば対応できると思います。
VueだけでなくReactでも使用できるのでバックエンドを作成するリソースとしてLaravelを選択することもありかもしれません。
