---
title: "FlutterでGoogle Mobile Adsを導入してみる"
emoji: "✨"
type: "tech"
topics: ["flutter", "Android"]
published: false
---

# まえがき

今回はgoogle_mobile_adsパッケージを使ってバナー広告を導入した手順を記録します。
導入先は現在、再開発しているスマホアプリです。

https://pub.dev/packages/google_mobile_ads

実は以前、作成したスマホアプリにadmob_flutterパッケージを使ってバナー広告を導入したことがあります。
ただ今ではadmob_flutterパッケージは更新が追いついておらず、Googleが公開しているgoogle_mobile_adsパッケージを推奨しています。

https://pub.dev/packages/admob_flutter/versions/3.0.0

リファレンスを見る限り導入方法は変わらないようなので、以前のコードやリファレンスを参考に進めていきます。
また今回のアプリはAndroid版のみ公開する予定なので、Android版だけを対象に進めていきます。
いずれはiPhone版も公開できればと思っていますので、その時はiPhone版を対象に進めた手順も改めて紹介できればと考えています。

ここからは以下のページから参考に進めていきます。

https://developers.google.com/ad-manager/mobile-ads-sdk/flutter/quick-start

それではいきましょう！！

# 始める

## Prerequisites(前提条件)

![](/images/flutter-mobile_ads/image01.png)
*(https://developers.google.com/ad-manager/mobile-ads-sdk/flutter/quick-start#prerequisites)*

Google Mobile Adsを導入するには以下の前提条件があります。
- Flutter 1.22.0以降
- Android
	- Android Studio 3.2以上
	- ターゲット: Android APIレベル20以降
	- compileSdkVersionを28以上に設定します

FlutterとAndroidStudioのバージョンは最新であれば前提条件をクリアしています。
<!-- ちなみにFlutterのバージョンによってAndroid APIレベルとcompileSdkVersionを設定するファイルの場所が違います。最新のバージョンであれば、インストールしたFlutterのディレクトリ以下のpackages/flutter_tools/gradle/flutter.gradleに -->

## Mobile Ads SDK をインポートする

![](/images/flutter-mobile_ads/image02.png)
*(https://developers.google.com/ad-manager/mobile-ads-sdk/flutter/quick-start#import_the_mobile_ads_sdk)*

リンクからgoogle_mobile_adsパッケージのHPに移動します。

![](/images/flutter-mobile_ads/image03.png)
*(https://pub.dev/packages/google_mobile_ads/install)*

ターミナルから以下のコマンドを実行します。

```bash
flutter pub add google_mobile_ads
```

または`pubspec.yaml`の`dependencies`のところに`google_mobile_ads: ^2.3.0`を入力してターミナルで`flutter pub get`を実行します。

```yaml
dependencies:
  google_mobile_ads: ^2.3.0
```

## プラットフォーム固有の設定

![](/images/flutter-mobile_ads/image04.png)
*(https://developers.google.com/ad-manager/mobile-ads-sdk/flutter/quick-start#platform_specific_setup)*

Androidの場合は`AndroidManifest.xml`を更新します。

![](/images/flutter-mobile_ads/image05.png)

更新する場所は下にあります。
赤線の下部に以下のコードを追加します。

```xml
<meta-data
    android:name="com.google.android.gms.ads.APPLICATION_ID"
    android:value="ca-app-pub-3940256099942544~3347511713"/>
```

`ca-app-pub-3940256099942544~3347511713`はテスト用のアプリIDです。
本番では本番用のアプリIDを入手して書き換えてください。

## Mobile Ads SDK を初期化する

![](/images/flutter-mobile_ads/image06.png)
*(https://developers.google.com/ad-manager/mobile-ads-sdk/flutter/quick-start#initialize_the_mobile_ads_sdk)*

`main.dart`のトップに`import 'package:google_mobile_ads/google_mobile_ads.dart';`を追加、`main()`以下に`MobileAds.instance.initialize();`を追加します。

```diff dart
+ import 'package:google_mobile_ads/google_mobile_ads.dart';
  import 'package:flutter/material.dart';

  void main() {
    WidgetsFlutterBinding.ensureInitialized();
+   MobileAds.instance.initialize();

    runApp(MyApp());
  }
```

## 広告フォーマットを選択

![](/images/flutter-mobile_ads/image07.png)
*(https://developers.google.com/ad-manager/mobile-ads-sdk/flutter/quick-start#select_an_ad_format)*

広告フォーマットを選択します。
今回はバナーを実装するので、「バナー広告を実装する」に移動します。

# バナー

## 常にテスト広告でテストする

ここでは公式サイトの手順は使いません。
以前からテストで使っていたAndroid用のテスト広告ユニットID `ca-app-pub-3940256099942544/6300978111`を使用します。

:::message
公式サイトの手順がよくわかリませんでした。
そこで以前のコードを参考に進めていきます。
わからなければ、諦めてわかる方法を見つけましょう！！
まずは思い通りに動かすことが重要です。
:::

## 広告をインスタンス化

![](/images/flutter-mobile_ads/image08.png)
*(https://developers.google.com/ad-manager/mobile-ads-sdk/flutter/banner/get-started#instantiate_ad)*

`AdManagerBannerAd`
- `adUnitId`
- `AdSize`
- `AdManagerAdRequest`
- `AdManagerBannerAdListener`

インスタンス化するにあたり、以上の4つのプロパティを設定します。
`adUnitId`は先述した`ca-app-pub-3940256099942544/6300978111`を使用します。
また`AdManagerAdRequest()`は例のまま使用します。

```dart
final AdManagerBannerAd myBanner = AdManagerBannerAd(
  adUnitId: 'ca-app-pub-3940256099942544/6300978111',
  sizes: [AdSize.banner],
  request: AdManagerAdRequest(),
  listener: AdManagerBannerAdListener(), // BannerAdListener()は間違い？
);
```

:::message
listener: BannerAdListener()は間違い？
公式サイトのコードの例はBannerAdListener()になっています。
しかし上記のようにAdManagerBannerAdListener()が正解では？
:::

### バナーのサイズ

![](/images/flutter-mobile_ads/image09.png)
*(https://developers.google.com/ad-manager/mobile-ads-sdk/flutter/banner/get-started#banner_sizes)*

標準バナーのサイズは以下の表のように5種類あります。

| サイズ（dp、WxH） | 説明 | AdSize定数 |
| ---- | ---- | ---- |
| 320x50 | 標準バナー | banner |
| 320x100 | バナー（大）| largeBanner |
| 320x250 | レクタングル（中）| mediumRectangle |
| 468×60 | フルサイズ バナー | fullBanner |
| 728x90 | リーダーボード | leaderboard |

またバナーサイズは自分で設定することもできます。
以下のように`AdSize`で設定できます。

```dart
final AdSize adSize = AdSize(300, 50);
```

### バナー広告のイベント

![](/images/flutter-mobile_ads/image10.png)
*(https://developers.google.com/ad-manager/mobile-ads-sdk/flutter/banner/get-started#banner_ad_events)*

バナー広告のイベントには以下の5つがあります。

- `onAdLoaded`：バナー広告が呼び出された時
- `onAdFailedToLoad`：バナー広告の読み込みに失敗した時
- `onAdOpened`：バナー広告が開かれた時
- `onAdClosed`：バナー広告が閉じられた時
- `onAdImpression`：バナー広告にインプレッションが発生したとき

`onAdFailedToLoad`が発生したときはエラーが返されます。

## 広告を読み込む

![](/images/flutter-mobile_ads/image11.png)
*(https://developers.google.com/ad-manager/mobile-ads-sdk/flutter/banner/get-started#load_ad)*

インスタンスを作成した`AdManagerBannerAd`を画面に表示するため`load()`を呼び出します。

## 表示広告

![](/images/flutter-mobile_ads/image12.png)
*(https://developers.google.com/ad-manager/mobile-ads-sdk/flutter/banner/get-started#display_ad)*

`AdWidget`を使用して広告を表示します。
まずは`AdWidget`のインスタンスを作成します。
そこで事前に作成した`AdManagerBannerAd`のインスタンス`myBanner`を`AdWidget`に渡します。

```dart
final AdWidget adWidget = AdWidget(ad: myBanner);
```

`AdWidget`はFlutterのウィジェットと同じように使用できます。
公式サイトではバナーを格納する`Container`のインスタンス`adContainer`を作成して、その中に小ウェジェットとしてインスタンス`adWidget`を渡します。

```dart
final Container adContainer = Container(
  alignment: Alignment.center,
  child: adWidget,
  width: size.width.toDouble(),
  height: size.height.toDouble(),
);
```

# 実装

ここまで公式サイトを見ながら、使用方法を見てきました。
しかし、私のレベルではどう実装すればいいのかわかりません。

## 