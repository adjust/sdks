## 요약

이 항목에서는 adjust의 Android SDK에 대해 설명합니다. adjust에 대한 자세한 내용은 [adjust.com]을 참조하십시오.

## 앱 예제

[`example` 디렉터리][example]에 앱 예제가 있습니다. Android 프로젝트를
열어 adjust SDK를 연동할 수 있는 방법의 예를 확인할 수 있습니다.

## 기본 설치

다음은 adjust SDK를 Android 프로젝트와 연동하기 위해 최소한으로 수행해야 하는
절차입니다. 여기서는 Android Studio를 Android 개발에
사용하고 Android API 레벨 9(Gingerbread) 이상을 대상으로 한다고 가정합니다.

[Maven Repository][maven]를 사용하는 경우에는 [3단계](#step3)부터 시작해도 됩니다.

### 1. SDK 다운로드 및 설치

[릴리스 페이지][releases]에서 최신 버전을 다운로드합니다. 압축 파일을
선택한 폴더에 풉니다.

### 2. Adjust 프로젝트 만들기

Android Studio 메뉴에서 `File a†’ Import Module...`을 선택합니다.

![][import_module]

`Source directory` 필드에서는 1단계에서 압축을 푼 폴더를 찾습니다.
`./android_sdk/Adjust/adjust` 폴더를 선택합니다.  작업을 완료하기 전에
모듈 이름 `:adjust`가 표시되는지 확인합니다.

![][select_module]

`adjust` 모듈을 나중에 Android Studio 프로젝트로
가져옵니다.

![][imported_module]

### <a id="step3"></a>3. 프로젝트에 adjust 라이브러리 추가

앱의 `build.gradle` 파일을 열고 `dependencies` 블록을 찾습니다. 다음
행을 추가합니다.

```
compile project(":adjust")
```

![][gradle_adjust]

Maven을 사용하는 경우 다음 행을 대신 추가합니다.

```
compile 'com.adjust.sdk:adjust-android:4.7.0'
```

### 4. Google Play 서비스 추가

2014년 8월 1일 이후로 Google Play Store의 앱은 [Google 광고 ID]
[google_ad_id]를 사용하여 장치를 고유하게 식별해야 합니다. adjust SDK에서
Google 광고 ID를 사용할 수 있게 하려면 [Google Play 서비스][google_play_services]를 연동해야 합니다. 이 작업을 아직 수행하지 않은 경우
다음 단계를 수행하십시오.

1. 앱의 `build.gradle` 파일을 열고 `dependencies` 블록을 찾습니다. 다음 행을
추가합니다.

    ```
    compile 'com.google.android.gms:play-services-analytics:8.4.0'
    ```

    ![][gradle_gps]

2. Google Play 서비스 버전 7 이상을 사용 중인 경우 이 단계를 건너뜁니다.
   Package Explorer에서 Android 프로젝트의 `AndroidManifest.xml`을
엽니다.  다음 `meta-data` 태그를 `<application>` 요소에
추가합니다.


    ```xml
    <meta-data android:name="com.google.android.gms.version"
               android:value="@integer/google_play_services_version" />
    ```

    ![][manifest_gps]

### 5. 권한 추가

Package Explorer에서 Android 프로젝트의 `AndroidManifest.xml`을 엽니다.
`INTERNET`에 대한 `uses-permission` 태그가 아직 없는 경우 이 태그를 추가합니다.

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

Google Play Store가 대상이 *아닌* 경우 다음 두 권한을 모두 추가합니다.

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
```

![][manifest_permissions]

Proguard를 사용 중인 경우 다음 행을 Proguard 파일에 추가합니다.

```
-keep class com.adjust.sdk.plugin.MacAddressUtil {
    java.lang.String getMacAddress(android.content.Context);
}
-keep class com.adjust.sdk.plugin.AndroidIdUtil {
    java.lang.String getAndroidId(android.content.Context);
}
-keep class com.google.android.gms.common.ConnectionResult {
    int SUCCESS;
}
-keep class com.google.android.gms.ads.identifier.AdvertisingIdClient {
    com.google.android.gms.ads.identifier.AdvertisingIdClient$Info getAdvertisingIdInfo (android.content.Context);
}
-keep class com.google.android.gms.ads.identifier.AdvertisingIdClient$Info {
    java.lang.String getId ();
    boolean isLimitAdTrackingEnabled();
}
```

Google Play Store가 대상이 *아닌* 경우 `com.google.android.gms` 규칙을 제거할
수 있습니다.

![][proguard]

**중요**: Proguard 파일의 `-overloadaggressively` 플래그를
사용할 경우 adjust SDK가 올바르게 작동하려면 다음 두 가지 시나리오 중 하나를 고려해야 합니다.

* `-overloadaggressively`가 필요하지 않은 경우 제거합니다.
* `-useuniqueclassmembernames` 플래그를 Proguard 파일에 추가합니다.

### <a id="broadcast_receiver"></a>6. 브로드캐스트 수신기 추가

`AndroidManifest.xml`에서 다음 `receiver` 태그를 `application` 태그에
추가합니다.

```xml
<receiver
    android:name="com.adjust.sdk.AdjustReferrerReceiver"
    android:exported="true" >
    <intent-filter>
        <action android:name="com.android.vending.INSTALL_REFERRER" />
    </intent-filter>
</receiver>
```

![][receiver]

이 브로드캐스트 수신기는 전환 트래킹을 개선하기 위해 설치 참조
페이지를 검색하는 데 사용됩니다.

다른 브로드캐스트 수신기를 `INSTALL_REFERRER` intent로 이미
사용 중인 경우 [이 지침][referrer]에 따라 adjust 수신기를
추가하십시오.

### 7. 앱과 adjust 연동

먼저 기본 세션 트래킹을 설정합니다.

#### 기본 설정

전역 android [응용 프로그램][android_application] 클래스를 사용하여
SDK를 초기화하는 것이 좋습니다. 앱에 이 클래스가 아직 없는 경우 다음 단계를 수행하십시오.

1. `Application`을 확장하는 클래스를 만듭니다.
    ![][application_class]

2. 앱의 `AndroidManifest.xml` 파일을 열고 `<application>` 요소를 찾습니다.
3. `android:name` 특성을 추가하고 새 응용 프로그램 클래스 이름으로 설정한 후 이름 앞에 점을 추가합니다.

    앱 예제에서는 이름이 `GlobalApplication`인 `Application` 클래스를 사용하므로, 매니페스트 파일은 다음과 같이 구성됩니다.
    ```xml
     <application
       android:name=".GlobalApplication"
       ... >
         ...
    </application>
    ```

    ![][manifest_application]

4. `Application` 클래스에서 `onCreate` 메서드를 추가하거나 만들고 다음 코드를
추가하여 adjust SDK를 초기화합니다.

    ```java
    import com.adjust.sdk.Adjust;
    import com.adjust.sdk.AdjustConfig;

    public class GlobalApplication extends Application {
        @Override
        public void onCreate() {
            super.onCreate();

            String appToken = "{YourAppToken}";
            String environment = AdjustConfig.ENVIRONMENT_SANDBOX;
            AdjustConfig config = new AdjustConfig(this, appToken, environment);
            Adjust.onCreate(config);
        }
    }
    ```

    ![][application_config]

    `{YourAppToken}`을 앱 토큰으로 대체합니다. 앱 토큰은 [대시보드]에서 찾을
    수 있습니다.

    앱을 테스트에 사용할지 아니면 프로덕션에 사용할 지에 따라 `environment`를
    다음 값 중 하나로 설정해야 합니다.

    ```java
    String environment = AdjustConfig.ENVIRONMENT_SANDBOX;
    String environment = AdjustConfig.ENVIRONMENT_PRODUCTION;
    ```

    **중요:** 이 값은 앱을 테스트하는 경우에만
    `AdjustConfig.ENVIRONMENT_SANDBOX`로 설정해야 합니다. 앱을 게시하기 전에
    environment를 `AdjustConfig.ENVIRONMENT_PRODUCTION`으로 설정해야 합니다. 개발 및 테스트를 다시 시작할 경우에는 `AdjustConfig.ENVIRONMENT_SANDBOX`로
    다시 설정하십시오.

    이 environment는 실제 트래픽과 테스트 장치의 테스트 트래픽을 구별하기 위해
    사용합니다. 이 값을 항상 의미 있게 유지해야 합니다! 이것은 특히 매출을 트래킹하는 경우에 중요합니다.


5. `ActivityLifecycleCallbacks` 인터페이스를 구현하는 비공개 클래스를 추가합니다.
이 인터페이스에 액세스할 권한이 없으면 앱의 대상 Android api 레벨이 14보다 낮기 때문입니다.
이 [지침][activity_resume_pause]에 따라 각 작업을 수동으로 업데이트해야 합니다.
이전에 앱의 각 작업에 대한 `Adjust.onResume` 및 `Adjust.onPause` 호출이 있었을 경우
각 호출을 제거해야 합니다.

    ![][activity_lifecycle_class]

6. `onActivityResumed(Activity activity)` 메서드를 편집하고 `Adjust.onResume()`에 호출을 추가합니다.
`onActivityPaused(Activity activity)` 메서드를 편집하고 `Adjust.onPause()`에 호출을 추가합니다.

    ![][activity_lifecycle_methods]

7. adjust SDK가 구성된 `onCreate()` 메서드를 추가하고 `registerActivityLifecycleCallbacks` 호출을 이전에 만든 `ActivityLifecycleCallbacks` 클래스의 인스턴스와 함께 추가합니다.

    ```java
    import com.adjust.sdk.Adjust;
    import com.adjust.sdk.AdjustConfig;

    public class GlobalApplication extends Application {
        @Override
        public void onCreate() {
            super.onCreate();

            String appToken = "{YourAppToken}";
            String environment = AdjustConfig.ENVIRONMENT_SANDBOX;
            AdjustConfig config = new AdjustConfig(this, appToken, environment);
            Adjust.onCreate(config);

            registerActivityLifecycleCallbacks(new AdjustLifecycleCallbacks());

            //...
        }
    }
    private static final class AdjustLifecycleCallbacks implements ActivityLifecycleCallbacks {
        @Override
        public void onActivityResumed(Activity activity) {
            Adjust.onResume();
        }

        @Override
        public void onActivityPaused(Activity activity) {
            Adjust.onPause();
        }
        //...
    }
    ```

    ![][activity_lifecycle_register]

#### adjust 로깅

다음 매개변수 중 하나를 사용하여 `AdjustConfig` 인스턴스에서 `setLogLevel`을 호출하면 테스트에 표시되는 로그의 양을 늘리거나 줄일 수
있습니다.

```java
config.setLogLevel(LogLevel.VERBOSE);   // enable all logging
config.setLogLevel(LogLevel.DEBUG);     // enable more logging
config.setLogLevel(LogLevel.INFO);      // the default
config.setLogLevel(LogLevel.WARN);      // disable info logging
config.setLogLevel(LogLevel.ERROR);     // disable warnings as well
config.setLogLevel(LogLevel.ASSERT);    // disable errors as well
```

### 8. 앱 작성

Android 앱을 작성하고 실행합니다. LogCat 뷰어에서 `tag:Adjust` 필터를 설정하여
다른 모든 로그를 숨길 수 있습니다. 앱이 시작된 후에 다음 adjust 로그가
표시됩니다. `Install tracked`

![][log_message]

## 추가 기능

adjust SDK를 프로젝트와 연동한 후에는 다음 기능을 사용할 수
있습니다.

### 9. 사용자 지정 이벤트 트래킹 추가

adjust를 사용하여 앱의 모든 이벤트를 트래킹할 수 있습니다. 버튼의 모든 탭을 트래킹하려면 [대시보드]에서 새 이벤트 토큰을 만들어야
합니다. 이벤트 토큰이 `abc123`일 경우, 버튼의 `onClick`
메서드에 다음 행을 추가하여 클릭을 트래킹할 수 있습니다.

```java
AdjustEvent event = new AdjustEvent("abc123");
Adjust.trackEvent(event);
```

이벤트 인스턴스를 사용하여 이벤트를 트래킹하기 전에 더 자세히
구성할 수 있습니다.

### 10. 콜백 매개변수 추가

[대시보드]에서 이벤트의 콜백 URL을 등록할 수 있습니다. 이벤트가
트래킹될 때마다 GET 요청이 해당 URL로 전송됩니다. 이 이벤트를 트래킹하기
전에 이벤트 인스턴스에서 `addCallbackParameter`를 호출하여 콜백 매개변수를
해당 이벤트에 추가할 수 있습니다. 그러면 해당 매개변수가 콜백 URL에 추가됩니다.

예를 들어 URL `http://www.adjust.com/callback`을 등록한 경우
이벤트를 다음과 같이 트래킹할 수 있습니다.

```java
AdjustEvent event = new AdjustEvent("abc123");

event.addCallbackParameter("key", "value");
event.addCallbackParameter("foo", "bar");

Adjust.trackEvent(event);
```

이 경우에는 이벤트가 트래킹되고 요청이 다음 주소로 전송됩니다.

```
http://www.adjust.com/callback?key=value&foo=bar
```

매개변수 값으로 사용할 수 있는 `{android_id}`와 같은 다양한 자리 표시자가 지원됩니다. 결과로 생성된 콜백에서
이 자리 표시자는 현재 장치의 AndroidID로 대체됩니다.
또한 사용자 지정 매개변수는 저장되지 않고 콜백에만
추가됩니다. 이벤트에 대한 콜백을 등록하지 않은 경우
해당 매개변수는 읽을 수 없습니다.

사용 가능한 값의 전체 목록을 포함한 URL 콜백 사용에 대한 자세한 내용은
[콜백 설명서][callbacks-guide]를 참조하십시오.


### 11. 파트너 매개변수

adjust 대시보드에서 활성화된 연동에 대해 네트워크 파트너로 전송할
매개변수도 추가할 수 있습니다.

이 매개변수는 위에서 설명한 콜백 매개변수의 경우와 비슷하지만,
`AdjustEvent` 인스턴스에서 `addPartnerParameter` 메서드를 호출해야 추가할 수 있습니다.

```java
AdjustEvent event = new AdjustEvent("abc123");

event.addPartnerParameter("key", "value");
event.addPartnerParameter("foo", "bar");

Adjust.trackEvent(event);
```

특별 파트너와 해당 파트너와의 연동에 대한 자세한 내용은 [특별 파트너
설명서][special-partners]를 참조하십시오.

### 12. 매출 트래킹 추가

사용자가 광고를 누르거나 인앱 구매를 통해 매출을 발생시킬 수 있는
경우 이벤트를 사용하여 해당 매출을 트래킹할 수 있습니다. 한 번 누를 때 0.01 유로의
매출이 발생한다고 가정할 경우 매출 이벤트를 다음과 같이 트래킹할 수 있습니다.

```java
AdjustEvent event = new AdjustEvent("abc123");
event.setRevenue(0.01, "EUR");
Adjust.trackEvent(event);
```

이것을 콜백 매개변수와 결합할 수도 있습니다.

통화 토큰을 설정하면 adjust에서 수신되는 매출이 선택한 보고 매출로 자동 변환됩니다. 통화 변환에 대한 자세한 내용은 [여기][currency-conversion]를 참조하십시오.

매출과 이벤트 트래킹에 대한 자세한 내용은 [이벤트 트래킹
설명서][event-tracking]를 참조하십시오.

### 13. 딥링크 리어트리뷰션 설정

앱을 열기 위해 사용하는 딥링크를 처리하도록 adjust SDK를 설정할
수 있습니다. adjust에서는 특정 adjust 관련 매개변수만 읽습니다. 딥링크를 사용하여
리타게팅 또는 재참여 캠페인을 운영할 계획인 경우 이 설정은 필수입니다.

딥링크를 허용하는 각 작업에 대해 `onCreate` 메서드를 찾고 다음 호출을
adjust에 추가합니다.

```java
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    Intent intent = getIntent();
    Uri data = intent.getData();
    Adjust.appWillOpenUrl(data);
    //...
}
```

### 14. 이벤트 버퍼링 사용

앱에서 이벤트 트래킹을 많이 사용하는 경우 일부 HTTP 요청을 지연하여 1분마다
하나의 배치로 보낼 수 있습니다. `AdjustConfig` 인스턴스를 통해
이벤트 버퍼링을 사용하도록 설정할 수 있습니다.

```java
AdjustConfig config = new AdjustConfig(this, appToken, environment);

config.setEventBufferingEnabled(true);

Adjust.onCreate(config);
```

### 15. 백그라운드에서 보내기

adjust SDK는 기본적으로 앱이 백그라운드에서 설정되어 있는 동안 HTTP 요청 보내기를 일시정지합니다.
이 설정을 `AdjustConfig` 인스턴스에서 변경할 수 있습니다.

```java
AdjustConfig config = new AdjustConfig(this, appToken, environment);

config.setSendInBackground(true);

Adjust.onCreate(config);
```

### <a id="attribution_changed_listener"></a>16. 어트리뷰션 변경에 대한 수신기 설정

트래커 어트리뷰션 변경에 대한 알림을 수신할 수신기를 등록할 수 있습니다. 어트리뷰션에
대해 다양한 소스가 고려되기 때문에 이 정보는 동시에 제공할 수
없습니다. 가장 간단한 방법은 익명 수신기를 하나 만드는
것입니다.

[해당 어트리뷰션 데이터 정책][attribution-data]을
고려하십시오.

SDK를 시작하기 전에 `AdjustConfig` 인스턴스를 사용하여 익명 수신기를 추가합니다.

```java
AdjustConfig config = new AdjustConfig(this, appToken, environment);

config.setOnAttributionChangedListener(new OnAttributionChangedListener() {
    @Override
    public void onAttributionChanged(AdjustAttribution attribution) {
    }
});

Adjust.onCreate(config);
```

또는 `Application` 클래스에서 `OnAttributionChangedListener`
인터페이스를 구현하고 수신기로 설정할 수 있습니다.

```java
AdjustConfig config = new AdjustConfig(this, appToken, environment);
config.setOnAttributionChangedListener(this);
Adjust.onCreate(config);
```

SDK에 최종 어트리뷰션 정보가 수신되면 수신기 함수가
호출됩니다. 수신기 함수를 통해 `attribution` 매개변수에 액세스할
수 있습니다. 다음은 매개변수 속성에 대한 정보를 간략히 요약하고 있습니다.

- `String trackerToken` 현재 설치의 트래커 토큰
- `String trackerName` 현재 설치의 트래커 이름
- `String network` 현재 설치의 network 그룹화 기준
- `String campaign` 현재 설치의 campaign 그룹화 기준
- `String adgroup` 현재 설치의 ad group 그룹화 기준
- `String creative` 현재 설치의 creative 그룹화 기준
- `String clickLabel` 현재 설치의 클릭 레이블

### 17. 트래킹하는 이벤트 및 세션의 수신기 설정

이벤트 또는 세션이 트래킹될 때 알림을 수신할 수신기를 등록할 수 있습니다.
성공적인 이벤트 트래킹 수신기, 실패한 이벤트 트래킹 수신기, 성공적인 세션 트래킹 수신기, 실패한 세션 트래킹 수신기와 같이 4개의 수신기가 있습니다.
`AdjustConfig` 개체를 만든 후에 수신기를 원하는 수만큼 추가할 수 있습니다.

```java
AdjustConfig config = new AdjustConfig(this, appToken, environment);

// set event success tracking delegate
config.setOnEventTrackingSucceededListener(new OnEventTrackingSucceededListener() {
    @Override
    public void onFinishedEventTrackingSucceeded(AdjustEventSuccess eventSuccessResponseData) {
        // ...
    }
});

// set event failure tracking delegate
config.setOnEventTrackingFailedListener(new OnEventTrackingFailedListener() {
    @Override
    public void onFinishedEventTrackingFailed(AdjustEventFailure eventFailureResponseData) {
        // ...
    }
});

// set session success tracking delegate
config.setOnSessionTrackingSucceededListener(new OnSessionTrackingSucceededListener() {
    @Override
    public void onFinishedSessionTrackingSucceeded(AdjustSessionSuccess sessionSuccessResponseData) {
        // ...
    }
});

// set session failure tracking delegate
config.setOnSessionTrackingFailedListener(new OnSessionTrackingFailedListener() {
    @Override
    public void onFinishedSessionTrackingFailed(AdjustSessionFailure sessionFailureResponseData) {
        // ...
    }
});

Adjust.onCreate(config);
```

수신기 함수는 SDK에서 서버로 패키지를 보내려고 시도한 후에 호출됩니다. 수신기 함수를 통해 수신기 전용 응답 데이터 개체에 액세스할 수 있습니다. 다음은 성공 세션 응답 데이터 개체 필드에 대한 정보를 간략히 요약하고 있습니다.

- `String message` 서버에서 전송된 메시지 또는 SDK에 의해 로깅된 오류
- `String timestamp` 서버에서 전송된 데이터의 타임스탬프
- `String adid` adjust에 의해 제공된 고유 장치 식별자
- `JSONObject jsonResponse` 서버에서 전송된 응답이 있는 JSON 개체

두 개의 이벤트 응답 데이터 개체에는 다음 정보가 포함됩니다.

- `String eventToken` 트래킹한 패키지가 이벤트인 경우 이벤트 토큰

그리고 이벤트 및 세션 실패 개체에는 다음 정보도 포함됩니다.

- `boolean willRetry` 나중에 패키지를 다시 보내려는 시도가 있을 것임을 나타냅니다.

### 18. 지연된 딥링크에 대한 수신기 설정

지연된 딥링크가 열리기 전에 알림을 수신할 수신기를 등록한 후 adjust SDK에서 딥링크를 열지 결정할 수 있습니다.
SDK를 시작하기 전에 `AdjustConfig` 인스턴스를 사용하여 익명 수신기를 추가합니다.

```java
AdjustConfig config = new AdjustConfig(this, appToken, environment);

// evaluate deeplink to be launched
config.setOnDeeplinkResponseListener(new OnDeeplinkResponseListener() {
    @Override
    public boolean launchReceivedDeeplink(Uri deeplink) {
        // ...
        if (allowAdjustSDKToOpenDeeplink(deeplink)) {
            return true;
        } else {
            return false;
        }
    }
});

Adjust.onCreate(config);
```

수신기 함수는 SDK에서 지연된 딥링크를 서버로부터 수신한 후와 딥링크를 열기 전에 호출됩니다.
수신기 함수를 통해 딥링크에 액세스할 수 있으며, 반환하는 부울식에 따라 SDK에서 딥싱크를 실행할지 여부가 결정됩니다.
예를 들어 딥링크를 SDK에서 지금 열지 않고 딥링크를 저장한 후 나중에 직접 열도록 할 수 있습니다.

### 19. 트래킹 사용 중지

`setEnabled`를 `false` 매개변수로 설정한 상태로 호출하면 adjust SDK에서 현재 장치의 모든 작업을 트래킹하지 않도록 할 수 있습니다. 이 설정은
세션 간에 기억됩니다.

```java
Adjust.setEnabled(false);
```

`isEnabled` 함수를 호출하여 adjust SDK가 현재 사용 가능한지 확인할 수 있습니다. 매개변수가 `true`로 설정된 `setEnabled`를 호출하면 adjust SDK를 언제든지 활성화할 수 있습니다.

### 20. 오프라인 모드

adjust SDK를 오프라인 모드로 전환하여 adjust 서버로 전송하는 작업을 일시 중단하고 트래킹된 데이터를 보관하여 나중에 보낼 수 있습니다. 오프라인 모드일 때는 모든 정보가 파일에 저장되므로 오프라인 모드에서 너무 많은 이벤트가 트리거되지 않도록 주의하십시오.

`setOfflineMode`를 `true` 매개변수로 설정한 상태로 호출하면 오프라인 모드를 활성화할 수 있습니다.

```java
Adjust.setOfflineMode(true);
```

또는 `setOfflineMode`를 `false`로 설정한 상태로 호출하면 오프라인 모드를 비활성화할 수 있습니다.
adjust SDK를 다시 온라인 모드로 전환하면 저장된 정보가 모두 올바른 시간 정보와 함께 adjust 서버로 전송됩니다.

트래킹 사용 중지와 달리 이 설정은 세션 간에 *기억되지
않습니다.* 따라서 앱을 오프라인 모드에서 종료한 경우에도 SDK는
항상 온라인 모드로 시작됩니다.

### 21. 장치 ID

Google Analytics와 같은 서비스를 사용하려면 중복 보고가 발생하지 않도록 장치 ID와 클라이언트 ID를 조정해야 합니다.

Google 광고 ID가 필요한 경우 제한 사항으로 인해 백그라운드 스레드에서만 ID를 읽을 수 있습니다.
 `getGoogleAdId` 함수를 컨텍스트와 `OnDeviceIdsRead` 인스턴스와 함께 호출하면 상황에 관계 없이 작동합니다.

```java
Adjust.getGoogleAdId(this, new OnDeviceIdsRead() {
    @Override
    public void onGoogleAdIdRead(String googleAdId) {
        // ...
    }
});
```

`OnDeviceIdsRead` 인스턴스의 `onGoogleAdIdRead` 메서드를 통해 Google 광고 ID에 `googleAdId` 변수로 액세스할 수 있습니다.

## 문제 해결

### "Session failed (Ignoring too frequent session. ...)" 오류가 나타납니다.

이 오류는 일반적으로 설치를 테스트할 때 발생합니다. 앱을 제거하고 다시 설치해도 새 설치를 트리거할 수 없습니다. 서버에서는 SDK가 로컬에서 집계된 세션 데이터를
유실했다고 판단하며 서버에 제공된 장치 관련 정보에 따라 오류 메시지를 무시합니다.

이 동작은 테스트 중에 불편을 초래할 수도 있지만, sandbox 동작이 프로덕션 환경과 최대한
일치하도록 하기 위해 필요합니다.

장치의 세션 데이터를 adjust 서버에서 재설정할 수 있습니다. 로그에서 다음 오류 메시지를 확인합니다.

```
Session failed (Ignoring too frequent session. Last session: YYYY-MM-DDTHH:mm:ss, this session: YYYY-MM-DDTHH:mm:ss, interval: XXs, min interval: 20m) (app_token: {yourAppToken}, adid: {adidValue})
```

아래에 `{yourAppToken}` 및 `{adidValue}` 값을 입력하고 다음 링크를 엽니다.

```
http://app.adjust.com/forget_device?app_token={yourAppToken}&adid={adidValue}
```

장치가 메모리에서 삭제되면 링크에서 `Forgot device`만 반환됩니다. 장치가 이미 메모리에서 삭제되었거나 값이 올바르지 않으면 `Device not found` 메시지가 반환됩니다.

### 브로드캐스트 수신기에서 설치 참조 페이지를 캡처하고 있습니까?

[설명서](#broadcast_receiver)의 지침을 따른 경우 브로드캐스트 수신기는 설치 참조 페이지를 adjust SDK와 adjust 서버로 보내도록 구성됩니다.

테스트 설치 참조 페이지를 수동으로 트리거하여 이 구성을 테스트할 수 있습니다.
`com.your.appid`를 앱 ID로 대체하고 Android Studio와 함께 제공되는 [adb](http://developer.android.com/tools/help/adb.html) 도구를 사용하여 다음 명령을 실행합니다.

```
adb shell am broadcast -a com.android.vending.INSTALL_REFERRER -n com.your.appid/com.adjust.sdk.AdjustReferrerReceiver --es "referrer" "adjust_reftag%3Dabc1234%26tracking_id%3D123456789%26utm_source%3Dnetwork%26utm_medium%3Dbanner%26utm_campaign%3Dcampaign"
```

이미 다른 브로드캐스트 수신기를 `INSTALL_REFERRER` intent로 사용 중이고 이 [설명서][referrer]의 지침을 따른 경우, `com.adjust.sdk.AdjustReferrerReceiver`를 브로드캐스트 수신기로 대체합니다.

`-n com.your.appid/com.adjust.sdk.AdjustReferrerReceiver` 매개변수를 제거하여 장치의 모든 앱에서 `INSTALL_REFERRER` intent를 수신하도록 할 수도 있습니다.

로그 레벨을 `verbose`로 설정하면 참조 페이지

````
V/Adjust: Reading query string (adjust_reftag=abc1234&tracking_id=123456789&utm_source=network&utm_medium=banner&utm_campaign=campaign) from reftag
```

와 SDK 패키지 처리기에 추가된 다음 클릭 패키지를 읽어서 로그를 볼 수 있습니다.

```
V/Adjust: Path:      /sdk_click
    ClientSdk: android4.6.0
    Parameters:
        app_token        abc123abc123
        click_time       yyyy-MM-dd'T'HH:mm:ss.SSS'Z'Z
        created_at       yyyy-MM-dd'T'HH:mm:ss.SSS'Z'Z
        environment      sandbox
        gps_adid         12345678-0abc-de12-3456-7890abcdef12
        needs_attribution_data 1
        referrer         adjust_reftag=abc1234&tracking_id=123456789&utm_source=network&utm_medium=banner&utm_campaign=campaign
        reftag           abc1234
        source           reftag
        tracking_enabled 1
```

앱을 시작하기 전에 이 테스트를 실시하면 전송할 패키지가 보이지 않습니다.
앱이 시작된 후에 패키지가 전송됩니다.

### 응용 프로그램 시작 시 이벤트를 트리거할 수 있습니까?

직관적으로 생각하는 것과는 다를 수 있습니다. 전역 `Application` 클래스의 `onCreate` 메서드는 응용 프로그램 시작 시에만 호출되는 것이 아니라 시스템 또는 응용 프로그램 이벤트가 앱에 의해 캡처될 때도 호출됩니다.

adjust SDK는 이 때 초기화할 준비가 되지만, 실제로 시작되지는 않습니다.
작업이 시작될 때, 즉 사용자가 실제로 앱을 시작할 경우에만 adjust SDK가 실제로 시작됩니다.

따라서 이 때 이벤트를 트리거하면 원하는 작업이 수행되지 않습니다.
이런 호출은 사용자가 앱을 시작하지 않은 경우에도 adjust SDK를 앱의 외부 요인에 따라 결정되는 시간에 시작하고 이벤트를 보냅니다.

따라서 응용 프로그램 시작 시 이벤트를 트리거하면 트래킹되는 설치 및 세션의 수가 부정확해집니다.

설치 후에 이벤트를 트리거하려면 [어트리뷰션 변경 수신기](#attribution_changed_listener)를 사용하십시오.

앱이 시작될 때 이벤트를 트리거하려면 시작된 작업의 `onCreate` 메서드를 사용하십시오.

[dashboard]:     http://adjust.com
[releases]:      https://github.com/adjust/adjust_android_sdk/releases
[import_module]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/01_import_module.png
[select_module]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/02_select_module.png
[imported_module]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/03_imported_module.png
[gradle_adjust]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/04_gradle_adjust.png
[gradle_gps]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/05_gradle_gps.png
[manifest_gps]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/06_manifest_gps.png
[manifest_permissions]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/07_manifest_permissions.png
[proguard]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/08_proguard_new.png
[receiver]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/09_receiver.png
[application_class]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/11_application_class.png
[manifest_application]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/12_manifest_application.png
[application_config]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/13_application_config.png
[log_message]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/15_log_message.png
[activity_lifecycle_class]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/16_activity_lifecycle_class.png
[activity_lifecycle_methods]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/17_activity_lifecycle_methods.png
[activity_lifecycle_register]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/18_activity_lifecycle_register.png

[referrer]:      doc/referrer.md
[attribution-data]:     https://github.com/adjust/sdks/blob/master/doc/attribution-data.md
[google_play_services]: http://developer.android.com/google/play-services/setup.html
[android_application]:  http://developer.android.com/reference/android/app/Application.html
[application_name]:     http://developer.android.com/guide/topics/manifest/application-element.html#nm
[google_ad_id]:         https://support.google.com/googleplay/android-developer/answer/6048248?hl=en
[callbacks-guide]:      https://docs.adjust.com/en/callbacks
[event-tracking]:       https://docs.adjust.com/en/event-tracking
[special-partners]:     https://docs.adjust.com/en/special-partners
[maven]:                http://maven.org
[example]:              https://github.com/adjust/android_sdk/tree/master/Adjust/example
[currency-conversion]:  https://docs.adjust.com/en/event-tracking/#tracking-purchases-in-different-currencies
[activity_resume_pause]: doc/activity_resume_pause.md

## 라이선스

adjust SDK는 MIT 라이선스에 따라 사용이 허가되었습니다.

Copyright (c) 2012-2015 adjust GmbH,
http://www.adjust.com

이로써 본 소프트웨어와 관련 문서 파일(이하 "소프트웨어")의 복사본을 받는 사람에게는 아래 조건에 따라 소프트웨어를 제한 없이 다룰 수 있는 권한이 무료로 부여됩니다. 이 권한에는 소프트웨어를 사용, 복사, 수정, 병합, 출판, 배포 및/또는 판매하거나 2차 사용권을 부여할 권리와 소프트웨어를 제공 받은 사람이 소프트웨어를 사용, 복사, 수정, 병합, 출판, 배포 및/또는 판매하거나 2차 사용권을 부여하는 것을 허가할 수 있는 권리가 제한 없이 포함됩니다.

위 저작권 고지문과 본 권한 고지문은 소프트웨어의 모든 복사본이나 주요 부분에 포함되어야 합니다.

소프트웨어는 상품성, 특정 용도에 대한 적합성 및 비침해에 대한 보증 등을 비롯한 어떤 종류의 명시적이거나 암묵적인 보증 없이 "있는 그대로" 제공됩니다. 어떤 경우에도 저작자나 저작권 보유자는 소프트웨어와 소프트웨어의 사용 또는 기타 취급에서 비롯되거나 그에 기인하거나 그와 관련하여 발생하는 계약 이행 또는 불법 행위 등에 관한 배상 청구, 피해 또는 기타 채무에 대해 책임지지 않습니다.
--END--
