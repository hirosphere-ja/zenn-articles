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

![](/images/flutter-mobile_ads/image08.png)
*(https://developers.google.com/ad-manager/mobile-ads-sdk/flutter/banner/get-started#always_test_with_test_ads)*

テスト広告でテストするためにはテスト広告ユニットが必要です。
Androidテスト広告ユニットから移動してテスト広告ユニットを確認します。

![](/images/flutter-mobile_ads/image09.png)
*(https://developers.google.com/ad-manager/mobile-ads-sdk/android/test-ads#enable_test_devices)*

バナーのテスト広告ユニットIDは`/6499/example/banner`となっています。
以降、これを使用します。
前のページに戻って次に移ります。

## 広告をインスタンス化

![](/images/flutter-mobile_ads/image10.png)
*(https://developers.google.com/ad-manager/mobile-ads-sdk/flutter/banner/get-started#instantiate_ad)*

AdManagerBannerAd
- adUnitId
- AdSize
- AdManagerAdRequest
- AdManagerBannerAdListener

インスタンス化するにあたり、以上の4つのプロパティを設定します。

```dart
final AdManagerBannerAd myBanner = AdManagerBannerAd(
  adUnitId: '/6499/example/banner',
  sizes: [AdSize.banner],
  request: AdManagerAdRequest(),
  listener: BannerAdListener(),
);
```

