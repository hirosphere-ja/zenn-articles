---
title: "Flutterのhooks_riverpodにチャレンジ"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["flutter"]
published: false
---
プロジェクトを新規作成したときにできるカウンターをhooks_riverpodを使って改造してみました。
幸いhooks_riverpodのドキュメントにちょうど具体例があったのでそれを参考にしました。

まずは適当なディレクトリにFlutterの新規プロジェクトを作成します。
作成すると以下のようなコードが作成されます（コメントは削除しています）。
```dart:main.dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: const MyHomePage(title: 'Flutter Demo Home Page'),
    );
  }
}
class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key, required this.title});
  final String title;
  @override
  State<MyHomePage> createState() => _MyHomePageState();
}
class _MyHomePageState extends State<MyHomePage> {
  int _counter = 0;
  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            const Text(
              'You have pushed the button this many times:',
            ),
            Text(
              '$_counter',
              style: Theme.of(context).textTheme.headline4,
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter,
        tooltip: 'Increment',
        child: const Icon(Icons.add),
      ),
    );
  }
}
```
hooks_riverpodを使うことの最大の目的は`StatefulWidget`を使わないことです。
（間違っていたらごめんなさい）。
そこで最初にできたコードから不要な部分をざっくり省きます。
（ここまできたらほぼ作り直し）。
```dart:main.dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: const TopPage(),
    );
  }
}

class TopPage extends StatelessWidget {
  const TopPage({super.key});
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Counter'),
      ),
      body: Column(),
    );
  }
}
```
ちなみに実行してみるとこんな感じです。
![](/images/400fdd83f2e5ea/image1.png =400x)
AppBarにタイトル「Counter」があるだけでメインの画面には何もありません。





```dart:main.dart
import 'package:example2/home_page_consumer.dart';
import 'package:flutter/material.dart';
import 'package:hooks_riverpod/hooks_riverpod.dart';

void main() {
  runApp(
    const ProviderScope(
      child: MyApp(),
    ),
  );
}
```

```dart:counter.dart
import 'package:hooks_riverpod/hooks_riverpod.dart';

class Counter extends StateNotifier<int> {
  Counter() : super(0);
  void increment() => state++;
  void decrement() => state--;
}

final counterProvider = StateNotifierProvider<Counter, int>(
  (ref) => Counter(),
);
```

```dart:home_page_consumer.dart
import 'package:example2/counter.dart';
import 'package:flutter/material.dart';
import 'package:hooks_riverpod/hooks_riverpod.dart';

class HomePageConsumer extends ConsumerWidget {
  const HomePageConsumer({super.key});
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return Scaffold(
      appBar: AppBar(),
      body: Consumer(
        builder: (context, ref, child) {
          final count = ref.watch(counterProvider);
          return Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              const SizedBox(
                height: 20,
              ),
              Center(
                child: Text(
                  '$count',
                  style: const TextStyle(
                    fontSize: 30,
                  ),
                ),
              ),
              const SizedBox(
                height: 20,
              ),
              TextButton(
                onPressed: ref.watch(counterProvider.notifier).increment,
                child: const Text(
                  "increment counter",
                  style: TextStyle(
                    fontSize: 30,
                  ),
                ),
              ),
              TextButton(
                onPressed: ref.watch(counterProvider.notifier).decrement,
                child: const Text(
                  "decrement counter",
                  style: TextStyle(
                    fontSize: 30,
                  ),
                ),
              ),
            ],
          );
        },
      ),
    );
  }
}
```


