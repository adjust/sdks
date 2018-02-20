## 요약

이 항목에서는 Adjust의 Xamarin SDK에 대해 설명합니다. Adjust에 대한 자세한 내용은 [adjust.com]을 참조하십시오.

## 목차

* [앱 예제](#example-apps)
* [기본 연동](#basic-integration)
   * [SDK 받기](#sdk-get)
   * [프로젝트에 SDK 추가](#sdk-add)
   * [앱에 SDK 프로젝트 레퍼런스 추가](#sdk-add-project)
   * [앱에 SDK DLL 레퍼런스 추가](#sdk-add-dll)
   * [Google Play 서비스 추가](#sdk-gps)
   * [권한 추가](#sdk-permissions)
   * [Proguard 설정](#sdk-proguard)
   * [앱에 SDK 연동](#sdk-integrate)
   * [세션 추적](#session-tracking)
      * [API 레벨 14 이상](#session-tracking-api14) 
      * [API 레벨 9-13](#session-tracking-api9) 
   * [Adjust 로그 기록(logging)](#adjust-logging)
   * [앱 빌드](#build-the-app)
* [부가 기능](#additional-features)
   * [이벤트 추적](#event-tracking)
      * [매출 추적](#revenue-tracking)
      * [매출 중복 제거](#revenue-deduplication)
      * [인앱 구매 검증](#iap-verification)
      * [콜백 파라미터](#callback-parameters)
      * [파트너 파라미터](#partner-parameters)
    * [세션 파라미터](#session-parameters)
      * [세션 콜백 파라미터](#session-callback-parameters)
      * [세션 파트너 파라미터](#session-partner-parameters)
      * [예약 시작(delay start)](#delay-start)
    * [어트리뷰션 콜백](#attribution-callback)
    * [세션 및 이벤트 콜백](#session-event-callbacks)
    * [추적 사용 중지](#disable-tracking)
    * [오프라인 모드](#offline-mode)
    * [이벤트 버퍼링(buffering)](#event-buffering)
    * [백그라운드 추적](#background-tracking)
    * [기기 ID](#device-ids)
      * [Google Play 서비스 광고 식별자](#di-gps-adid)
      * [Adjust 기기 식별자](#di-adid)
    * [푸시 토큰(push token)](#push-token)
    * [사전 설치 트래커(pre-installed trackers)](#pre-installed-trackers)
    * [딥링크](#deeplinking)
        * [기본 딥링크](#deeplinking-standard)
        * [거치(deferred) 딥링크](#deeplinking-deferred)
        * [딥링크를 통한 리어트리뷰션](#deeplinking-reattribution)
* [라이선스](#license)

## <a id="example-apps">앱 예제

[`Android` 디렉토리][demo-app-android]에 안드로이드 앱 예제가 있습니다. Xamarin Studio 프로젝트를 열어 Adjust SDK 연동 방법의 예를 확인할 수 있습니다.

## <a id="basic-integration">기본 연동

다음은 Adjust SDK를 Xamarin 프로젝트와 연동하기 위해 최소한으로 수행해야 하는 절차입니다. 여기서는 Xamarin Studio 또는 Visual Studio를 안드로이드 개발에 사용한다고 가정합니다.

### <a id="sdk-get">SDK 받기

Adjust의 [releases 페이지][releases]에서 최신 버전을 다운로드하세요. 원하는 위치에 압축 파일을 풀면 됩니다.

Adjust 바인딩 DLL를 사용하려면 [이 단계](#sdk-add-dll)에 나오는 내용을 따르십시오.

### <a id="sdk-add">프로젝트를 SDK에 추가

솔루션에 '기존 프로젝트를 추가(add an existing project)'하도록 선택합니다.

![][add_android_binding]

`AdjustSdk.Xamarin.Android` 프로젝트 파일을 선택한 다음 `Open`을 클릭합니다.

![][select_android_binding]

이제 Adjust 안드로이드 바인딩이 솔루션에 서브모듈로 추가되었습니다. 

![][submodule_android_binding]

### <a id="sdk-add-project">앱에 SDK 프로젝트 레퍼런스 추가

Adjust 안드로이드 바인딩을 솔루션에 추가한 다음에는 안드로이드 앱 프로젝트에서 해당 레퍼런스 연결도 추가해야 합니다. 

![][reference_android_binding]

레퍼런스를 프로젝트 레퍼런스를 통하지 않고 Adjust SDK에 추가하려면 이 단계를 그대로 통과하여 DLL 레퍼런스로 앱에 추가할 수도 있습니다. 아래에서 이 방법 설명을 확인할 수 있습니다. 

### <a id="sdk-add-dll">앱에 SDK DLL 레퍼런스 추가

다음 단계로 안드로이드 프로젝트 내 properties 폴더에서 프로퍼티 바인딩 DLL에 레퍼런스를 추가해야 합니다. 레퍼런스 창에서 먼저 `.Net Assembly`를, 그 다음 다운로드한 `AdjustSdk.Xamarin.Android.dll`을 선택합니다.

![][select_android_dll]

### <a id="sdk-gps">Google Play 서비스 추가

2014년 8월 1일 자로 Google Play Store의 앱은 Google 광고 ID를 사용하여 장치를 고유하게 식별해야 합니다. Adjust SDK에서 Google 광고 ID를 사용할 수 있게 하려면 Google Play 서비스를 연동해야 합니다. 이 작업을 아직 수행하지 않은 경우 다음 단계를 수행하십시오.

1. 안드로이드 앱 프로젝트 내 `Packages` 폴더에서 `Add Packages`를 선택합니다.

	![][add_packages]

2. `Xamarin Google Play Services - Analytics`를 찾아 앱에 추가합니다.

	![][add_gps_to_app]

3. 안드로이드 앱 프로젝트에 Google Play Services Analytics 추가가 끝나면 `Packages` 폴더의 내용물이 다음과 같이 보일 것입니다. 

	![][gps_added]

### <a id="sdk-add-gps">권한 추가

`Properties` 폴더에서 안드로이드 앱 프로젝트의 `AndroidManifest.xml` 파일을 엽니다. `INTERNET` 권한이 없을 경우 역시 추가합니다.

![][permission_internet]

Google Play Store가 대상이 **아닌** 경우, `INTERNET` 및 `ACCESS_WIFI_STATE` 권한을 추가합니다.

![][permission_wifi_state]

### <a id="sdk-proguard"></a>Proguard 설정

Proguard를 사용 중인 경우 다음 행을 Proguard 파일에 추가합니다.

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

**Google Play Store가 대상이 아닌** 경우 `com.google.android.gms` 규칙을 제거할 수 있습니다.

### <a id="sdk-integrate">앱에 SDK 연동

먼저 기본 세션 트래킹을 설정합니다.

SDK 초기화를 위해 전역 안드로이드 [Application][android_application] 클래스 사용을 권장합니다. 앱에 없을 경우 `Application` 확장 클래스를 생성하십시오.

![][application_class]

`Application` 클래스에서 `onCreate` 매서드를 찾거나 만든 후 아래 코드를 추가하여 Adjust SDK를 초기화합니다. 

```cs
using Com.Adjust.Sdk;

// ...

string appToken = "{YourAppToken}";
string environment = AdjustConfig.EnvironmentSandbox;

var config = new AdjustConfig(this, appToken, environment);

Adjust.OnCreate(config);
```

`{YourAppToken}`을 앱 토큰으로 대체합니다. 앱 토큰은 [대시보드][adjust.com]에서 찾을 수 있습니다.

앱을 테스트에 사용할지 아니면 프로덕션에 사용할지에 따라 `environment`를 다음 값 중 하나로 설정해야 합니다.

```cs
const String environment = AdjustConfig.EnvironmentSandbox;
const String environment = AdjustConfig.EnvironmentProduction;
```

**중요:** 이 값은 앱을 테스트하는 경우에만 `AdjustConfig.EnvironmentSandbox`로 설정해야 합니다. 앱을 게시하기 전에 environment를 `AdjustConfig.EnvironmentProduction`으로 설정해야 합니다. 개발 및 테스트를 다시 시작할 경우 `AdjustConfig.EnvironmentSandbox`로 다시 설정하십시오.

이 environment는 실제 트래픽과 테스트 장치의 테스트 트래픽을 구별하기 위해 사용합니다. 이 값을 항상 유의미하게 유지해야 합니다! 매출을 추적하는 경우에 특히 중요합니다.

### <a id="session-tracking"></a>세션 추적

**주의:** 이 단계는 **매우 중요**하므로 **앱에 제대로 구현했는지 반드시 확인하세요**. 구현하게 되면 Adjust SDK가 제공하는 세션 추적을 앱에서 올바르게 활성화할 수 있습니다.

### <a id="session-tracking-api14"></a>API 레벨 14 이상

1. `Java.Lang.Object` 및 `IActivityLifecycleCallbacks` 인터페이스를 구현하는 비공개 클래스를 추가합니다. 이 인터페이스에 액세스할 권한이 없다면, 앱의 대상 안드로이드 API 레벨이 14보다 낮기 때문이며 이 경우 다음 [지침](#session-tracking-api9)에 따라 각 작업을 수동으로 업데이트해야 합니다. 이전에 앱의 각 작업에 대한 `Adjust.onResume` 및 `Adjust.onPause` 호출이 있었을 경우 각 호출을 제거해야 합니다.

2. `onActivityResumed(Activity activity)` 메서드를 편집하고 `Adjust.onResume()`에 호출을 추가합니다. `onActivityPaused(Activity activity)` 메서드를 편집하고 `Adjust.onPause()`에 호출을 추가합니다.

3. Adjust SDK 환경 구성을 마쳤으면 `RegisterActivityLifecycleCallbacks` 호출을 앞서 만든 `ActivityLifecycleCallbacks` 클래스의 인스턴스와 함께 추가합니다.

```cs

using Com.Adjust.Sdk;

    // ...

    [Application(AllowBackup = true)]
    public class GlobalApplication : Application
    {
        public GlobalApplication(IntPtr javaReference, JniHandleOwnership transfer) : base(javaReference, transfer)
        {
        }

        public override void OnCreate()
        {
            base.OnCreate();

            string appToken = "{YourAppToken}";
            string environment = AdjustConfig.EnvironmentSandbox;

            var config = new AdjustConfig(this, appToken, environment);

            Adjust.OnCreate(config);

            RegisterActivityLifecycleCallbacks(new ActivityLifecycleCallbacks());
        }

        private class ActivityLifecycleCallbacks : Java.Lang.Object, IActivityLifecycleCallbacks
        {
            public void OnActivityPaused(Activity activity)
            {
                Adjust.OnPause();
            }

            public void OnActivityResumed(Activity activity)
            {
                Adjust.OnResume()
            }

            public void OnActivityCreated(Activity activity, Bundle savedInstanceState)
            {
            }

            public void OnActivityDestroyed(Activity activity)
            {
            }

            public void OnActivitySaveInstanceState(Activity activity, Bundle outState)
            {
            }

            public void OnActivityStarted(Activity activity)
            {
            }

            public void OnActivityStopped(Activity activity)
            {
            }
        }
    }
    ```

    ![][session_tracking_new]

### <a id="session-tracking-api9"></a>API 레벨 9-13

앱 최소 지원 API 레벨이 `9`와 `13` 사이일 경우, 장기 연동 절차를 간소화하기 위해 `14` 이상으로의 업그레이드를 고려해 주십시오. 주요 버전의 최신 시장 점유율을 확인하려면 안드로이드 [대시보드][adjust.com]를 참조하십시오.

API 저레벨을 여전히 대상으로 삼는 경우, 올바른 세션 추적을 위해 작업 재개 혹은 중지가 있을 때마다 Adjust SDK 메서드 몇 가지를 호출해야 합니다. 그렇게 하지 않으면 SDK가 세션 시작이나 종결을 놓칠 수 있습니다. 이 작업을 위해서는 **앱 내 각 작업 시 다음 단계를 수행해야 합니다**.  

1. 작업 소스 파일을 엽니다.
2. 파일 맨 위에 `using` 문을 추가합니다.
3. 작업 `OnResume` 메소드에서 `Adjust.OnResume` 메서드를 호출합니다. 필요 시에는 해당 메소드를 직접 만듭니다.
4. 작업 `OnPause` 메소드에서 `Adjust.OnPause` 메소드를 호출합니다. 필요 시에는 해당 메소드를 직접 만듭니다. 

단계 수행 후 작업이 다음과 같이 보여야 합니다.

```cs

using Com.Adjust.Sdk;

// ...

[Activity(Label = "Example", MainLauncher = true)]
public class MainActivity : Activity
{
    protected override void OnCreate(Bundle savedInstanceState)
    {
        base.OnCreate(savedInstanceState);
    }

    protected override void OnResume()
    {
        base.OnResume();

        Adjust.OnResume();
    }

    protected override void OnPause()
    {
        base.OnPause();

        Adjust.OnPause();
    }
}
```

![][session_tracking_old]

**모든** 앱 작업에서 이같은 단계를 반복해야 하므로 신규 작업 생성 시에 잊지 마세요. 코딩 유형에 따라 작업 전체에서 이를 공통 슈퍼클래스로 설정해야 할 수도 있습니다.

### <a id="adjust-logging">Adjust 로그 기록

다음 파라미터 중 하나를 사용하여 `AdjustConfig` 인스턴스에서 `SetLogLevel`을 호출하면 테스트에 표시되는 로그의 양을 늘리거나 줄일 수 있습니다.

```cs
config.SetLogLevel(LogLevel.Verbose); 	// 모든 로그 활성화
config.SetLogLevel(LogLevel.Debug);   	// 더 많은 로그 활성화
config.SetLogLevel(LogLevel.Info);    	// 기본값
config.SetLogLevel(LogLevel.Warn);    	// info 로그 비활성화
config.SetLogLevel(LogLevel.Error);   	// 경고 역시 비활성화
config.SetLogLevel(LogLevel.Assert);  	// 오류 역시 비활성화
config.SetLogLevel(LogLevel.Supress);	// 모든 로그 비활성화
```

프로덕션 단계의 앱이 Adjust SDK 로그를 출력하지 않게 하려면 `LogLevel.Supress`를 선택하고 `AdjustConfig` 개체에서 생성자를 초기화해야 합니다. 이 생성자에서 supress 로그 레벨 모드를 활성화할 수 있습니다.

```cs

using Com.Adjust.Sdk;

// ...

string appToken = "{YourAppToken}";
string environment = AdjustConfig.EnvironmentSandbox;

AdjustConfig config = new AdjustConfig(this, appToken, environment, true);
config.SetLogLevel(LogLevel.Supress);

Adjust.OnCreate(config);
```

### <a id="build-your-app">앱 빌드

안드로이드 앱을 작성하고 실행합니다. 빌드가 성공적이면 콘솔에서 SDK 로그를 읽을 수 있습니다. 앱이 최초 런칭된 다음 `Install tracked`가 정보 로그로 표시됩니다. 

![][run]

## <a id="additional-features">추가 기능

Adjust SDK를 프로젝트와 연동한 후에는 다음 기능을 사용할 수 있습니다.

### <a id="event-tracking">이벤트 추적

Adjust를 사용하여 앱 이벤트를 추적할 수 있습니다. 특정 버튼을 클릭하는 경우 이를 매번 추적하려면 [대시보드][adjust.com]에서 해당 이벤트 토큰과 연결되는 새 이벤트 토큰을 만들어야 합니다. 이벤트 토큰이 `abc123`일 경우, 버튼의 `Click` 메서드에 다음 라인을 추가하여 클릭을 추적할 수 있습니다.

```cs
AdjustEvent eventClick = new AdjustEvent("abc123");

Adjust.TrackEvent(eventClick);
```

버튼을 클릭하면 로그에 `Event tracked`가 표시됩니다. 

이벤트 인스턴스는 추적할 이벤트 환경을 실제 추적 전에 더 자세히 설정할 때 사용할 수 있습니다.

### <a id="revenue-tracking">매출 추적

사용자가 광고를 누르거나 인앱 구매를 할 때 매출이 발생하는 경우, 이벤트를 사용하여 해당 매출을 추적할 수 있습니다. 예를 들어 광고를 한 번 누를 때 0.01 유로의 매출이 발생한다면 매출 이벤트를 다음과 같이 추적할 수 있습니다.

```cs
AdjustEvent eventRevenue = new AdjustEvent("abc123");

adjustEvent.SetRevenue(0.01, "EUR");

Adjust.TrackEvent(adjustEvent);
```

통화 토큰을 설정할 경우, Adjust가 자동으로 들어오는 매출을 미리 지정한 보고용 통화로 전환해 줍니다. 통화 전환에 관한 자세한 내용은 [여기][currency-conversion]에서 확인하세요.

매출 및 이벤트 추적에 관한 자세한 내용은 [이벤트 추적 설명서][event-tracking]에서 확인하세요.

### <a id="revenue-deduplication">매출 중복 제거

거래 ID를 선택 사항으로 추가하여 매출 중복 추적을 피할 수 있습니다. 가장 최근에 사용한 거래 ID 10개를 기억하며, 중복 거래 ID로 이루어진 매출 이벤트는 집계하지 않습니다. 인앱 구매 추적 시 특히 유용합니다. 사용 예는 아래에 나와 있습니다.

인앱 구매를 추적할 경우, 거래가 완료되고 아이템을 구매했을 때만 `TrackEvent`를 호출해야 한다는 사실을 기억하십시오. 그렇게 해야 실제로 발생하지 않은 매출을 추적하는 일을 피할 수 있습니다.

```cs
AdjustEvent adjustEvent = new AdjustEvent("abc123");

adjustEvent.SetRevenue(0.01, "EUR");
adjustEvent.SetOrderId("YourOrderId");

Adjust.TrackEvent(adjustEvent);
```

### <a id="iap-verification">인앱 구매 검증

인앱 구매 검증을 가능하게 해 줄 Xamarin 구매 SDK가 현재 개발 중이며 곧 출시될 예정입니다. 자세한 내용을 확인하려면 support@adjust.com으로 연락해 주십시오.

### <a id="callback-parameters">콜백 파라미터

[대시보드][adjust.com]에서 이벤트 콜백 URL을 등록할 수 있습니다. 이벤트를 추적할 때마다 GET 요청이 해당 URL로 전송됩니다. 이벤트를 추적하기 전에 이벤트 인스턴스에서 `AddCallbackParameter`를 호출하여 콜백 파라미터를 해당 이벤트에 추가할 수 있습니다. 그러면 해당 파라미터가 콜백 URL에 추가됩니다.

예를 들어 URL `http://www.adjust.com/callback`을 등록했다면 이벤트를 다음과 같이 추적할 수 있습니다.

```cs
AdjustEvent adjustEvent = new AdjustEvent("abc123");

adjustEvent.AddCallbackParameter("key", "value");
adjustEvent.AddCallbackParameter("foo", "bar");

Adjust.TrackEvent(adjustEvent);
```

이 경우에는 이벤트를 추적하고 다음 주소로 요청이 전송됩니다.

```
http://www.adjust.com/callback?key=value&foo=bar
```

Adjust는 파라미터 값으로 사용할 수 있는 `{gps_adid}`와 같은 다양한 자리 표시자(placeholder)를 지원합니다. 그 결과로 생성한 콜백에서 이 자리 표시자는 현재 기기의 Google Play 서비스 ID로 대체됩니다. 사용자 지정 파라미터는 저장되지 않으며 콜백에만 추가된다는 사실도 기억해 주십시오. 이벤트에 대한 콜백을 등록하지 않은 경우 해당 파라미터는 읽을 수 없습니다.

사용 가능한 값의 전체 목록을 포함한 URL 콜백 사용에 대한 자세한 내용은 [콜백 설명서][callbacks-guide]를 참조하십시오.

### <a id="partner-parameters">파트너 파라미터

Adjust 대시보드에서 활성화된 연동에 대해 네트워크 파트너로 전송할 파라미터도 추가할 수 있습니다.

위에서 설명한 콜백 매개변수의 경우와 비슷하지만, `AdjustEvent` 인스턴스에서 `AddPartnerParameter` 메서드를 호출해야 추가할 수 있습니다.

```cs
AdjustEvent adjustEvent = new AdjustEvent("abc123");

adjustEvent.AddPartnerParameter("key", "value");
adjustEvent.AddPartnerParameter("foo", "bar");

Adjust.TrackEvent(adjustEvent);
```

특별 파트너와 그 연동에 대한 자세한 내용은 [특별 파트너 설명서][special-partners]를 참조하십시오.

### <a id="session-parameters">세션 파라미터

일부 파라미터는 Adjust SDK 이벤트 및 세션 발생시마다 전송을 위해 저장합니다. 어느 파라미터든 한 번 저장하면 로컬에 바로 저장되므로 매번 새로 추가할 필요가 없습니다. 같은 파라미터를 두 번 저장해도 효력이 없습니다.

이 세션 파라미터는 설치 시에도 전송할 수 있도록 Adjust SDK 런칭 전에도 호출할 수 있습니다. 설치 시에 전송하지만 필요한 값은 런칭 후에야 들어갈 수 있게 하고 싶다면 Adjust SDK 런칭 시 [예약 시작](#delay-start)을 걸 수 있습니다. 

### <a id="session-callback-parameters"> 세션 콜백 파라미터

[이벤트](#callback-parameters)에 등록한 콜백 파라미터는 Adjust SDK 전체 이벤트 및 세션 시 전송할 목적으로 저장할 수 있습니다.

세션 콜백 파라미터는 이벤트 콜백 파라미터와 비슷한 인터페이스를 지녔지만, 이벤트에 키, 값을 추가하는 대신 `Adjust.AddSessionCallbackParameter(String key, String value)` 메서드를 호출하여 추가합니다.

```cs
Adjust.AddSessionCallbackParameter("foo", "bar");
```

세션 콜백 파라미터는 이벤트에 추가된 콜백 파라미터와 합쳐지며, 이벤트에 추가된 콜백 파라미터가 우선권을 지닙니다. 그러나 세션에서와 같은 키로 이벤트에 콜백 파라미터를 추가한 경우 새로 추가한 콜백 파라미터가 우선권을 가집니다.

원하는 키를 `Adjust.RemoveSessionCallbackParameter(String key)` 메서드로 전달하여 특정 세션 콜백 파라미터를 제거할 수 있습니다.

```cs
Adjust.RemoveSessionCallbackParameter("foo");
```

세션 콜백 파라미터의 키와 값을 전부 없애고 싶다면 `Adjust.ResetSessionCallbackParameters()` 메서드로 재설정하면 됩니다.

```cs
Adjust.ResetSessionCallbackParameters();
```

### <a id="session-partner-parameters"> 세션 파트너 파라미터

Adjust SDK 내 모든 이벤트 및 세션에서 전송되는 [세션 콜백 파라미터](#session-callback-parameters)가 있는 것처럼, 세션 파트너 파라미터도 있습니다.

이들 파라미터는 Adjust [대시보드](adjust.com)에서 연동을 활성화한 네트워크 파트너에게 전송할 수 있습니다.

세션 파트너 파라미터는 이벤트 파트너 파라미터와 인터페이스가 비슷하지만, 이벤트에 키, 값을 추가하는 대신 `Adjust.AddSessionPartnerParameter(String key, String value)` 메서드를 호출하여 추가합니다.

```cs
Adjust.AddSessionPartnerParameter("foo", "bar");
```

세션 파트너 파라미터는 이벤트에 추가한 파트너 파라미터와 합쳐지며, 이벤트에 추가된 파트너 파라미터가 우선순위를 지닙니다. 그러나 세션에서와 같은 키로 이벤트에 파트너 파라미터를 추가한 경우, 새로 추가한 파트너 파라미터가 우선권을 가집니다.

원하는 키를 `Adjust.RemoveSessionPartnerParameter(String key)` 메서드로 전달하여 특정 세션 파트너 파라미터를 제거할 수 있습니다.

```cs
Adjust.RemoveSessionPartnerParameter("foo");
```

세션 파트너 파라미터의 키와 값을 전부 없애고 싶다면 `Adjust.ResetSessionPartnerParameters()` 메서드로 재설정하면 됩니다.

```cs
Adjust.ResetSessionPartnerParameters();
```

### <a id="delay-start"> 예약 시작

Adjust SDK에 예약 시작을 걸면 앱이 고유 식별자 등의 세션 파라미터를 얻어 설치 시에 전송할 시간을 벌 수 있습니다.

`AdjustConfig` 인스턴스의 `SetDelayStart` 메서드에서 예약 시작 시각을 초 단위로 설정하세요.

```cs
config.SetDelayStart(5.5);
```

이 경우 Adjust SDK는 최초 인스톨 세션 및 생성된 이벤트를 5.5초간 기다렸다가 전송합니다. 이 시간이 지난 후, 또는 그 사이에 `Adjust.SendFirstPackages()`을 호출했을 경우 모든 세션 파라미터가 지연된 인스톨 세션 및 이벤트에 추가되며 Adjust SDK는 원래대로 돌아옵니다.

**Adjust SDK의 최대 지연 예약 시작 시간은 10초입니다**.

### <a id="attribution-callback">어트리뷰션 콜백

콜백을 등록하여 트래커 어트리뷰션 변경 알림을 받을 수 있습니다. 어트리뷰션에서 고려하는 소스가 각각 다르기 때문에 이 정보는 동시간에 제공할 수 없습니다. 어트리뷰션 콜백은 선택 사항이며, 앱에 구현하려면 다음과 같이 하면 됩니다. 

[해당 어트리뷰션 데이터 정책][attribution-data]을 반드시 고려하세요.

1. 어플리케이션 클래스에서 `IOnAttributionChangedListener` 인터페이스를 구현합니다. 

    ```cs
    [Application (AllowBackup = true)]
	public class GlobalApplication : Application, IOnAttributionChangedListener
	{
	    ...
	}
	```
2. `OnAttributionChanged` 콜백을 오버라이드(override)합니다. 이 콜백은 어트리뷰션에 변동이 생기면 촉발됩니다. 

	```cs
	public void OnAttributionChanged(AdjustAttribution attribution)
	{
	    Console.WriteLine("Attribution changed!");
	    Console.WriteLine("New attribution: {0}", attribution.ToString ());
	}
	```

3. 어플리케이션 클래스 인스턴스를 `AdjustConfig` 개체 내 수신기(listner)로 설정합니다. 

	```cs
	AdjustConfig config = new AdjustConfig(this, yourAppToken, environment);

	config.SetOnAttributionChangedListener(this);

	Adjust.OnCreate (config);
	```

콜백 함수는 SDK가 최종 어트리뷰션 데이터를 접수한 후 호출됩니다. 콜백 함수에서는 `attribution` 파라미터에 액세스할 수 있습니다. 해당 파라미터에 대한 개요는 다음과 같습니다.

- `string TrackerToken` 현재 설치된 트래커 토큰.
- `string TrackerName` 현재 설치된 트래커 이름.
- `string Network` 현재 설치된 네트워크 그룹화 기준.
- `string Campaign` 현재 설치된 캠페인 그룹화 기준.
- `string Adgroup` 현재 설치된 광고 그룹 그룹화 기준.
- `string Creative` 현재 설치된 크리에이티브 그룹화 기준.
- `string ClickLabel` 현재 설치된 클릭 레이블.
- `string Adid` Adjust 기기 식별자.


### <a id="session-event-callbacks">세션 및 이벤트 콜백

콜백을 등록하여 이벤트나 세션 추적 시 알림을 받을 수 있습니다. 어트리뷰션 콜백과 마찬가지로 개인 설정 클래스에서 수행해야 하며 여기서 각 콜백 메서드에 해당하는 인터페이스를 구현할 수 있습니다.

아래는 이벤트 추적 성공 시의 콜백 함수이며, 이를 구현하려면 다음 단계를 수행하십시오. `IOnEventTrackingSucceededListener` 인터페이스를 구현하는 `AdjustConfig` 인스턴스에 수신기 개체를 설정해야 합니다. 

```cs
[Application(AllowBackup = true)]
public class GlobalApplication : Application, IOnEventTrackingSucceededListener
{
    public override void OnCreate()
    {
        base.OnCreate();

        string appToken = "{YourAppToken}";
        string environment = AdjustConfig.EnvironmentSandbox;
        AdjustConfig config = new AdjustConfig(this, appToken, environment);

        // Set event tracking success callback.
        config.SetOnEventTrackingSucceededListener(this);

        Adjust.OnCreate(config);
    }

    public void OnFinishedEventTrackingSucceeded(AdjustEventSuccess eventSuccess)
    {
        Console.WriteLine("Event tracking succeeded! " + eventSuccess.ToString());
    }
}
```

이벤트 추적 실패 시에는 `IOnEventTrackingFailedListener` 인터페이스를 구현하는 `AdjustConfig` 인스턴스에 수신기 개체를 설정해야 합니다. 

```cs
[Application(AllowBackup = true)]
public class GlobalApplication : Application, IOnEventTrackingFailedListener
{
    public override void OnCreate()
    {
        base.OnCreate();

        string appToken = "{YourAppToken}";
        string environment = AdjustConfig.EnvironmentSandbox;
        AdjustConfig config = new AdjustConfig(this, appToken, environment);

        // Set event tracking failure callback.
        config.SetOnEventTrackingFailedListener(this);

        Adjust.OnCreate(config);
    }

    public void OnFinishedEventTrackingFailed(AdjustEventFailure eventFailure)
    {
        Console.WriteLine("Event tracking failed! " + eventFailure.ToString());
    }
}
```

세션 추적 성공 시에는 `IOnSessionTrackingSucceededListener` 인터페이스를 구현하는 `AdjustConfig` 인스턴스에 수신기 개체를 설정해야 합니다. 

```cs
[Application(AllowBackup = true)]
public class GlobalApplication : Application, IOnSessionTrackingSucceededListener
{
    public override void OnCreate()
    {
        base.OnCreate();

        string appToken = "{YourAppToken}";
        string environment = AdjustConfig.EnvironmentSandbox;
        AdjustConfig config = new AdjustConfig(this, appToken, environment);

        // Set session tracking success callback.
        config.SetOnSessionTrackingSucceededListener(this);

        Adjust.OnCreate(config);
    }

    public void OnFinishedSessionTrackingSucceeded(AdjustSessionSuccess sessionSuccess)
    {
        Console.WriteLine("Session tracking succeeded! " + sessionSuccess.ToString());
    }
}
```

세션 추적 실패 시에는 `IOnSessionTrackingFailedListener` 인터페이스를 구현하는 `AdjustConfig` 인스턴스에 수신기 개체를 설정해야 합니다. 

```cs
[Application(AllowBackup = true)]
public class GlobalApplication : Application, IOnSessionTrackingFailedListener
{
    public override void OnCreate()
    {
        base.OnCreate();

        string appToken = "{YourAppToken}";
        string environment = AdjustConfig.EnvironmentSandbox;
        AdjustConfig config = new AdjustConfig(this, appToken, environment);

        // Set session tracking failure callback.
        config.SetOnSessionTrackingFailedListener(this);

        Adjust.OnCreate(config);
    }

    public void OnFinishedSessionTrackingFailed(AdjustSessionFailure sessionFailure)
    {
        Console.WriteLine("Session tracking failed! " + sessionFailure.ToString());
    }
}
```

콜백 함수는 SDK가 서버에 패키지 전송을 시도한 후에 호출됩니다. 해당 콜백에서 수신기에 대한 특정 응답 데이터 개체에 액세스할 수 있습니다. 세션 응답 데이터 필드에 등장하는 프로퍼티 개요는 다음과 같습니다.

- `String Message` 서버에서 전송한 메시지 또는 SDK가 기록한 오류
- `String Timestamp` 서버에서 전송한 데이터의 타임스탬프
- `String Adid` Adjust가 제공하는 고유 기기 식별자
- `JSONObject JsonResponse` 서버로부터의 응답이 있는 JSON 개체

이벤트 응답 데이터 개체 두 가지에는 다음 정보가 포함됩니다.

- `String EventToken` 추적 패키지가 이벤트인 경우 이벤트 토큰

그리고 이벤트 및 세션 실패 개체에는 다음 정보도 포함됩니다.

- `bool WillRetry` 나중에 패키지 재전송 시도가 있을 것임을 나타냅니다.

### <a id="disable-tracking">추적 사용 중지

`Enabled` 프로퍼티를 `false` 파라미터로 설정한 상태로 호출하면 Adjust SDK에서 현재 기기의 모든 작업 추적을 중지할 수 있습니다. **이 설정은 세션 간에 기억**되지만, 활성화하려면 최초 세션이 끝나야 합니다.

```cs
Adjust.Enabled = false;
```

`Enabled` 프로퍼티를 사용하면 Adjust SDK가 현재 사용 가능한지 확인할 수 있습니다. 파라미터가 `true`로 설정된 `Enabled`를 호출하면 Adjust SDK를 언제든 활성화할 수 있습니다.

### <a id="offline-mode">오프라인 모드

Adjust SDK를 오프라인 모드로 전환하여 서버로 전송하는 작업을 일시 중단하고 추적 데이터를 보관하여 나중에 보낼 수 있습니다. 오프라인 모드에서는 모든 정보가 파일에 저장되므로, 이때 너무 많은 이벤트를 촉발(trigger)하지 않도록 주의하십시오.

`SetOfflineMode`를 `true`로 설정하여 호출하면 오프라인 모드를 활성화할 수 있습니다.

```cs
Adjust.SetOfflineMode (true);
```

반대로, `SetOfflineMode`를 `false`로 설정한 상태로 호출하면 오프라인 모드를 비활성화할 수 있습니다. Adjust SDK를 다시 온라인 모드로 전환하면 저장된 정보가 모두 올바른 시간 정보와 함께 Adjust 서버로 전송됩니다.

추적 사용 중지와 달리 이 설정은 세션 간에 **기억되지 않습니다.** 따라서 앱을 오프라인 모드에서 종료한 경우에도 SDK는 항상 온라인 모드로 시작됩니다.

### <a id="event-buffering">이벤트 버퍼링

앱이 이벤트 추적을 많이 사용하는 경우, 매 분마다 배치(batch) 하나씩만 보내도록 하기 위해 일부 HTTP 요청을 지연시키고자 할 경우가 있을 수 있습니다. 

`AdjustConfig` 인스턴스로 이벤트 버퍼링을 적용할 수 있습니다.

```cs
AdjustConfig config = new AdjustConfig(this, yourAppToken, environment);

config.SetEventBufferingEnabled((Java.Lang.Boolean)true);

Adjust.OnCreate(config);
```

설정된 사항이 없을 경우 이벤트 버퍼링은 **사용 중지 상태가 기본값입니다**

### <a id="background-tracking">백그라운드 추적

Adjust SDK 기본값 행위는 **앱이 백그라운드에 있을 동안에는 HTTP 요청 전송을 잠시 중지**하는 것입니다. `AdjustConfig` 인스턴스에서 이 설정을 바꿀 수 있습니다.

```cs
AdjustConfig config = new AdjustConfig(this, yourAppToken, environment);

config.SetSendInBackground(true);

Adjust.OnCreate(config);
```

설정된 사항이 없을 경우 백그라운드 전송은 **사용 중지 상태가 기본값입니다**

### <a id="device-ids"></a>기기 ID

Adjust SDK로 기기 식별자 몇 가지를 얻을 수 있습니다.

### <a id="di-gps-adid"></a>Google Play 서비스 광고 식별자

Google Analytics와 같은 몇몇 서비스를 사용하려면 중복 보고가 발생하지 않도록 기기 ID와 클라이언트 ID를 조정해야 합니다.

Google 광고 ID가 필요한 경우 제한 사항으로 인해 백그라운드 스레드에서만 ID를 읽을 수 있습니다. `getGoogleAdId` 함수를 컨텍스트 및 `OnDeviceIdsRead` 인스턴스와 함께 호출하면 상황에 관계 없이 작동합니다.

```cs

using Com.Adjust.Sdk;

// ...

[Application(AllowBackup = true)]
public class GlobalApplication : Application, IOnDeviceIdsRead
{
    public override void OnCreate()
    {
        base.OnCreate();

        string appToken = "{YourAppToken}";
        string environment = AdjustConfig.EnvironmentSandbox;

        var config = new AdjustConfig(this, appToken, environment);

        Adjust.OnCreate(config);

        Adjust.GetGoogleAdId(this, this);
    }

    public void OnGoogleAdIdRead(string googleAdId)
    {
        Console.WriteLine("Google Ad Id read: " + googleAdId);
    }
}
```

`OnDeviceIdsRead` 인스턴스의 `onGoogleAdIdRead` 메서드를 통해 Google 광고 ID에 `googleAdId` 변수로 액세스할 수 있습니다.

### <a id="di-adid"></a>Adjust 기기 식별자

Adjust 백엔드는 앱을 인스톨한 각 기기에서 고유한 **Adjust 기기 식별자** (**adid**)를 생성합니다. 이 식별자를 얻으려면 `Adjust` 인스턴스에서 다음 메서드를 호출하면 됩니다.

```cs
String adid = Adjust.Adid;
```

**주의**: **adid** 관련 정보는 Adjust 백엔드가 앱 설치를 추적한 후에만 얻을 수 있습니다. 그 순간부터 Adjust SDK는 기기 **adid** 정보를 갖게 되며 이 메서드로 억세스할 수 있습니다. 따라서 SDK가 초기화되고 앱 설치 추적이 성공적으로 이루어지기 전에는 **adid** 억세스가 **불가능합니다**.

### <a id="user-attribution"></a>사용자 어트리뷰션

[어트리뷰션 콜백 섹션](#attribution-callback)에서 설명한 대로, 이 콜백은 변동이 있을 때마다 새로운 어트리뷰션 관련 정보를 전달할 목적으로 촉발됩니다. 사용자의 현재 어트리뷰션 값 관련 정보에 언제든 억세스하고 싶다면 `Adjust` 인스턴스의 다음 메서드를 호출하면 됩니다.

```cs
AdjustAttribution attribution = Adjust.Attribution;
```

**주의**: 현재 어트리뷰션 관련 정보는 Adjust 백엔드가 앱 인스톨을 추적하여 최초 어트리뷰션 콜백이 촉발된 후에만 얻을 수 있습니다. 그 순간부터 Adjus SDK는 사용자 어트리뷰션 정보를 갖게 되며 이 메서드로 억세스할 수 있습니다. 따라서 SDK가 초기화되고 최초 어트리뷰션 콜백이 촉발되기 전에는 사용자 어트리뷰션 값 억세스가 **불가능합니다**. 

### <a id="push-token"></a>푸시 토큰

푸시 알림 토큰을 전송하려면 **토큰을 받았거나 또는 토큰 값에 변화가 생길 때마다** 아래와 같이 Adjust에 대한 호출을 추가하세요.

```cs
Adjust.SetPushToken(pushNotificationsToken);
```

### <a id="pre-installed-trackers">사전 설치 트래커

Adjust SDK를 사용하여 앱이 사전 설치된 기기를 지닌 사용자를 인식하고 싶다면 다음 절차를 따르세요.

1. [대시보드][adjust.com]에 새 트래커를 생성합니다.

2. 애플리케이션 클래스를 열고 `AdjustConfig` 인스턴스의 기본값 트래커를 다음과 같이 설정합니다.

    ```cs
    AdjustConfig config = new AdjustConfig(this, yourAppToken, environment);

    config.SetDefaultTracker("{TrackerToken}");

    Adjust.OnCreate(config);
    ```

`{TrackerToken}`을 2에서 생성한 트래커 토큰으로 대체합니다. 대시보드에서는 (`http://app.adjust.com/`을 포함하는) 트래커 URL을 표시한다는 사실을 명심하세요. 소스코드에서는 전체 URL을 표시할 수 없으며 6자로 이루어진 토큰만을 명시해야 합니다.

3. 앱 빌드를 실행하세요. 앱 로그 출력에서 다음 라인을 볼 수 있을 것입니다.

    ```
    Default tracker: 'abc123'
    ```

### <a id="deeplinking"></a>딥링크

URL에서 앱으로 딥링크를 거는 옵션이 있는 Adjust 트래커 URL을 사용하고 있다면, 딥링크 URL과 그 내용 관련 정보를 얻을 가능성이 있습니다. 해당 URL 클릭 시 사용자가 이미 앱을 설치한 상태(기본 딥링크)일 수도, 아직 앱을 설치하지 않은 상태(거치 딥링크)일 수도 있습니다. 기본 딥링크 상황에서 안드로이드는 딥링크 내용에 관한 정보 인출을 기본 지원합니다. 안드로이드는 거치 딥링크를 기본 지원하지 않지만, Adjust SDK에서 딥링크 정보를 얻는 데 필요한 도구를 제공합니다.

### <a id="deeplinking-standard">기본 딥링크

사용자가 앱을 설치하고 `deep_link` 파라미터가 들어간 Adjust 트래커 URL을 클릭 시 런칭하도록 만들려 하는 경우에는 앱에 딥링크를 활성화시켜야 합니다. 이는 원하는 **고유 스킴명(unique scheme name)**을 선택하여 사용자가 링크를 클릭하고 앱이 열릴 때 런칭할 작업을 배정하여 이루어집니다. 딥링크를 클릭하고 앱이 열릴 때 런칭시킬 액티비티 클래스에서 몇 가지 속성을 지정하면 됩니다. 적절한 인텐트 필터를 설정하고 스킴명을 붙여야 합니다.

```cs
[Activity(Label = "Example", MainLauncher = true)]
	[IntentFilter
	 (new[] { Intent.ActionView },
		Categories = new[] { Intent.CategoryDefault, Intent.CategoryBrowsable },
		DataScheme = "adjustExample")]
	public class MainActivity : Activity
	{
	    // ...
	}
```

이 설정이 끝나면 배정한 스킴명을 Adjust 트래커 URL의 `deep_link` 파라미터에 사용해야 합니다. 딥링크에 정보가 추가되지 않은 트래커 URL은 아래와 같이 보일 것입니다.

```
https://app.adjust.com/abc123?deep_link=adjustExample%3A%2F%2F
```

URL 내 `deep_link` 파라미터 값은 **URL 인코딩이 되어야 한다**는 사실을 명심하세요.

앱이 위와 같이 설정된 상황에서 이 트래커 URL을 클릭하면 `MainActivity` 인텐트와 함께 앱이 런칭됩니다. `MainActivity` 클래스에서 `deep_link` 파라미터 내용이 자동으로 제공됩니다. URL에서 인코딩이 된 상태라 해도 내용이 전달된 후에는 **인코딩이 이루어지지 않습니다**.

액티비티 내 `launchMode` 설정에 따라 `deep_link` 파라미터 내용 정보가 액티비티 파일 내 적절한 위치로 전달됩니다. `launchMode` 프로퍼티에 대한 더 자세한 정보는 [안드로이드 문서](https://developer.android.com/guide/topics/manifest/activity-element.html)에서 확인하세요.  

딥링크 내용 정보는 `Intent` 개체를 통해 원하는 작업 내 `OnCreate` 메서드 또는 `OnNewIntent` 메서드로 전달됩니다. 앱을 런칭하고 두 개 메서드 중 하나가 촉발되면 클릭 URL 내 `deep_link` 파라미터에서 실제 딥링크가 전달되도록 할 수 있습니다. 이렇게 하면 이 정보를 사용하여 앱에서 추가 로직을 수행할 수 있게 됩니다.    

딥링크 내용은 이들 두 개 메서드에서 다음과 같이 추출할 수 있습니다.

```cs
using Android.Content;
using Com.Adjust.Sdk;

// ...

protected override void OnCreate(Bundle savedInstanceState)
{
    base.OnCreate(savedInstanceState);

    Intent intent = this.Intent;
    var data = intent.Data;

    // data.ToString() -> This is your deep_link parameter value.
}
```

```cs
using Android.Content;
using Com.Adjust.Sdk;

// ...

protected override void OnNewIntent(Android.Content.Intent intent)
{
    base.OnNewIntent(intent);

    var data = intent.Data;

    // data.ToString() -> This is your deep_link parameter value.
}
```

### <a id="deeplinking-deferred">거치 딥링크

거치 딥링크는 사용자가 `deep_link` 파라미터가 들어 있는 Adjust 트래커 URL을 클릭했으나, 그 시점에 앱이 기기에 설치되어 있지 않은 경우에 발생합니다. 클릭 후 사용자는 Play Store로 재이동하여 앱을 다운로드하게 됩니다. 링크를 처음 연 후 `deep_link` 파라미터 내용이 앱으로 전달됩니다. 

거치 딥링크에서 `deep_link` 파라미터 내용 정보를 얻으려면 `AdjustConfig` 개체에 수신기를 설치해야 합니다. 수신기 개체가 `IOnDeeplinkResponseListener` 인터페이스를 구현하고 `LaunchReceivedDeeplink` 메서드를 오버라이드해야 합니다. Adjust SDK가 백엔드에서 딥링크 정보를 얻으면 수신기가 촉발됩니다.

```cs
[Application(AllowBackup = true)]
public class GlobalApplication : Application, IOnDeeplinkResponseListener
{
    public override void OnCreate()
    {
        base.OnCreate();

        string appToken = "{YourAppToken}";
        string environment = AdjustConfig.EnvironmentSandbox;
        AdjustConfig config = new AdjustConfig(this, appToken, environment);

        // Set deferred deeplink callback.
        config.SetOnDeeplinkResponseListener(this);

        Adjust.OnCreate(config);
    }

    public bool LaunchReceivedDeeplink(Android.Net.Uri deeplink)
    {
        Console.WriteLine("Deferred deeplink arrived! URL = " + deeplink.ToString());

        return true;
        // return false;
    }
}
```

Adjust SDK가 백엔드에서 딥링크 내용 정보를 수신하면 그 내용이 수신기에 전달되며 `bool` 리턴값을 요청합니다. 리턴값은 (기본 딥링크에서와 마찬가지로) Adjust SDK가 스킴명을 배정한 액티비티를 딥링크에서 런칭할 것인지 여부를 표시합니다.    

리턴값을 `true`로 설정하면 작업이 런칭되어 [기본 딥링크](#deeplinking-standard)에서 설명한 것과 똑같은 결과를 구현합니다. SDK가 작업을 런칭하기를 원하지 않는다면, 수신기에서 `false`를 리턴하여 딥링크 내용을 토대로 앱에서 다음 작업을 어떻게 실행할 지 스스로 정할 수 있습니다.

### <a id="deeplinking-reattribution">딥링크를 통한 리어트리뷰션

Adjust는 딥링크를 사용하여 광고 캠페인 리인게이지먼트(re-engagement)를 수행할 수 있게 해줍니다. 이에 대한 자세한 정보는 [관련 문서](https://docs.adjust.com/en/deeplinking/#manually-appending-attribution-data-to-a-deep-link)를 참조하세요. 

이 기능을 사용 중이라면, 사용자를 올바로 리어트리뷰트하기 위해 앱에서 호출을 한 가지 더 수행해야 합니다.

앱에서 딥링크 내용을 수신했다면, `Adjust.AppWillOpenUrl` 메서드 호출을 추가하세요. 이 호출이 이루어지면 Adjust SDK는 딥링크 내에 새로운 어트리뷰션 정보가 있는지 확인하고, 새 정보가 있으면 Adjust 백엔드로 송신합니다. 딥링크 정보가 담긴 Adjust 트래커 URL을 클릭한 사용자를 리어트리뷰트해야 할 경우, 앱에서 해당 사용자의 새 어트리뷰션 정보로 [어트리뷰션 콜백](#attribution-callback)이 촉발되는 것을 확인할 수 있습니다. 

`Adjust.AppWillOpenUrl` 호출은 다음과 같이 이루어집니다.

```cs
using Android.Content;
using Com.Adjust.Sdk;

// ...

protected override void OnCreate(Bundle savedInstanceState)
{
    base.OnCreate(savedInstanceState);

    Intent intent = this.Intent;
    var data = intent.Data;

    // data.ToString() -> This is your deep_link parameter value.

    Adjust.AppWillOpenUrl(data);
}
```

```cs
using Android.Content;
using Com.Adjust.Sdk;

// ...

protected override void OnNewIntent(Android.Content.Intent intent)
{
    base.OnNewIntent(intent);

    var data = intent.Data;

    // data.ToString() -> This is your deep_link parameter value.

    Adjust.AppWillOpenUrl(data);
}
```

[dashboard]:	http://adjust.com
[adjust.com]:	http://adjust.com

[releases]: 		https://github.com/adjust/xamarin_sdk/releases
[event-tracking]: 	https://docs.adjust.com/en/event-tracking
[callbacks-guide]: 	https://docs.adjust.com/en/callbacks
[attribution-data]: 	https://github.com/adjust/sdks/blob/master/doc/attribution-data.md
[special-partners]: 	https://docs.adjust.com/en/special-partners
[demo-app-android]:	/./Android
[android-dashboard]:    http://developer.android.com/about/dashboards/index.html
[android_application]:	http://developer.android.com/reference/android/app/Application.html
[currency-conversion]:	https://docs.adjust.com/en/event-tracking/#tracking-purchases-in-different-currencies
[android-launch-modes]:	https://developer.android.com/guide/topics/manifest/activity-element.html

[reattribution-with-deeplinks]: https://docs.adjust.com/en/deeplinking/#manually-appending-attribution-data-to-a-deep-link

[run]: 			https://github.com/adjust/sdks/blob/master/Resources/xamarin/android/run.png
[gps_added]: 		https://github.com/adjust/sdks/blob/master/Resources/xamarin/android/gps_added.png
[add_packages]: 	https://github.com/adjust/sdks/blob/master/Resources/xamarin/android/add_packages.png
[add_gps_to_app]: 	https://github.com/adjust/sdks/blob/master/Resources/xamarin/android/add_gps_to_app.png
[application_class]: 	https://github.com/adjust/sdks/blob/master/Resources/xamarin/android/application_class.png
[select_android_dll]: 	https://github.com/adjust/sdks/blob/master/Resources/xamarin/android/select_android_dll.png
[permission_internet]: 	https://github.com/adjust/sdks/blob/master/Resources/xamarin/android/permission_internet.png
[add_android_binding]:	https://github.com/adjust/sdks/blob/master/Resources/xamarin/android/add_android_binding.png
[session_tracking_old]:	https://github.com/adjust/sdks/blob/master/Resources/xamarin/android/session_tracking_old.png
[session_tracking_new]:	https://github.com/adjust/sdks/blob/master/Resources/xamarin/android/session_tracking_new.png

[submodule_ios_binding]:     https://github.com/adjust/sdks/blob/master/Resources/xamarin/ios/submodule_ios_binding.png
[permission_wifi_state]:     https://github.com/adjust/sdks/blob/master/Resources/xamarin/android/permission_wifi_state.png
[select_android_binding]:    https://github.com/adjust/sdks/blob/master/Resources/xamarin/android/select_android_binding.png

[submodule_android_binding]: https://github.com/adjust/sdks/blob/master/Resources/xamarin/android/submodule_android_binding.png
[reference_android_binding]: https://github.com/adjust/sdks/blob/master/Resources/xamarin/android/reference_android_binding.png

## <a id="license">라이선스

Adjust SDK는 MIT 라이선스에 따라 사용이 허가되었습니다.

Copyright (c) 2012-2017 Adjust GmbH,
http://www.adjust.com

이로써 본 소프트웨어와 관련 문서 파일(이하 "소프트웨어")의 복사본을 받는 사람에게는 아래 조건에 따라 소프트웨어를 제한 없이 다룰 수 있는 권한이 무료로 부여됩니다. 이 권한에는 소프트웨어를 사용, 복사, 수정, 병합, 출판, 배포 및/또는 판매하거나 2차 사용권을 부여할 권리와 소프트웨어를 제공 받은 사람이 소프트웨어를 사용, 복사, 수정, 병합, 출판, 배포 및/또는 판매하거나 2차 사용권을 부여하는 것을 허가할 수 있는 권리가 제한 없이 포함됩니다.

위 저작권 고지문과 본 권한 고지문은 소프트웨어의 모든 복사본이나 주요 부분에 포함되어야 합니다.

소프트웨어는 상품성, 특정 용도에 대한 적합성 및 비침해에 대한 보증 등을 비롯한 어떤 종류의 명시적이거나 암묵적인 보증 없이 "있는 그대로" 제공됩니다. 어떤 경우에도 저작자나 저작권 보유자는 소프트웨어와 소프트웨어의 사용 또는 기타 취급에서 비롯되거나 그에 기인하거나 그와 관련하여 발생하는 계약 이행 또는 불법 행위 등에 관한 배상 청구, 피해 또는 기타 채무에 대해 책임지지 않습니다.
--END--
