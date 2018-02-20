## 요약

이 항목에서는 Adjust™의 Xamarin iOS SDK에 대해 설명합니다. Adjust에 대한 자세한 내용은 adjust.com을 참조하십시오.

## 목차

* [앱 예제](#example-apps)
* [기본 연동](#basic-integration)
   * [SDK 받기](#sdk-get)
   * [프로젝트에 SDK 추가](#sdk-add)
   * [앱에 SDK 프로젝트 레퍼런스 추가](#sdk-add-project)
   * [앱에 SDK DLL 레퍼런스 추가](#sdk-add-dll)
   * [앱에 SDK 연동](#sdk-integrate) 
   * [Adjust 로그 기록](#adjust-logging)
   * [추가 설정](#additional-settings)
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
      * [iOS 광고 식별자](#di-idfa)
      * [Adjust 기기 식별자](#di-adid)
   * [푸시 토큰(push token)](#push-token)
   * [사전 설치 트래커(pre-installed trackers)](#pre-installed-trackers)
   * [딥링크](#deeplinking)
      * [기본 딥링크](#deeplinking-standard)
      * [iOS 8 이하 버전 딥링크](#deeplinking-setup-old)
      * [iOS 9 이상 버전 딥링크](#deeplinking-setup-new)
      * [거치(deferred) 딥링크](#deeplinking-deferred)
      * [딥링크를 통한 리어트리뷰션](#deeplinking-reattribution)
* [라이선스](#license)

## <a id="example-apps">앱 예제

[`iOS` 디렉토리][demo-app-android]에 iOS 앱 예제가 있습니다. Xamarin Studio 프로젝트를 열어 Adjust SDK 연동 방법의 예를 확인할 수 있습니다.

## <a id="basic-integration">기본 연동

다음은 Adjust SDK를 Xamarin 프로젝트와 연동하기 위해 최소한으로 수행해야 하는 절차입니다. 여기서는 Xamarin Studio 또는 Visual Studio를 iOS 개발에 사용한다고 가정합니다.

### <a id="sdk-get">SDK 받기

Adjust의 [releases 페이지][releases]에서 최신 버전을 다운로드하세요. 원하는 위치에 압축 파일을 풀면 됩니다.

Adjust 바인딩 DLL를 사용하려면 [이 단계](#sdk-add-dll)에 나오는 내용을 따르십시오.

### <a id="sdk-add">프로젝트를 SDK에 추가

솔루션에 '기존 프로젝트를 추가(add an existing project)'하도록 선택합니다.

![][add_ios_binding]

`AdjustSdk.Xamarin.iOS` 프로젝트 파일을 선택한 다음 `Open`을 클릭합니다.

![][select_ios_binding]

이제 Adjust iOS 바인딩이 솔루션에 서브모듈로 추가되었습니다.

![][submodule_ios_binding]

### <a id="sdk-add-project">앱에 SDK 프로젝트 레퍼런스 추가

Adjust iOS 바인딩을 솔루션에 추가한 다음에는 iOS 앱 프로젝트에서 해당 레퍼런스 연결도 추가해야 합니다. 

![][reference_ios_binding]

레퍼런스를 프로젝트 레퍼런스를 통하지 않고 Adjust SDK에 추가하려면 이 단계를 그대로 통과하여 DLL 레퍼런스로 앱에 추가할 수도 있습니다. 이 방법 설명은 아래에서 확인할 수 있습니다. 

### <a id="sdk-add-dll">앱에 SDK DLL 레퍼런스 추가

다음 단계로 iOS 프로젝트 내 properties 폴더에서 프로퍼티 바인딩 DLL에 레퍼런스를 추가해야 합니다. 레퍼런스 창에서 먼저 `.Net Assembly`를, 그 다음 다운로드한 `AdjustSdk.Xamarin.iOS.dll`을 선택합니다.

![][select_ios_dll]

### <a id="sdk-integrate">앱에 SDK 연동

먼저 기본 세션 추적을 설정합니다.

앱 델리게이트에서 소스 파일을 열고 파일 맨 위에 `using` 문을 추가합니다. 그 다음  앱 델리게이트의 `FinishedLaunching` 메서드에서 다음 호출을 `Adjust`에 추가합니다. 

```cs
using AdjustBindingsiOS;

// ...

string yourAppToken = "{YourAppToken}";
string environment = AdjustConfig.EnvironmentSandbox;

var config = ADJConfig.ConfigWithAppToken(yourAppToken, environment);

Adjust.AppDidLaunch(adjustConfig);
```

`{YourAppToken}`을 앱 토큰으로 대체합니다. 앱 토큰은 [대시보드][adjust.com]에서 찾을 수 있습니다. 

앱을 테스트에 사용할지 아니면 프로덕션에 사용할지에 따라 `environment`를 다음 값 중 하나로 설정해야 합니다.

```cs
string environment = AdjustConfig.EnvironmentSandbox;
string environment = AdjustConfig.EnvironmentProduction;
```

**중요:** 이 값은 앱을 테스트하는 경우에만 `AdjustConfig.EnvironmentSandbox`로 설정해야 합니다. 앱을 게시하기 전에 environment를 `AdjustConfig.EnvironmentProduction`으로 설정해야 합니다. 개발 및 테스트를 다시 시작할 경우 `AdjustConfig.EnvironmentSandbox`로 다시 설정하십시오.

이 environment는 실제 트래픽과 테스트 장치의 테스트 트래픽을 구별하기 위해 사용합니다. 이 값을 항상 유의미하게 유지해야 합니다! 매출을 추적하는 경우에 특히 중요합니다.

### <a id="adjust-logging">Adjust 로그 기록

다음 파라미터 중 하나를 사용하여 `ADJConfig` 인스턴스에서 `LogLevel` 프로퍼티를 설정하면 테스트에 표시되는 로그의 양을 늘리거나 줄일 수 있습니다.

```cs
config.LogLevel = ADJLogLevel.Verbose;  // 모든 로그 활성화
config.LogLevel = ADJLogLevel.Debug;    // 더 많은 로그 활성화
config.LogLevel = ADJLogLevel.Info;     // 기본값
config.LogLevel = ADJLogLevel.Warn;     // info 로그 비활성화
config.LogLevel = ADJLogLevel.Error;    // 경고 역시 비활성화
config.LogLevel = ADJLogLevel.Assert;   // 오류 역시 비활성화
config.LogLevel = ADJLogLevel.Suppress; // 모든 로그 비활성화
```

프로덕션 단계의 앱이 Adjust SDK 로그를 출력하지 않게 하려면 `ADJLogLevel.Suppress`를 선택하고 `ADJConfig` 개체에서 생성자를 초기화해야 합니다. 이 생성자에서 supress 로그 레벨 모드를 활성화할 수 있습니다.

```cs
using AdjustBindingsiOS;

// ...

string yourAppToken = "{YourAppToken}";
string environment = AdjustConfig.EnvironmentSandbox;

var config = ADJConfig.ConfigWithAppToken(yourAppToken, environment);
config.LogLevel = ADJLogLevel.Suppress;

Adjust.AppDidLaunch(adjustConfig);
```

### <a id="additional-settings">추가 설정

Xamarin iOS 앱 프로젝트가 Adjust iOS 바인딩 카테고리를 인식할 수 있게 하려면 `iOS Build`에 mtouch 아규먼트(argument)를 추가해야 합니다. `Project Options`에 있는 `Build` 섹션에서 해당 아규먼트를 찾을 수 있습니다. `-ObjC` 아규먼트를 포함한 쿼트 문자열 앞에 `-gcc_flags`를 추가하면 됩니다. 

![][additional_flags]

### <a id="build-your-app">앱 빌드

앱을 작성하고 실행합니다. 빌드가 성공적이면 콘솔에서 SDK 로그를 읽을 수 있습니다. 앱이 최초 런칭된 다음 `Install tracked`가 정보 로그로 표시됩니다.

![][run]

## <a id="additional-features">추가 기능

Adjust SDK를 프로젝트에 연동한 후에는 다음 기능을 사용할 수 있습니다.

### <a id="event-tracking">이벤트 추적

Adjust로 이벤트를 추적할 수 있습니다. 특정 버튼의 모든 탭을 추적하려는 경우 `abc123`와 같은 관련 이벤트 토큰이 있는 새 이벤트 토큰을 [대시보드](adjust.com)에서 만듭니다. 그런 다음 버튼의 `TouchUpInside` 핸들러에 다음 행을 추가하여 클릭을 추적할 수 있습니다.

```cs
var adjustEvent = ADJEvent.EventWithEventToken("abc123");

Adjust.TrackEvent(adjustEvent);
```

버튼을 누르면 `Event tracked`가 로그에 나타납니다.

### <a id="revenue-tracking">매출 추적

사용자가 광고를 누르거나 인앱 구매를 통해 매출을 발생시킬 수 있는 경우 이벤트를 사용하여 해당 수익을 추적할 수 있습니다. 한 번 누를 때 0.01 유로의 수익이 발생한다고 가정할 경우 매출 이벤트를 다음과 같이 추적할 수 있습니다.

```cs
var adjustEvent = ADJEvent.EventWithEventToken("abc123");

adjustEvent.SetRevenue(0.01, "EUR");

Adjust.TrackEvent(adjustEvent);
```

통화 토큰을 설정하면 들어오는 매출을 Adjust가 자동으로 미리 지정한 보고용 통화로 전환해 줍니다. 통화 전환에 관한 자세한 내용은 [여기][currency-conversion]에서 확인하세요.

매출 및 이벤트 추적에 대한 자세한 내용은 [이벤트 추적 설명서][event-tracking]를 참조하십시오.

### <a id="revenue-deduplication"></a>매출 중복 제거

거래 ID를 선택 사항으로 추가하여 수익 중복 추적을 피할 수 있습니다. 가장 최근에 사용한 거래 ID 10개를 기억하며, 중복 거래 ID로 이루어진 매출 이벤트는 집계하지 않습니다. 인앱 구매 추적 시 특히 유용합니다. 사용 예는 아래에 나와 있습니다.

인앱 구매를 추적하려면 상태가 `SKPaymentTransactionState.Purchased`로 변경된 경우에만 `UpdatedTransaction`에서 `finishTransaction` 후에 `trackEvent`를 호출해야 합니다. 이렇게 해야 실제로 발생하지 않은 매출을 추적하는 오류를 막을 수 있습니다.

```cs
public void UpdatedTransactions(SKPaymentQueue queue, SKPaymentTransaction[] transactions)
{
    foreach (SKPaymentTransaction transaction in transactions) 
    {
        switch (transaction.TransactionState)
        {
            case SKPaymentTransactionState.Purchased:
                SKPaymentQueue.DefaultQueue.FinishTransaction(transaction);

                var adjustEvent = ADJEvent.EventWithEventToken("abc123");

                adjustEvent.SetRevenue(0.01, "{currency}");
                adjustEvent.SetTransactionId(transaction.TransactionIdentifier);

                Adjust.TrackEvent(adjustEvent);

                break;

                // More cases
        }
    }
}
```

### <a id="iap-verification">인앱 구매 검증

인앱 구매 검증을 가능하게 해 줄 Xamarin 구매 SDK가 현재 개발 중이며 곧 출시될 예정입니다. 자세한 내용을 확인하려면 support@adjust.com으로 연락해 주십시오.

### <a id="callback-parameters">콜백 파라미터

[대시보드][adjust.com]에서 이벤트 콜백 URL을 등록할 수 있습니다. 이벤트를 추적할 때마다 GET 요청이 해당 URL로 전송됩니다. 이벤트를 추적하기 전에 이벤트 인스턴스에서 `AddCallbackParameter`를 호출하여 콜백 파라미터를 해당 이벤트에 추가할 수 있습니다. 그러면 해당 파라미터가 콜백 URL에 추가됩니다.

예를 들어 URL `http://www.adjust.com/callback`을 등록했다면 이벤트를 다음과 같이 추적할 수 있습니다.

```csharp
var adjustEvent = ADJEvent.EventWithEventToken("abc123");

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

위에서 설명한 콜백 매개변수의 경우와 비슷하지만, `ADJEvent` 인스턴스에서 `AddPartnerParameter` 메서드를 호출해야 추가할 수 있습니다.

```cs
var adjustEvent = ADJEvent.EventWithEventToken("abc123");

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

세션 콜백 파라미터는 이벤트 콜백 파라미터와 비슷한 인터페이스를 지녔지만, 이벤트에 키, 값을 추가하는 대신 `Adjust`의 `AddSessionCallbackParameter` 메서드를 호출하여 추가합니다.

```cs
Adjust.AddSessionCallbackParameter("foo", "bar");
```

세션 콜백 파라미터는 이벤트에 추가된 콜백 파라미터와 합쳐지며, 이벤트에 추가된 콜백 파라미터가 우선권을 지닙니다. 그러나 세션에서와 같은 키로 이벤트에 콜백 파라미터를 추가한 경우 새로 추가한 콜백 파라미터가 우선권을 가집니다.

원하는 키를 `RemoveSessionCallbackParameter` 메서드로 전달하여 특정 세션 콜백 파라미터를 제거할 수 있습니다.

```cs
Adjust.RemoveSessionCallbackParameter("foo");
```

세션 콜백 파라미터의 키와 값을 전부 없애고 싶다면 `ResetSessionCallbackParameters` 메서드로 재설정하면 됩니다.

```cs
Adjust.ResetSessionCallbackParameters();
```

### <a id="session-partner-parameters"> 세션 파트너 파라미터

Adjust SDK 내 모든 이벤트 및 세션에서 전송되는 [세션 콜백 파라미터](#session-callback-parameters)가 있는 것처럼, 세션 파트너 파라미터도 있습니다.

이들 파라미터는 Adjust [대시보드](adjust.com)에서 연동을 활성화한 네트워크 파트너에게 전송할 수 있습니다.

세션 파트너 파라미터는 이벤트 파트너 파라미터와 인터페이스가 비슷하지만, 이벤트에 키, 값을 추가하는 대신 `Adjust`의 `AddSessionPartnerParameter` 메서드를 호출하여 추가합니다.

```cs
Adjust.AddSessionPartnerParameter("foo", "bar");
```

세션 파트너 파라미터는 이벤트에 추가한 파트너 파라미터와 합쳐지며, 이벤트에 추가된 파트너 파라미터가 우선순위를 지닙니다. 그러나 세션에서와 같은 키로 이벤트에 파트너 파라미터를 추가한 경우, 새로 추가한 파트너 파라미터가 우선권을 가집니다.

원하는 키를 `RemoveSessionPartnerParameter` 메서드로 전달하여 특정 세션 파트너 파라미터를 제거할 수 있습니다.

```cs
Adjust.RemoveSessionPartnerParameter("foo");
```

세션 파트너 파라미터의 키와 값을 전부 없애고 싶다면 `ResetSessionPartnerParameters` 메서드로 재설정하면 됩니다.

```cs
Adjust.ResetSessionPartnerParameters();
```

### <a id="delay-start"> 예약 시작

Adjust SDK에 예약 시작을 걸면 앱이 고유 식별자 등의 세션 파라미터를 얻어 설치 시에 전송할 시간을 벌 수 있습니다.

`ADJConfig` 인스턴스의 `DelayStart` 프로퍼티로 예약 시작 시각을 초 단위로 설정하세요.

```objc
config.DelayStart = 5.5;
```

이 경우 Adjust SDK는 최초 인스톨 세션 및 생성된 이벤트를 5.5초간 기다렸다가 전송합니다. 이 시간이 지난 후, 또는 그 사이에 `Adjust.SendFirstPackages()`을 호출했을 경우 모든 세션 파라미터가 지연된 인스톨 세션 및 이벤트에 추가되며 Adjust SDK는 원래대로 돌아옵니다.

**Adjust SDK의 최대 지연 예약 시작 시간은 10초입니다**.

### <a id="attribution-callback">어트리뷰션 콜백

콜백을 등록하여 트래커 어트리뷰션 변경 알림을 받을 수 있습니다. 어트리뷰션에서 고려하는 소스가 각각 다르기 때문에 이 정보는 동시간에 제공할 수 없습니다. 어트리뷰션 콜백은 선택 사항이며, 앱에 구현하려면 다음과 같이 하면 됩니다. 

[해당 어트리뷰션 데이터 정책][attribution-data]을 반드시 고려하세요.

1. `AppDelegate.cs`를 열고 `AdjustDelegate`에서 상속되는 클래스를 생성한 다음 `AdjustAttributionChanged` 메서드를 오버라이드합니다. 
    
    ```cs
    public class AdjustDelegateXamarin : AdjustDelegate
    {
        public override void AdjustAttributionChanged (ADJAttribution attribution)
        {
            Console.WriteLine ("Attribution changed!");
            Console.WriteLine ("New attribution: {0}", attribution.ToString ());
        }
    }
    ```

2. 이 `AppDelegate` 클래스에 `AdjustDelegateXamarin` 타입 프라이빗 필드를 추가합니다. 

    ```cs
    private AdjustDelegateXamarin adjustDelegate = null;
    ```

3. `ADJConfig` 인스턴스로 델리게이트를 초기화 및 설정합니다.

    ```cs
    adjustDelegate = new AdjustDelegateXamarin();
    
    // ...
    
    adjustConfig.Delegate = adjustDelegate;
    ```
    
위임 콜백은 `ADJConfig` 인스턴스를 써서 구성하므로, `Adjust.AppDidLaunch(adjustConfig)`를 호출하기 전에 `Delegate` 프로퍼티를 호출해야 합니다.

SDK에 최종 프로퍼티 데이터가 수신되면 위임 함수가 호출됩니다. 위임 함수를 통해 `attribution` 파라미터에 액세스할 수 있습니다. 각 파라미터 속성에 대한 개요는 다음과 같습니다.

- `string TrackerToken` 현재 어트리뷰션의 트래커 토큰.
- `string TrackerName` 현재 어트리뷰션의 트래커 이름.
- `string Network` 현재 어트리뷰션의 네트워크 그룹화 기준.
- `string Campaign` 현재 어트리뷰션의 캠페인 그룹화 기준.
- `string Adgroup` 현재 어트리뷰션의 광고 그룹 그룹화 기준.
- `string Creative` 현재 어트리뷰션의 크리에이티브 그룹화 기준.
- `string ClickLabel` 현재 어트리뷰션의 클릭 레이블.
- `string Adid` Adjust 기기 식별자.

### <a id="session-event-callbacks">세션 및 이벤트 콜백

콜백을 등록하여 이벤트나 세션 추적 시 알림을 받을 수 있습니다. 어트리뷰션 콜백과 마찬가지로 `AdjustDelegate`에서 상속된 개인 설정 클래스에서 수행해야 합니다.

아래는 이벤트 추적 성공 시의 콜백 함수입니다. 이를 구현하려면 다음 단계를 수행하십시오. 

```cs
public class AdjustDelegateXamarin : AdjustDelegate
{
    public override void AdjustEventTrackingSucceeded(ADJEventSuccess eventSuccessResponseData)
    {
        Console.WriteLine("Event tracking succeeded! Info: " + eventSuccessResponseData.ToString());
    }
}
```

다음은 이벤트 추적 실패 시에 구현하는 콜백 함수입니다.

```cs
public class AdjustDelegateXamarin : AdjustDelegate
{
    public override void AdjustEventTrackingFailed(ADJEventFailure eventFailureResponseData)
    {
        Console.WriteLine("Event tracking failed! Info: " + eventFailureResponseData.ToString());
    }
}
```

세선 추적 성공의 경우입니다.

```cs
public class AdjustDelegateXamarin : AdjustDelegate
{
    public override void AdjustSessionTrackingSucceeded(ADJSessionSuccess sessionSuccessResponseData)
    {
        Console.WriteLine("Session tracking succeeded! Info: " + sessionSuccessResponseData.ToString());
    }
}
```

그리고 세션 추적 실패의 경우입니다.

```cs
public class AdjustDelegateXamarin : AdjustDelegate
{
    public override void AdjustSessionTrackingFailed(ADJSessionFailure sessionFailureResponseData)
    {
        Console.WriteLine("Session tracking failed! Info: " + sessionFailureResponseData.ToString());
    }
}
```

콜백 함수는 SDK에서 서버로 패키지를 보내려고 시도한 후에 호출됩니다. 콜백에서 전용 응답 데이터 개체에 액세스할 수 있습니다. 세션 응답 데이터 프로퍼티에 대한 개요는 다음과 같습니다.

- `string Message` 서버에서 전송한 메시지 또는 SDK가 기록한 오류.
- `string Timestamp` 서버에서 전송한 데이터의 타임스탬프.
- `string Adid` Adjust가 제공하는 고유 기기 식별자.
- `NSDictionary<string, object> JsonResponse` 서버로부터의 응답이 있는 JSON 개체.

이벤트 응답 데이터 개체 두 가지에는 다음 정보가 포함됩니다.

- `string EventToken` 추적 패키지가 이벤트인 경우 이벤트 토큰

그리고 이벤트 및 세션 실패 개체에는 다음 정보도 포함됩니다.

- `bool WillRetry` 나중에 패키지 재전송 시도가 있을 것임을 나타냅니다.

### <a id="disable-tracking">추적 사용 중지

`SetEnabled` 메서드를 `false` 파라미터로 설정한 상태로 호출하면 Adjust SDK에서 현재 기기의 모든 작업 추적을 중지할 수 있습니다. **이 설정은 세션 간에 기억**되지만, 활성화하려면 최초 세션이 끝나야 합니다.

```cs
Adjust.SetEnabled(false);
```

`IsEnabled` 프로퍼티를 사용하면 Adjust SDK가 현재 활성화 상태인지 확인할 수 있습니다. 사용중 파라미터가 `true`로 설정된 `SetEnabled`를 호출하면 Adjust SDK를 언제든 활성화할 수 있습니다.

### <a id="offline-mode">오프라인 모드

Adjust SDK를 오프라인 모드로 전환하여 서버로 전송하는 작업을 일시 중단하고 추적 데이터를 보관하여 나중에 보낼 수 있습니다. 오프라인 모드에서는 모든 정보가 파일에 저장되므로, 이때 너무 많은 이벤트를 촉발(trigger)하지 않도록 주의하십시오.

`true` 파라미터로 `SetOfflineMode`를 호출하면 오프라인 모드를 활성화할 수 있습니다.

```cs
Adjust.SetOfflineMode(true);
```

반대로, `SetOfflineMode`를 `false`로 설정한 상태로 호출하면 오프라인 모드를 비활성화할 수 있습니다. Adjust SDK를 다시 온라인 모드로 전환하면 저장된 정보가 모두 올바른 시간 정보와 함께 Adjust 서버로 전송됩니다.

추적 사용 중지와 달리 이 설정은 세션 간에 **기억되지 않습니다.** 따라서 앱을 오프라인 모드에서 종료한 경우에도 SDK는 항상 온라인 모드로 시작됩니다.

### <a id="event-buffering">이벤트 버퍼링

앱이 이벤트 추적을 많이 사용하는 경우, 매 분마다 배치(batch) 하나씩만 보내도록 하기 위해 일부 HTTP 요청을 지연시키고자 할 경우가 있을 수 있습니다. `AdjustConfig` 인스턴스로 이벤트 버퍼링을 적용할 수 있습니다.

```cs
var config = ADJConfig.ConfigWithAppToken(yourAppToken, environment);

config.EventBufferingEnabled = true;

Adjust.AppDidLaunch(config);
```

설정된 사항이 없을 경우 이벤트 버퍼링은 **사용 중지 상태가 기본값입니다**

### <a id="background-tracking">백그라운드 추적

Adjust SDK 기본값 행위는 **앱이 백그라운드에 있을 동안에는 HTTP 요청 전송을 잠시 중지**하는 것입니다. `ADJConfig` 인스턴스에서 이 설정을 바꿀 수 있습니다.

```cs
var config = ADJConfig.ConfigWithAppToken(yourAppToken, environment);

config.SendInBackground = true;

Adjust.AppDidLaunch(config);
```

설정된 사항이 없을 경우 백그라운드 전송은 **사용 중지 상태가 기본값입니다**

### <a id="device-ids"></a>기기 ID

Adjust SDK로 기기 식별자 몇 가지를 얻을 수 있습니다.

### <a id="di-gps-adid"></a>iOS 광고 식별자

Google Analytics와 같은 몇몇 서비스를 사용하려면 중복 보고가 발생하지 않도록 기기 ID와 클라이언트 ID를 조정해야 합니다.

IDFA 기기 식별자를 얻으려면 `Adjust` 인스턴스에서 `idfa` 프로퍼티에 억세스하세요.

```cs
string idfa = Adjust.Idfa;
```

### <a id="di-adid"></a>Adjust 기기 식별자

Adjust 백엔드는 앱을 설치한 기기에서 고유한 **Adjust 기기 식별자** (**adid**)를 생성합니다. 이 식별자를 얻으려면 `Adjust` 인스턴스에서 다음 프로퍼티에 억세스하면 됩니다.

```cs
stirng adid = Adjust.Adid;
```

**주의**: **adid** 관련 정보는 Adjust 백엔드가 앱 설치를 추적한 후에만 얻을 수 있습니다. 그 순간부터 Adjust SDK는 기기 **adid** 정보를 갖게 되며 이 메서드로 억세스할 수 있습니다. 따라서 SDK가 초기화되고 앱 설치 추적이 성공적으로 이루어지기 전에는 **adid** 억세스가 **불가능합니다**.

### <a id="user-attribution"></a>사용자 어트리뷰션

[어트리뷰션 콜백 섹션](#attribution-callback)에서 설명한 대로, 변동이 있을 때마다 새로운 어트리뷰션 관련 정보를 전달하기 위해 어트리뷰션 콜백이 촉발됩니다. 사용자의 현재 어트리뷰션 관련 정보에 언제든 억세스하고 싶다면 `Adjust` 인스턴스의 다음 프로퍼티를 통해 가능합니다.

```cs
ADJAttribution attribution = Adjust.Attribution;
```

### <a id="push-token"></a>푸시 토큰

푸시 알림 토큰을 전송하려면 **토큰을 받았거나 또는 토큰이 업데이트될 때마다** 아래와 같이 Adjust에 대한 호출을 추가하세요.

```cs
NSData pushNotificationsToken;	// Obtain and assign your push notification token as NSData type.

Adjust.SetDeviceToken(pushNotificationsToken);
```

### <a id="pre-installed-trackers">사전 설치 트래커

Adjust SDK를 사용하여 앱이 사전 설치된 기기를 지닌 사용자를 인식하고 싶다면 다음 절차를 따르세요.

1. [대시보드][adjust.com]에 새 트래커를 생성합니다.

2. 앱 델리게이트를 열고 `ADJConfig` 인스턴스의 기본값 트래커를 다음과 같이 설정합니다.

    ```cs
    var config = ADJConfig.ConfigWithAppToken(yourAppToken, environment);

    config.DefaultTracker = "{TrackerToken}";

    Adjust.AppDidLaunch(config);
    ```

`{TrackerToken}`을 2에서 생성한 트래커 토큰으로 대체합니다. 대시보드에서는 (`http://app.adjust.com/`을 포함하는) 트래커 URL을 표시한다는 사실을 명심하세요. 소스코드에서는 전체 URL을 표시할 수 없으며 6자로 이루어진 토큰만을 명시해야 합니다.

3. 앱 빌드를 실행하세요. 앱 로그 출력에서 다음 라인을 볼 수 있을 것입니다.

    ```
    Default tracker: 'abc123'
    ```

### <a id="deeplinking"></a>딥링크

URL에서 앱으로 딥링크를 거는 옵션이 있는 Adjust 트래커 URL을 사용하고 있다면, 딥링크 URL과 그 내용 관련 정보를 얻을 가능성이 있습니다. 해당 URL 클릭 시 사용자가 이미 앱을 설치한 상태(기본 딥링크)일 수도, 아직 앱을 설치하지 않은 상태(거치 딥링크)일 수도 있습니다. 

iOS 딥링크 및 앱에서 이를 활성화시키는 방법에 대해 보다 자세한 정보를 보시려면 [iOS SDK README][ios-readme-deeplinking]를 참조해 주십시오. 

#### <a id="deeplinking-standard">기본 딥링크

사용자가 앱을 설치하고 딥링크 정보가 들어간 트래커 URL을 클릭할 경우, 앱이 열리고 딥링크 내용이 앱으로 전달되어 이를 분석하고 다음 행동을 결정하게 됩니다. iOS SDK README에서 확인하신 대로, iOS에서 딥링크는 현재 (iOS 8 이하 버전인 경우) 사용자 설정 URL 스킴 설정을 사용하거나 (iOS 9 이상 버전인 경우) 유니버설 링크를 사용합니다.

앱에 어떤 상황을 사용하고자 하는지에 따라 (또는 다양한 기기를 지원하기 위해 두 가지 다 사용하려 할 경우) 앱이 다음 상황 중 하나 또는 두 가지 다를 취급할 수 있도록 설정해야 합니다. 

#### <a id="deeplinking-setup-old">iOS 8 이하 버전 딥링크

iOS 8 이하 버전 기기에서는 앱의 `Info.plist` 파일에서 사용자 설정 URL 스킴을 설정해야 합니다. `Info.plist` 파일을 열고 `Advanced` 탭으로 간 다음 `Identifier` 필드에 `Bundle ID` 값을, 그리고 `URL Schemes` 필드에 앱이 취급할 사용자 URL 스킴을 입력합니다.

![][deeplinking_custom_url_scheme]

이 설정을 마치면, 선택한 스킴명이 들어있는 딥링크 정보가 담긴 트래커 URL을 사용자가 클릭하여 앱이 열릴 때마다 `AppDelegate` 클래스의 `OpenUrl` 메서드가 호출되어 해당 딥링크 정보를 얻을 수 있습니다. 

```cs
public override bool OpenUrl(UIApplication application, NSUrl url, string sourceApplication, NSObject annotation)
{
    // url -> This is your deep link content.

    return true;
}
```

이렇게 하면 iOS 8 이하 버전을 사용하는 iOS 기기에서 딥링크를 성공적으로 설정할 수 있습니다. 

#### <a id="deeplinking-setup-new">iOS 9 이상 버전 딥링크

iOS 9 이상 버전 기기에서 딥링크를 설정하려면 앱이 Apple 유니버설 링크를 취급하도록 해야 합니다. iOS SDK README의 설명대로 하셨다면 Apple Developer 포털에서 앱의 `Associated Domains`가 활성화 상태일 것입니다. 앱 프로젝트에서 활성화하려면 먼저 `Entitlements.plist`를 연 다음 `Associated Domains`에서 `Enable Associated Domains`체크박스에 표시하여 해당 부분을 활성화하십시오. 그 다음 Adjust 대시보드에서 생성된 도메인을 추가하기만 하면 됩니다.

![][deeplinking_universal_links]

이 설정을 마치고 나면, 유니버설 링크를 클릭하여 앱이 열릴 때마다 `AppDelegate` 클래스의 `ContinueUserActivity` 메서드가 호출되어 해당 딥링크 정보를 얻을 수 있습니다.

```cs
public override bool ContinueUserActivity(UIApplication application, NSUserActivity userActivity, UIApplicationRestorationHandler completionHandler)
{
    if (userActivity.ActivityType == NSUserActivityType.BrowsingWeb)
    {
        // userActivity.WebPageUrl -> This is your deep link content.
    }

    return true;
}
```

이렇게 하면 iOS 9 이상 버전을 사용하는 iOS 기기에서 딥링크를 성공적으로 설정할 수 있습니다. 

#### <a id="deeplinking-deferred">거치 딥링크 

거치 딥링크에서 URL 콘텐츠 관련 정보를 얻기 위해서는 어트리뷰션, 이벤트 및 세션 콜백과 마찬가지 방식으로 콜백 메서드를 구현해야 합니다. `AdjustDelegate`에서 상속된 클래스에서 `AdjustDeeplinkResponse`를 오버라이드하면 됩니다. 

```cs
public class AdjustDelegateXamarin : AdjustDelegate
{
    public override bool AdjustDeeplinkResponse(NSUrl deeplink)
    {
        Console.WriteLine("adjust: Deferred deep link received! URL = " + deeplink.ToString());

        return true;
        // return false;
    }
}
```

거치 딥링크의 경우 `AdjustDeeplinkResponse` 메서드에서 리턴값을 통해 추가 설정을 한 가지 할 수 있습니다. Adjust SDK가 거치 딥링크 정보를 받고 나면 이 URL을 SDK에서 열지 혹은 열지 않을지를 결정할 수 있습니다. 리턴값을 `true`로 설정하면 URL이 열리며, `false`를 리턴하면 SDK에서 아무 것도 실행되지 않습니다. 

콜백을 실행하지 않을 경우, **Adjust SDK는 항상 기본값으로 딥링크를 엽니다**.

#### <a id="deeplinking-reattribution">딥링크를 통한 리어트리뷰션

Adjust는 딥링크를 사용하여 광고 캠페인 리인게이지먼트(re-engagement)를 수행할 수 있게 해줍니다. 이에 대한 자세한 정보는 [관련 문서][reattribution-with-deeplinks]를 참조하세요. 

이 기능을 사용 중이라면, 사용자를 올바로 리어트리뷰트하기 위해 앱에서 Adjust SDK에 호출을 하나 더 수행해야 합니다.

앱에서 딥링크 내용을 수신했다면, `Adjust.AppWillOpenUrl` 메서드 호출을 추가하세요. 이 호출이 이루어지면 Adjust SDK는 딥링크 내에 새로운 어트리뷰션 정보가 있는지 확인하고, 새 정보가 있으면 Adjust 백엔드로 송신합니다. 딥링크 정보가 담긴 Adjust 트래커 URL을 클릭한 사용자를 리어트리뷰트해야 할 경우, 앱에서 해당 사용자의 새 어트리뷰션 정보로 [어트리뷰션 콜백](#attribution-callback)이 촉발되는 것을 확인할 수 있습니다. 

모든 iOS 버전에서 딥링크 리어트리뷰션을 지원하기 위한 `Adjust.AppWillOpenUrl` 호출은 다음과 같이 이루어집니다.

```cs
public override bool OpenUrl(UIApplication application, NSUrl url, string sourceApplication, NSObject annotation)
{
    // url -> This is your deep link content.
    Adjust.AppWillOpenUrl(url);

    return true;
}
```

```cs
public override bool ContinueUserActivity(UIApplication application, NSUserActivity userActivity, UIApplicationRestorationHandler completionHandler)
{
    if (userActivity.ActivityType == NSUserActivityType.BrowsingWeb)
    {
        // userActivity.WebPageUrl -> This is your deep link content.
        Adjust.AppWillOpenUrl(userActivity.WebPageUrl);
    }

    return true;
}
```

[dashboard]: 	http://adjust.com
[adjust.com]:	http://adjust.com

[releases]:             https://github.com/adjust/xamarin_sdk/releases
[demo-app-ios]:         /./iOS
[event-tracking]:       https://docs.adjust.com/en/event-tracking
[callbacks-guide]:      https://docs.adjust.com/en/callbacks
[special-partners]:     https://docs.adjust.com/en/special-partners
[attribution-data]:     https://github.com/adjust/sdks/blob/master/doc/attribution-data.md
[currency-conversion]:  https://docs.adjust.com/en/event-tracking/#tracking-purchases-in-different-currencies

[ios-readme-deeplinking]:	        https://github.com/adjust/ios_sdk/#deeplink-reattributions
[reattribution-with-deeplinks]:   https://docs.adjust.com/en/deeplinking/#manually-appending-attribution-data-to-a-deep-link

[run]: 			        https://github.com/adjust/sdks/blob/master/Resources/xamarin/ios/run.png
[select_ios_dll]: 	https://github.com/adjust/sdks/blob/master/Resources/xamarin/ios/select_ios_dll.png
[add_ios_binding]: 	https://github.com/adjust/sdks/blob/master/Resources/xamarin/ios/add_ios_binding.png
[additional_flags]:	https://github.com/adjust/sdks/blob/master/Resources/xamarin/ios/additional_flags.png

[select_ios_binding]: 		https://github.com/adjust/sdks/blob/master/Resources/xamarin/ios/select_ios_binding.png
[submodule_ios_binding]: 	https://github.com/adjust/sdks/blob/master/Resources/xamarin/ios/submodule_ios_binding.png
[reference_ios_binding]: 	https://github.com/adjust/sdks/blob/master/Resources/xamarin/ios/reference_ios_binding.png

[deeplinking_universal_links]:	https://github.com/adjust/sdks/blob/master/Resources/xamarin/ios/deeplinking_universal_links.png
[deeplinking_custom_url_scheme]:	https://github.com/adjust/sdks/blob/master/Resources/xamarin/ios/deeplinking_custom_url_scheme.png

## <a id="license">라이선스

Adjust SDK는 MIT 라이선스에 따라 사용이 허가됩니다.

Copyright (c) 2012-2017 adjust GmbH, http://www.adjust.com

이로써 본 소프트웨어와 관련 문서 파일(이하 "소프트웨어")의 복사본을 받는 사람에게는 아래 조건에 따라 소프트웨어를 제한 없이 다룰 수 있는 권한이 무료로 부여됩니다. 이 권한에는 소프트웨어를 사용, 복사, 수정, 병합, 출판, 배포 및/또는 판매하거나 2차 사용권을 부여할 권리와 소프트웨어를 제공 받은 사람이 소프트웨어를 사용, 복사, 수정, 병합, 출판, 배포 및/또는 판매하거나 2차 사용권을 부여하는 것을 허가할 수 있는 권리가 제한 없이 포함됩니다.

위 저작권 고지문과 본 권한 고지문은 소프트웨어의 모든 복사본이나 주요 부분에 포함되어야 합니다.

소프트웨어는 상품성, 특정 용도에 대한 적합성 및 비침해에 대한 보증 등을 비롯한 어떤 종류의 명시적이거나 암묵적인 보증 없이 "있는 그대로" 제공됩니다. 어떤 경우에도 저작자나 저작권 보유자는 소프트웨어와 소프트웨어의 사용 또는 기타 취급에서 비롯되거나 그에 기인하거나 그와 관련하여 발생하는 계약 이행 또는 불법 행위 등에 관한 배상 청구, 피해 또는 기타 채무에 대해 책임지지 않습니다.
--END--