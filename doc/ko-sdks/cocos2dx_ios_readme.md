## 개요

Adjust™의 Cocos2d-x 소프트웨어 개발 도구(SDK)입니다. Adjust™에 대해 더 자세한 내용은 [adjust.com]에서 찾아보실 수 있습니다.

## Table of contents

* [기본 연동기능](#basic-integration)
   * [SDK 받기](#sdk-get)
   * [프로젝트에 SDK 추가](#sdk-add)
   * [프로젝트에 프레임워크 추가](#sdk-frameworks)
   * [추가 링커 플래그 추가](#sdk-linker-flags)
   * [앱에 SDK 연동](#sdk-integrate)
   * [Adjust 로그 기록(logging)](#sdk-logging)
   * [세션 추적](#sdk-session-tracking)
   * [앱 제작](#sdk-build)
* [부가 기능](#additional-features)
      * [수익 추적](#revenue-tracking)
      * [수익 중복 제거](#revenue-deduplication)
      * [인앱 구매 검증](#iap-verification)
      * [콜백 파라미터](#callback-parameters)
      * [파트너 파라미터](#partner-parameters)
    * [세션 파라미터](#session-parameters)
      * [세션 콜백 파라미터](#session-callback-parameters)
      * [세션 파트너 파라미터](#session-partner-parameters)
      * [예약 시작(delay start)](#delay-start)
      * [속성 콜백](#attribution-callback)
      * [세션 및 이벤트 콜백](#session-event-callbacks)
      * [추적 해제](#disable-tracking)
      * [오프라인 모드](#offline-mode)
      * [이벤트 버퍼링(buffering)](#event-buffering)
      * [배경 추적](#background-tracking)
      * [기기 ID](#device-ids)
        * [iOS 광고 식별자(identifier)](#di-idfa)
        * [Adjust 기기 식별자](#di-adid)
        * [유저 속성](#user-attribution)
    * [푸시 토큰(push token)](#push-token)
    * [사전 설치 트래커(pre-installed trackers)](#pre-installed-trackers)
    * [딥링크](#deeplinking)
        * [기본 딥링크](#deeplinking-standard)
        * [거치(deferred) 딥링크](#deeplinking-deferred)
        * [iOS 앱 용 딥링크](#deeplinking-ios)
* [라이선스](#license)

## <a id="basic-integration">기본 연동기능

Adjust SDK을 Cocos2d-x iOS 프로젝트에 연동하는 방법.

### <a id="sdk-get">SDK 받기

Adjust 웹 사이트 [자료실 페이지][releases]에서 최신 버전을 다운로드한 후 압축 파일을 원하는 디렉토리에 풀면 됩니다.

### <a id="sdk-add">프로젝트에 SDK 추가

`Adjust` 폴더에 있는 파일을 iOS 프로젝트에 추가하면 됩니다.

![][add-ios-files]

### <a id="sdk-frameworks">프로젝트에 프레임워크 추가

프로젝트에 AdjustSdk, AdSupport, 그리고 iAd 프레임워크를 추가해야 합니다.

프로젝트에 `AdjustSdk.framework`를 추가하기 전에 프로젝트 안에 있는  `proj.ios_mac` 폴더에 먼저 복사하는 걸 잊지 마십시오.

Project Navigator에서 프로젝트를 선택하십시오. 메인 화면 보기 왼쪽에서 타겟을 선택하면 됩니다. 그 다음 `Build Phases` 탭에서 `Link Binary With Libraries` 그룹을 펴십시오. 섹션 맨 아랫부분에서 `+` 버튼을 클릭하여 `AdjustSdk.framework`을 선택한 후 `Add` 버튼을 클릭합니다. `AdSupport.framework` 및 `iAd.framework`를 추가할 때도 이 과정을 반복하면 됩니다.

![][add-the-frameworks]

### <a id="sdk-linker-flags">추가 링커 플래그 추가

`AdjustSdk.framework`에서 카테고리를 지원하려면 링커 플래그를 추가해야 합니다. 프로젝트 설정 내 `Build Settings` 부분에서 `Other Linker Flags` 옵션을 찾아 `-ObjC` 플래그를 추가하면 됩니다.

![][add-other-linker-flags]

### <a id="sdk-integrate">앱에 SDK 연동

Project Navigator에서 앱 델리게이트(delegate)의 소스파일을 열어 맨 위에 임포트(import)문을 추가한 후, `applicationDidFinishLaunching` 메소드에 다음과 같이 Adjust 호출 라인을 추가하세요.

```cpp
#include "Adjust/Adjust2dx.h"
// ...
std::string appToken = "{YourAppToken}";
std::string environment = AdjustEnvironmentSandbox2dx;

AdjustConfig2dx adjustConfig = AdjustConfig2dx(appToken, environment);

Adjust2dx::start(adjustConfig);
```

![][add-adjust2dx]

`{YourAppToken}`을 해당 앱 토큰으로 바꾸면 됩니다. [대시보드]에서 찾을 수 있습니다.

테스트 빌드인지 제작 빌드인지 여부에 따라 `environment`를 다음 값 중 하나로 설정해야 합니다.

```cpp
std::string environment = AdjustEnvironmentSandbox2dx;
std::string environment = AdjustEnvironmentProduction2dx;
```

**주의:** 앱을 테스트하는 경우 이 값을 `AdjustEnvironmentSandbox2dx`로 설정해야 합니다. 앱을 게시하기 전에는 `AdjustEnvironmentProduction2dx`로 바꾸는 걸 잊지 마세요. 개발 및 테스트 단계로 돌아가면 설정을 다시 `AdjustEnvironmentSandbox2dx`에 맞춰야 합니다.

이는 실제 트래픽과 테스트 기기 상의 테스트 트래픽을 구분하기 위해서입니다. 언제나 그렇지만, 특히 수익 추적시에는 이 값을 유의미하게 두는 게 대단히 중요합니다.

### <a id="sdk-logging">Adjust 로그 기록

`AdjustConfig2dx`인스턴스에서 아래 파라미터 중 하나로 `setLogLevel`을 호출하여 테스트에서 확인하는 로그 기록 분량을 늘이거나 줄일 수 있습니다.

```cpp
adjustConfig.setLogLevel(AdjustLogLevel2dxVerbose);     // enable all logging
adjustConfig.setLogLevel(AdjustLogLevel2dxDebug);       // enable more logging
adjustConfig.setLogLevel(AdjustLogLevel2dxInfo);        // the default
adjustConfig.setLogLevel(AdjustLogLevel2dxWarn);        // disable info logging
adjustConfig.setLogLevel(AdjustLogLevel2dxError);       // disable warnings as well
adjustConfig.setLogLevel(AdjustLogLevel2dxAssert);      // disable errors as well
adjustConfig.setLogLevel(AdjustLogLevel2dxSuppress);    // disable all log output
```

Suppress 로그 레벨을 사용하고자 할 경우, 사용 여부 지정 기능의 `bool`을 추가로 받는 생성자(constructor)를 써서 `AdjustConfig2dx` 인스턴스를 초기화해야 합니다.

```cpp
std::string appToken = "{YourAppToken}";
std::string environment = AdjustEnvironmentSandbox2dx;

AdjustConfig2dx adjustConfig = AdjustConfig2dx(appToken, environment, true);
adjustConfig.setLogLevel(AdjustLogLevel2dxSuppress);
```

## <a id="sdk-build">앱 빌드

앱 빌드를 실행해 보세요. 빌드가 성공적이면 콘솔에서 SDK 로그를 주의 깊게 읽어보세요. 최초 런칭 후 정보 로그 시작 부분에 `Install tracked`가 보일 겁니다.

![][run]

## <a id="additional-features">부가 기능

Adjust SDK를 프로젝트에 연동하면 다음과 같은 기능을 이용할 수 있습니다.

### <a id="event-tracking">Event tracking

With Adjust, you can track any event that you want. 

Suppose you want to track every tap on a button. If you create a new event token in your [dashboard] - let's say that event token is `abc123` - you can add the following line in your button’s click handler method to track the click:

```cpp
AdjustEvent2dx adjustEvent = AdjustEvent2dx("abc123");
Adjust2dx::trackEvent(adjustEvent);
```

### <a id="revenue-tracking">Revenue tracking

Adjust로 원하는 어떤 이벤트든 추적할 수 있습니다.

버튼을 탭할 때마다 이를 추적하고 싶다고 해봅시다. [대시보드]에 새 이벤트 토큰을 만듭니다 - 편의상 토큰을 `abc123`이라고 합시다 - 다음 라인을 버튼 클릭 핸들러 메소드에 붙이면 클릭 추적이 가능합니다.

```cpp
AdjustEvent2dx adjustEvent = AdjustEvent2dx("abc123");

adjustEvent.setRevenue(0.01, "EUR");

Adjust2dx::trackEvent(adjustEvent);
```

통화 토큰 설정 시, Adjust는 들어오는 수익을 선택한 방식에 따라 자동으로 보고 수익으로 바꿔줍니다. 통화 전환에 대한 자세한 내용은 [여기서 확인하세요][currency-conversion].

수익 및 이벤트 추적에 대한 자세한 내용은 [이벤트 추적 지침에서 확인하세요][event-tracking].

### <a id="revenue-deduplication"></a>Revenue deduplication

중복 수익 추적을 피하기 위해 거래 ID(transaction ID)를 선택 사항으로 추가할 수 있습니다. 가장 최근의 10개 거래 ID가 기억되며, 중복 거래 ID로부터의 수익 이벤트는 합산되지 않습니다. 인앱 구매 추적에 특히 유용합니다. 실제 예는 아래를 참조하세요.

인앱 구매를 추적하시려면, 거래 완결 시에 한해 `trackEvent`를 호출하세요. 이렇게 하면 실제로 생성되지 않은 수익을 추적하는 걸 막을 수 있습니다.

```cpp
AdjustEvent2dx adjustEvent = AdjustEvent2dx("abc123");

adjustEvent.setRevenue(0.01, "EUR");
adjustEvent.setTransactionId("transactionID");

Adjust2dx::trackEvent(adjustEvent);
```

**주의**: 거래 ID는 iOS 용어이며, 성공적으로 완결된 안드로이드 인앱 구매 용 고유 식별자는 **주문 ID(Order ID)**입니다.

### <a id="iap-verification">인앱 구매 검증

인앱 구매 검증은 Cocos2d-x Purchase SDK로 할 수 있는데, 이는 현재 개발 중이며 곧 시중에 내놓을 예정입니다. 자세한 정보는 support@adjust.com으로 문의해 주십시오.

### <a id="callback-parameters">Callback parameters

[대시보드] 내 이벤트에 사용할 콜백 URL을 등록할 수 있습니다. 이벤트를 추적할 때마다 해당 URL로 GET 요청을 전송합니다. 콜백 파라미터를 이벤트에 추가하시려면 추적하기 전 해당 이벤트의 `addCallbackParameter` 메소드를 호출하면 됩니다. 그러면 이 파라미터를 콜백 URL에 덧붙이게 됩니다.

예를 들어 이벤트 토큰 `abc123`에 해당하는 이벤트 추적을 위해  `http://www.adjust.com/callback`라는 URL을 등록했다고 가정합시다. 그러면 다음 라인을 실행하면 됩니다.

```cpp
AdjustEvent2dx adjustEvent = AdjustEvent2dx("abc123");

adjustEvent.addCallbackParameter("key", "value");
adjustEvent.addCallbackParameter("foo", "bar");

Adjust2dx::trackEvent(adjustEvent);
```

이 경우, 이벤트를 추적하여 다음 URL로 요청을 보냅니다.

```
http://www.adjust.com/callback?key=value&foo=bar
```

Adjust는 파라미터 값으로 사용하는 `{idfa}` 등의 다양한 플레이스홀더(placeholder)를 지원합니다. 결과 콜백 시 `{idfa}`는 현재 기기 광고자 ID로 대체됩니다. Adjust는 고객 파라미터를 저장하지 않으며 콜백에 덧붙일 뿐입니다. 이벤트 콜백을 등록하지 않으셨다면 이들 파라미터는 아예 읽히지 않습니다. 가능한 전체 값 목록을 비롯하여 자세한 URL 콜백 사용법은 [콜백 지침][callbacks-guide]에서 확인하세요.

### <a id="partner-parameters">파트너 파라미터

네트워크 파트너에게 전송할 파라미터도 추가할 수 있습니다. 현재 Adjust 대시보드에서 활성화된 연동 부분에서 사용합니다.

이들 파라미터는 위에 설명한 콜백 파라미터와 비슷하게 작동하지만, `AdjustEvent2dx` 인스턴스의 `addPartnerParameter` 메소드를 호출하여 추가합니다.

```cpp
AdjustEvent2dx adjustEvent = AdjustEvent2dx("abc123");

adjustEvent.addPartnerParameter("key", "value");
adjustEvent.addPartnerParameter("foo", "bar");

Adjust2dx::trackEvent(adjustEvent);
```

특별 파트너 및 그 연동 방법에 대한 자세한 내용은 [특별 파트너 지침][special-partners]에서 확인하세요.

### <a id="session-parameters">세션 파라미터

Adjust SDK 매 이벤트 및 세션마다 전송되도록 저장하는 파라미터가 있습니다. 이들 파라미터중 어느 하나라도 이미 추가했다면 로컬에 저장되므로 다시 추가하지 않아도 됩니다. 같은 파라미터를 두 번 저장해도 그 효과는 나타나지 않습니다.

이들 세션 파라미터는 Adjust SDK를 런칭하여 인스톨 시에 전송하도록 하기 전에 호출할 수 있습니다. 인스톨 시 전송해야 하나 런칭 후에만 그 필요값을 얻을 수 있는 경우에는 Adjust SDK 최초 런칭 시 [예약 시작](#delay-start) 기능을 사용하면 됩니다.

### <a id="session-callback-parameters"> 세션 콜백 파라미터

[이벤트](#callback-parameters)용으로 등록한 콜백 파라미터는 Adjust SDK 내 어떤 이벤트나 세션에서든 저장하여 전송할 수 있습니다.

세션 콜백 파라미터는 이벤트 콜백 파라미터와 인터페이스가 비슷하지만, 이벤트에 키와 값을 추가하는 대신 `Adjust2dx` 인스턴스에서 `addSessionCallbackParameter` 메소드를 호출하여 추가합니다.

```cpp
Adjust2dx::addSessionCallbackParameter("foo", "bar");
```

세션 콜백 파라미터는 이벤트에 추가한 콜백 파라미터와 병합됩니다. 이벤트에 추가된 콜백 파라미터가 세션 콜백 파라미터보다 우선순위를 지닙니다. 그러나 세션에서와 같은 키로 이벤트에 콜백 파라미터를 추가한 경우 새로 추가한 콜백 파라미터가 우선권을 가집니다.

원하는 키를 `Adjust2dx` 인스턴스의 `removeSessionCallbackParameter` 메소드로 전달하여 특정 세션 콜백 파라미터를 제거할 수 있습니다.

```cpp
Adjust2dx::removeSessionCallbackParameter("foo");
```

세션 콜백 파라미터의 키와 값을 전부 없애고 싶다면 `Adjust2dx` 인스턴스의 `resetSessionCallbackParameters` 메소드로 재설정하면 됩니다.

```cpp
Adjust2dx::resetSessionCallbackParameters();
```

### <a id="session-partner-parameters">세션 파트너 파라미터

Adjust SDK 내 모든 이벤트 및 세션에서 전송되는 [세션 콜백 파라미터](#session-callback-parameters)가 있는 것처럼, 세션 파트너 파라미터도 있습니다. 이들 파라미터는 Adjust [대시보드]에서 연동되고 활성화된 네트워크 파트너에게 전송할 수 있습니다.

세션 파트너 파라미터는 이벤트 파트너 파라미터와 인터페이스가 비슷하지만, 인터페이스가 비슷하지만, 이벤트에 키와 값을 추가하는 대신 `Adjust2dx` 인스턴스에서 `addSessionPartnerParameter` 메소드를 호출하여 추가합니다.

```cpp
Adjust2dx::addSessionPartnerParameter("foo", "bar");
```

세션 파트너 파라미터는 이벤트에 추가한 파트너 파라미터와 병합됩니다. 이벤트에 추가된 파트너 파라미터가 세션 파트너 파라미터보다 우선순위를 지닙니다. 그러나 세션에서와 같은 키로 이벤트에 파트너 파라미터를 추가한 경우, 새로 추가한 파트너 파라미터가 우선권을 가집니다.

원하는 키를 `Adjust2dx` 인스턴스의 `removeSessionPartnerParameter` 메소드로 전달하여 특정 세션 파트너 파라미터를 제거할 수 있습니다.

```cpp
Adjust2dx::removeSessionPartnerParameter("foo");
```

세션 파트너 파라미터의 키와 값을 전부 없애고 싶다면 `Adjust2dx` 인스턴스의 `resetSessionPartnerParameters` 메소드로 재설정하면 됩니다.

```cpp
Adjust2dx::resetSessionPartnerParameters();
```

### <a id="delay-start">예약 시작

Adjust SDK에 예약 시작을 걸면 앱이 고유 식별자 등의 세션 파라미터를 얻어 인스톨 시에 전송할 시간을 벌 수 있습니다.

`AdjustConfig2dx` 인스턴스의 `setDelayStart` 메소드에서 예약 시작 시각을 초 단위로 설정하세요.

```cpp
config.setDelayStart(5.5);
```

이 경우 Adjust SDK는 최초 인스톨 세션 및 생성된 이벤트를 5.5초간 기다렸다가 전송합니다. 이 시간이 지난 후, 또는 그 사이에 `Adjust2dx::sendFirstPackages()`을 호출했을 경우 모든 세션 파라미터가 지연된 인스톨 세션 및 이벤트에 추가되며 Adjust SDK는 원래대로 돌아옵니다.

**Adjust SDK의 최대 지연 예약 시작 시간은 10초입니다**.

### <a id="attribution-callback">속성 콜백

Adjust는 속성을 어떻게 변화시켜도 콜백을 보낼 수 있습니다. 속성에서 고려하는 소스가 각각 다르기 때문에, 이 정보는 동시간에 제공할 수 없습니다. 다음 단계를 거쳐 어플리케이션에 속성 콜백을 시행하세요.

1. `AdjustAttribution2dx` 파라미터를 받는 빈 메소드를 생성합니다.

2. `AdjustConfig2dx` 인스턴스를 생성한 후 이전에 만든 메소드를 파라미터로 하여 `setAttributionCallback` 메소드를 호출합니다.

SDK가 최종 속성 데이터를 받고 나면 콜백 기능이 호출됩니다. 콜백 기능이 작동하는 동안 `attribution` 파라미터에 억세스할 수 있습니다. 각 특성을 간단히 요약하면 다음과 같습니다.

- `std::string trackerToken` 현재 인스톨 트래커 토큰.
- `std::string trackerName` 현재 인스톨 트래커 이름.
- `std::string network` 현재 인스톨 네트워크 그루핑 레벨.
- `std::string campaign` 현재 인스톨 캠페인 그루핑 레벨.
- `std::string adgroup` 현재 인스톨 광고 그루핑 레벨.
- `std::string creative` 현재 인스톨 크리에이티브 그루핑 레벨.
- `std::string clickLabel` 현재 인스톨 클릭 레벨.
- `std::string adid` Adjust 기기 식별자.

```cpp
#include "Adjust/Adjust2dx.h"

//...

static void attributionCallbackMethod(AdjustAttribution2dx attribution) {
    // Printing all attribution properties.
    CCLOG("\nAttribution changed!");
    CCLOG("\nTracker token: %s", attribution.getTrackerToken().c_str());
    CCLOG("\nTracker name: %s", attribution.getTrackerName().c_str());
    CCLOG("\nNetwork: %s", attribution.getNetwork().c_str());
    CCLOG("\nCampaign: %s", attribution.getCampaign().c_str());
    CCLOG("\nAdgroup: %s", attribution.getAdgroup().c_str());
    CCLOG("\nCreative: %s", attribution.getCreative().c_str());
    CCLOG("\nClick label: %s", attribution.getClickLabel().c_str());
    CCLOG("\nAdid: %s", attribution.getAdid().c_str());
    CCLOG("\n");
}

// ...

bool AppDelegate::applicationDidFinishLaunching() {
    std::string appToken = "{YourAppToken}";
    std::string environment = AdjustEnvironmentSandbox2dx;

    AdjustConfig2dx adjustConfig = AdjustConfig2dx(appToken, environment);
    adjustConfig.setLogLevel(AdjustLogLevel2dxVerbose);
    adjustConfig.setAttributionCallback(attributionCallbackMethod);

    Adjust2dx::start(adjustConfig);

    // ...
}
```

[적용 가능한 속성 데이터 정책][attribution-data]을 고려하는 걸 잊지 마세요.

### <a id="session-event-callbacks">세션 및 이벤트 콜백

콜백을 등록하여 이벤트/세션 추적에 성공하거나 실패할 경우 알림을 받을 수 있습니다.

[위에 설명한 속성 콜백](#attribution-callback)과 같은 단계를 거쳐 다음과 같이 이벤트 추적 성공 시 콜백 기능을 수행하세요.

```cpp
#include "Adjust/Adjust2dx.h"

//...

static void eventSuccessCallbackMethod(AdjustEventSuccess2dx eventSuccess) {
    CCLOG("\nEvent successfully tracked!");
    CCLOG("\nADID: %s", eventSuccess.getAdid().c_str());
    CCLOG("\nMessage: %s", eventSuccess.getMessage().c_str());
    CCLOG("\nTimestamp: %s", eventSuccess.getTimestamp().c_str());
    CCLOG("\nEvent token: %s", eventSuccess.getEventToken().c_str());
    CCLOG("\nJSON response: %s", eventSuccess.getJsonResponse().c_str());
    CCLOG("\n");
}

// ...

bool AppDelegate::applicationDidFinishLaunching() {
    std::string appToken = "{YourAppToken}";
    std::string environment = AdjustEnvironmentSandbox2dx;

    AdjustConfig2dx adjustConfig = AdjustConfig2dx(appToken, environment);
    adjustConfig.setLogLevel(AdjustLogLevel2dxVerbose);
    adjustConfig.setEventSuccessCallback(eventSuccessCallbackMethod);

    Adjust2dx::start(adjustConfig);

    // ...
}
```

아래는 이벤트 추적 실패 시 콜백 기능입니다.

```cpp
#include "Adjust/Adjust2dx.h"

//...

static void eventFailureCallbackMethod(AdjustEventFailure2dx eventFailure) {
    CCLOG("\nEvent tracking failed!");
    CCLOG("\nADID: %s", eventFailure.getAdid().c_str());
    CCLOG("\nMessage: %s", eventFailure.getMessage().c_str());
    CCLOG("\nTimestamp: %s", eventFailure.getTimestamp().c_str());
    CCLOG("\nWill retry: %s", eventFailure.getWillRetry().c_str());
    CCLOG("\nEvent token: %s", eventFailure.getEventToken().c_str());
    CCLOG("\nJSON response: %s", eventFailure.getJsonResponse().c_str());
    CCLOG("\n");
}

// ...

bool AppDelegate::applicationDidFinishLaunching() {
    std::string appToken = "{YourAppToken}";
    std::string environment = AdjustEnvironmentSandbox2dx;

    AdjustConfig2dx adjustConfig = AdjustConfig2dx(appToken, environment);
    adjustConfig.setLogLevel(AdjustLogLevel2dxVerbose);
    adjustConfig.setEventFailureCallback(eventFailureCallbackMethod);

    Adjust2dx::start(adjustConfig);

    // ...
}
```

다음은 세션 추적 성공 시 콜백입니다.

```cpp
#include "Adjust/Adjust2dx.h"

//...

static void sessionSuccessCallbackMethod(AdjustSessionSuccess2dx sessionSuccess) {
    CCLOG("\nSession successfully tracked!");
    CCLOG("\nADID: %s", sessionSuccess.getAdid().c_str());
    CCLOG("\nMessage: %s", sessionSuccess.getMessage().c_str());
    CCLOG("\nTimestamp: %s", sessionSuccess.getTimestamp().c_str());
    CCLOG("\nJSON response: %s", sessionSuccess.getJsonResponse().c_str());
    CCLOG("\n");
}

// ...

bool AppDelegate::applicationDidFinishLaunching() {
    std::string appToken = "{YourAppToken}";
    std::string environment = AdjustEnvironmentSandbox2dx;

    AdjustConfig2dx adjustConfig = AdjustConfig2dx(appToken, environment);
    adjustConfig.setLogLevel(AdjustLogLevel2dxVerbose);
    adjustConfig.setSessionSuccessCallback(sessionSuccessCallbackMethod);

    Adjust2dx::start(adjustConfig);

    // ...
}
```

그리고 세션 추적 실패 시 콜백은 아래와 같습니다.

```cpp
#include "Adjust/Adjust2dx.h"

//...

static void sessionFailureCallbackMethod(AdjustSessionFailure2dx sessionFailure) {
    CCLOG("\nSession tracking failed!");
    CCLOG("\nADID: %s", sessionFailure.getAdid().c_str());
    CCLOG("\nMessage: %s", sessionFailure.getMessage().c_str());
    CCLOG("\nTimestamp: %s", sessionFailure.getTimestamp().c_str());
    CCLOG("\nWill retry: %s", sessionFailure.getWillRetry().c_str());
    CCLOG("\nJSON response: %s", sessionFailure.getJsonResponse().c_str());
    CCLOG("\n");
}

// ...

bool AppDelegate::applicationDidFinishLaunching() {
    std::string appToken = "{YourAppToken}";
    std::string environment = AdjustEnvironmentSandbox2dx;

    AdjustConfig2dx adjustConfig = AdjustConfig2dx(appToken, environment);
    adjustConfig.setLogLevel(AdjustLogLevel2dxVerbose);
    adjustConfig.setSessionFailureCallback(sessionFailureCallbackMethod);

    Adjust2dx::start(adjustConfig);

    // ...
}
```

SDK가 서버에 패키지 전송 시도 시 콜백 기능이 호출됩니다. 콜백 기능이 작동하는 동안 콜백용 응답 데이터 객체에 억세스할 수 있습니다. 세션 응답 데이터 특성을 간단히 요약하면 아래와 같습니다.

- `std::string message` 서버로부터의 메시지 또는 SDK에 기록된 에러.
- `std::string timestamp` 서버로부터의 시간 스탬프.
- `std::string adid` Adjust가 제공하는 고유 기기 식별자.
- `std::string jsonResponse` 서버로부터의 응답을 담은 JSON 객체.

이벤트 응답 데이터 객체 두 가지에 다 들어있는 특성은 아래와 같습니다.

- `std::string eventToken` 추적한 패키지가 이벤트일 경우 이벤트 토큰.

이벤트 및 세션 객체 실패 시에는 다음 특성이 들어갑니다.

- `std::string willRetry` 패키지를 나중에 다시 전송 시도한다는 의미입니다.

### <a id="disable-tracking">추적 해제

적용 파라미터를 `false`로 설정한 `Adjust2dx::setEnabled` 메소드를 인보크(invoke)하여 Adjust SDK를 추적 해제할 수 있습니다. 이 설정은 **세션과 세션 사이에서 기억됩니다**. 하지만 첫 번째 세션 후에만 적용할 수 있습니다.

```cpp
Adjust2dx::setEnabled(false);
```

`Adjust2dx::isEnabled()` 메소드로 Adjust SDK가 현재 사용중인 지 검증할 수 있습니다. 적용 파라미터를 `true`로 설정한  `Adjust2dx::setEnabled` 메소드를 인보크하여 Adjust SDK를 언제든 활성화할 수 있습니다.

### <a id="offline-mode">오프라인 모드

추적한 데이터를 추후 전송하기 위해 유지하면서 서버로의 전송을 연기하기 위해 Adjust SDK를 오프라인 모드로 둘 수 있습니다. 오프라인 모드일 때는 모든 정보가 파일에 저장되므로 이벤트를 지나치게 많이 트리거(trigger)하지 않는 게 좋습니다.

파라미터가 `true`로 설정된 `Adjust2dx::setOfflineMode` 메소드를 호출하여 오프라인 모드를 활성화할 수 있습니다.

```cpp
Adjust2dx::setOfflineMode(true);
```

반대로, 파라미터가 `false`로 설정된 `Adjust2dx::setOfflineMode` 메소드를 호출하면 오프라인 모드를 비활성화할 수 있습니다. Adjust SDK가 온라인 모드로 돌아오면, 저장된 모든 정보가 올바른 시간 정보와 함께 서버로 전송됩니다.

추적 해체와 달리, 세션과 세션 사이에서 **이 설정은 기억되지 않습니다**. 이는 앱이 오프라인 모드로 꺼진다고 해도 SDK는 언제나 온라인 모드에서 시작한다는 뜻입니다.

### <a id="event-buffering">이벤트 버퍼링

앱이 이벤트 추적을 많이 사용하는 경우, 매 분마다 배치 하나씩만 보내도록 하기 위해 일부 HTTP 요청을 지연시키고자 할 경우가 있을 수 있습니다. `AdjustConfig2dx` 인스턴스에서 `setEventBufferingEnabled` 메소드를 호출하여 이벤트 버퍼링을 적용할 수 있습니다.

```cpp
// ...

bool AppDelegate::applicationDidFinishLaunching() {
    std::string appToken = "{YourAppToken}";
    std::string environment = AdjustEnvironmentSandbox2dx;

    AdjustConfig2dx adjustConfig = AdjustConfig2dx(appToken, environment);
    adjustConfig.setLogLevel(AdjustLogLevel2dxVerbose);
    adjustConfig.setEventBufferingEnabled(true);

    Adjust2dx::start(adjustConfig);

    // ...
}
```

여기에 설정한 내용이 없으면 이벤트 버퍼링은 **기본값으로 해제됩니다**.

### <a id="background-tracking">배경 추적

Adjust SDK 기본값 행위는 **앱이 배경에 있을 동안에는 HTTP 요청 전송을 잠시 중지**하는 것입니다. `AdjustConfig2dx` 인스턴스에서 `setSendInBackground` 메소드를 호출하면 이를 바꿀 수 있습니다.

```cpp
// ...

bool AppDelegate::applicationDidFinishLaunching() {
    std::string appToken = "{YourAppToken}";
    std::string environment = AdjustEnvironmentSandbox2dx;

    AdjustConfig2dx adjustConfig = AdjustConfig2dx(appToken, environment);
    adjustConfig.setLogLevel(AdjustLogLevel2dxVerbose);
    adjustConfig.setSendInBackground(true);

    Adjust2dx::start(adjustConfig);

    // ...
}
```

여기에 설정한 내용이 없으면 배경에서의 전송은 **기본값으로 해제됩니다**.

### <a id="device-ids">기기 ID

(Google Analytics 등) 일부 기기의 경우 중복 보고를 막기 위해 기기 및 클라이언트 ID를 조정해야 합니다.

### <a id="di-idfa">iOS 광고 식별자

`Adjust2dx` 인스턴스에서 `getIdfa()` 메소드를 인보크하여 iOS 기기에서 IDFA 값에 억세스할 수 있습니다.

```cpp
std::string idfa = Adjust2dx::getIdfa();
```

### <a id="di-adid"></a>Adjust 기기 식별자

Adjust 백엔드는 앱을 인스톨한 기기에서 고유한 **Adjust 기기 식별자** (**adid**)를 생성합니다. 이 식별자를 얻으려면 `Adjust2dx` 인스턴스에서 다음 메소드를 호출하면 됩니다.

```cpp
std::string adid = Adjust2dx::getAdid();
```

**주의**: **adid** 관련 정보는 Adjust 백엔드가 앱 인스톨을 추적한 후에만 얻을 수 있습니다. 그 순간부터 Adjust SDK는 기기 **adid** 정보를 갖게 되며 이 메소드로 억세스할 수 있습니다. 따라서 SDK가 초기화되고 앱 인스톨 추적이 성공적으로 이루어지기 전에는 **adid** 억세스가 **불가능합니다**.

### <a id="user-attribution"></a>User attribution

[속성 콜백 섹션](#attribution-callback)에서 설명한 바와 같이, 이 콜백은 변동이 있을 때마다 새로운 속성 관련 정보를 전달할 목적으로 촉발됩니다. 유저의 현재 속성 값 관련 정보를 언제든 억세스하고 싶다면, `Adjust2dx` 인스턴스의 다음 메소드를 호출하면 됩니다.

```cpp
AdjustAttribution2dx attribution = Adjust2dx::getAttribution();
```

**주의**: 유저의 현재 속성 값 관련 정보는 Adjust 백엔드가 앱 인스톨을 추적하여 최초 속성 콜백이 촉발된 후에만 얻을 수 있습니다. 그 순간부터 Adjus SDK는 유저 속성 값 정보를 갖게 되며 이 메소드로 억세스할 수 있습니다. 따라서 SDK가 초기화되고 최초 속성 콜백이 촉발되기 전에는 유저 속성 값 억세스가 **불가능합니다**. 

### <a id="push-token">푸시 토큰

Adjust로 푸시 알림 토큰을 전송하려면 **앱에서 토큰을 받거나 업데이트가 있을 때마다** Adjust로 다음 호출을 추가하세요.

```cpp
Adjust2dx::setDeviceToken("YourPushNotificationToken");
```

### <a id="pre-installed-trackers">사전 설치 트래커

Adjust SDK를 사용하여 앱이 사전 설치된 기기를 지닌 유저를 인식하고 싶다면 아래 절차를 따르세요.

1. [대시보드]에 새 트래커를 생성합니다.

2. 앱 델리케이트를 열고 `AdjustConfig2dx` 인스턴스의 기본값 트래커를 다음과 같이 설정합니다.

    ```cpp
    AdjustConfig2dx adjustConfig = AdjustConfig2dx(appToken, environment);

    adjustConfig.setDefaultTracker("{TrackerToken}");

    Adjust2dx::adjustConfig(config);
    ```

  `{TrackerToken}`을 1에서 생성한 트래커 토큰으로 대체합니다. 대시보드에서는 (`http://app.adjust.com/`을 포함하는) 트래커 URL을 표시한다는 사실을 명심하세요. 소스코드에서는 전체 URL을 표시할 수 없으며 6자로 이루어진 토큰만을 명시해야 합니다.

3. 앱 빌드를 실행하세요. 앱 로그 출력 시 다음과 같은 라인을 볼 수 있을 것입니다.

    ```
    Default tracker: 'abc123'
    ```

### <a id="deeplinking">딥링크

URL에서 앱으로 딥링크를 거는 옵션이 있는 Adjust 트래커 URL을 사용하고 있다면, 딥링크 URL과 그 내용 관련 정보를 얻을 가능성이 있습니다. 딥링크에는두 가지 경우가 있는데, 기본 딥링크와 거치 딥링크입니다.

기본 딥링크는 유저가 이미 앱을 인스톨한 경우입니다. 이 때 안드로이드는 딥링크 내용에 관한 정보 인출을 기본 지원합니다.

거치 딥링크는 유저가 앱을 인스톨하지 않았을 경우입니다. 안드로이드는 거치 딥링크를 기본 지원하지 않지만, Adjust SDK는 거치 딥링크 정보를 인출하는 방법을 제공합니다.

생성한 Xcode 프로젝트에서 딥링크 취급을 앱 내 **기본 레벨**로 설정해야 합니다. 

### <a id="deeplinking-standard">기본 딥링크

불행히도 이 경우 Cocos2d-x C++ 코드에서 딥링크 정보가 전달되지 않습니다. 앱에서 딥링크 취급을 구동시키면, 딥링크 관련 정보를 기본 레벨로 받게 됩니다. iOS 앱에서 딥링크를 구동하는 법에 관한 자세한 정보는 아래 장에서 확인하세요.

### <a id="deeplinking-deferred">거치 딥링크

거치 딥링크인 경우 URL 내용 정보를 받으려면, URL 내용을 전달하는 `std::string` 파라미터를 받는 `AdjustConfig2dx` 객체에 콜백 메소드를 설정해야 합니다. `setDeferredDeeplinkCallback` 메소드를 호출하여 `AdjustConfig2dx` 객체 인스턴스에 설정하면 됩니다.

```cpp
#include "Adjust/Adjust2dx.h"

//...

static bool deferredDeeplinkCallbackMethod(std::string deeplink) {
    CCLOG("\nDeferred deep link received!");
    CCLOG("\nURL: %s", deeplink.c_str());
    CCLOG("\n");

    Adjust2dx::appWillOpenUrl(deeplink);

    return true;
}

// ...

bool AppDelegate::applicationDidFinishLaunching() {
    std::string appToken = "{YourAppToken}";
    std::string environment = AdjustEnvironmentSandbox2dx;

    AdjustConfig2dx adjustConfig = AdjustConfig2dx(appToken, environment);
    adjustConfig.setLogLevel(AdjustLogLevel2dxVerbose);
    adjustConfig.setDeferredDeeplinkCallback(deferredDeeplinkCallbackMethod);

    Adjust2dx::start(adjustConfig);

    // ...
}
```

<a id="deeplinking-deferred-open">거치 딥링크인 경우 콜백 메소드에서 설정이 하나 더 필요합니다. Adjust SDK가 거치 딥링크 정보를 받으면, SDK가 이 URL을 열 것인지를 선택할 수 있습니다. 이를 위해서는 거치 딥링크 콜백 메소드의 리턴 값을 설정하세요.

설정 내용이 없을 경우 **Adjust SDK는 URL을 언제나 기본값으로 런칭합니다**.

### <a id="deeplinking-ios">iOS 앱 용 딥링크

**기본 Xcode 프로젝트에서 수행해야 합니다.**

iOS 앱이 딥링크를 기본 레벨에서 취급하도록 설정하려면, 공식 iOS SDK README에 있는 [지침][ios-deeplinking]을 따르세요.

[adjust]:       http://adjust.com
[dashboard]:    http://adjust.com
[adjust.com]:   http://adjust.com

[releases]:             https://github.com/adjust/cocos2dx_sdk/releases
[deeplinking]:          https://docs.adjust.com/en/tracker-generation/#deeplinking
[event-tracking]:       https://docs.adjust.com/en/event-tracking
[callbacks-guide]:      https://docs.adjust.com/en/callbacks
[ios-deeplinking]:      https://github.com/adjust/ios_sdk/#deeplinking
[special-partners]:     https://docs.adjust.com/en/special-partners
[attribution-data]:     https://github.com/adjust/sdks/blob/master/doc/attribution-data.md
[currency-conversion]:  https://docs.adjust.com/en/event-tracking/#tracking-purchases-in-different-currencies

[run]:                    https://raw.github.com/adjust/sdks/master/Resources/cocos2dx/ios/run.png
[add-ios-files]:          https://raw.github.com/adjust/sdks/master/Resources/cocos2dx/ios/add-ios-files.png
[add-adjust2dx]:          https://raw.github.com/adjust/sdks/master/Resources/cocos2dx/ios/add-adjust2dx.png
[add-the-frameworks]:     https://raw.github.com/adjust/sdks/master/Resources/cocos2dx/ios/add-the-frameworks.png
[add-other-linker-flags]: https://raw.github.com/adjust/sdks/master/Resources/cocos2dx/ios/add-other-linker-flags.png

## <a id="license">라이선스

Adjust SDK는 MIT 허가서에 따라 라이선스를 얻었습니다.

Copyright (c) 2012-2017 Adjust GmbH,
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