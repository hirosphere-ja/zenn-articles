---
title: "Laravelで文字列のプライマリーキーに必要なコード"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["PHP","Laravel"]
published: false
published_at: 2021-07-04 09:00
---
:::message
この記事は2021年7月4日に個人ブログへ投稿した記事の転載です。
:::
文字列のプライマリーキーを使おうとしてエラーの発生から解決までの道のりを紹介します。
ここではプライマリーキーを`string_id`と仮定しています。

# エラーが発生
一覧表示から詳細を表示するコードを書いて詳細表示しようとしたら以下のエラーが発生。
```sql
SQLSTATE[HY000]: General error: 1 no such column: id
```
エラーメッセージを見たら`id`がありませんとおっしゃっていらっしゃいます。
調べたらプライマリーキーが`id`以外の場合は明示的に指定すること必要でした。
以下のコードをModelファイルに追加しました。
```php
protected $primaryKey='string_id';
```

# 新たなエラーが発生
そうしたら次は文字列で設定しているプライマリーキーが一覧表示で0と表示される現象に遭遇。
詳細表示しようとしたら以下のエラーが発生。
```
Trying to get property 'string_id' of non-object
```
これは`string_id`がないってことだと思い・・・、ってさっき設定したじゃん！！
小一時間ほど調べましてやっと解決策を見つけました。
プライマリーキーが文字列の場合は以下の設定を追加することがわかりました。
```php
protected $keyType='string';
```

# まとめ
文字列のプライマリーキーでは以下の2つのコードが必要だとわかりました。
```php
protected $primaryKey='string_id';
protected $keyType='string';
```
