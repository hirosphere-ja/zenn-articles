---
title: "Modelでリレーションを設定"
---
:::message
この記事は2021年7月18日に個人ブログへ投稿した記事の転載です。
:::
usstocksテーブルとmarketsテーブルに対応したModelでリレーションの設定について説明します。
:::message
当サイトでは米国株を題材に作成しています。
:::
# Modelの作成
まずはusstocksテーブルとmarketsテーブルに対応したModelファイルを作成します。
usstocksテーブルに対応したModel名を`Usstock`として、marketsテーブルに対応したModel名を`Market`とします。

`php artisan make:model`がコマンド名で`Usstocks`がファイル名となります。

```
php artisan make:model Usstock
php artisan make:model Market
```

Laravelのプロジェクトを作成するとデフォルトでUserという名前のModelファイルがありファイル名は大文字で始まっています。
それにしたがって作成します。

# Modelの設定
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

次回はcontrollerを作成してJSON形式で表示できるようにします。
