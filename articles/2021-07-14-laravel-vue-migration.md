---
title: "マイグレーションファイルを作成"
---
:::message
この記事は2021年7月14日に個人ブログへ投稿した記事の転載です。
:::
LaravelでVueを使う時のリレーションについて複数回に分けて説明します。
:::message
当サイトでは米国株を題材に作成しています。
:::
今回はデータベースのテーブルを作成するためにマイグレーションファイルを作成して、使用するusstocksテーブルとmarketsテーブルについて説明します。

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

次回はusstocksテーブルとmarketsテーブルに対応したModelでリレーションの設定を説明します。
