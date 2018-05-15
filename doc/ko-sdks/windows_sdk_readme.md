## 요약

이 항목에서는 adjust의 Windows SDK에 대해 설명합니다. adjust에 대한 자세한 내용은 [adjust.com](http://adjust.com)을 참조하십시오.

## 앱 예제

[`Adjust` 디렉터리][example]에 다양한 앱 예제가 있습니다. `AdjustWP80Example`은 Windows Phone 8.0, `AdjustWP81Example`은 Windows Phone 8.1, `AdjustWSExample`은 Windows Store 앱의 예입니다. 이런 프로젝트 예제를 사용하여 adjust SDK를 앱과 연동할 수 있는 방법을 확인할 수 있습니다.

## 기본 설치

다음은 adjust SDK를 Windows Phone 또는 Windows Store 프로젝트에 연동하기 위해 최소한으로 수행해야 하는 절차입니다. 이 항목에서는 최신 NuGet 패키지 관리자가 설치된 Visual Studio 2013 이상을 사용한다고 가정합니다. Windows Phone 8.0 또는 Windows 8을 지원하는 이전 버전도 사용할 수 있습니다. 스크린샷에는 Windows Universal 앱 연동 프로세스가 나와 있지만, Windows Store 또는 Phone 앱 모두 절차는 매우 비슷합니다. Windows Phone 8.0과의 차이점은 설명에 계속 나와 있습니다.

### 1. NuGet을 사용하여 Adjust 패키지를 설치합니다.

Visual Studio 메뉴에서 `TOOLS a†’ Library Package Manager a†’ Package
Manager Console`을 선택하여 Package Manager Console 뷰를 엽니다.

![][nuget_click]

`PM>` 프롬프트 뒤에 다음 행을 입력하고 `<Enter>` 키를 눌러 [Adjust 패키지][NuGet]를 설치합니다.

```
Install-Package Adjust
```

![][nuget_install]

Windows Phone 또는 Windows Store 프로젝트용 NuGet 패키지 관리자를 통해 Adjust 패키지를 설치할 수도 있습니다.

### 2. 기능 추가(Windows Phone 8.0에만 해당)

Solution Explorer에서 `Properties\WMAppManifest.xml` 파일을 열고 Capabilities 탭으로 전환한 후 `ID_CAP_IDENTITY_DEVICE` 확인란을 선택합니다.

![][wp_capabilities]

### 3. adjust를 앱과 연동

Solution Explorer에서 `App.xaml.cs` 파일을 엽니다. `using
AdjustSdk;` 문을 파일 맨 위에 추가합니다.

#### Windows Phone 8.0

앱의 `Application_Launching` 메서드에서 `AppDidLaunch` 메서드를 호출합니다. 그러면 adjust에 응용 프로그램 시작을 알립니다.

```cs
using AdjustSdk;

public partial class App : Application
{
    private void Application_Launching(object sender, LaunchingEventArgs e)
    {
        string appToken = "{YourAppToken}";
        string environment = AdjustConfig.EnvironmentSandbox;
        var config = new AdjustConfig(appToken, environment);
        Adjust.ApplicationLaunching(config);
        // ...
    }

    private void Application_Activated(object sender, ActivatedEventArgs e)
    {
        Adjust.ApplicationActivated();
        // ...
    }

    private void Application_Deactivated(object sender, DeactivatedEventArgs e)
    {
        Adjust.ApplicationDeactivated();
        // ...
    }
}
```

![][wp_app_integration]

#### 범용 앱

앱의 `OnLaunched` 메서드에서 `AppDidLaunch` 메서드를 호출합니다. 그러면 adjust에 응용 프로그램 시작을 알립니다.

```cs
using AdjustSdk;

sealed partial class App : Application
{
    protected override void OnLaunched(LaunchActivatedEventArgs e)
    {
        string appToken = "{YourAppToken}";
        string environment = AdjustConfig.EnvironmentSandbox;
        var config = new AdjustConfig(appToken, environment);
        Adjust.ApplicationLaunching(config);
        // ...
    }
}
```

![][ws_app_integration]

### 4. adjust 설정 업데이트

`{YourAppToken}` 자리 표시자를 앱 토큰으로 변경합니다. 앱 토큰은 [대시보드]에 있습니다.

앱을 테스트에 사용할지 아니면 프로덕션에 사용할 지에 따라 `environment` 매개변수를 다음 값 중 하나로 설정해야 합니다.

```cs
string environment = AdjustConfig.EnvironmentSandbox;
string environment = AdjustConfig.EnvironmentProduction;
```

**중요:** 이 값은 앱을 테스트하는 경우에만 `AdjustConfig.EnvironmentSandbox`로 설정해야 합니다. 앱을 게시하기 직전에 environment를 `AdjustConfig.EnvironmentProduction`으로 설정해야 합니다. 개발 및 테스트를 다시 시작할 경우에는 `AdjustConfig.EnvironmentSandbox`로 다시 설정하십시오.

이 environment는 실제 트래픽과 테스트 장치의 테스트 트래픽을 구별하기 위해 사용합니다. 이 값을 항상 의미 있게 유지해야 하고, 특히 매출을 트래킹하는 경우에 중요합니다.

#### Adjust 로깅

adjust 라이브러리에서 컴파일된 로그를 `released` 모드에서 보려면 로그 출력을 `debug` 모드에서 테스트하는 동안 앱으로 리디렉션해야 합니다.

adjust SDK에 다른 호출을 하기 전에 `Adjust.SetupLogging` 메서드를 호출하십시오.

```cs
Adjust.SetupLogging(logDelegate: msg => System.Diagnostics.Debug.WriteLine(msg));
// ...
var config = new AdjustConfig(appToken, environment);
Adjust.ApplicationLaunching(config);
// ...
```

다음 값 중 하나를 사용하여 `SetupLogging` 메서드의 두 번째 인수인 `logLevel`을 설정하여 테스트에서 표시되는 로그의 양을 늘리거나 줄일 수 있습니다.

```cs
logLevel: LogLevel.Verbose  // enable all logging
logLevel: LogLevel.Debug    // enable more logging
logLevel: LogLevel.Info     // the default
logLevel: LogLevel.Warn     // disable info logging
logLevel: LogLevel.Error    // disable warnings as well
logLevel: LogLevel.Assert   // disable errors as well
```

#### Windows Phone 8.0

앱의 `Application_Launching` 메서드에서 adjust SDK에 다른 호출을 하기 전에 `SetupLogging` 메서드를 호출하십시오.

```cs
using AdjustSdk;

public partial class App : Application
{
    private void Application_Launching(object sender, LaunchingEventArgs e)
    {
        Adjust.SetupLogging(logDelegate: msg => System.Diagnostics.Debug.WriteLine(msg),
            logLevel: LogLevel.Verbose);
        // ...
        var config = new AdjustConfig(appToken, environment);
        Adjust.ApplicationLaunching(config);
        // ...
    }
    // ...
}
```

#### 범용 앱

앱의 `OnLaunched` 메서드에서 adjust SDK에 다른 호출을 하기 전에 `SetupLogging` 메서드를 호출하십시오.

```cs
using AdjustSdk;

sealed partial class App : Application
{
    protected override void OnLaunched(LaunchActivatedEventArgs e)
    {
        Adjust.SetupLogging(logDelegate: msg => System.Diagnostics.Debug.WriteLine(msg),
            logLevel: LogLevel.Verbose);
        // ...
        var config = new AdjustConfig(appToken, environment);
        Adjust.ApplicationLaunching(config);
        // ...
    }
}
```

### 5. 앱 작성

메뉴에서 `DEBUG a†’ Start Debugging`을 선택합니다. 앱이 시작된 후에 `Tracked session start` 디버그 로그가 Output 뷰에 표시됩니다.

![][run_app]

## 추가 기능

adjust SDK를 프로젝트와 연동한 후에는 다음 기능을 사용할 수 있습니다.

### 6. 사용자 지정 이벤트 트래킹 추가

adjust를 사용하여 앱의 모든 이벤트를 트래킹할 수 있습니다. 버튼의 모든 탭을 트래킹하려면 [대시보드]에서 새 이벤트 토큰을 만들어야 합니다. 이벤트 토큰이 `abc123`일 경우 버튼의 `Button_Click` 메서드에 다음 행을 추가하여 클릭을 트래킹할 수 있습니다.

```cs
var adjustEvent = new AdjustEvent("abc123");
Adjust.trackEvent(adjustEvent);
```

트래킹을 시작하기 전에 이벤트 인스턴스를 사용하여 추가 구성을 수행할 수 있습니다.

### 7. 콜백 매개변수 추가

[대시보드]에서 이벤트의 콜백 URL을 등록할 수 있습니다. 이벤트가 트래킹될 때마다 GET 요청이 이 URL로 전송됩니다. 이 이벤트를 트래킹하기 전에 이벤트 인스턴스에서 `AddCallbackParameter`를 호출하여 콜백 매개변수를 해당 이벤트에 추가할 수도 있습니다. 그러면 해당 매개변수가 지정된 콜백 URL에 추가됩니다.

예를 들어 `http://www.adjust.com/callback` URL을 등록한 다음 이벤트를 다음과 같이 트래킹할 수 있습니다.

```cs
var adjustEvent = new AdjustEvent("abc123");

adjustEvent.addCallbackParameter("key", "value");
adjustEvent.addCallbackParameter("foo", "bar");

Adjust.trackEvent(adjustEvent);
```

이 경우에는 이벤트가 트래킹되고 요청이 다음 주소로 전송됩니다.

```
http://www.adjust.com/callback?key=value&foo=bar
```

매개변수 값으로 사용할 수 있는 `{win_adid}`와 같은 다양한 자리 표시자가 지원됩니다. 결과로 생성되는 콜백에서 이 자리 표시자는 현재 장치의 Windows 광고 ID로 변경됩니다.
또한 사용자 지정 매개변수는 저장되지 않고 콜백에만 추가됩니다. 이벤트에 대한 콜백을 등록하지 않은 경우 해당 매개변수는 읽을 수 없습니다.

사용 가능한 값의 전체 목록을 포함한 URL 콜백 사용에 대한 자세한 내용은 [콜백 설명서][callbacks-guide]를 참조하십시오.

### 8. 파트너 매개변수

adjust 대시보드에서 활성화된 연동에 대해, 네트워크 파트너로 전송할 매개변수도 추가할 수 있습니다.

이 매개변수는 위에서 설명한 콜백 매개변수의 경우와 비슷하지만, `AdjustEvent` 인스턴스에서 `AddPartnerParameter` 메서드를 호출해야 추가할 수 있습니다.

```cs
var adjustEvent = new AdjustEvent("abc123");

adjustEvent.addPartnerParameter("key", "value");
adjustEvent.addPartnerParameter("foo", "bar");

Adjust.trackEvent(adjustEvent);
```

특별 파트너와 해당 파트너와의 연동에 대한 자세한 내용은 [특별 파트너 설명서][special-partners]를 참조하십시오.

### 9. 매출 트래킹 추가

사용자가 광고를 누르거나 인앱 구매를 통해 매출을 발생시키는 경우 이벤트를 사용하여 해당 매출을 트래킹할 수 있습니다. 한 번 누를 때 0.01 유로의 매출이 발생한다고 가정할 경우 매출 이벤트를 다음과 같이 트래킹할 수 있습니다.

```cs
var adjustEvent = new AdjustEvent("abc123");
adjustEvent.setRevenue(0.01, "EUR");
Adjust.trackEvent(adjustEvent);
```

이것을 콜백 매개변수와 결합할 수도 있습니다.

통화 토큰을 설정하면 adjust에서 수신되는 매출이 선택한 보고 매출로 자동 변환됩니다. 통화 변환에 대한 자세한 내용은 [여기][currency-conversion]를 참조하십시오.

매출 및 이벤트 트래킹에 대한 자세한 내용은 [이벤트 트래킹 설명서][event-tracking]를 참조하십시오.

### 10. 딥링크 리어트리뷰션 설정

앱을 여는 데 사용하는 딥링크(Windows Phone 8.0에서는 URI 연결, 그리고 범용 앱에서는 URI 활성화라고도 함)를 처리하도록 adjust SDK를 설정할 수 있습니다. adjust에서는 adjust 관련 매개변수만 읽습니다. 딥링크를 사용하여 리타게팅 또는 재참여 캠페인을 진행할 계획인 경우 이 설정은 필수입니다.

#### Windows Phone 8.0

딥링크를 처리하기 위해 만든 `UriMapperBase` 클래스의 `MapUri` 메서드에서 `AppWillOpenUrl` 메서드를 호출합니다.

```cs
using AdjustSdk;

public class AssociationUriMapper : UriMapperBase
{
    public override Uri MapUri(Uri uri)
    {
        Adjust.AppWillOpenUrl(uri);
        //...
    }
}
```

#### 범용 앱

앱의 `OnActivated` 메서드에서 `AppWillOpenUrl` 메서드를 호출합니다.

```cs
using AdjustSdk;

public partial class App : Application
{
    protected override void OnActivated(IActivatedEventArgs args)
    {
        if (args.Kind == ActivationKind.Protocol)
        {
            var eventArgs = args as ProtocolActivatedEventArgs;

            if (eventArgs != null)
            {
                Adjust.AppWillOpenUrl(eventArgs.Uri);
            }
        }
        //...
    }
}
```

### 11. 이벤트 버퍼링 사용

앱에서 이벤트 트래킹을 많이 사용하는 경우 일부 HTTP 요청을 연기하여 1분마다 하나의 배치로 보낼 수 있습니다. `AdjustConfig` 인스턴스를 통해 이벤트 버퍼링을 사용하도록 설정할 수 있습니다.

```cs
var config = new AdjustConfig(appToken, environment);

config.setEventBufferingEnabled(true);

Adjust.ApplicationLaunching(config);
```

## 12. GDPR(일반 개인정보 보호법) 상의 잊힐 권리

유럽연합(EU) 일반 개인정보 보호법 제 17조에 의거하여, 사용자가 잊힐 권리를 행사하였을 경우  Adjust에 이를 통보할 수 있습니다. 다음 매서드를 호출하면 Adjust SDK는 사용자가 잊힐 권리를 사용하기로 했음을 Adjust 백엔드에 전달합니다:

```objc
[Adjust gdprForgetMe];
```

이 정보를 받는 즉시 Adjust는 사용자의 데이터를 삭제하며 Adjust SDK는 해당 사용자 추적을 중단합니다. 향후 이 기기로부터 어떤 요청도 Adjust에 전송되지 않습니다.

### 13. 어트리뷰션 변경 수신기 설정

트래커 어트리뷰션 변경에 대한 알림을 수신할 수신기를 등록할 수 있습니다. 어트리뷰션에 대해 다양한 소스가 고려되기 때문에 이 정보는 동시에 제공할 수 없습니다. 가장 간단한 방법은 익명 수신기를 하나 만드는 것입니다.

[해당 어트리뷰션 데이터 정책][attribution-data]을 고려하십시오.

SDK를 시작하기 전에 `AdjustConfig` 인스턴스를 사용하여 `AttributionChanged` 위임을 `Action<AdjustAttribution>` 시그너처와 함께 설정합니다.

```cs
var config = new AdjustConfig(appToken, environment);

config.AttributionChanged = (attribution) => 
    System.Diagnostics.Debug.WriteLine("attribution: " + attribution);
    
Adjust.ApplicationLaunching(config);
```

또는 `Application` 클래스에서 `AttributionChanged` 위임 인터페이스를 구현하고 위임으로 설정할 수 있습니다.

```cs
var config = new AdjustConfig(appToken, environment);
config.AttributionChanged = AdjustAttributionChanged;
Adjust.ApplicationLaunching(config);

private void AdjustAttributionChanged(AdjustAttribution attribution) 
{
    //...
}
```

SDK에서 최종 어트리뷰션 정보를 수신하면 위임 함수가 호출됩니다. 수신기 함수를 통해 `attribution` 매개변수에 액세스할 수 있습니다. 매개변수 속성에 대한 개요는 다음과 같습니다.

- `string TrackerToken` 현재 설치의 트래커 토큰.
- `string TrackerName` 현재 설치의 트래커 이름.
- `string Network` 현재 설치의 network 그룹화 기준.
- `string Campaign` 현재 설치의 campaign 그룹화 기준.
- `string Adgroup` 현재 설치의 ad group 그룹화 기준.
- `string Creative` 현재 설치의 creative 그룹화 기준.
- `string ClickLabel` 현재 설치의 클릭 레이블.

[adjust.com]: http://www.adjust.com
[dashboard]: http://www.adjust.com
[nuget]: http://nuget.org/packages/Adjust
[nuget_click]: https://raw.github.com/adjust/adjust_sdk/master/Resources/windows/01_nuget_console_click.png
[nuget_install]: https://raw.github.com/adjust/adjust_sdk/master/Resources/windows/02_nuget_install.png
[wp_capabilities]: https://raw.github.com/adjust/adjust_sdk/master/Resources/windows/03_windows_phone_capabilities.png
[wp_app_integration]: https://raw.github.com/adjust/adjust_sdk/master/Resources/windows/04_wp_app_integration.png
[ws_app_integration]: https://raw.github.com/adjust/adjust_sdk/master/Resources/windows/05_ws_app_integration.png
[run_app]: https://raw.github.com/adjust/adjust_sdk/master/Resources/windows/06_run_app.png
[attribution-data]: https://github.com/adjust/sdks/blob/master/doc/attribution-data.md

[dashboard]:     http://adjust.com
[releases]:      https://github.com/adjust/adjust_android_sdk/releases
[import_module]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/01_import_module.png
[select_module]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/02_select_module.png
[imported_module]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/03_imported_module.png
[gradle_adjust]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/04_gradle_adjust.png
[gradle_gps]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/05_gradle_gps.png
[manifest_gps]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/06_manifest_gps.png
[manifest_permissions]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/07_manifest_permissions.png
[proguard]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/08_proguard.png
[receiver]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/09_receiver.png
[application_class]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/11_application_class.png
[manifest_application]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/12_manifest_application.png
[application_config]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/13_application_config.png
[activity]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/14_activity.png
[log_message]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/15_log_message.png

[callbacks-guide]:      https://docs.adjust.com/en/callbacks
[event-tracking]:       https://docs.adjust.com/en/event-tracking
[special-partners]:     https://docs.adjust.com/en/special-partners
[example]:              https://github.com/adjust/windows_sdk/tree/master/Adjust
[currency-conversion]:  https://docs.adjust.com/en/event-tracking/#tracking-purchases-in-different-currencies

## 라이선스

adjust-sdk는 MIT 라이선스에 따라 사용이 허가됩니다.

Copyright (c) 2015 adjust GmbH, http://www.adjust.com

이로써 본 소프트웨어와 관련 문서 파일(이하 "소프트웨어")의 복사본을 받는 사람에게는 아래 조건에 따라 소프트웨어를 제한 없이 다룰 수 있는 권한이 무료로 부여됩니다. 이 권한에는 소프트웨어를 사용, 복사, 수정, 병합, 출판, 배포 및/또는 판매하거나 2차 사용권을 부여할 권리와 소프트웨어를 제공 받은 사람이 소프트웨어를 사용, 복사, 수정, 병합, 출판, 배포 및/또는 판매하거나 2차 사용권을 부여하는 것을 허가할 수 있는 권리가 제한 없이 포함됩니다.

위 저작권 고지문과 본 권한 고지문은 소프트웨어의 모든 복사본이나 주요 부분에 포함되어야 합니다.

소프트웨어는 상품성, 특정 용도에 대한 적합성 및 비침해에 대한 보증 등을 비롯한 어떤 종류의 명시적이거나 암묵적인 보증 없이 "있는 그대로" 제공됩니다. 어떤 경우에도 저작자나 저작권 보유자는 소프트웨어와 소프트웨어의 사용 또는 기타 취급에서 비롯되거나 그에 기인하거나 그와 관련하여 발생하는 계약 이행 또는 불법 행위 등에 관한 배상 청구, 피해 또는 기타 채무에 대해 책임지지 않습니다.

--END--
