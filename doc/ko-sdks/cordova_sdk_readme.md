## 요약

이 항목에서는 adjust의 Cordova SDK에 대해 설명합니다. adjust에 대한 자세한 내용은 [adjust.com]을 참조하십시오.

참고 현재 Cordova용 SDK 4.3.0에서는 Android 플랫폼 버전 `4.0.0 and higher`과 iOS 플랫폼 버전 `3.0.0 and higher`을 지원합니다.

## 기본 설치

다음은 adjust SDK를 Cordova 프로젝트와 연동하기 위해 최소한으로 수행해야 하는 절차입니다. 여기서는 명령줄 인터페이스를 사용한다고 가정합니다.

### 1. SDK 다운로드 및 설치

[릴리스 페이지][releases]에서 최신 버전을 다운로드합니다. 압축 파일을 선택한 폴더에 풉니다.

### 2. 프로젝트에 SDK 추가

iOS 및/또는 Android를 프로젝트 플랫폼으로 추가한 후 다음 명령을 프로젝트 폴더에 입력합니다.

```
> cordova plugin add path_to_folder/cordova_sdk
Installing "com.adjust.sdk" for android
Installing "com.adjust.sdk" for ios
```

또는 adjust SDK를 `npm` [리포지토리][npm-repo]에서 직접 플러그인으로 다운로드할 수도 있습니다. 이렇게 하려면 프로젝트 폴더에서 다음 명령을 실행합니다.

```
> cordova plugin add com.adjust.sdk
Fetching plugin "com.adjust.sdk" via npm
Installing "com.adjust.sdk" for android
Installing "com.adjust.sdk" for ios
```

### 3. 앱과 연동

adjust 플러그인은 Cordova 이벤트 `deviceready`, `resume` 및 `pause`에 자동으로 등록됩니다.

#### 기본 설정

`deviceready` 이벤트를 수신한 후 `index.js` 파일에 다음 코드를 추가하여 adjust SDK를 초기화합니다.

```javascript
var adjustConfig = new AdjustConfig("{YourAppToken}", AdjustConfig.EnvironmentSandbox);

Adjust.create(adjustConfig);
```

`{YourAppToken}`을 앱 토큰으로 변경합니다. 앱 토큰은 [대시보드]에서 찾을 수 있습니다.

앱을 테스트에 사용할지 아니면 프로덕션에 사용할 지에 따라 `environment`을 다음 값 중 하나로 설정해야 합니다.

```javascript
AdjustConfig.EnvironmentSandbox
AdjustConfig.EnvironmentProduction
```

**중요:** 이 값은 앱을 테스트하는 경우에만 `AdjustConfig.EnvironmentSandbox`로 설정해야 합니다. 앱을 게시하기 직전에 environment를 `AdjustConfig.EnvironmentProduction`으로 설정해야 합니다. 개발 및 테스트를 다시 시작할 경우에는 `AdjustConfig.EnvironmentSandbox`로 다시 설정하십시오.

이 environment는 실제 트래픽과 테스트 장치의 테스트 트래픽을 구별하기 위해 사용합니다. 이 값을 항상 의미 있게 유지해야 합니다! 이것은 특히 매출을 트래킹하는 경우에 중요합니다.

#### adjust 로깅

다음 매개변수 중 하나를 사용하여 `AdjustConfig` 인스턴스에서 `setLogLevel`을 호출하면 테스트에 표시되는 로그의 양을 늘리거나 줄일 수 있습니다.

```javascript
adjustConfig.setLogLevel(AdjustConfig.LogLevelVerbose);   // enable all logging
adjustConfig.setLogLevel(AdjustConfig.LogLevelDebug);     // enable more logging
adjustConfig.setLogLevel(AdjustConfig.LogLevelInfo);      // the default
adjustConfig.setLogLevel(AdjustConfig.LogLevelWarn);      // disable info logging
adjustConfig.setLogLevel(AdjustConfig.LogLevelError);     // disable warnings as well
adjustConfig.setLogLevel(AdjustConfig.LogLevelAssert);    // disable errors as well
```

### 4. Google Play 서비스

2014년 8월 1일 이후로 Google Play Store의 앱은 [Google 광고 ID][google_ad_id]를 사용하여 장치를 고유하게 식별해야 합니다. adjust SDK에서 Google 광고 ID를 사용할 수 있게 하려면 [Google Play 서비스][google_play_services]를 연동해야 합니다.

adjust SDK 플러그인은 Google Play 서비스를 앱에 기본적으로 추가합니다.

Proguard를 사용 중인 경우 다음 행을 Proguard 파일에 추가하십시오.

````
-keep class com.adjust.sdk.** { *; }
-keep class com.google.android.gms.common.** { *; }
-keep class com.google.android.gms.ads.identifier.** { *; }
```

Google Play 서비스를 앱에 사용하지 않으려면 adjust SDK 플러그인의 `plugin.xml` 파일을 편집하여 제거하십시오. `plugins/com.adjust.sdk` 폴더로 이동하여 `plugin.xml` 파일을 여십시오.
Google Play 서비스 종속성을 추가하는 다음 행을 `<platform name="android">`의 일부로 찾을 수 있습니다.

```xml
<framework src="com.google.android.gms:play-services-analytics:+" />
```

Google Play 서비스를 제거하려면 이 행을 제거하고 변경 사항을 저장한 후 앱을 다시 만드십시오.

## 추가 기능

adjust SDK를 프로젝트와 연동한 후에는 다음 기능을 사용할 수 있습니다.

### 5. 사용자 지정 이벤트 트래킹 추가

adjust를 사용하여 이벤트를 트래킹할 수 있습니다. `abc123`과 같은 관련 이벤트 토큰이 있는 새 이벤트 토큰을 대시보드에서 만들어야 합니다. 
그런 다음 앱에서 다음 행을 추가하여 관심 있는 이벤트를 트래킹합니다.

```javascript
var adjustEvent = new AdjustEvent("abc123");
Adjust.trackEvent(adjustEvent);
```

이벤트 인스턴스를 사용하여 이벤트를 트래킹하기 전에 더 자세히 구성할 수 있습니다.

### 6. 매출 트래킹 추가

사용자가 광고를 누르거나 인앱 구매를 통해 매출을 발생시킬 수 있는 경우 이벤트를 사용하여 해당 매출을 트래킹할 수 있습니다. 한 번 누를 때 0.01 유로의 매출이 발생한다고 가정할 경우 매출 이벤트를 다음과 같이 트래킹할 수 있습니다.

```javascript
var adjustEvent = new AdjustEvent("abc123");
adjustEvent.setRevenue(0.01, "EUR");
Adjust.trackEvent(adjustEvent);
```

이것을 콜백 매개변수와 결합할 수도 있습니다.

통화 토큰을 설정하면 adjust에 수신되는 매출이 선택한 보고 매출으로 자동 변환됩니다. 통화 변환에 대한 자세한 내용은 [여기][currency-conversion]를 참조하십시오.

매출과 이벤트 트래킹에 대한 자세한 내용은 [이벤트 트래킹 설명서][event-tracking]를 참조하십시오.

### 7. 콜백 매개변수 추가

[대시보드]에서 이벤트의 콜백 URL을 등록할 수 있습니다. 이벤트가 트래킹될 때마다 GET 요청이 해당 URL로 전송됩니다. 이 이벤트를 트래킹하기 전에 이벤트 인스턴스에서 `addCallbackParameter`를 호출하여 콜백 매개변수를 해당 이벤트에 추가할 수 있습니다. 그러면 해당 매개변수가 콜백 URL에 추가됩니다.

예를 들어 URL `http://www.adjust.com/callback`을 등록한 경우 이벤트를 다음과 같이 트래킹할 수 있습니다.

```javascript
var adjustEvent = new AdjustEvent("abc123");

adjustEvent.addCallbackParameter("key", "value");
adjustEvent.addCallbackParameter("foo", "bar");

Adjust.trackEvent(adjustEvent);
```

이 경우에는 이벤트가 트래킹되고 요청이 다음 주소로 전송됩니다.

```
http://www.adjust.com/callback?key=value&foo=bar
```

매개변수 값으로 사용할 수 있는 `{android_id}`와 같은 다양한 자리 표시자가 지원됩니다. 결과로 생성된 콜백에서 이 자리 표시자는 현재 장치의 AndroidID로 변경됩니다.
또한 사용자 지정 매개변수는 저장되지 않고 콜백에만 추가됩니다. 이벤트에 대한 콜백을 등록하지 않은 경우 해당 매개변수는 읽을 수 없습니다.

사용 가능한 값의 전체 목록을 포함한 URL 콜백 사용에 대한 자세한 내용은 [콜백 설명서][callbacks-guide]를 참조하십시오.

### 8. 파트너 매개변수

adjust 대시보드에서 활성화된 연동에 대해 네트워크 파트너로 전송할 매개변수도 추가할 수 있습니다.

이 매개변수는 위에서 설명한 콜백 매개변수의 경우와 비슷하지만, `AdjustEvent` 인스턴스에서 `addPartnerParameter` 메서드를 호출해야 추가할 수 있습니다.

```javascript
var adjustEvent = new AdjustEvent("abc123");

adjustEvent.addPartnerParameter("key", "value");
adjustEvent.addPartnerParameter("foo", "bar");

Adjust.trackEvent(adjustEvent);
```

특별 파트너와 해당 파트너와의 연동에 대한 자세한 내용은 [특별 파트너 설명서][special-partners]를 참조하십시오.

### 9. 어트리뷰션 변경 수신기 설정

트래커 어트리뷰션 변경에 대한 알림을 수신할 수신기를 등록할 수 있습니다. 어트리뷰션에 대해 다양한 소스가 고려되기 때문에 이 정보는 동시에 제공할 수 없습니다. 가장 간단한 방법은 익명 수신기를 하나 만드는 것입니다.

[해당 어트리뷰션 데이터 정책][attribution-data]을 고려하십시오.

SDK를 시작하기 전에 `AdjustConfig` 인스턴스를 사용하여 익명 수신기를 추가합니다.

```javascript
var adjustConfig = new AdjustConfig(appToken, environment);

adjustConfig.setCallbackListener(function(attribution) {
});

Adjust.create(adjustConfig);
```

SDK에 최종 어트리뷰션 정보가 수신되면 수신기 함수가 호출됩니다. 수신기 함수를 통해 `attribution` 매개변수에 액세스할 수 있습니다. 매개변수 속성에 대한 개요는 다음곽 같습니다.

- `trackerToken`    현재 설치의 트래커 토큰
- `trackerName`    현재 설치의 트래커 이름
- `network`       현재 설치의 network 그룹화 기준
- `campaign`        현재 설치의 campaign 그룹화 기준
- `adgroup`        현재 설치의 ad group 그룹화 기준
- `creative`        현재 설치의 creative 그룹화 기준
- `clickLabel`        현재 설치의 클릭 레이블

### 10. 딥링크 리어트리뷰션 설정

앱을 열기 위해 사용하는 딥링크를 처리하도록 adjust SDK를 설정할 수 있습니다. adjust에서는 특정 adjust 관련 매개변수만 읽습니다. 딥링크를 사용하여 리타게팅 또는 재참여 캠페인을 운영할 계획인 경우 이 설정은 필수입니다.

앱 스키마 이름을 설정하려면 `Custom URL Scheme` 플러그인을 사용하십시오. 이 플러그인은 [여기][custom-url-scheme]에서 찾을 수 있습니다.

이 플러그인을 성공적으로 연동한 후 이 [섹션][custom-url-scheme-usage]에서 설명하는 플러그인과 함께 사용하는 콜백 메서드에서 `Adjust` 인스턴스의 `appWillOpenUrl` 메서드에 호출을 추가하고 `url`을 매개변수로 전달하십시오.

```javascript
function handleOpenURL(url) {
    setTimeout(function () {
        Adjust.appWillOpenUrl(url);
    }, 300);
};

```

생성된 기본 프로젝트에서 직접 딥링크 연결 리어트리뷰션을 사용하려면 다음 단계를 수행하십시오.

#### iOS

XCode Project Navigator에서 Application Delegate의 소스 파일을 엽니다. `openURL` 메서드를 찾거나 추가하고 다음 호출을 adjust에 추가합니다.

```objc
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url
  sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
{
    [Adjust appWillOpenUrl:url];
    Bool canHandle = [self someLogic:url];
    return canHandle;
}
```

#### Android

딥링크를 허용하는 각 작업에 대해 `onCreate` 또는 `onNewIntent` 메서드를 찾고 다음 호출을 adjust에 추가합니다.

###### 시작 모드가 `standard`인 작업의 경우

```java
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    Intent intent = getIntent();
    Uri data = intent.getData();
    Adjust.appWillOpenUrl(data);
    // ...
}
```

###### 시작 모드가 `singleTop`인 작업의 경우

```java
protected void onNewIntent(Intent intent) {
    super.onNewIntent(intent);

    Uri data = intent.getData();
    Adjust.appWillOpenUrl(data);
    // ...
}
```

작업 시작 모드에 대한 자세한 내용은 이 [페이지][google-launch-modes]를 참조하십시오.

### 11. 이벤트 버퍼링 사용

앱에서 이벤트 트래킹을 많이 사용하는 경우 일부 HTTP 요청을 지연하여 1분마다 하나의 배치로 보낼 수 있습니다. `AdjustConfig` 인스턴스를 통해 이벤트 버퍼링을 사용하도록 설정할 수 있습니다.

```javascript
var adjustConfig = new AdjustConfig(appToken, environment);
adjustConfig.setEventBufferingEnabled(true);
Adjust.create(adjustConfig);
```

### 12. 트래킹 사용 중지

`setEnabled` 매개변수를 `false`로 설정한 상태로 호출하면 adjust SDK에서 현재 장치의 모든 작업을 트래킹하지 않도록 할 수 있습니다. 이 설정은 세션 간에 기억됩니다.

```javascript
Adjust.setEnabled(false);
```

`isEnabled` 함수를 호출하여 adjust SDK가 현재 사용 가능한지 확인할 수 있습니다. `setEnabled` 매개변수를 `true`로 설정한 상태로 호출하면 adjust SDK를 언제든지 활성화할 수 있습니다.

SDK 사용 가능 여부를 나타내는 부울식을 수신할 함수를 전달하여 `isEnabled`를 호출해야 합니다.

```javascript
Adjust.isEnabled(function(isEnabled) {
    if (isEnabled) {
        // SDK is enabled.
    } else {
        // SDK is disabled.
    }
});
```

### 13. 오프라인 모드

adjust SDK를 오프라인 모드로 전환하여 adjust 서버로 전송하는 작업을 일시 중단하고 트래킹된 데이터를 보관하여 나중에 보낼 수 있습니다. 오프라인 모드일 때는 모든 정보가 파일에 저장되므로 오프라인 모드에서 너무 많은 이벤트가 트리거되지 않도록 주의하십시오.

`setOfflineMode` 매개변수를 `true`로 설정한 상태로 호출하면 오프라인 모드를 활성화할 수 있습니다.

```javascript
Adjust.setOfflineMode(true);
```

반대로 `setOfflineMode` 매개변수를 `false`로 설정한 상태로 호출하면 오프라인 모드를 비활성화할 수 있습니다.
adjust SDK를 다시 온라인 모드로 전환하면 저장된 정보가 모두 올바른 시간 정보와 함께 adjust 서버로 전송됩니다.

트래킹 사용 중지와 달리 이 설정은 세션 간에 *기억되지 않습니다.* 
따라서 앱을 오프라인 모드에서 종료한 경우에도 SDK는 항상 온라인 모드로 시작됩니다.

[adjust.com]:               http://adjust.com
[dashboard]:                http://adjust.com
[releases]:                 https://github.com/adjust/cordova_sdk/releases
[npm-repo]:                 https://www.npmjs.com/package/com.adjust.sdk
[attribution-data]:         https://github.com/adjust/sdks/blob/master/doc/attribution-data.md
[callbacks-guide]:          https://docs.adjust.com/en/callbacks
[event-tracking]:           https://docs.adjust.com/en/event-tracking
[special-partners]:         https://docs.adjust.com/en/special-partners
[currency-conversion]:      https://docs.adjust.com/en/event-tracking/#tracking-purchases-in-different-currencies
[google-launch-modes]:      http://developer.android.com/guide/topics/manifest/activity-element.html#lmode
[google_play_services]:     http://developer.android.com/google/play-services/index.html
[google_ad_id]:             https://developer.android.com/google/play-services/id.html
[custom-url-scheme]:        https://github.com/EddyVerbruggen/Custom-URL-scheme
[custom-url-scheme-usage]:  https://github.com/EddyVerbruggen/Custom-URL-scheme#3-usage

## 라이선스

adjust SDK는 MIT 라이선스에 따라 사용이 허가되었습니다.

Copyright (c) 2012-2015 adjust GmbH, http://www.adjust.com

이로써 본 소프트웨어와 관련 문서 파일(이하 "소프트웨어")의 복사본을 받는 사람에게는 아래 조건에 따라 소프트웨어를 제한 없이 다룰 수 있는 권한이 무료로 부여됩니다. 이 권한에는 소프트웨어를 사용, 복사, 수정, 병합, 출판, 배포 및/또는 판매하거나 2차 사용권을 부여할 권리와 소프트웨어를 제공 받은 사람이 소프트웨어를 사용, 복사, 수정, 병합, 출판, 배포 및/또는 판매하거나 2차 사용권을 부여하는 것을 허가할 수 있는 권리가 제한 없이 포함됩니다.

위 저작권 고지문과 본 권한 고지문은 소프트웨어의 모든 복사본이나 주요 부분에 포함되어야 합니다.

소프트웨어는 상품성, 특정 용도에 대한 적합성 및 비침해에 대한 보증 등을 비롯한 어떤 종류의 명시적이거나 암묵적인 보증 없이 "있는 그대로" 제공됩니다. 어떤 경우에도 저작자나 저작권 보유자는 소프트웨어와 소프트웨어의 사용 또는 기타 취급에서 비롯되거나 그에 기인하거나 그와 관련하여 발생하는 계약 이행 또는 불법 행위 등에 관한 배상 청구, 피해 또는 기타 채무에 대해 책임지지 않습니다.
--END--
