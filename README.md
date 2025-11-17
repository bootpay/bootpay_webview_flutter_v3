# bootpay_webview_flutter 플러터 라이브러리

[![pub package](https://img.shields.io/pub/v/bootpay_webview_flutter.svg)](https://pub.dev/packages/bootpay_webview_flutter)

webview_flutter를 부트페이가 Fork 떠서 만든 웹뷰입니다. 이미 결제모듈이 동작하는 웹사이트에 webview로 링크만 연결하여 사용하실 웹앱 flutter 개발자분께서는 해당 모듈의 웹뷰를 사용하시면 쉽게 결제 진행이 가능하십니다.

## 주요 특징

- ✅ **한국 결제 환경 완벽 지원**: 30+ 한국 결제앱 (카카오페이, 네이버페이, 토스, 신한/KB/삼성카드 등)
- ✅ **최신 webview_flutter 기반**: Android v4.4.0, iOS v3.23.2
- ✅ **앱투앱 결제 자동 처리**: Intent scheme, Custom URL scheme 자동 처리
- ✅ **ISP/앱카드 지원**: 모든 카드사 앱카드 및 ISP 완벽 지원
- ✅ **간편결제 지원**: PAYCO, SSG페이 등

## 설치하기
``pubspec.yaml`` 파일에 아래 모듈을 추가해주세요
```yaml
...
dependencies:
 ...
 bootpay_webview_flutter: last_version
...
```

## 설정하기

### Android
따로 설정하실 것이 없습니다.

### iOS
** {your project root}/ios/Runner/Info.plist **
``CFBundleURLName``과 ``CFBundleURLSchemes``의 값은 개발사에서 고유값으로 지정해주셔야 합니다. 외부앱(카드사앱)에서 다시 기존 앱으로 앱투앱 호출시 필요한 스키마 값입니다.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    ...

    <key>NSAppTransportSecurity</key>
    <dict>
        <key>NSAllowsArbitraryLoads</key>
        <true/>
    </dict>
    <key>CFBundleURLTypes</key>
    <array>
        <dict>
            <key>CFBundleTypeRole</key>
            <string>Editor</string>
            <key>CFBundleURLName</key>
            <string>kr.co.bootpaySample</string> 
            <key>CFBundleURLSchemes</key>
            <array>
                <string>bootpaySample</string> 
            </array>
        </dict>
    </array>

    ...
</dict>
</plist>
```

## 사용예제

```dart 
import 'package:flutter/material.dart';
import 'dart:async';

import 'dart:io';
import 'package:bootpay_webview_flutter/webview_flutter.dart';

void main() => runApp(MaterialApp(home: WebViewExample()));

class WebViewExample extends StatefulWidget {
  @override
  _WebViewExampleState createState() => _WebViewExampleState();
}

class _WebViewExampleState extends State<WebViewExample> {
  late final WebViewController _controller;


  @override
  void initState() {
    super.initState();

    // #docregion platform_features
    late final PlatformWebViewControllerCreationParams params;
    if (WebViewPlatform.instance is BTWebKitWebViewPlatform) {
      params = WebKitWebViewControllerCreationParams(
        allowsInlineMediaPlayback: true,
        mediaTypesRequiringUserAction: const <PlaybackMediaTypes>{},
      );
    } else {
      params = const PlatformWebViewControllerCreationParams();
    }

    final WebViewController controller =
    WebViewController.fromPlatformCreationParams(params);
    // #enddocregion platform_features

    controller
      ..setJavaScriptMode(JavaScriptMode.unrestricted)
      ..setBackgroundColor(const Color(0x00000000))
      ..setNavigationDelegate(
        NavigationDelegate(
          onProgress: (int progress) {
            debugPrint('WebView is loading (progress : $progress%)');
          },
          onPageStarted: (String url) {
            debugPrint('Page started loading: $url');
          },
          onPageFinished: (String url) {
            debugPrint('Page finished loading: $url');
          },
          onWebResourceError: (WebResourceError error) {
            debugPrint('''
Page resource error:
  code: ${error.errorCode}
  description: ${error.description}
  errorType: ${error.errorType}
  isForMainFrame: ${error.isForMainFrame}
          ''');
          },
          onNavigationRequest: (NavigationRequest request) {
            if (request.url.startsWith('https://www.youtube.com/')) {
              debugPrint('blocking navigation to ${request.url}');
              return NavigationDecision.prevent;
            }
            debugPrint('allowing navigation to ${request.url}');
            return NavigationDecision.navigate;
          },
        ),
      )
      ..addJavaScriptChannel(
        'Toaster',
        onMessageReceived: (JavaScriptMessage message) {
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(content: Text(message.message)),
          );
        },
      )
      ..loadRequest(Uri.parse('https://flutter.dev'));

    // #docregion platform_features
    if (controller.platform is AndroidWebViewController) {
      AndroidWebViewController.enableDebugging(true);
      (controller.platform as AndroidWebViewController)
          .setMediaPlaybackRequiresUserGesture(false);
    }
    // #enddocregion platform_features

    _controller = controller;
  }

  @override
  Widget build(context) {
    return Scaffold(
      backgroundColor: Colors.green,
      body: WebViewWidget(controller: _controller),
    );
  }
}
```

## 지원하는 결제 앱

### 카드사 앱카드
- 신한카드 (shinhan-sr-ansimclick)
- KB카드 (kb-acp)
- 삼성카드 (mpocket.online.ansimclick, samsungpay)
- 현대카드 (hdcardappcardansimclick)
- 롯데카드 (lotteappcard, lottesmartpay)
- NH농협카드 (nhappvardansimclick)
- 우리카드 (com.wooricard.wcard)
- 하나카드 (cloudpay)
- 씨티카드 (citispay)

### 간편결제
- 카카오페이 (kakaotalk)
- 네이버페이 (naversearchapp)
- 토스 (supertoss)
- PAYCO (payco)
- SSG페이 (shinsegaeeasypayment)
- L.pay (lpayapp)

### 인증/결제
- 뱅크페이 (kfc-bankpay)
- ISP (kfc-ispmobile)
- PASS (SKT/KT/LG U+)

## 버전 정보

| 패키지 | 버전 | 기반 |
|--------|------|------|
| bootpay_webview_flutter | 4.11.0 | webview_flutter 4.13.0 |
| bootpay_webview_flutter_android | 4.4.0 | webview_flutter_android 4.10.5 |
| bootpay_webview_flutter_wkwebview | 3.23.2 | webview_flutter_wkwebview 3.23.2 |
| bootpay_webview_flutter_platform_interface | 2.14.0 | webview_flutter_platform_interface 2.14.0 |

## 변경 이력

자세한 변경 이력은 [CHANGELOG.md](CHANGELOG.md)를 참조해주세요.

## Documentation

[부트페이 개발매뉴얼](https://docs.bootpay.co.kr)을 참조해주세요

## 기술문의

[채팅](https://bootpay.channel.io/)으로 문의

## License

[MIT License](https://opensource.org/licenses/MIT).

