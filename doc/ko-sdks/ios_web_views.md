## 개요

WebView를 사용하는 iOS 앱을 위한 Adjust.com™의 iOS SDK 설명서입니다. Adjust.com™에 관한 더 자세한 내용은 [Adjust.com]에서 확인할 수 있습니다.

[WebViewJavascriptBridge][web_view_js_bridge] 플러그인을 사용하여 자바스크립트를 네이티브 Objective-C 호출과 양방향으로 연결해 줍니다. 이 플러그인은 `MIT 허가서`에 따라 라이선스를 얻었습니다. 

## 목차

* [기본 연동](#basic-integration)
    * [네이티브 Adjust iOS SDK 추가](#native-add)
    * [프로젝트에 AdjustBridge 추가](#bridge-add)
    * [앱에 AdjustBridge 연동](#bridge-integrate-app)
    * [WebView에 AdjustBridge 연동](#bridge-integrate-web)
    * [기본 설정](#basic-setup)
    * [AdjustBridge 로그 기록](#bridge-logging)
    * [앱 빌드](#build-the-app)
* [부가 기능](#additional-features)
    * [이벤트 추적](#event-tracking)
        * [매출 추적](#revenue-tracking)
        * [콜백 파라미터](#callback-parameters)
        * [파트너 파라미터](#partner-parameters)
    * [어트리뷰션 콜백](#attribution-callback)
    * [이벤트 및 세션 콜백](#event-session-callbacks)
    * [이벤트 버퍼링](#event-buffering)
    * [추적 비활성화](#disable-tracking)
    * [오프라인 모드](#offline-mode)
    * [백그라운드 추적](#background-tracking)
    * [기기 ID](#device-ids)
    * [딥링크](#deeplinking)
        * [지연 딥링크 콜백](#deferred-deeplinking-callback)
* [라이선스](#license)

## <a id="basic-integration">기본 연동

### <a id="native-add">기본 Adjust iOS SDK 추가

WebView에서 Adjust SDK를 사용하려면 Adjust의 네이티브 iOS SDK를 앱에 추가해야 합니다. 네이티브 안드로이드 SDK를 설치하려면 [iOS SDK README](https://github.com/adjust/sdks/blob/master/doc/ko-sdks/ios_sdk_readme.md#basic-integration)에서 `기본 연동` 부분을 참조하십시오.

### <a id="bridge-add">프로젝트에 AdjustBridge 추가

Xcode 내 `Project Navigator`에서 `Supporting Files` 그룹을 (또는 원하는 다른 그룹을) 찾습니다. Finder에서 `AdjustBridge` 하위 디렉토리를 Xcode `Supporing Files` 그룹으로 드래그합니다.

![][bridge_drag]

`Choose options for adding these files` 다이얼로그에서 'Copy items into destination group's folder'에 체크 표시한 다음, 상단 라디오 버튼을 'Create groups for any added folders'로 선택합니다.

![][bridge_add]

### <a id="bridge-integrate-app">앱에 AdjustBridge 연동

Project Navigator에서 View Controller 소스 파일을 엽니다. 파일 맨 위에 `import` 문을 추가합니다. WebView Delegate의 `viewDidLoad` 또는 `viewWillAppear` 메서드에서 다음 호출을 `AdjustBridge`에 추가합니다..

```objc
#import "Adjust.h"
// Or #import <AdjustSdk/Adjust.h>
// (depends on the way you have chosen to add our native iOS SDK)
// ...

- (void)viewWillAppear:(BOOL)animated {
    UIWebView *webView = [[UIWebView alloc] initWithFrame:self.view.bounds];
    // or with WKWebView:
    // WKWebView *webView = [[NSClassFromString(@"WKWebView") alloc] initWithFrame:self.view.bounds];

    AdjustBridge *adjustBridge = [[AdjustBridge alloc] init];
    [adjustBridge loadUIWebViewBridge:webView];
    // optionally you can add a web view delegate so that you can also capture its events
    // [adjustBridge loadUIWebViewBridge:webView webViewDelegate:(UIWebViewDelegate*)self];
    
    // or with WKWebView:
    // [adjustBridge loadWKWebViewBridge:webView];
    // optionally you can add a web view delegate so that you can also capture its events
    // [adjustBridge loadWKWebViewBridge:webView wkWebViewDelegate:(id<WKNavigationDelegate>)self];
}

// ...
```

![][bridge_init_objc]

### <a id="bridge-integrate-web">WebView에 AdjustBrige 연동

WebView에 자바스크립트 브릿지를 추가하려면 [README][wvjsb_readme] 섹션 `4` 에 나와 있는 `WebViewJavascriptBridge` 플러그인처럼 설정이 이루어져야 합니다. 아래 자바스크립트 코드를 추가하여 Adjust iOS 웹 브릿지를 초기화하세요. 

```js
function setupWebViewJavascriptBridge(callback) {
    if (window.WebViewJavascriptBridge) {
        return callback(WebViewJavascriptBridge);
    }

    if (window.WVJBCallbacks) {
        return window.WVJBCallbacks.push(callback);
    }

    window.WVJBCallbacks = [callback];

    var WVJBIframe = document.createElement('iframe');
    WVJBIframe.style.display = 'none';
    WVJBIframe.src = 'wvjbscheme://__BRIDGE_LOADED__';
    document.documentElement.appendChild(WVJBIframe);

    setTimeout(function() { document.documentElement.removeChild(WVJBIframe) }, 0)
}

setupWebViewJavascriptBridge(function(bridge) {
    // AdjustBridge initialisation will be added in this method.
})
```

![][bridge_init_js]

### <a id="basic-setup">기본 설정

HTML 파일에서 아래 레퍼런스를 Adjust 자바스크립트 파일에 추가합니다. 

```html
<script type="text/javascript" src="adjust.js"></script>
<script type="text/javascript" src="adjust_event.js"></script>
<script type="text/javascript" src="adjust_config.js"></script>
```

추가된 레퍼런스는 HTML 파일에서 Adjust SDK 초기화에 사용할 수 있습니다. 

```js
setupWebViewJavascriptBridge(function(bridge) {
    // ...

    var yourAppToken = '{YourAppToken}'
    var environment = AdjustConfig.EnvironmentSandbox
    var adjustConfig = new AdjustConfig(bridge, yourAppToken, environment)

    Adjust.appDidLaunch(adjustConfig)

    // ...
})
```

![][bridge_init_js_xcode]

`{YourAppToken}` 을 해당 앱 토큰으로 바꾸면 됩니다. 앱 토큰은 [대시보드]에서 찾아볼 수 있습니다. 

테스트 빌드인지 제작 빌드인지 여부에 따라 `environment`를 다음 값 중 하나로 설정해야 합니다.

```js
var environment = AdjustConfig.EnvironmentSandbox
var environment = AdjustConfig.EnvironmentProduction
```

**주의:** 앱을 테스트하는 경우 이 값을 `AdjustConfig.EnvironmentSandbox`로 설정해야 합니다. 앱을 게시할 땐 `AdjustEnvironmentProduction`으로 바꾸는 걸 잊지 마세요. 개발 및 테스트 단계로 돌아가면 설정을 다시 `AdjustEnvironmentSandbox`에 맞춰야 합니다.

이는 실제 트래픽과 테스트 기기 상의 테스트 트래픽을 구분하기 위해서입니다. 언제나 그렇지만, 특히 매출 추적 시에는 이 값을 유의미하게 두는 게 대단히 중요합니다.

### <a id="bridge-logging">AdjustBridge 로그 기록

`AdjustConfig` 인스턴스에서 아래 파라미터 중 하나로 `setLogLevel`을 호출하여 테스트에서 확인하는 로그 분량을 늘이거나 줄일 수 있습니다. 

```objc
adjustConfig.setLogLevel(AdjustConfig.LogLevelVerbose) // enable all logging
adjustConfig.setLogLevel(AdjustConfig.LogLevelDebug)   // enable more logging
adjustConfig.setLogLevel(AdjustConfig.LogLevelInfo)    // the default
adjustConfig.setLogLevel(AdjustConfig.LogLevelWarn)    // disable info logging
adjustConfig.setLogLevel(AdjustConfig.LogLevelError)   // disable warnings as well
adjustConfig.setLogLevel(AdjustConfig.LogLevelAssert)  // disable errors as well
```

### <a id="build-the-app">앱 빌드

앱 빌드를 실행해 보세요. 빌드가 성공적이면 콘솔에서 SDK 로그를 주의 깊게 읽어 보세요. 최초 런칭 후 정보 로그 `Install tracked`가 보일 겁니다. 

![][bridge_install_tracked]

## <a id="additional-features">부가 기능

Adjust SDK를 프로젝트에 연동하면 다음과 같은 기능을 이용할 수 있습니다.

### <a id="event-tracking">이벤트 추적

Adjust로 원하는 어떤 이벤트든 추적할 수 있습니다. 특정한 버튼을 탭할 때마다 이를 추적하도록 하고 싶다면, [대시보드]에 새 이벤트 토큰을 만듭니다. 편의상 토큰을 `abc123`이라고 합시다. 버튼 내 `onclick` 메서드에 아래와 같은 라인을 추가하면 탭 추적이 가능합니다. 

```js
var adjustEvent = new AdjustEvent('abc123')
Adjust.trackEvent(adjustEvent)
```

사용자가 버튼을 탭할 때마다 로그에 `Event tracked`가 보일 것입니다.

이벤트 인스턴스를 사용하여 추적 시작 이전부터 이벤트 환경 설정을 할 수 있습니다.

#### <a id="revenue-tracking">매출 추적

사용자가 광고를 탭하거나 인앱 구매를 할 때 매출이 발생하는 경우, 해당 이벤트를 통해 매출을 추적할 수 있습니다. 탭 한 번 당 1센트의 수익이 난다고 가정해 봅시다. 매출을 추적하려면 아래와 같이 하면 됩니다. 

```js
var adjustEvent = new AdjustEvent('abc123')
adjustEvent.setRevenue(0.01, 'EUR')

Adjust.trackEvent(adjustEvent)
```

물론 콜백 파라미터와 결합할 수도 있습니다.

통화 토큰 설정 시, Adjust는 들어오는 매출을 선택한 방식에 따라 자동으로 변환해 줍니다. 통화 전환에 대한 자세한 내용은 [여기에서 확인하세요][currency-conversion]. 

매출 및 이벤트 추적에 대한 자세한 내용은 [이벤트 추적 설명서에서 확인하세요][event-tracking]. 

#### <a id="callback-parameters">콜백 파라미터

[대시보드] 내 이벤트에 사용할 콜백 URL을 등록할 수 있습니다. 이벤트를 추적할 때마다 해당 URL로 GET 요청을 전송합니다. 콜백 파라미터를 이벤트에 추가하려면 추적하기 전 해당 이벤트의 `addCallbackParameter` 메서드를 호출하면 됩니다. 그러면 이 파라미터를 콜백 URL에 덧붙이게 됩니다. 

예를 들어 이벤트 토큰 `abc123`에 해당하는 이벤트 추적을 위해 `http://www.adjust.com/callback`이라는 URL을 등록했다고 가정합시다. 그러면 아래와 같이 이벤트를 추적할 수 있습니다. 

```js
var adjustEvent = new AdjustEvent('abc123')
adjustEvent.addCallbackParameter('key', 'value')
adjustEvent.addCallbackParameter('foo', 'bar')

Adjust.trackEvent(adjustEvent)
```

이 경우, 이벤트를 추적하여 다음 URL로 요청을 보냅니다. 

    http://www.adjust.com/callback?key=value&foo=bar

Adjust는 파라미터 값으로 사용하는 `{idfa}` 등의 다양한 플레이스홀더(placeholder)를 지원합니다. 이 플레이스홀더는 결과 콜백 시에 현재 기기 광고자 ID로 대체됩니다. Adjust는 고객 파라미터를 저장하지 않으며 콜백에 덧붙일 뿐임을 기억해 주십시오. 이벤트 콜백을 등록하지 않았다면 이들 파라미터는 아예 읽히지 않습니다.  

가능한 전체 값 목록을 비롯하여 자세한 URL 콜백 사용법은 [콜백 설명서][callbacks-guide]에서 확인하세요.

#### <a id="partner-parameters">파트너 파라미터

네트워크 파트너에게 전송할 파라미터도 추가할 수 있습니다. Adjust 대시보드에서 활성화된 연동에서 사용합니다. 

이들 파라미터는 위에 설명한 콜백 파라미터와 비슷하게 작동하지만, `AdjustEvent` 인스턴스의 `addPartnerParameter` 메서드를 호출하여 추가합니다. 

```js
var adjustEvent = new AdjustEvent('abc123')
adjustEvent.addPartnerParameter('key', 'value')
adjustEvent.addPartnerParameter('foo', 'bar')

Adjust.trackEvent(adjustEvent)
```

특별 파트너 및 그 연동 방법에 대한 자세한 내용은 [특별 파트너 설명서][special-partners]에서 확인하세요. 

### <a id="attribution-callback">어트리뷰션 콜백

Adjust에서는 어트리뷰션에 변동이 생길 때 알려 주는 콜백을 등록할 수 있습니다. 어트리뷰션 대상으로 분류하는 소스가 각기 다르기 때문에 이 정보는 동시간에 제공할 수 없습니다. 

Adjust에서 [적용 가능한 어트리뷰션 데이터 정책][attribution-data]을 고려하는 걸 잊지 마세요. 

콜백 메서드 환경설정에는 `AdjustConfig` 인스턴스를 사용하므로, `Adjust.onCreate(adjustConfig)`를 호출하기 전에 먼저 `setAttributionCallback`을 호출해야 합니다. 

```js
adjustConfig.setAttributionCallback(function(attribution) {
    // In this example, we're just displaying alert with attribution content.
    alert('Tracker token = ' + attribution.trackerToken + '\n' +
          'Tracker name = ' + attribution.trackerName + '\n' +
          'Network = ' + attribution.network + '\n' +
          'Campaign = ' + attribution.campaign + '\n' +
          'Adgroup = ' + attribution.adgroup + '\n' +
          'Creative = ' + attribution.creative + '\n' +
          'Click label = ' + attribution.clickLabel)
})
```

SDK가 최종 어트리뷰션 데이터를 받고 나면 콜백 메서드가 촉발됩니다. 콜백 시에는 `attribution` 파라미터에 억세스할 수 있습니다. 각 특성을 간단히 요약하면 다음과 같습니다. 

- `var trackerToken` 현재 설치 트래커 토큰.
- `var trackerName` 현재 설치 트래커 이름.
- `var network` 현재 설치 네트워크 그루핑 수준.
- `var campaign` 현재 설치 캠페인 그루핑 수준.
- `var adgroup` 현재 설치 광고 그루핑 수준.
- `var creative` 현재 설치 크리에이티브 그루핑 수준.
- `var clickLabel` 현재 설치 클릭 수준.

### <a id="event-session-callbacks">이벤트 및 세션 콜백

Adjust에서는 이벤트나 세션 추적 시 알림을 받는 리스너(listner)를 등록할 수 있습니다. 리스너에는 네 가지가 있습니다. 성공 이벤트 추적용 리스너, 실패 이벤트 추적용 리스너, 성공 세션 추적용 리스너, 그리고 실패 세션 추적용 리스너입니다. 

다음 단계를 거쳐 성공 이벤트 콜백을 수행하세요. 

```js
adjustConfig.setEventSuccessCallback(function(eventSuccess) {
    // ...
})
```

실패 이벤트 추적 시 위임(delegate) 콜백은 다음과 같습니다. 

```js
adjustConfig.setEventFailureCallback(function(eventFailure) {
    // ...
})
```

성공 세션 추적은 다음과 같습니다.

```js
adjustConfig.setSessionSuccessCallback(function(sessionSuccess) {
    // ...
})
```

그리고 실패 세션 추적은 다음과 같습니다.

```js
adjustConfig.setSessionFailureCallback(function(sessionFailure) {
    // ...
})
```

SDK가 서버에 패키지 전송 시도 시 콜백 메서드가 호출됩니다. 콜백 메서드에서는 콜백용 응답 데이터 객체에 억세스할 수 있습니다. 세션 응답 데이터 특성을 간단히 요약하면 아래와 같습니다. 

- `var message` 서버로부터의 메시지 또는 SDK에 기록된 에러.
- `var timeStamp` 서버로부터의 타임 스탬프.
- `var adid` Adjust가 제공하는 고유 기기 식별자.
- `var jsonResponse` 서버로부터의 응답을 담은 JSON 객체.

이벤트 응답 데이터 객체 두 가지에 다 들어있는 속성은 아래와 같습니다.

- `var eventToken` 추적한 패키지가 이벤트일 경우의 이벤트 토큰.

실패 이벤트 및 세션 객체에는 다음 속성이 들어갑니다.

- `var willRetry` 패키지를 나중에 다시 전송 시도한다는 의미입니다.

### <a id="event-buffering">이벤트 버퍼링

앱이 이벤트 추적을 많이 사용할 때 일부 HTTP 요청을 지연시켜 매 분마다 배치(batch) 단위로 같이 전송하고자 할 경우가 있을 수 있습니다. `AdjustConfig` 인스턴스를 사용하면 이러한 이벤트 버퍼링 기능을 활성화할 수 있습니다

```js
adjustConfig.setEventBufferingEnabled(true)
```

### <a id="disable-tracking">추적 비활성화

`false` 파라미터를 지닌 `setEnabled`를 호출하여 Adjust SDK가 기기의 최근 활동을 추적하지 않도록 만들 수 있습니다. **이 설정은 세션과 세션 사이에서 기억됩니다**. 하지만 첫 번째 세션 후에만 적용할 수 있습니다. 

```js
Adjust.setEnabled(false)
```

<a id="is-enabled">">`isEnabled` 기능을 호출하여 Adjust SDK가 현재 사용 중인지 확인할 수 있습니다. 적용 파라미터를 `true`로 설정한 `setEnabled`를 불러 내면 Adjust SDK를 언제든 활성화할 수 있습니다. 

```js
Adjust.isEnabled(function(isEnabled) {
    if (isEnabled) {
        // SDK is enabled.    
    } else {
        // SDK is disabled.
    }
})
```

### <a id="offline-mode">오프라인 모드

추후에 전송하기 위해 추적한 데이터를 유지하면서 서버로의 전송을 잠시 미뤄 두기 위해 Adjust SDK를 오프라인 모드로 둘 수 있습니다. 오프라인 모드일 때는 모든 정보가 파일에 저장되므로 이벤트를 지나치게 많이 촉발시키지 않는 게 좋습니다.

`true` 파라미터를 지닌 `setOfflineMode`를 호출하면 오프라인 모드를 활성화할 수 있습니다. 

```js
Adjust.setOfflineMode(true)
```

이와 반대로, `false` 파라미터를 지닌 `setOfflineMode`를 호출하면 오프라인 모드를 비활성화할 수 있습니다. Adjust SDK가 온라인 모드로 돌아오면, 저장된 모든 정보가 정확한 시간 정보와 함께 서버로 전송됩니다.

추적 비활성화와 달리, 이 설정은 **세션과 세션 사이에서 기억되지 않습니다**. 이는 앱이 오프라인 모드로 꺼진다고 해도 SDK는 언제나 온라인 모드에서 시작한다는 뜻입니다.

### <a id="background-tracking">백그라운드 추적

Adjust SDK 기본값 설정에서는, 앱이 백그라운드에 있으면 HTTP 요청 전송이 잠시 중지됩니다. 이 설정은 `AdjustConfig` 인스턴스에서 바꿀 수 있습니다.
 
```js
adjustConfig.setSendInBackground(true)
```

### <a id="device-ids">기기 ID

Google Analytics 등 일부 서비스에서는 중복 보고를 막기 위해 기기 및 클라이언트 ID를 조정해야 합니다.

기기 Google 광고 식별자를 얻으려면 `getIdfa` 기능을 호출하면 됩니다. 

```js
Adjust.getIdfa(function(idfa) {
    // ...
});
```

### <a id="deeplinking">딥링크

Adjust SDK에서는 개인 설정된 URL 스킴을 통해 앱을 열도록 하기 위해 딥링크를 설정할 수 있습니다.

딥링크를 사용하여 리타겟팅 또는 리인게이지먼트 캠페인을 실시하려 할 경우, Adjust 캠페인의 특정 파라미터를 딥링크에 삽입해야 합니다. 딥링크를 사용하여 리타겟팅 또는 리인게이지먼트 캠페인을 실시하는 방법에 관해 더 자세히 알아보려면 [Adjust 공식 자료][reattribution-deeplinks] 를 참조하세요.

Project Navigator에서 앱 델리게이트의 소스 파일을 엽니다. `openURL` 및`application:continueUserActivity:restorationHandler:`을 찾거나 추가한 다음, WebView에 표시된 뷰 컨트롤러에 있는 `AdjustBridge` 레퍼런스에 추가하세요. 

```objc
#import "Adjust.h"
// Or #import <AdjustSdk/Adjust.h>
// (depends on the way you have chosen to add our native iOS SDK)
// ...

- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url
  sourceApplication:(NSString *)sourceApplication annotation:(id)annotation {
    // This is how AdjustBridge is accessed in our example app.
    // Of course, you can choose on your own how to access it.
    [self.uiWebViewExampleController.adjustBridge sendDeeplinkToWebView:url];
    
    // Your logic whether URL should be opened or not.
    BOOL shouldOpen = [self yourLogic:url];
    
    return shouldOpen;
}

// ...

- (BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity 
 restorationHandler:(void (^)(NSArray *restorableObjects))restorationHandler {
    if ([[userActivity activityType] isEqualToString:NSUserActivityTypeBrowsingWeb]) {
        [self.uiWebViewExampleController.adjustBridge sendDeeplinkToWebView:[userActivity webpageURL]];
    }

    // Your logic whether URL should be opened or not.
    BOOL shouldOpen = [self yourLogic:url];
    
    return shouldOpen;
}
```

이 호출을 양 쪽 메서드에 추가하면 (이전 개인 설정 URL 스킴을 사용하는) iOS 8 이하 버전 및 (`유니버설 링크`를 사용하는) iOS 9 이상 버전 모두에서 딥링크 리어트리뷰션을 지원할 수 있게 됩니다. 

**주의:** 앱에서 유니버설 링크를 활성화하려면 네이티브 iOS SDK README에서 [유니버설 링크 설명서][ios_sdk_ulinks]를 참조하십시오. 

딥링크 URL 정보를 WebView에 다시 읽어 들이려면 `deeplink`이라는 브릿지에 등록해야 합니다. 이 메서드는 사용자가 딥링크 정보가 담긴 트래커 URL을 클릭하여 앱이 열리면 Adjust SDK에 의해 촉발됩니다. 

```js
setupWebViewJavascriptBridge(function(bridge) {
    bridge.registerHandler('deeplink', function(data, responseCallback) {
        // In this example, we're just displaying alert with deeplink URL content.
        alert('Deeplink:\n' + data)
    })
})
```

#### <a id="deferred-deeplinking-callback">지연 딥링크 콜백

지연 딥링크가 열리기 전에 이를 알려줄 콜백 메서드를 등록하여 Adjust SDK가 이를 열 것인지 결정하도록 만들 수 있습니다.

이 콜백은 `AdjustConfig` 인스턴스에서도 설정할 수 있습니다.:

```js
adjustConfig.setDeferredDeeplinkCallback(function(deferredDeeplink) {
    // In this example, we're just displaying alert with deferred deeplink URL content.
    alert('Deferred deeplink:\n' + deferredDeeplink)
})
```

SDK가 서버로부터 지연 딥링크를 받으면 SDK가 이를 열려고 하기 전에 이 콜백 기능이 호출됩니다. 

`AdjustConfig` 인스턴스에서 설정하는 경우, `setOpenDeferredDeeplink` 메서드를 호출하면 Adjust SDK에 링크를 열거나 열지 않도록 전할 수 있습니다

```js
adjustConfig.setOpenDeferredDeeplink(true)
// Or if you don't want our SDK to open the link:
adjustConfig.setOpenDeferredDeeplink(false)
```

특별한 언급이 없을 시 Adjust SDK는 기본값으로 다음 링크를 엽니다.

[dashboard]:  http://adjust.com
[adjust.com]: http://adjust.com

[wvjsb_readme]:             https://github.com/marcuswestin/WebViewJavascriptBridge#usage
[ios_sdk_ulinks]:           https://github.com/adjust/ios_sdk/#universal-links
[callbacks-guide]:          https://docs.adjust.com/en/callbacks
[event-tracking]:           https://docs.adjust.com/en/event-tracking/#tracking-purchases-and-revenues
[attribution-data]:         https://github.com/adjust/sdks/blob/master/doc/attribution-data.md
[special-partners]:         https://docs.adjust.com/en/special-partners
[basic_integration]:        https://github.com/adjust/ios_sdk/#basic-integration
[web_view_js_bridge]:       https://github.com/marcuswestin/WebViewJavascriptBridge
[currency-conversion]:      https://docs.adjust.com/en/event-tracking/#tracking-purchases-in-different-currencies
[event-tracking-guide]:     https://docs.adjust.com/en/event-tracking/#reference-tracking-purchases-and-revenues
[reattribution-deeplinks]:  https://docs.adjust.com/en/deeplinking/#manually-appending-attribution-data-to-a-deep-link

[bridge_add]:             https://raw.githubusercontent.com/adjust/sdks/master/Resources/ios/bridge/bridge_add.png
[bridge_drag]:            https://raw.githubusercontent.com/adjust/sdks/master/Resources/ios/bridge/bridge_drag.png
[bridge_init_js]:         https://raw.githubusercontent.com/adjust/sdks/master/Resources/ios/bridge/bridge_init_js.png
[bridge_init_objc]:       https://raw.githubusercontent.com/adjust/sdks/master/Resources/ios/bridge/bridge_init_objc.png
[bridge_init_js_xcode]:   https://raw.githubusercontent.com/adjust/sdks/master/Resources/ios/bridge/bridge_init_js_xcode.png
[bridge_install_tracked]: https://raw.githubusercontent.com/adjust/sdks/master/Resources/ios/bridge/bridge_install_tracked.png

## <a id="license">라이선스

Adjust SDK는 MIT 허가서에 따라 라이선스를 얻었습니다.

Copyright (c) 2012-2016 adjust GmbH,
http://www.adjust.com

Permission is hereby granted, free of charge, to any person obtaining a copy of
this software and associated documentation files (the "Software"), to deal in
the Software without restriction, including without limitation the rights to
use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies
of the Software, and to permit persons to whom the Software is furnished to do
so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
