---
title: "ControllerとResourceを作成してJSON形式で取得"
---
:::message
この記事は2021年7月22日に個人ブログへ投稿した記事の転載です。
:::
ControllerとResourceを作成してJSON形式で取得できるようにします。
:::message
当サイトでは米国株を題材に作成しています。
:::
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
