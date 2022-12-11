---
title: "FlutterでDriftを使うとColumnが使えなくなる件"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["flutter"]
published: true
---

FlutterでUIを作っている時に`Column`ウィジェットを使おうとしたらエラーが出ました。
当たり前に使っているのにエラーが出て困惑してしまいました。
エラーメッセージからDriftが影響しているようです。
原因がわかったので調べて解決することができました。
エラーから解決までの流れを記録しておきます。

Driftを使っている状態でレイアウトの`Column`ウィジェットを使うと以下のエラーが出ました。
ちなみにエラーには赤い波線が引かれます。

![](/images/flutter-column_drift/code_before.png)

エラーが出ている`Column`クラスにマウスカーソルを合わせてみます。
そうすると2つのエラーメッセージが出てきます。

![](/images/flutter-column_drift/code_error.png)

2つ目のエラーメッセージに注目します。
エラーメッセージを読むとDriftのソースコード`dsl.dart`とFlutterのソースコード`basic.dart`の両方でクラス名`Column`が使われているようです。
これで原因は分かりましたが解決方法が分かりません。
こんな時はググってみるべし！！
とりあえず適当と思われるキーワード「flutter column drift」で検索してみました。
検索結果はこちら⬇️

![](/images/flutter-column_drift/search.png)

結論から言うと上から4つ目の検索結果を参考にしました。
理由はちょっと見にくいですがリンク元がmoorのGitHubのIssuesだったからです。
moorとはDriftの以前の名前です。
GitHubのIssuesであれば求めている結果に近い可能性が高いと考えました。
早速見てみましょう。

![](/images/flutter-column_drift/result.png)

検索結果先の表題は「Column class clashes with Flutter」で直訳すると「コラムクラスがFlutterと衝突する」となります。
表題から見ても求めているもののようです。
ページを見ていくとすぐに解決策が見つかりました。

```dart
import 'package:moor_flutter/moor_flutter.dart' hide Column;
```

インポートした`drift.dart`に`hide Column`を追加すれば良さそうです。
解決策を参考に以下のようにコードを追加してみます。

```dart
import 'package:drift/drift.dart' hide Column;
```

![](/images/flutter-column_drift/code_after.png)

追加するとエラーの警告が消えました。
赤い波線も無くなっています。

DriftはデータベースSQLiteを扱いやすくするためのパッケージです。
データベースは「テーブル」「カラム」「レコード」「フィールド」という4つの要素から構成されいます。
要素の1つに「カラム」があることからも`Column`がクラス名で使われていてもおかしくないですね。

Drift以外のデータベース関連のパッケージを使っていても同様なエラーが出るかもしれません。
みなさんの参考になれば幸いです。
