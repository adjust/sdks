## 개요

WebView를 사용하는 안드로이드 앱을 위한 adjust.com™의 Android SDK 지침입니다. adjust.com™에 관한 더 자세한 내용은 [adjust.com]에서 확인하실 수 있습니다.

## 목차

* [기본 연동기능](#basic-integration)
    * [기본(native) adjust Android SDK 추가](#native-add)
    * [프로젝트에 AdjustBridge 추가](#bridge-add)
    * [앱에 AdjustBridge 연동](#bridge-integrate-app)
    * [WebView에 AdjustBridge 연동](#bridge-integrate-web)
    * [AdjustBridge 로그 기록(logging)](#adjust-logging)
    * [앱 빌드](#build-the-app)
* [부가 기능](#additional-features)
    * [이벤트 추적](#event-tracking)
        * [수익 추적](#revenue-tracking)
        * [콜백 파라미터](#callback-parameters)
        * [파트너 파라미터](#partner-parameters)
    * [속성 콜백](#attribution-callback)
    * [이벤트 및 세션 콜백](#event-session-callbacks)
    * [이벤트 버퍼링](#event-buffering)
    * [추적 해제](#disable-tracking)
    * [오프라인 모드](#offline-mode)
    * [배경 추적](#background-tracking)
    * [기기 ID](#device-ids)
    * [딥링크](#deeplink)
        * [기본 딥링크 시나리오](#deeplinking-standard)
        * [거치(deferred) 딥링크 시나리오](#deeplinking-deferred)
        * [딥링크를 통한 리어트리뷰션(reattribution, 광고효과재분석)](#deeplinking-reattribution)
* [라이선스](#license)

## <a id="basic-integration"></a>기본 연동기능

### <a id="native-add">기본 adjust 안드로이드 SDK 추가

WebView에서 adjust SDK를 사용하려면 adjust의 기본 안드로이드 SDK를 앱에 추가해야 합니다. 기본 안드로이드 SDK를 인스톨하려면 [안드로이드 SDK README][android-sdk-basic-integration]에서 `기본 연동기능` 부분을 참고하십시오.

### <a id="bridge-add">프로젝트에 AdjustBridge 추가

AdjustBridge `.java` 파일을 프로젝트에 복사하세요. 선택한 패키지로 추가할 수 있습니다만, 소스 파일에서 adjust가 기본값으로 지정한 이름 붙이기 방식을 따르는 걸 권장합니다. 이렇게 하려면 프로젝트에 `com.adjust.sdk.bridge` 패키지를 생성한 다음 AdjustBridge `.java` 파일을 복사하면 됩니다.

AdjustBridge `.js` 파일을 프로젝트에 복사한 다음, `assets` 폴더에 추가하세요.

![][bridge_add]

### <a id="bridge-integrate-app"></a>앱에 AdjustBridge 연동

AdjustBridge 인스턴스를 올바로 초기화하려면, 인스턴스 
뿐만 아니라 HTML 페이지를 불러오는 `WebView` 객체에 `Application` 레퍼런스를 전달해야 합니다.

커스텀 `Application` 클라스를 정의하여 `onCreate` 메소드 안에 레퍼런스를 전달하는 게 좋습니다.

```java
public class GlobalApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();

        AdjustBridge.setApplicationContext(this);
    }
}
```

`WebView` 객체가 레퍼런스를 얻게 만든 다음 AdjustBridge로 전달하면 됩니다.

```java
public class MainActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        WebView webView = (WebView) findViewById(R.id.webView);
        AdjustBridge.setWebView(webView);

        // ...
    }
}
```

커스텀 `Application` 클라스를 추가하고 싶지 않다면 앱 레퍼런스를 전달해도 됩니다.

```java
public class MainActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        WebView webView = (WebView) findViewById(R.id.webView);
        
        AdjustBridge.setApplicationContext(getApplication());
        AdjustBridge.setWebView(webView);

        // ...
    }
}
```

이 단계를 마치면 AdjustBridge는 adjust 기본 SDK 및 WebView에 표시되는 페이지 간의 통신에 필요한 내용을 전부 갖추게 됩니다. 이제 WebView를 올바로 초기화하면 AdjustBridge 자바 인터페이스를 사용할 수 있습니다.

```java
public class MainActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        WebView webView = (WebView) findViewById(R.id.webView);
        
        AdjustBridge.setApplicationContext(getApplication());
        AdjustBridge.setWebView(webView);

        webView.getSettings().setJavaScriptEnabled(true);
        webView.setWebChromeClient(new WebChromeClient());
        webView.setWebViewClient(new WebViewClient());
        webView.addJavascriptInterface(AdjustBridge.getDefaultInstance(), "AdjustBridge");

        try {
            webView.loadUrl("file:///android_asset/AdjustExample-WebView.html");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

이걸 완료하면 AdjustBridge를 앱에 성공적으로 추가하여 자바스크립트 연결을 통해 adjust 기본 안드로이드 SDK와 WebView 간 통신이 가능하게 됩니다.

### <a id="bridge-integrate-web"></a>WebView에 AdjustBridge 연동

HTML 파일에서 adjust 자바스크립트 파일에 레퍼런스를 추가하세요.

```html
<script type="text/javascript" src="adjust.js"></script>
<script type="text/javascript" src="adjust_event.js"></script>
<script type="text/javascript" src="adjust_config.js"></script>
```

자바스크립트 파일에 레퍼런스를 추가하고 나면 HTML 파일에 사용하여 adjust SDK를 초기화할 수 있습니다.

```js
// ...

var yourAppToken = '{YourAppToken}'
var environment = AdjustConfig.EnvironmentSandbox
var adjustConfig = new AdjustConfig(AdjustBridge, yourAppToken, environment)

Adjust.onCreate(adjustConfig)

// ...
```

![][bridge_init_js_android]

`{YourAppToken}`을 해당 앱 토큰으로 바꾸면 됩니다. [대시보드]에서 찾을 수 있습니다.

테스트 빌드인지 제작 빌드인지 여부에 따라 `environment`를 다음 값 중 하나로 설정해야 합니다.

```js
var environment = AdjustConfig.EnvironmentSandbox
var environment = AdjustConfig.EnvironmentProduction
```

**주의:** 앱을 테스트하는 경우 이 값을 `AdjustConfig.EnvironmentSandbox`로 설정해야 합니다. 앱을 게시하기 전에는 `AdjustEnvironmentProduction`으로 바꾸는 걸 잊지 마세요. 개발 및 테스트 단계로 돌아가면 설정을 다시 `AdjustEnvironmentSandbox`에 맞춰야 합니다.

이는 실제 트래픽과 테스트 기기 상의 테스트 트래픽을 구분하기 위해서입니다. 언제나 그렇지만, 특히 수익 추적시에는 이 값을 유의미하게 두는 게 대단히 중요합니다.

### <a id="adjust-logging">AdjustBridge 로그 기록

`AdjustConfig`인스턴스에서 아래 파라미터 중 하나로 `setLogLevel`을 호출하여 테스트에서 확인하는 로그 기록 분량을 늘이거나 줄일 수 있습니다.

```objc
adjustConfig.setLogLevel(AdjustConfig.LogLevelVerbose) // enable all logging
adjustConfig.setLogLevel(AdjustConfig.LogLevelDebug)   // enable more logging
adjustConfig.setLogLevel(AdjustConfig.LogLevelInfo)    // the default
adjustConfig.setLogLevel(AdjustConfig.LogLevelWarn)    // disable info logging
adjustConfig.setLogLevel(AdjustConfig.LogLevelError)   // disable warnings as well
adjustConfig.setLogLevel(AdjustConfig.LogLevelAssert)  // disable errors as well
```

### <a id="build-the-app">앱 빌드

앱 빌드를 실행해 보세요. 빌드가 성공적이면 콘솔에서 SDK 로그를 주의 깊게 읽어보세요. 최초 런칭 후 정보 로그 시작 부분에 `Install tracked`가 보일 겁니다.

![][bridge_install_tracked]

## <a id="additional-features">부가 기능

Adjust SDK를 프로젝트에 연동하면 다음과 같은 기능을 이용할 수 있습니다.

### <a id="event-tracking">이벤트 추적


Adjust로 원하는 어떤 이벤트든 추적할 수 있습니다. 버튼을 탭할 때마다 이를 추적하고 싶다고 해봅시다. [대시보드]에 새 이벤트 토큰을 만듭니다. 편의상 토큰을 `abc123`이라고 합시다. 버튼 내 `onclick` 메소드에 아래와 같은 라인을 추가하면 탭 추적이 가능합니다.

```js
var adjustEvent = new AdjustEvent('abc123')
Adjust.trackEvent(adjustEvent)
```

유저가 버튼을 탭할 때마다 로그에 `Event tracked`가 보일 것입니다.

이벤트 인스턴스를 사용하여 추적 시작 이전부터 이벤트 환경 설정을 할 수 있습니다.

#### <a id="revenue-tracking">수익 추적

유저가 광고를 탭하거나 인앱 구매를 할 때 수익이 창출되는 경우, 해당 이벤트를 통해 수익을 추적할 수 있습니다. 탭 한 번 당 1센트의 수익이 난다고 가정해 봅시다. 그러면 아래와 같이 하면 됩니다.

```js
var adjustEvent = new AdjustEvent('abc123')
adjustEvent.setRevenue(0.01, 'EUR')

Adjust.trackEvent(adjustEvent)
```

물론 콜백 파라미터와 결합할 수도 있습니다.

통화 토큰 설정 시, Adjust는 들어오는 수익을 선택한 방식에 따라 자동으로 보고 수익으로 바꿔줍니다. 통화 전환에 대한 자세한 내용은 [여기서 확인하세요][currency-conversion].

수익 및 이벤트 추적에 대한 자세한 내용은 [이벤트 추적 지침에서 확인하세요][event-tracking].

#### <a id="callback-parameters">콜백 파라미터

[대시보드] 내 이벤트에 사용할 콜백 URL을 등록할 수 있습니다. 이벤트를 추적할 때마다 해당 URL로 GET 요청을 전송합니다. 콜백 파라미터를 이벤트에 추가하시려면 추적하기 전 해당 이벤트의 `addCallbackParameter` 메소드를 호출하면 됩니다. 그러면 이 파라미터를 콜백 URL에 덧붙이게 됩니다.

예를 들어 이벤트 토큰 `abc123`에 해당하는 이벤트 추적을 위해 `http://www.adjust.com/callback`라는 URL을 등록했다고 가정합시다. 그러면 다음 라인을 실행하면 됩니다.

```js
var adjustEvent = new AdjustEvent('abc123')
adjustEvent.addCallbackParameter('key', 'value')
adjustEvent.addCallbackParameter('foo', 'bar')

Adjust.trackEvent(adjustEvent)
```

이 경우, 이벤트를 추적하여 다음 URL로 요청을 보냅니다.

    http://www.adjust.com/callback?key=value&foo=bar

Adjust는 파라미터 값으로 사용하는 `{gps_adid}` 등의 다양한 플레이스홀더(placeholder)를 지원합니다. 이 플레이스홀더는 결과 콜백 시에 현재 기기 Google Play Services ID로 대체됩니다. Adjust는 고객 파라미터를 저장하지 않으며 콜백에 덧붙일 뿐임을 기억해 주십시오. 이벤트 콜백을 등록하지 않으셨다면 이들 파라미터는 아예 읽히지 않습니다. 

가능한 전체 값 목록을 비롯하여 자세한 URL 콜백 사용법은 [콜백 지침][callbacks-guide]에서 확인하세요.

#### <a id="partner-parameters">파트너 파라미터

네트워크 파트너에게 전송할 파라미터도 추가할 수 있습니다. 현재 Adjust 대시보드에서 활성화된 연동 부분에서 사용합니다.

이들 파라미터는 위에 설명한 콜백 파라미터와 비슷하게 작동하지만, `AdjustEvent` 인스턴스의 `addPartnerParameter` 메소드를 호출하여 추가합니다.

```js
var adjustEvent = new AdjustEvent('abc123')
adjustEvent.addPartnerParameter('key', 'value')
adjustEvent.addPartnerParameter('foo', 'bar')

Adjust.trackEvent(adjustEvent)
```

특별 파트너 및 그 연동 방법에 대한 자세한 내용은 [특별 파트너 지침][special-partners]에서 확인하세요.

### <a id="attribution-callback">속성 콜백

Adjust는 속성을 어떻게 변화시켜도 콜백을 보낼 수 있습니다. 속성에서 고려하는 소스가 각각 다르기 때문에, 이 정보는 동시간에 제공할 수 없습니다.

[적용 가능한 속성 데이터 정책][attribution-data]을 고려하는 걸 잊지 마세요.

콜백 메소드 환경설정에는 `AdjustConfig` 인스턴스를 사용하므로, `Adjust.onCreate(adjustConfig)`를 호출하기 전에 먼저 `setAttributionCallback`을 호출해야 합니다.

```js
function attributionCallback(attribution) {
    alert('Tracker token = ' + attribution.trackerToken + '\n' +
          'Tracker name = ' + attribution.trackerName + '\n' +
          'Network = ' + attribution.network + '\n' +
          'Campaign = ' + attribution.campaign + '\n' +
          'Adgroup = ' + attribution.adgroup + '\n' +
          'Creative = ' + attribution.creative + '\n' +
          'Click label = ' + attribution.clickLabel)
}

// ...

var yourAppToken = '{YourAppToken}'
var environment = AdjustConfig.EnvironmentSandbox
var adjustConfig = new AdjustConfig(AdjustBridge, yourAppToken, environment)

adjustConfig.setAttributionCallback(attributionCallback)

Adjust.onCreate(adjustConfig)

// ...
```

SDK가 최종 속성 데이터를 받고 나면 콜백 기능이 호출됩니다. 콜백 기능이 작동하는 동안 `attribution` 파라미터에 억세스할 수 있습니다. 각 특성을 간단히 요약하면 다음과 같습니다.

- `var trackerToken` 현재 인스톨 트래커 토큰.
- `var trackerName` 현재 인스톨 트래커 이름.
- `var network` 현재 인스톨 네트워크 그루핑 레벨.
- `var campaign` 현재 인스톨 캠페인 그루핑 레벨.
- `var adgroup` 현재 인스톨 광고 그루핑 레벨.
- `var creative` 현재 인스톨 크리에이티브 그루핑 레벨.
- `var clickLabel` 현재 인스톨 클릭 레벨.

### <a id="event-session-callbacks">이벤트 및 세션 콜백

이벤트나 세션 추적 시 알림을 받는 리스너(listner)를 등록할 수 있습니다. 리스너에는 네 가지가 있습니다 - 성공 이벤트 추적용 리스너, 실패 이벤트 추적용 리스너, 성공 세션 추적용 리스너, 그리고 실패 세션 추적용 리스너입니다.

다음 단계를 거쳐 이벤트 성공 콜백 기능을 수행하세요.

```js
function eventSuccessCallback(eventSuccess) {
    alert('Message = ' + eventSuccess.message + '\n' +
          'Timestamp = ' + eventSuccess.timestamp + '\n' +
          'Adid = ' + eventSuccess.adid + '\n' +
          'Event token = ' + eventSuccess.eventToken)
}

// ...

adjustConfig.setEventSuccessCallback(eventSuccessCallback)
```

이벤트 실패 추적 시 델리케이트 콜백 기능은 다음과 같습니다.

```js
function eventFailureCallback(eventFailure) {
    alert('Message = ' + eventFailure.message + '\n' +
          'Timestamp = ' + eventFailure.timestamp + '\n' +
          'Adid = ' + eventFailure.adid + '\n' +
          'Event token = ' + eventFailure.eventToken + '\n' +
          'Will retry = ' + eventFailure.willRetry)
}

adjustConfig.setEventFailureCallback(eventFailureCallback)
```

세션 성공 추적은 아래와 같이 합니다.

```js
function sessionSuccessCallback(sessionSuccess) {
    alert('Message = ' + sessionSuccess.message + '\n' +
          'Timestamp = ' + sessionSuccess.timestamp + '\n' +
          'Adid = ' + sessionSuccess.adid)
}

adjustConfig.setSessionSuccessCallback(sessionSuccessCallback)
```

세션 실패 추적은 다음과 같습니다.

```js
function sessionFailureCallback(sessionFailure) {
    alert('Message = ' + sessionFailure.message + '\n' +
          'Timestamp = ' + sessionFailure.timestamp + '\n' +
          'Adid = ' + sessionFailure.adid + '\n' +
          'Will retry = ' + sessionFailure.willRetry)
}

adjustConfig.setSessionFailureCallback(sessionFailureCallback)
```

SDK가 서버에 패키지 전송 시도 시 콜백 메소드가 호출됩니다. 콜백 기능이 작동하는 동안 콜백용 응답 데이터 객체에 억세스할 수 있습니다. 세션 응답 데이터 특성을 간단히 요약하면 아래와 같습니다.

- `var message` 서버로부터의 메시지 또는 SDK에 기록된 에러.
- `var timeStamp` 서버로부터의 시간 스탬프.
- `var adid` adjust가 제공하는 고유 기기 식별자.
- `var jsonResponse` 서버로부터의 응답을 담은 JSON 객체.

이벤트 응답 데이터 객체 두 가지에 다 들어있는 특성은 아래와 같습니다.

- `var eventToken` 추적한 패키지가 이벤트일 경우 이벤트 토큰.

이벤트 및 세션 객체 실패 시에는 다음 특성이 들어갑니다.

- `var willRetry` 패키지를 나중에 다시 전송 시도한다는 의미입니다.

### <a id="disable-tracking">추적 해제

적용 파라미터를 `false`로 설정한 `setEnabled`를 호출하여 adjust SDK를 추적 해제할 수 있습니다. **이 설정은 세션과 세션 사이에서 기억됩니다**. 하지만 첫 번째 세션 후에만 적용할 수 있습니다.

```js
Adjust.setEnabled(false)
```

<a id="is-enabled">`isEnabled` 기능을 호출하여 adjust SDK가 현재 사용중인 지 검증할 수 있습니다. 적용 파라미터를 `true`로 설정한 `setEnabled`를 인보크하여 adjust SDK를 언제든 활성화할 수 있습니다.

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

추적한 데이터를 추후 전송하기 위해 유지하면서 서버로의 전송을 연기하기 위해 adjust SDK를 오프라인 모드로 둘 수 있습니다. 오프라인 모드일 때는 모든 정보가 파일에 저장되므로 이벤트를 지나치게 많이 트리거(trigger)하지 않는 게 좋습니다.

파라미터가 `true`로 설정된 `setOfflineMode`를 호출하여 오프라인 모드를 활성화할 수 있습니다.

```js
Adjust.setOfflineMode(true)
```

반대로, 파라미터가 `false`로 설정된 `setOfflineMode`를 호출하면 오프라인 모드를 비활성화할 수 있습니다. adjust SDK가 온라인 모드로 돌아오면, 저장된 모든 정보가 올바른 시간 정보와 함께 서버로 전송됩니다.

추적 해체와 달리, 이 설정은 **세션과 세션 사이에서 기억되지 않습니다**. 이는 앱이 오프라인 모드로 꺼진다고 해도 SDK는 언제나 온라인 모드에서 시작한다는 뜻입니다.

### <a id="event-buffering">이벤트 버퍼링

앱이 이벤트 추적을 많이 사용하는 경우, 매 분마다 배치 하나씩만 보내도록 하기 위해 일부 HTTP 요청을 지연시키고자 할 경우가 있을 수 있습니다. `AdjustConfig` 인스턴스에서 이벤트 버퍼링을 적용할 수 있습니다.

```js
adjustConfig.setEventBufferingEnabled(true)
```

### <a id="background-tracking">배경 추적

Adjust SDK 기본값 행위는 앱이 배경에 있을 동안에는 HTTP 요청 전송을 잠시 중지하는 것입니다. `AdjustConfig` 인스턴스에서 이를 바꿀 수 있습니다.

```js
adjustConfig.setSendInBackground(true)
```

### <a id="device-ids">기기 ID

(Google Analytics 등) 일부 기기의 경우 중복 보고를 막기 위해 기기 및 클라이언트 ID를 조정해야 합니다.

기기 Google 광고 식별자를 얻으려면 `getGoogleAdId` 기능을 호출하면 됩니다.

```js
Adjust.getGoogleAdId(function(googleAdId) {
    // ...
});
```

### <a id="deeplink">딥링크

딥링크 메커니즘에 관해 더 자세히 읽어보려면 adjust 공식 [안드로이드 SDK README][android-readme-deeplinking]를 참조하십시오.

WebView에 딥링크 정보를 포함시키려면 기본 안드로이드 앱에서 호출을 해야 합니다. 공식 SDK README 내 딥링크 관련 섹션에서 설명한 바와 같이, 딥링크 내용은 액티비티 생애주기 (activity lifecycle) 메소드 한두개로 전달할 수 있습니다. WebView 페이지로 딥링크 정보를 전달하려면 적당한 메소드를 오버라이딩한 다음 `AdjustBridge.deeplinkReceived` 메소드로의 호출을 추가하면 됩니다.

```java
@Override
protected void onNewIntent(Intent intent) {
    super.onNewIntent(intent);

    Uri data = intent.getData();
    AdjustBridge.deeplinkReceived(data);
}
```

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    WebView webView = (WebView) findViewById(R.id.webView);
    
    AdjustBridge.setApplicationContext(getApplication());
    AdjustBridge.setWebView(webView);

    webView.getSettings().setJavaScriptEnabled(true);
    webView.setWebChromeClient(new WebChromeClient());
    webView.setWebViewClient(new WebViewClient());
    webView.addJavascriptInterface(AdjustBridge.getDefaultInstance(), "AdjustBridge");

    try {
        webView.loadUrl("file:///android_asset/AdjustExample-WebView.html");

        Intent intent = getIntent();
        Uri data = intent.getData();

        AdjustBridge.deeplinkReceived(data);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

#### <a id="deeplinking-standard">기본 딥링크 시나리오

딥링크 URL 정보를 WebView로 전달하려면, `adjust_deeplink`라는 콜백 메소드를 HTML 스크립트에 등록해야 합니다. 딥링크 정보가 들어 있는 트래커 URL을 클릭하여  앱이 열리면, 이 메소드가 adjust SDK에 의해 촉발됩니다.

```js
function adjust_deeplink(deeplink) {
    alert('Deeplink content:\n' + deeplink)
}
```

#### <a id="deeplinking-deferred">디퍼 딥링크 시나리오

디퍼드 딥링크가 열려 adjust SDK가 이를 열 것인지 결정하기 전에 알림을 얻으려면 콜백 메소드를 등록하면 됩니다.

이 콜백은 `AdjustConfig` 인스턴스에서도 설정됩니다.

```js
adjustConfig.setDeferredDeeplinkCallback(function(deferredDeeplink) {
    // In this example, we're just displaying alert with deferred deeplink URL content.
    alert('Deferred deeplink:\n' + deferredDeeplink)
})
```

SDK가 서버로부터 디퍼드 딥링크를 받아 이를 열기 전에 콜백 기능이 호출됩니다.

`AdjustConfig` 인스턴스에서 설정을 다르게 하면, SDK에 이 링크를 열 것인지 아닌지를 알릴 수 있습니다. `setOpenDeferredDeeplink` 메소드를 호출하면 됩니다.

```js
adjustConfig.setOpenDeferredDeeplink(true)
// Or if you don't want our SDK to open the link:
adjustConfig.setOpenDeferredDeeplink(false)
```

기본값으로 특정한 설정을 하지 않았다면, adjust SDK는 링크를 열게 됩니다.

#### <a id="deeplinking-reattribution">딥링크를 통한 리어트리뷰션

AdjustBridge로, 딥링크 사용에 따른 유저 리어트리뷰션이 별도의 설치나 구성 노력을 할 필요 없이 바로 지원됩니다. 이 주제에 관해 더 자세한 정보는 [공식 안드로이드 SDK README][android-sdk-reattribution]에서 확인하세요.

[dashboard]:   http://adjust.com
[adjust.com]:  http://adjust.com

[callbacks-guide]:               https://docs.adjust.com/en/callbacks
[special-partners]:              https://docs.adjust.com/en/special-partners
[event-tracking-guide]:          https://docs.adjust.com/en/event-tracking
[android-sdk-deeplinking]:       https://github.com/adjust/android_sdk/blob/master/README.md#deeplinking
[android-sdk-reattribution]:     https://github.com/adjust/android_sdk#deeplinking-reattribution
[android-sdk-basic-integration]: https://github.com/adjust/android_sdk/blob/master/README.md#basic-integration

[bridge_add]:              https://raw.githubusercontent.com/adjust/sdks/master/Resources/android/bridge/bridge_add.png
[bridge_init_js_android]:  https://raw.githubusercontent.com/adjust/sdks/master/Resources/android/bridge/bridge_init_js_android.png
[bridge_install_tracked]:  https://raw.githubusercontent.com/adjust/sdks/master/Resources/android/bridge/bridge_install_tracked.png

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
