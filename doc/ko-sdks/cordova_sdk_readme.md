## 요약

이 항목에서는 Adjust의 Cordova SDK에 대해 설명합니다. Adjust에 대한 자세한 내용은 [adjust.com]을 참조하십시오.

참고: 현재 Cordova용 SDK 4.3.0에서는 Android 플랫폼 버전 `4.0.0` 이상과 iOS 플랫폼 버전 `3.0.0` 이상을 지원합니다.

## 목차

* [앱 예제](#example-app)
* [기본 연동](#basic-integration)
   * [SDK 다운로드 및 설치](#sdk-get)
* [앱에 SDK 연동](#sdk-integrate)
   * [Adjust 로그](#adjust-logging)
   * [Adjust 프로젝트 설정](#adjust-project-settings)
     * [안드로이드 권한](#android-permissions)
     * [Google Play Services](#android-gps)
     * [안드로이드 설치 참조 페이지](#android-broadcast-receiver)
     * [iOS 프레임워크](#android-permissions)
* [부가 기능](#additional-features)
   * [이벤트 추적](#event-tracking)
     * [수익 추적](#revenue-tracking)
     * [수익 중복 제거](#revenue-deduplication)
     * [인앱 구매 검증](#iap-verification)
     * [콜백 매개변수](#callback-parameters)
     * [파트너 매개변수](#partner-parameters)
   * [세션 매개변수](#session-parameters)
     * [세션 콜백 매개변수](#session-callback-parameters)      
     * [세션 파트너 매개변수](#session-partner-parameters)
     * [예약 시작(delay start)](#delay-start)
   * [속성 콜백](#attribution-callback)
   * [세션 및 이벤트 콜백](#session-event-callbacks)
   * [추적 사용 중지](#disable-tracking)
   * [오프라인 모드](#offline-mode)
   * [이벤트 버퍼링](#event-buffering)
   * [배경 추적](#background-tracking)
   * [기기 ID](#device-ids)
     * [iOS 광고 식별자](#di-idfa)
     * [Google Play Service 광고 식별자](#di-gps-adid)
     * [Adjust 기기 식별자](#di-adid)
   * [사용자 속성](#user-attribution)
   * [푸시 토큰(push token)](#push-token)
   * [사전 설치 트래커(pre-installed trackers)](#pre-installed-trackers)
   * [딥링크](#deeplinking)
     * [기본 딥링크](#deeplinking-standard)
     * [Android & iOS 8 이하 버전 딥링크](#deeplinking-android-ios-old)
     * [iOS 9 이상 버전 딥링크](#deeplinking-ios-new)
     * [거치(deferred) 딥링크](#deeplinking-deferred)
     * [딥링크를 통한 재어트리뷰션](#deeplinking-reattribution)
* [라이선스](#license)

## <a id="example-app"></a>앱 예제 

Adjust SDK를 앱에 연동하는 방법의 예시를 [`예제` 디렉토리][example]에서 확인할 수 있습니다. 크기 문제 때문에 업로드된 앱 예제에는 플랫폼이 붙어있지 않습니다. 따라서 앱을 다운로드한 다음에는 원하는 플랫폼에서 앱을 만들 수 있도록 ‘script’ 폴더에서 알맞은 스크립트를 구동해 주시기 바랍니다.

```
sh cordova-test-ios.sh
sh cordova-test-android.sh
```

**참고**: iOS 플랫폼에서 앱 예제를 처음 만들어 본 다음에는 코드 사인 오류가 발생하므로 앱을Xcode 프로젝트에서 열어 설정 내 ‘Signing’ 부분에서 ‘Team’을 선택해야 한다는 사실을 기억해 주십시오. 이 작업이 끝나면 Xcode에서 앱을 구동하거나 빌드 스크립트를 재구동하세요.  

## <a id="basic-integration">기본 설치

다음은 Adjust SDK를 Cordova 프로젝트와 연동하기 위해 최소한으로 수행해야 하는 절차입니다.

### <a id="sdk-get">SDK 다운로드 및 설치

`npm` [리포지토리][npm-repo]나 [릴리스 페이지][releases]에서 최신 버전을 다운로드합니다. 

### <a id="sdk-add">프로젝트에 SDK 추가

Adjust SDK를 `npm` [리포지토리][npm-repo]에서 직접 플러그인으로 다운로드할 수 있습니다. 이렇게 하려면 프로젝트 폴더에서 다음 명령을 실행합니다.

```
> cordova plugin add com.adjust.sdk
Fetching plugin "com.adjust.sdk" via npm
Installing "com.adjust.sdk" for android
Installing "com.adjust.sdk" for ios
```

릴리스 페이지에서 다운로드할 경우에는 압축파일을 원하는 폴더에 푼 후 다음 명령을 프로젝트 폴더에 입력합니다.

```
> cordova plugin add path_to_folder/cordova_sdk
Installing "com.adjust.sdk" for android
Installing "com.adjust.sdk" for ios
```

<a id="sdk-cocoon">**참고:** **Adjust SDK v4.10.2**에서 시작할 경우, `npm` 플러그인과 `master` 브랜치는 `cocoon.io`와 호환이 가능합니다. 따라서 SDK를 반드시 `cocoon` 브랜치에서 사용하지 않아도 됩니다.

### <a id="sdk-integrate">앱에 SDK 연동

Adjust 플러그인은 Cordova 이벤트 `deviceready`, `resume` 및 `pause`에 자동으로 등록됩니다.

`deviceready` 이벤트를 수신한 후 `index.js` 파일에 다음 코드를 추가하여 Adjust SDK를 초기화합니다.

```js
var adjustConfig = new AdjustConfig("{YourAppToken}", AdjustConfig.EnvironmentSandbox);

Adjust.create(adjustConfig);
```

`{YourAppToken}`을 앱 토큰으로 변경합니다. 앱 토큰은 [대시보드]에서 찾을 수 있습니다.

앱을 테스트에 사용할지 아니면 프로덕션에 사용할 지에 따라 `environment`를 다음 값 중 하나로 설정해야 합니다.

```javascript
AdjustConfig.EnvironmentSandbox
AdjustConfig.EnvironmentProduction
```

**중요:** 이 값은 앱을 테스트하는 경우에만 `AdjustConfig.EnvironmentSandbox`로 설정해야 합니다. 앱을 게시하기 전에는 environment를 `AdjustConfig.EnvironmentProduction`으로 설정해야 합니다. 개발 및 테스트를 다시 시작할 때 `AdjustConfig.EnvironmentSandbox`로 되돌려 놓으십시오.

이 environment는 실제 트래픽과 테스트 장치의 테스트 트래픽을 구별하기 위해 사용합니다. 이 값은 항상 의미 있게 유지해야 합니다! 수익을 추적할 때 특히 중요합니다.

### <a id="sdk-logging">Adjust 로그

다음 매개변수 중 하나를 사용하여 `AdjustConfig` 인스턴스에서 `setLogLevel`을 호출하면 테스트에 표시되는 로그의 양을 늘리거나 줄일 수 있습니다.

```js
adjustConfig.setLogLevel(AdjustConfig.LogLevelVerbose);   // 모든 로그 활성
adjustConfig.setLogLevel(AdjustConfig.LogLevelDebug);     // 더 많은 로그 활성
adjustConfig.setLogLevel(AdjustConfig.LogLevelInfo);      // 기본값
adjustConfig.setLogLevel(AdjustConfig.LogLevelWarn);      // info 로그 비활성
adjustConfig.setLogLevel(AdjustConfig.LogLevelError);     // 경고 역시 비활성
adjustConfig.setLogLevel(AdjustConfig.LogLevelAssert);    // 오류 역시 비활성
adjustConfig.setLogLevel(AdjustConfig.LogLevelSuppress);  // 모든 로그 비활성
```

### <a id="adjust-project-settings">Adjust 프로젝트 설정

Adjust SDK를 앱에 추가하고 나면 Adjust SDK가 제대로 작동할 수 있도록 몇 가지 수정 작업이 수행됩니다. 이 과정에서 수행하는 모든 작업은 Adjust SDK 플러그인 내 `plugin.xml` 파일에 기록됩니다. Adjust SDK를 앱에 추가한 후 추가로 수행하는 작업을 아래에 설명해 놓았습니다.

#### <a id="android-permissions">안드로이드 권한

Adjust SDK는 안드로이드 매니페스트(manifest) 파일에 `INTERNET`과 `ACCESS_WIFI_STATE`이라는 두 가지 권한을 추가합니다. 이 설정은 Adjust SDK 플러그인 내 `plugin.xml` 파일에서 확인할 수 있습니다.

```xml
<config-file target="AndroidManifest.xml" parent="/manifest">
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
</config-file>
```

`INTERNET` 권한은 SDK가 언제든 필요로 할 가능성이 있습니다. `ACCESS_WIFI_STATE` 권한은 앱이 Google Play Store를 대상으로 하지 않으며 Google Play Services를 사용하지 않을 때 필요합니다. Google Play Store를 대상으로 하며 Google Play Services를 사용하는 경우 Adjust SDK는 이 권한을 필요로 하지 않으며, 앱에서 필요하지 않다면 제거할 수 있습니다. 

#### <a id="android-gps"Google Play Services

2014년 8월 1일 이후로 Google Play Store에 있는 앱은 [Google 광고 ID][google_ad_id]를 사용하여 장치를 고유하게 식별해야 합니다. Adjust SDK에서 Google 광고 ID를 사용할 수 있게 하려면 [Google Play Services][google_play_services]를 연동해야 합니다.

Adjust SDK 플러그인은 Google Play Services를 앱에 기본적으로 추가합니다. `plugin.xml` 파일 속에 있는 아래 라인이 수행합니다.

```xml
<framework src="com.google.android.gms:play-services-analytics:+" />
```

다른 Cordova 플러그인을 사용할 때도 Google Play Services를 앱에 기본으로 들여오는 경우가 있습니다. 이 때 Adjust SDK 및 다른 플러그인이 Google Play Services와 충돌하여 빌드 타임 오류를 일으킬 수 있습니다. Google Play Services가 Adjust SDK의 일부로 반드시 앱에 있어야 하는 건 아닙니다. **Google Play Services 라이브러리의 애널리틱스 부분이 앱에 연동되어 있기만 하면** Adjust SDK가 필요한 정보를 읽어들일 수 있습니다. Google Play Services를 다른 Cordova 플러그인의 일부로 앱에 추가하기로 했다면, 위의 해당 라인을 Adjust SDK 내 `plugin.xml` 파일에서 제거하면 됩니다.

Google Play Services 라이브러리의 애널리틱스가 제대로 앱에 추가되어 Adjust SDK가 올바로 읽어들일 수 있는 상태인지 확인하려면, SDK를 ‘sandbox’ 모드에 놓은 채로 앱을 작동하여 로그를 ‘verbose’ 레벨에 놓아야 합니다. 그런 다음 앱에서 세션이나 이벤트를 추적하여 이 때 verbose 로그에 등장하는 매개변수 목록을 살펴봅니다. `gps_adid`라는 매개변수가 보인다면, 앱에 Google Play Services 라이브러리의 애널리틱스를 제대로 추가하여 SDK가 필요한 정보를 읽어들이고 있다는 뜻입니다.

#### <a id="android-proguard">Proguard 설정

Proguard를 사용 중인 경우 다음 행을 Proguard 파일에 추가하십시오.

```
-keep public class com.adjust.sdk.** { *; }
-keep class com.google.android.gms.common.ConnectionResult {
    int SUCCESS;
}
-keep class com.google.android.gms.ads.identifier.AdvertisingIdClient {
    com.google.android.gms.ads.identifier.AdvertisingIdClient$Info getAdvertisingIdInfo(android.content.Context);
}
-keep class com.google.android.gms.ads.identifier.AdvertisingIdClient$Info {
    java.lang.String getId();
    boolean isLimitAdTrackingEnabled();
}
-keep class dalvik.system.VMRuntime {
    java.lang.String getRuntime();
}
-keep class android.os.Build {
    java.lang.String[] SUPPORTED_ABIS;
    java.lang.String CPU_ABI;
}
-keep class android.content.res.Configuration {
    android.os.LocaledList getLocales();
    java.util.Locale locale;
}
-keep class android.os.LocaledList {
    java.util.Locale get(int);
}
```

#### <a id="android-broadcast-receiver">안드로이드 설치 참조 수신기

Adjust 설치 참조 수신기는 앱에 기본으로 추가됩니다. 더 자세한 정보는 기본 [안드로이드 SDK README][broadcast-receiver]를 참고하십시오. 이 설정은 Adjust SDK 플러그인 내 `plugin.xml` 파일에서 찾아볼 수 있습니다.

```xml
<config-file target="AndroidManifest.xml" parent="/manifest/application">
    <receiver
        android:name="com.adjust.sdk.AdjustReferrerReceiver"
        android:exported="true">
        <intent-filter>
            <action android:name="com.android.vending.INSTALL_REFERRER" />
        </intent-filter>
    </receiver>
</config-file>
```

사용자가 INSTALL_REFERRER 인텐트를 사용하는 자신의 수신기를 가지고 있을 경우 Adjust 브로드캐스트 수신기를 매니페스트 파일에 추가하지 않아도 된다는 사실을 기억해 주십시오. 제거해도 상관없습니다. 그러나 [안드로이드 설명서][broadcast-receiver-custom]에 나온 대로 Adjust 브로드캐스트 수신기에 대한 호출을 자신의 수신기에 추가해 주십시오.

#### <a id="ios-frameworks">iOS 프레임워크

Adjust SDK 플러그인은 생성된 Xcode 프로젝트에 3개의 iOS 프레임워크를 추가합니다.

* `iAd.framework` - iAd 캠페인을 구동하는 경우
* `AdSupport.framework` - iOS 광고 Id (IDFA) 인식용
* `AdjustSdk.framework` - 기본 iOS SDK 프레임워크

이 설정은 Adjust SDK 플러그인 내 `plugin.xml` 파일에서 찾아볼 수 있습니다.

```xml
<framework src="src/iOS/AdjustSdk.framework" custom="true" />
<framework src="AdSupport.framework" weak="true" />
<framework src="iAd.framework" weak="true" />
```

iAd 캠페인을 구동하지 않는 경우 `iAd.framework` 의존성을 임의로 제거해도 됩니다.

## <a id="additional-features">추가 기능

adjust SDK를 프로젝트와 연동한 후에는 다음 기능을 사용할 수 있습니다.

### <a id="event-tracking">이벤트 추적

Adjust를 사용하여 앱의 모든 이벤트를 추적할 수 있습니다. 버튼의 모든 탭을 추적하려면 [대시보드]에서 새 이벤트 토큰을 만들어야 합니다. 예를 들어 이벤트 토큰이 `abc123`일 경우, 버튼의 클릭 핸들러 메소드에 다음 라인을 추가하여 클릭을 추적할 수 있습니다.

```js
var adjustEvent = new AdjustEvent("abc123");
Adjust.trackEvent(adjustEvent);
```

#### <a id="revenue-tracking">수익 추적

사용자가 광고를 누르거나 인앱 구매를 통해 매출을 발생시킬 수 있는 경우 이벤트를 사용하여 해당 매출을 추적할 수 있습니다. 한 번 누를 때 0.01 유로의 매출이 발생한다고 가정할 경우 수익 이벤트는 다음과 같이 추적하면 됩니다.

```js
var adjustEvent = new AdjustEvent("abc123");

adjustEvent.setRevenue(0.01, "EUR");

Adjust.trackEvent(adjustEvent);
```

통화 토큰을 설정하면 adjust에 수신되는 매출이 선택한 보고 매출으로 자동 변환됩니다. 통화 변환에 대한 [자세한 내용은 여기][currency-conversion]를 참조하십시오.

#### <a id="revenue-deduplication"></a>수익 중복 제거

거래 ID를 선택 사항으로 추가하여 수익 중복 추적을 피할 수 있습니다. 가장 최근에 사용한 거래 ID 10개를 기억하며, 중복 거래 ID로 이루어진 수익 이벤트는 집계하지 않습니다. 인앱 구매 추적 시 특히 유용합니다. 사용 예는 아래에 나와 있습니다.

인앱 구매를 추적할 경우, 거래가 완료되고 아이템을 구매했을 때만 `trackEvent`를 호출해야 한다는 사실을 기억하십시오. 그렇게 해야 실제로 발생하지 않은 수익을 추적하는 일을 피할 수 있습니다.

```js
var adjustEvent = new AdjustEvent("abc123");

adjustEvent.setRevenue(0.01, "EUR");
adjustEvent.setTransactionId("{YourTransactionId}");

Adjust.trackEvent(adjustEvent);
```

**주의**: 거래 ID는 iOS 용어이며, 성공적으로 완결된 안드로이드 인앱 구매 용 고유 식별자는 **주문 ID(Order ID)**입니다.

#### <a id="iap-verification">인앱 구매 검증

인앱 구매의 유효성을 확인하려면 Adjust의 서버 측 수신 확인 도구인 구매 검증(Purchase Verification)을 사용해 보세요. Cordova 구매 SDK에 대한 자세한 사항은 [여기][cordova-purchase-sdk]에서 확인할 수 있습니다.

#### <a id="callback-parameters">콜백 매개변수

You can also register a callback URL for that event in your [dashboard][dashboard] and we will send a GET request to that URL whenever the event gets tracked. In that case you can also put some key-value pairs in an object and pass it to the `trackEvent` method. We will then append these named parameters to your callback URL.
[대시보드][dashboard]에서 이벤트 콜백 URL을 등록할 수 있습니다. 이벤트를 추적할 때마다 GET 요청이 해당 URL로 전송됩니다. 키-값 쌍을 오브젝트에 삽입하여 `trackEvent` 메소드로 전달할 수도 있게 됩니다. 그러면 해당 매개변수가 콜백 URL에 추가됩니다.

예를 들어 이벤트 토큰 ‘abc123’에 해당하는 이벤트 추적을 위해 `http://www.adjust.com/callback`이라는 URL을 등록한 경우 다음 라인을 실행하면 됩니다.

```js
var adjustEvent = new AdjustEvent("abc123");

adjustEvent.addCallbackParameter("key", "value");
adjustEvent.addCallbackParameter("foo", "bar");

Adjust.trackEvent(adjustEvent);
```

이 경우 이벤트를 추적하여 아래 주소로 요청을 보냅니다.

```
http://www.adjust.com/callback?key=value&foo=bar
```

Adjust는 매개변수 값으로 사용하는 iOS 용의 `{idfa}`나 안드로이드 용의 `{gps_adid}` 등 다양한 자리표시자(placeholder)를 지원합니다. 결과 콜백 시 `{idfa}`는 현재 iOS 기기 광고자 ID로, `{gps_adid}`는 현재 안드로이드 기기 Google 광고 ID로 대체됩니다. Adjust는 고객 매개변수를 저장하지 않으며 콜백에 덧붙일 뿐입니다. 이벤트 콜백을 등록하지 않으셨다면 이들 매개변수는 아예 읽히지 않습니다. 

가능한 전체 값 목록을 비롯하여 자세한 URL 콜백 사용법은 [콜백 지침][callbacks-guide]에서 확인하세요.

#### <a id="partner-parameters">파트너 매개변수

위에 설명한 콜백 매개변수와 마찬가지로, 선택한 네트워크 파트너에게 전송할 매개변수도 추가할 수 있습니다. 

위에 설명한 콜백 매개변수의 경우와 비슷하지만, `AdjustEvent` 인스턴스에서 `addPartnerParameter` 메소드를 호출해야 추가할 수 있습니다.

```js
var adjustEvent = new AdjustEvent("abc123");

adjustEvent.addPartnerParameter("key", "value");
adjustEvent.addPartnerParameter("foo", "bar");

Adjust.trackEvent(adjustEvent);
```

특별 파트너 및 네트워크에 대한 자세한 내용은 [특별 파트너 설명서][special-partners]를 참조하십시오.

### <a id="session-parameters">세션 매개변수

일부 매개변수는 Adjust SDK 이벤트 및 세션 발생시마다 전송을 위해 저장합니다. 어느 매개변수이든 한 번 저장하면 로컬에 바로 저장되므로 매번 새로 추가할 필요가 없습니다. 같은 매개변수를 두 번 저장해도 효력이 없습니다.

이 세션 매개변수는 설치 시에도 전송할 수 있도록 Adjust SDK 런칭 전에도 호출할 수 있습니다. 설치 시에 전송하지만 필요한 값은 런칭 후에야 들어갈 수 있게 하고 싶다면 Adjust SDK 런칭 시 [예약 시작](#delay-start)을 걸 수 있습니다.

#### <a id="session-callback-parameters">세션 콜백 매개변수

[이벤트](#callback-parameters)에 등록한 콜백 매개변수는, Adjust SDK 전체 이벤트 및 세션 시 전송할 목적으로 저장할 수도 있습니다.

세션 콜백 매개변수는 이벤트 콜백 매개변수와 비슷한 인터페이스를 지녔지만, 이벤트에 키, 값을 추가하는 대신 `Adjust` 인스턴스에 있는 `addSessionCallbackParameter` 메소드를 호출하여 추가합니다.

```js
Adjust.addSessionCallbackParameter("foo", "bar");
```

세션 콜백 매개변수는 이벤트에 추가된 콜백 매개변수와 합쳐지며, 이벤트에 추가된 콜백 매개변수가 우선권을 지닙니다. 그러나 세션에서와 같은 키로 이벤트에 콜백 매개변수를 추가한 경우 새로 추가한 콜백 매개변수가 우선권을 가집니다.

원하는 키를 `Adjust` 인스턴스의 `removeSessionCallbackParameter` 메소드로 전달하여 특정 세션 콜백 매개변수를 제거할 수 있습니다.

```js
Adjust.removeSessionCallbackParameter("foo");
```

세션 콜백 매개변수의 키와 값을 전부 없애고 싶다면, `Adjust` 인스턴스의 `resetSessionCallbackParameters` 메소드로 재설정하면 됩니다.

```js
Adjust.resetSessionCallbackParameters();
```

#### <a id="session-partner-parameters">세션 파트너 매개변수

Adjust SDK 내 모든 이벤트 및 세션에서 전송되는 [세션 콜백 매개변수](#session-callback-parameters)가 있는 것처럼, 세션 파트너 매개변수도 있습니다.

이들 매개변수는 Adjust [대시보드]에서 연동 및 활성화된 네트워크 파트너에게 전송할 수 있습니다.

세션 파트너 매개변수는 이벤트 파트너 매개변수와 인터페이스가 비슷하지만, 이벤트에 키와 값을 추가하는 대신 `Adjust` 인스턴스에서 `addSessionPartnerParameter` 메소드를 호출하여 추가합니다.

```js
Adjust.addSessionPartnerParameter("foo", "bar");
```

세션 파트너 매개변수는 이벤트에 추가한 파트너 매개변수와 합쳐지며, 이벤트에 추가된 파트너 매개변수가 우선순위를 지닙니다. 그러나 세션에서와 같은 키로 이벤트에 파트너 매개변수를 추가한 경우, 새로 추가한 파트너 매개변수가 우선권을 가집니다.

원하는 키를 `Adjust` 인스턴스의 `removeSessionPartnerParameter` 메소드로 전달하여 특정 세션 파트너 매개변수를 제거할 수 있습니다.

```js
Adjust.removeSessionPartnerParameter("foo");
```

세션 파트너 매개변수의 키와 값을 전부 없애고 싶다면 `Adjust` 인스턴스의 `resetSessionPartnerParameters` 메소드로 재설정하면 됩니다.

```js
Adjust.resetSessionPartnerParameters();
```

#### 예약 시작

Adjust SDK에 예약 시작을 걸면 앱이 고유 식별자 등의 세션 매개변수를 얻어 설치 시 전송할 시간을 벌 수 있습니다.

`AdjustConfig` 인스턴스의 `setDelayStart` 메소드에서 예약 시작 시각을 초 단위로 설정하세요.

```js
adjustConfig.setDelayStart(5.5);
```

이 경우 Adjust SDK는 최초 인스톨 세션 및 생성된 이벤트를 5.5초간 기다렸다가 전송합니다. 이 시간이 지난 후, 또는 그 사이에 ‘Adjust’ 인스턴스의 `sendFirstPackages()`을 호출하면 모든 세션 매개변수가 지연된 인스톨 세션 및 이벤트에 추가되며 Adjust SDK는 원래대로 돌아옵니다.

**Adjust SDK의 최대 지연 예약 시작 시간은 10초입니다.**

### <a id="attribution-callback">속성 콜백

수신기를 등록하여 트래커 속성 변경 알림을 받을 수 있습니다. 속성에서 고려하는 소스가 각각 다르기 때문에 이 정보는 동시간에 제공할 수 없습니다. 가장 간단한 방법은 익명의 단일 수신기를 생성하여 **사용자 속성값이 바뀔 때마다** 호출되도록 하는 것입니다.

SDK를 시작하기 전에 `AdjustConfig`로 익명 수신기를 추가합니다.

```js
var adjustConfig = new AdjustConfig(appToken, environment);

adjustConfig.setAttributionCallbackListener(function(attribution) {
    // Printing all event success properties.
    console.log("Attribution changed!");
    console.log(attribution.trackerToken);
    console.log(attribution.trackerName);
    console.log(attribution.network);
    console.log(attribution.campaign);
    console.log(attribution.adgroup);
    console.log(attribution.creative);
    console.log(attribution.clickLabel);
    console.log(attribution.adid);
});

Adjust.create(adjustConfig);
```

수신기 함수에서는 `attribution` 매개변수에 억세스할 수 있습니다. 해당 매개변수에 대한 개요는 다음과 같습니다.

- `trackerToken`    현재 설치된 트래커 토큰.
- `trackerName`     현재 설치된 트래커 이름.
- `network`         현재 설치된 네트워크 그룹화 수준.
- `campaign`        현재 설치된 캠페인 그룹화 수준.
- `adgroup`         현재 설치된 ad group 그룹화 수준.
- `creative`        현재 설치된 크리에이티브 그룹화 수준.
- `clickLabel`      현재 설치된 클릭 레이블.
- `adid`            Adjust 장치 식별자.

[해당 속성 데이터 정책][attribution-data]을 반드시 고려하세요.

### <a id="session-event-callbacks">세션 및 이벤트 콜백

콜백을 등록하여 이벤트나 세션 추적 시 알림을 받을 수 있습니다. 

속성 콜백과 동일한 절차를 거쳐 다음과 같이 이벤트 성공 추적 콜백 함수를 적용합니다. 

```js
var adjustConfig = new AdjustConfig(appToken, environment);

adjustConfig.setEventTrackingSucceededCallbackListener(function(eventSuccess) {
    // Printing all event success properties.
    console.log("Event tracking succeeded!");
    console.log(eventSuccess.message);
    console.log(eventSuccess.timestamp);
    console.log(eventSuccess.eventToken);
    console.log(eventSuccess.adid);
    console.log(eventSuccess.jsonResponse);
});

Adjust.create(adjustConfig);
```

아래는 이벤트 추적 실패 콜백 함수입니다.

```js
var adjustConfig = new AdjustConfig(appToken, environment);

adjustConfig.setEventTrackingFailedCallbackListener(function(eventFailure) {
    // Printing all event failure properties.
    console.log("Event tracking failed!");
    console.log(eventFailure.message);
    console.log(eventFailure.timestamp);
    console.log(eventFailure.eventToken);
    console.log(eventFailure.adid);
    console.log(eventFailure.willRetry);
    console.log(eventFailure.jsonResponse);
});

Adjust.create(adjustConfig);
```

세션 추적 성공 콜백 함수는 다음과 같습니다.

```js
var adjustConfig = new AdjustConfig(appToken, environment);

adjustConfig.setSessionTrackingSucceededCallbackListener(function(sessionSuccess) {
    // Printing all session success properties.
    console.log("Session tracking succeeded!");
    console.log(sessionSuccess.message);
    console.log(sessionSuccess.timestamp);
    console.log(sessionSuccess.adid);
    console.log(sessionSuccess.jsonResponse);
});

Adjust.create(adjustConfig);
```

그리고 세션 추적 실패 콜백 함수입니다.

```js
var adjustConfig = new AdjustConfig(appToken, environment);

adjustConfig.setSessionTrackingFailedCallbackListener(function(sessionFailure) {
    // Printing all session failure properties.
    console.log("Session tracking failed!");
    console.log(sessionFailure.message);
    console.log(sessionFailure.timestamp);
    console.log(sessionFailure.adid);
    console.log(sessionFailure.willRetry);
    console.log(sessionFailure.jsonResponse);
});

Adjust.create(adjustConfig);
```

수신기 함수는 SDK가 서버에 패키지 전송을 시도한 후에 호출됩니다. 수신기 함수에서 수신기에 대한 특정 응답 데이터 개체에 억세스할 수 있습니다. 세션 응답 데이터 개체 필드 개요는 다음과 같습니다.

- `var message` 서버에서 전송한 메시지 또는 SDK가 기록한 오류.
- `var timestamp` 서버에서 전송한 데이터의 타임스탬프.
- `var adid` Adjust가 제공하는 고유 장치 식별자.
- `var jsonResponse` 서버로부터의 응답이 있는 JSON 개체.

이벤트 응답 데이터 개체 두 가지에는 다음 정보가 포함됩니다.

- `var eventToken` 추적 패키지가 이벤트인 경우 이벤트 토큰.

그리고 이벤트 및 세션 실패 개체에는 다음 정보도 포함됩니다.

- `var willRetry` 나중에 패키지 재전송 시도가 있을 것임을 나타냅니다.

### <a id="disable-tracking">추적 사용 중지

‘Adjust’ 인스탄스의 `setEnabled` 메소드를 `false` 매개변수로 설정한 상태로 호출하면 Adjust SDK에서 현재 장치의 모든 작업 추적을 중지할 수 있습니다. **이 설정은 세션과 세션 사이에 기억됩니다**. 하지만 첫 번째 세션 후에만 적용할 수 있습니다.

```js
Adjust.setEnabled(false);
```

‘Adjust’ 인스턴스의 `isEnabled` 메소드로 Adjust SDK가 현재 사용 가능한지 확인할 수 있습니다. 매개변수가 `true`로 설정된 `setEnabled`를 호출하면 Adjust SDK를 언제든지 활성화할 수 있습니다.

### <a id="offline-mode">오프라인 모드

Adjust SDK를 오프라인 모드로 전환하여 서버로 전송하는 작업을 일시 중단하고 추적 데이터를 보관하여 나중에 보낼 수 있습니다. 오프라인 모드일 때는 모든 정보가 파일에 저장되므로, 너무 많은 이벤트가 촉발되지 않도록 주의하십시오.

‘Adjust’ 인스턴스의 `setOfflineMode` 메소드를 `true` 매개변수로 설정한 상태로 호출하면 오프라인 모드를 활성화할 수 있습니다.

```js
Adjust.setOfflineMode(true);
```

또는 `setOfflineMode`를 `false`로 설정한 상태로 호출하면 오프라인 모드를 비활성화할 수 있습니다. Adjust SDK를 다시 온라인 모드로 전환하면 저장된 정보가 모두 올바른 시간 정보와 함께 Adjust 서버로 전송됩니다.

추적 사용 중지와 달리 이 설정은 세션 간에 **기억되지 않습니다.** 따라서 앱을 오프라인 모드에서 종료한 경우에도 SDK는 항상 온라인 모드로 시작됩니다.

### <a id="event-buffering">이벤트 버퍼링

앱이 이벤트 추적을 많이 사용하는 경우, 매 분마다 배치 하나씩만 보내도록 하기 위해 일부 HTTP 요청을 지연시키고자 할 경우가 있을 수 있습니다. `setEventBufferingEnabled` 메소드를 호출하면 `AdjustConfig` 인스턴스로 이벤트 버퍼링을 적용할 수 있습니다.

```js
var adjustConfig = new AdjustConfig(appToken, environment);

adjustConfig.setEventBufferingEnabled(true);

Adjust.create(adjustConfig);
```

### <a id="background-tracking">배경 추적

Adjust SDK 기본값 행위는 **앱이 배경에 있을 동안에는 HTTP 요청 전송을 잠시 중지**하는 것입니다. `setSendInBackground` 메소드를 호출하면 `AdjustConfig` 인스턴스에서 이를 바꿀 수 있습니다.

```js
var adjustConfig = new AdjustConfig(appToken, environment);

adjustConfig.setSendInBackground(true);

Adjust.create(adjustConfig);
```

여기에 설정한 내용이 없으면 배경에서의 전송은 **기본값으로 해제됩니다**.

### <a id="device-ids">기기 ID

Google Analytics 등 일부 기기의 경우 중복 보고를 막기 위해 기기 및 클라이언트 ID를 조정해야 합니다.

### <a id="di-idfa">iOS 광고 식별자

IDFA를 얻으려면 `Adjust` 인스턴스에서 `getIdfa` 메소드를 호출하세요. 값을 얻으려면 해당 메소드에 콜백을 전달해야 합니다.

```js
Adjust.getIdfa(function(idfa) {
    // Use idfa value.
});
```

### <a id="di-gps-adid">Google Play Services 광고 식별자

Google Advertising ID를 얻으려면 ‘Adjust’ 인스턴스의 `getGoogleAdId` 메소드를 호출하세요. 값을 얻으려면 해당 메소드에 콜백을 전달해야 합니다.

```js
Adjust.getGoogleAdId(function(googleAdId) {
    // Use googleAdId value.
});
```

콜백 메소드에서는 `googleAdId` 변수로 Google Advertising ID에 억세스할 수 있습니다.

### <a id="di-adid"></a>Adjust 장치 식별자

Adjust 백엔드는 앱을 인스톨한 장치에서 고유한 **Adjust 장치 식별자** (**adid**)를 생성합니다. 이 식별자를 얻으려면 `Adjust` 인스턴스에서 `getAdid` 메소드를 호출하세요. 값을 얻으려면 해당 메소드에 콜백을 전달해야 합니다.

```js
Adjust.getAdid(function(adid) {
    // Use adid value.
});
```
**주의**: **adid** 관련 정보는 Adjust 백엔드가 앱 인스톨을 추적한 후에만 얻을 수 있습니다. 그 순간부터 Adjust SDK는 장치 **adid** 정보를 갖게 되며 이 메소드로 억세스할 수 있습니다. 따라서 SDK가 초기화되고 앱 인스톨 추적이 성공적으로 이루어지기 전에는 **adid** 억세스가 **불가능합니다**.

### <a id="user-attribution"></a>사용자 속성

[속성 콜백 섹션](#attribution-callback)에서 설명한 대로, 이 콜백은 변동이 있을 때마다 새로운 속성 관련 정보를 전달할 목적으로 촉발됩니다. 사용자의 현재 속성 값 관련 정보에 언제든 억세스하고 싶다면, `Adjust` 인스턴스의 `getAttribution` 메소드를 호출하면 됩니다.

```js
Adjust.getAttribution(function(attribution) {
    // Use attribution object in same way like in attribution callback.
});
```

**주의**: 유저의 현재 속성 값 관련 정보는 Adjust 백엔드가 앱 인스톨을 추적하여 최초 속성 콜백이 촉발된 후에만 얻을 수 있습니다. 그 순간부터 Adjus SDK는 유저 속성 값 정보를 갖게 되며 이 메소드로 억세스할 수 있습니다. 따라서 SDK가 초기화되고 최초 속성 콜백이 촉발되기 전에는 유저 속성 값 억세스가 **불가능합니다**. 

### <a id="push-token">푸시 토큰

푸시 알림 토큰을 전송하려면 **앱에서 토큰을 받거나 값 변화가 있을 때마다** 아래와 같이 Adjust에 대한 호출을 추가하세요.

```js
Adjust.setDeviceToken("YourPushNotificationToken");
```

### <a id="pre-installed-trackers">사전 설치 트래커

Adjust SDK를 사용하여 앱이 사전 설치된 장치를 지닌 사용자를 인식하게 하고 싶다면 다음 절차를 따르세요.

1. 대시보드[dashboard]에 새 트래커를 생성합니다.
2. 앱 델리게이트를 열고 `AdjustConfig` 인스턴스의 기본값 트래커를 다음과 같이 설정합니다.

    ```js
    var adjustConfig = new AdjustConfig(appToken, environment);

    adjustConfig.setDefaultTracker("{TrackerToken}");
    
    Adjust.create(adjustConfig);
    ```

  `{TrackerToken}`을 2에서 생성한 트래커 토큰으로 대체합니다. 대시보드에서는 `http://app.adjust.com/`을 포함하는 트래커 URL을 표시한다는 사실을 명심하세요. 소스코드에서는 전체 URL을 표시할 수 없으며 6자로 이루어진 토큰만을 명시해야 합니다.

3. 앱 빌드를 실행하세요. 앱 로그 출력 시 다음과 같은 라인을 볼 수 있을 것입니다.

    ```
    Default tracker: 'abc123'
```

### <a id="deeplinking">딥링크

URL에서 앱으로 딥링크를 거는 옵션이 있는 Adjust 트래커 URL을 사용하고 있다면, 딥링크 URL과 그 내용 관련 정보를 얻을 가능성이 있습니다. 해당 URL 클릭 시 사용자가 이미 앱을 설치한 상태(기본 딥링크)일 수도, 앱을 설치하지 않은 상태(거치 딥링크)일 수도 있습니다. 

#### <a id="deeplinking-standard">기본 딥링크

기본 딥링크는 플랫폼에 따라 달라지는 기능이며, 지원하려면 앱에 몇 가지 설정을 추가해야 합니다. 사용자가 앱을 이미 설치한 상태에서 트래커 URL을 열면 앱이 열리며 딥링크 내용이 앱에 전송되어 이를 분석하고 다음에 어떤 행동을 취할 지 결정할 수 있습니다. 

**iOS 관련 알림**: iOS 9가 출시되면서 Apple이 앱 내 딥링크 취급 방식을 바꿨습니다. 둘 중 어떤 딥링크를 앱에 사용하려는 가에 따라 (또는 더 다양한 장치를 지원하기 위해 양쪽을 다 사용할 경우에) 앱에서 그에 맞춰 설정을 다시 해줘야 합니다. 

#### <a id="deeplinking-android-ios-old">안드로이드 및 iOS 8 이하 버전 딥링크

앱에서 안드로이드 및 iOS 8 이하 버전용 딥링크를 지원하려면, ‘개인 맞춤 URL 스킴’ 플러그인을 사용하면 됩니다. [여기][custom-url-scheme]에서 찾을 수 있습니다.

이 플러그인을 성공적으로 연동하고 나면, 사용자 장치에 있는 앱을 연 URL의 내용에 이 [섹션] [custom-url-scheme-usage]에서 설명한 플러그인에 사용하는 콜백 메소드로 억세스할 수 있습니다. 

```js
function handleOpenURL(url) {
    setTimeout(function () {
        // Check content of the url object and get information about the URL.
    }, 300);
};
```

이 플러그인 연동을 완료하면 **안드로이드 및 iOS 8 이하 버전**에서 딥링크를 취급할 수 있습니다. 

#### <a id="deeplinking-ios-new">iOS 9 이상 버전 딥링크

**iOS 9** 출시와 더불어 Apple는 위에서 설명한 기존 개인 맞춤 URL 스킴 방식의 딥링크에 대한 지원을 줄이고 ‘유니버설 링크’를 더 지원하겠다고 밝혔습니다. 앱이 iOS 9 이상 버전 용 딥링크를 지원하게 만들고 싶다면 유니버설 링크 취급에 지원을 추가해야 합니다.

우선 Adjust 대시보드에서 앱에 사용할 유니버설 링크를 활성화해야 합니다. 활성화하는 방법은 기본 iOS SDK [README][enable-ulinks]에서 찾아볼 수 있습니다.

대시보드에서 유니버설 링크를 활성화한 다음에는 앱 상에서 지원을 추가해야 합니다. 이 [플러그인][plugin-ulinks]을 cordova 앱에 추가하세요. 제대로 연동시키는 방법이 정확하게 설명되어 있으므로 플러그인 README를 주의 깊게 읽어주세요. 

**주의**: README에 나오는 정보 중 도메인과 웹사이트가 있어야 한다든가 도메인 루트에 파일을 업로드해야 한다는 내용은 전부 무시하셔도 됩니다. 이 부분은 Adjust가 알아서 해결해 드리오니 README에서 이 부분은 건너 뛰셔도 무방합니다. 또한 안드로이드 플랫폼에서는 딥링크 시 여전히 ‘개인 맞춤 URL 스킴’ 플러그인을 사용하고 있으므로 이 플러그인 관련 설명을 따르지 않아도 됩니다.

Adjust 대시보드에서 앱에 사용할 유니버설 링크를 성공적으로 활성화한 다음 `Cordova 유니버설 링크 플러그인` 연동을 마무리하려면 다음 절차를 거쳐야 합니다.
 
### `config.xml` 파일 편집

`config.xml`에 다음 엔트리를 추가해야 합니다.

```xml
<widget>
    <universal-links>
        <host name="[hash].adj.st" scheme="https" event="adjustDeepLinking" />
    </universal-links>
</widget>
```

`[hash]` 값을 Adjust 대시보드에서 생성한 값으로 대체해야 합니다. 이벤트 이름은 원하는 대로 붙이면 됩니다. 

### 플러그인에서 `ul_web_hooks/ios/` 내용 체크

앱에 있는 `Cordova 유니버설 링크 플러그인` 설치 디렉토리에서 ‘ul_web_hooks/ios/` 폴더 내용을 확인합니다. `[hash].ulink.adjust.com#apple-app-site-association`이라는 이름으로 생성된 파일을 볼 수 있을 것입니다. 이 파일 내용은 아래와 같습니다.

```
{
  "applinks": {
    "apps": [],
    "details": [
      {
        "appID": "<YOUR_TEAM_ID_FROM_MEMBER_CENTER>.com.adjust.examples",
        "paths": [
          "/ulink/*"
        ]
      }
    ]
  }
}
```

### 플러그인을 `index.js` 파일에 연동하기

`deviceready` 이벤트가 촉발된 후, `config.xml` 파일에서 정의한 이벤트에 가입하여 이벤트 촉발 시에 활성화될 콜백 메소드를 정의합니다. 안드로이드에서는 딥링크를 취급하는 데 이 플러그인이 필요하지 않으므로, 이벤트 가입은 앱이 iOS 장치에서 구동할 때만 필요합니다.

```js
// ...

var app = {
    initialize: function() {
        this.bindEvents();
    },

    bindEvents: function() {
        document.addEventListener('deviceready', this.onDeviceReady, false);
    },

    onDeviceReady: function() {
        if (device.platform == "iOS") {
            universalLinks.subscribe('adjustDeepLinking', app.didLaunchAppFromLink);
        }
    },

    didLaunchAppFromLink: function(eventData) {
        // Check content of the eventData.url object and get information about the URL.
    }
}
// ...
```

이 단계를 전부 마치면 iOS 9 이상 버전에서 딥링크에 필요한 지원을 성공적으로 추가한 것입니다. 

### <a id="deeplinking-deferred">거치 딥링크

안드로이드와 iOS에서는 거치 딥링크를 바로 지원하지 않지만, Adjust SDK를 사용하면 가능합니다.
 
거치 딥링크에서 URL 내용 정보를 얻으려면, 해당 정보가 전달되는 매개변수를 접수할 `AdjustConfig` 오브젝트에 대한 콜백 메소드를 설정해야 합니다. 이를 위해서는 `setDeeplinkCallbackListener` 메소드를 호출해야 합니다. 

```js
var adjustConfig = new AdjustConfig(appToken, environment);

adjustConfig.setDeferredDeeplinkCallbackListener(function(deeplink) {
    console.log("Deferred deep link URL content: " + deeplink);
});

Adjust.create(adjustConfig);
```

거치 딥링크에서는 `AdjustConfig` 오브젝트에 대해 한 가지 설정이 더 필요합니다. Adjust SDK가딥링크 정보를 얻고 나면 SDK가 이 URL을 열지 열지 않을지를 선택할 수 있습니다. 환경 설정 오브젝트에서 `setShouldLaunchDeeplink` 메소드를 호출하여 이 옵션을 설정할 수 있습니다.

```js
var adjustConfig = new AdjustConfig(appToken, environment);

adjustConfig.setShouldLaunchDeeplink(true);
// or adjustConfig.setShouldLaunchDeeplink(false);

adjustConfig.setDeeplinkCallbackListener(function(deeplink) {
    console.log("Deferred deep link URL content: " + deeplink);
});

Adjust.create(adjustConfig);
```

설정한 내용이 없을 경우 **Adjust SDK는 항상 URL 런칭을 기본값으로 선택합니다**.

#### <a id="deeplinking-reattribution">딥링크를 통한 재어트리뷰션

Adjust는 딥링크를 사용하여 광고 캠페인 리인게이지먼트(re-engagement)를 수행할 수 있게 해줍니다. 이에 대한 자세한 정보는 [관련 문서][reattribution-with-deeplinks]를 참조하세요.

이 기능을 사용 중이라면, 사용자를 올바로 리어트리뷰트하기 위해 앱에서 Adjust SDK에 대한 호출을 하나 더 수행해야 합니다.

앱에서 딥링크 내용을 수신했다면 ‘Adjust’ 인스턴스에서 `Adjust.appWillOpenUrl` 메소드 호출을 추가하세요. 이 호출이 이루어지면 Adjust SDK는 딥링크 내에 새로운 어트리뷰션 정보가 있는지 확인하고, 새 정보가 있으면 Adjust 백엔드로 송신합니다. 딥링크 정보가 담긴 Adjust 트래커 URL을 클릭한 유저를 리어트리뷰트해야 할 경우, 앱에서 해당 사용자의 새 속성 정보로 [속성 콜백](#attribution-callback)이 촉발되는 것을 확인할 수 있습니다.

위에 설명한 코드 예시에서 `appWillOpenUrl` 메소드 호출은 다음과 같이 이루어집니다.

```js
function handleOpenURL(url) {
    setTimeout(function () {
        // Check content of the url object and get information about the URL.
        Adjust.appWillOpenUrl(url);
    }, 300);
};
```

```js
// ...

var app = {
    initialize: function() {
        this.bindEvents();
    },

    bindEvents: function() {
        document.addEventListener('deviceready', this.onDeviceReady, false);
    },

    onDeviceReady: function() {
        if (device.platform == "iOS") {
            universalLinks.subscribe('adjustDeepLinking', app.didLaunchAppFromLink);
        }
    },

    didLaunchAppFromLink: function(eventData) {
        // Check content of the eventData.url object and get information about the URL.
        Adjust.appWillOpenUrl(eventData.url);
    }
}
// ...
```

[dashboard]:    http://adjust.com
[adjust.com]:   http://adjust.com

[example]:      ./example
[releases]:     https://github.com/adjust/cordova_sdk/releases
[npm-repo]:     https://www.npmjs.com/package/com.adjust.sdk

[google-ad-id]:         https://developer.android.com/google/play-services/id.html
[enable-ulinks]:        https://github.com/adjust/ios_sdk#deeplinking-setup-new
[plugin-ulinks]:        https://github.com/nordnet/cordova-universal-links-plugin
[event-tracking]:       https://docs.adjust.com/en/event-tracking
[callbacks-guide]:      https://docs.adjust.com/en/callbacks
[attribution-data]:     https://github.com/adjust/sdks/blob/master/doc/attribution-data.md
[special-partners]:     https://docs.adjust.com/en/special-partners
[custom-url-scheme]:    https://github.com/EddyVerbruggen/Custom-URL-scheme
[broadcast-receiver]:   https://github.com/adjust/android_sdk#sdk-broadcast-receiver

[google-launch-modes]:    http://developer.android.com/guide/topics/manifest/activity-element.html#lmode
[currency-conversion]:    https://docs.adjust.com/en/event-tracking/#tracking-purchases-in-different-currencies
[cordova-purchase-sdk]:   https://github.com/adjust/cordova_purchase_sdk
[google-play-services]:   http://developer.android.com/google/play-services/index.html

[custom-url-scheme-usage]:      https://github.com/EddyVerbruggen/Custom-URL-scheme#3-usage
[broadcast-receiver-custom]:    https://github.com/adjust/android_sdk/blob/master/doc/english/referrer.md
[reattribution-with-deeplinks]: https://docs.adjust.com/en/deeplinking/#manually-appending-attribution-data-to-a-deep-link

## <a id="license">라이선스

Adjust SDK는 MIT 라이선스에 따라 사용이 허가되었습니다.

Copyright (c) 2012-2015 adjust GmbH, http://www.adjust.com

이로써 본 소프트웨어와 관련 문서 파일(이하 "소프트웨어")의 복사본을 받는 사람에게는 아래 조건에 따라 소프트웨어를 제한 없이 다룰 수 있는 권한이 무료로 부여됩니다. 이 권한에는 소프트웨어를 사용, 복사, 수정, 병합, 출판, 배포 및/또는 판매하거나 2차 사용권을 부여할 권리와 소프트웨어를 제공 받은 사람이 소프트웨어를 사용, 복사, 수정, 병합, 출판, 배포 및/또는 판매하거나 2차 사용권을 부여하는 것을 허가할 수 있는 권리가 제한 없이 포함됩니다.

위 저작권 고지문과 본 권한 고지문은 소프트웨어의 모든 복사본이나 주요 부분에 포함되어야 합니다.

소프트웨어는 상품성, 특정 용도에 대한 적합성 및 비침해에 대한 보증 등을 비롯한 어떤 종류의 명시적이거나 암묵적인 보증 없이 "있는 그대로" 제공됩니다. 어떤 경우에도 저작자나 저작권 보유자는 소프트웨어와 소프트웨어의 사용 또는 기타 취급에서 비롯되거나 그에 기인하거나 그와 관련하여 발생하는 계약 이행 또는 불법 행위 등에 관한 배상 청구, 피해 또는 기타 채무에 대해 책임지지 않습니다.
--END--
