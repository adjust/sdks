## 요약

이 항목에서는 adjust의 Marmalade SDK에 대해 설명합니다. adjust에 대한 자세한 내용은 [adjust.com]을 참조하십시오.

## 목차

* [앱 예제](#example-app)
* [기본 연동](#basic-integration)
    * [SDK 다운로드 및 설치](#sdk-get)
    * [프로젝트에 SDK 추가](#sdk-add)
    * [앱과 연동](#sdk-integrate)
        * [Adjust 로깅](#adjust-logging)
    * [Android 매니페스트](#android-manifest)
    * [Google Play 서비스](#google-play-services)
* [추가 기능](#additional-features)
    * [이벤트 트래킹](#event-tracking)
        * [매출 트래킹](#revenue-tracking)
            * [매출 중복 제거](#revenue-deduplication)
            * [인앱 구매 유효성 검사](#iap-verification)
        * [콜백 매개변수](#callback-parameters)
        * [파트너 매개변수](#partner-parameters)
    * [어트리뷰션 콜백](#attribution-callback)
    * [세션 및 이벤트 콜백](#session-event-callbacks)
    * [트래킹 사용 중지](#disable-tracking)
    * [오프라인 모드](#offline-mode)
    * [장치 ID](#device-ids)
    * [딥링크 연결](#deeplinking)
        * [기본 딥링크 연결 시나리오](#deeplinking-standard)
        * [지연된 딥링크 연결 시나리오](#deeplinking-deferred)
        * [Android 작업에 대한 구조 설정](#scheme-android)
        * [iOS에서 사용자 지정 URL 구조 설정](#scheme-ios)
* [라이선스](#license)

## <a id="example-app"></a>앱 예제

[`Example` 디렉터리][example-app]에 앱 예제가 있습니다. 앱 예제를 사용하여 adjust SDK를 연동할 수 있는 방법을 확인할 수 있습니다.

## <a id="basic-integration">기본 연동

다음은 adjust SDK를 Marmalade 프로젝트와 연동하기 위해 최소한으로 수행해야 하는 절차입니다.

**참고**: Marmalade용 SDK 4.7.0은 **Marmalade 8.3.0p3**을 사용하여 제작되었으므로 이 Marmalade 버전 이상을 사용하는 것이 좋습니다. Marmalade extension을 직접 다시 작성하려는 경우에는 특히 그렇습니다.

### <a id="sdk-get">1. SDK 다운로드 및 설치

[릴리스 페이지][releases]에서 최신 버전을 다운로드합니다. 압축 파일을 원하는 폴더에 풉니다.

### <a id="sdk-add">2. 프로젝트에 SDK 추가

`AdjustMarmalade`를 하위 프로젝트로 추가합니다.  프로젝트의 `.mkb` 파일을 편집하고 다음 행을 추가합니다.

````
subproject path_to_folder/adjust/AdjustMarmalade
```

### <a id="sdk-integrate">3. 앱과 연동

`AdjustMarmalade.h` 헤더를 `main` 함수가 있는 프로젝트에 추가합니다. `main` 함수에서 `adjust_config` 구조를 초기화하고 `adjust_Start` 함수를 호출합니다.

```cpp
#include "AdjustMarmalade.h"

int main()
{
    const char* app_token = "{YourAppToken}";
    const char* environment = "sandbox";
    const char* log_level = "verbose";

    adjust_config* config = new adjust_config(app_token, environment);
    config->set_log_level(log_level);

    adjust_Start(config);

    // ...
}
```

`{YourAppToken}`을 앱 토큰으로 변경합니다. 앱 토큰은 [대시보드]에서 찾을 수 있습니다.

앱을 테스트에 사용할지 아니면 프로덕션에 사용할 지에 따라 `environment`를 다음 값 중 하나로 설정해야 합니다.

```cpp
const char* environment = "sandbox";
const char* environment = "production";
```

**중요:** 이 값은 앱을 테스트하는 경우에만 `sandbox`로 설정해야 합니다. 앱을 게시하기 직전에 environment를 `production`으로 설정해야 합니다. 개발 및 테스트를 다시 시작하면 `sandbox`로 다시 설정하십시오.

이 environment는 실제 트래픽과 테스트 장치의 테스트 트래픽을 구별하기 위해 사용합니다. 이 값을 항상 의미 있게 유지해야 합니다! 이것은 특히 매출을 트래킹하는 경우에 중요합니다.

#### <a id="sdk-logging">Adjust 로깅

다음 매개변수 중 하나를 사용하여 `adjust_config` 인스턴스에서 `set_log_level`을 호출하여 테스트에 표시되는 로그의 양을 늘리거나 줄일 수 있습니다.

```cpp
config->set_log_level("verbose"); // enable all logging
config->set_log_level("debug");   // enable more logging
config->set_log_level("info");    // the default
config->set_log_level("warn");    // disable info logging
config->set_log_level("error");   // disable warnings as well
config->set_log_level("assert");  // disable errors as well
```

### <a id="android-manifest">4. Android 매니페스트

Android용 Marmalade 앱을 adjust SDK와 함께 사용하려면 앱의 `AndroidManifest.xml` 파일을 변경해야 합니다.

adjust SDK는 Android `INSTALL_REFERRER` intent로 사용할 자체 브로드캐스트 수신기와 SDK에 필요한 권한을 자동으로 추가합니다.

다음 사항을 기억하십시오.

- 사용자 지정 브로드캐스트 수신기를 `INSTALL_REFERRER` intent로 사용하는 경우, adjust [설명서][custom-receiver]의 설명에 따라 필요한 호출을 adjust SDK에 추가하십시오. 이렇게 하면 `AdjustMarmalade.mkf` 파일로 이동하여 adjust 사용자 지정 브로드캐스트 수신기를 앱의 매니페스트 파일에 자동으로 추가하는 행을 제거할 수 있습니다.

    ```xml
    android-extra-application-manifest="adjust_broadcast_receiver.xml"
    ```
    
- adjust SDK extension은 기본적으로 `INTERNET` 및 `ACCESS_WIFI_STATE` 권한을 앱의 매니페스트 파일에 추가합니다. `ACCESS_WIFI_STATE` 권한은 앱에서 *Google Play 서비스를 사용하지 않는* 경우에만 필요합니다. 이런 경우 해당 권한이 다른 작업에 필요하지 않으면 제거해도 됩니다.

### <a id="google-play-services">5. Google Play 서비스

2014년 8월 1일 이후로 Google Play Store의 모든 앱은 [Google 광고 ID][google_ad_id]를 사용하여 장치를 고유하게 식별해야 합니다. adjust SDK에서 Google 광고 ID를 사용할 수 있게 하려면 [Google Play 서비스][google_play_services]를 연동해야 합니다.

Google Play 서비스를 Marmalade 앱에 추가하려면 앱의 `.mkb` 파일을 편집하고 `s3eGooglePlayServices`를 `subprojects` 목록에 추가해야 합니다. 또한 다음 행을 `.mkb` 파일의 `deployment` 목록에 추가해야 합니다.

```
android-extra-strings='(gps_app_id,your.app.package)'
```

Google Play 서비스가 Marmalade 앱에 성공적으로 연동됩니다.

## <a id="additional-features">추가 기능

Adjust SDK를 프로젝트에 연동하면 다음 기능을 사용할 수 있습니다.

### <a id="event-tracking">6. 이벤트 트래킹

adjust를 사용하여 원하는 이벤트를 모두 트래킹할 수 있습니다. 버튼의 모든 탭을 트래킹하려는 경우 [대시보드]에서 새 이벤트 토큰을 만드십시오. 이벤트 토큰이 `abc123`일 경우 다음 행을 버튼의 클릭 핸들러 메서드에 추가하면 클릭을 트래킹할 수 있습니다.

```cpp
adjust_event* event = new adjust_event("abc123");
adjust_TrackEvent(event);
```

#### <a id="revenue-tracking">매출 트래킹

사용자가 광고를 누르거나 인앱 구매를 통해 매출을 발생시킬 수 있는 경우 이벤트를 사용하여 해당 매출을 트래킹할 수 있습니다. 한 번 누를 때 0.01 유로의 매출이 발생한다고 가정할 경우 매출 이벤트를 다음과 같이 트래킹할 수 있습니다.

```cpp
adjust_event* event = new adjust_event("abc123");
event->set_revenue(0.01, "EUR");

adjust_TrackEvent(event);
```

##### <a id="revenue-deduplication"></a>매출 중복 제거

**참고**: 현재 이 기능은 iOS에서만 제공됩니다.

중복되는 매출을 트래킹하지 않도록 트랜잭션 ID(선택 사항)를 추가할 수도 있습니다. 마지막 10개의 트랜잭션 ID가 저장되고, 트랜잭션 ID가 중복되는 매출 이벤트는 건너뜁니다. 이 기능은 인앱 구매 트래킹에 특히 유용합니다. 아래 예를 참조하십시오.

인앱 구매를 트래킹하려면 트랜잭션이 완료되고 품목이 구매된 경우에만 `adjust_TrackEvent`를 호출해야 합니다. 이렇게 해야 실제로 발생하지 않은 매출을 트래킹하는 오류를 방지할 수 있습니다.

```cpp
adjust_event* event = new adjust_event("abc123");

event->set_revenue(0.01, "EUR");
event->set_transaction_id("transaction_id");

adjust_TrackEvent(event);
```

##### <a id="iap-verification">인앱 구매 유효성 검사

adjust의 서버측 수신 확인 메커니즘을 사용하여 앱에서 이루어진 인앱 구매의 유효성을 검사하려는 경우 Marmalade용 adjust 구매 SDK가 곧 공개될 예정이므로 조금만 더 기다려 주십시오.

### <a id="callback-parameters">콜백 매개변수

[대시보드][dashboard]에서 해당 이벤트의 콜백 URL을 등록할 수도 있으며, 이렇게 하면 이벤트가 트래킹될 때마다 GET 요청이 해당 URL로 전송됩니다. 이 경우 몇 개의 키-값 쌍을 개체에 포함하여 `adjust_TrackEvent` 메서드에 전달할 수도 있습니다. 그러면 명명된 매개변수들이 콜백 URL에 추가됩니다.

예를 들어 이벤트 토큰 `abc123`을 사용하여 이벤트에 대해 `http://www.adjust.com/callback` URL을 등록하고 다음 행을 실행한다고 가정합니다.

```cpp
adjust_event* event = new adjust_event("abc123");

event->add_callback_parameter("key", "value");
event->add_callback_parameter("foo", "bar");

adjust_TrackEvent(event);
```

이 경우에는 이벤트가 트래킹되고 요청이 다음 주소로 전송됩니다.

```
http://www.adjust.com/callback?key=value&foo=bar
```

iOS의 `{idfa}` 또는 Android의 `{gps_adid}`처럼 매개변수 값으로 사용할 수 있는 다양한 자리 표시자가 지원됩니다.  그 결과로 생성되는 콜백에서는 iOS의 경우 `{idfa}` 자리 표시자가 현재 장치의 광고주 ID로 변경되고 Android의 경우 `{gps_adid}`가 현재 Android 장치의 Google 광고로 변경됩니다. 또한 사용자 지정 매개변수는 저장되지 않고 콜백에만 추가됩니다. 이벤트에 대한 콜백을 등록하지 않은 경우 해당 매개변수는 읽을 수 없습니다.

### <a id="partner-parameters">파트너 매개변수

위에서 언급한 콜백 매개변수와 마찬가지로, adjust에서 선택된 네트워크 파트너로 전송할 매개변수도 추가할 수 있습니다. 해당 네트워크는 adjust 대시보드에서 활성화할 수 있습니다.

추가할 파트너 매개변수에 대해 `adjust_event` 인스턴스에서 `add_partner_parameter` 메서드를 호출해야 합니다.

```cpp
adjust_event* event = new adjust_event("abc123");

event->add_partner_parameter("key", "value");
event->add_partner_parameter("foo", "bar");

adjust_TrackEvent(event);
```

특별 파트너와 네트워크에 대한 자세한 내용은 [특별 파트너 설명서][special-partners]를 참조하십시오.

### <a id="attribution-callback">7. 어트리뷰션 콜백

adjust는 어트리뷰션 변경 시 콜백을 보낼 수도 있습니다. 어트리뷰션에 대해 다양한 소스가 고려되기 때문에 이 정보는 동시에 제공할 수 없습니다. 콜백(선택 사항)을 응용 프로그램에서 구현하려면 다음 단계를 수행하십시오.

1. 유형이 `adjust_attribution_data*`인 매개변수를 수신하는 void 메서드를 만듭니다.

2. `adjust_config`의 인스턴스를 만든 후 이전에 만든 메서드를 매개변수로 사용하여 이 인스턴스의 `set_attribution_callback` 메서드를 호출합니다.

SDK에서 최종 어트리뷰션 데이터를 수신하면 콜백 함수가 호출됩니다. 
콜백 함수를 통해 `attribution` 매개변수에 액세스할 수 있습니다. 
매개변수 속성에 대한 개요는 다음과 같습니다.

- `char* tracker_token` 현재 설치의 트래커 토큰.
- `char* tracker_name` 현재 설치의 트래커 이름.
- `char* network` 현재 설치의 network 그룹화 수준.
- `char* campaign` 현재 설치의 campaign 그룹화 수준.
- `char* ad_group` 현재 설치의 ad group 그룹화 수준.
- `char* creative` 현재 설치의 creative 그룹화 수준.
- `char* click_label` 현재 설치의 클릭 레이블.

```cpp
#include "AdjustMarmalade.h"

//...

static void trace_attribution_data(adjust_attribution_data* attribution) 
{
    // Printing all attribution properties.
    IwTrace(ADJUSTMARMALADE, ("Attribution changed!"));
    IwTrace(ADJUSTMARMALADE, (attribution->tracker_token));
    IwTrace(ADJUSTMARMALADE, (attribution->tracker_name));
    IwTrace(ADJUSTMARMALADE, (attribution->network));
    IwTrace(ADJUSTMARMALADE, (attribution->campaign));
    IwTrace(ADJUSTMARMALADE, (attribution->ad_group));
    IwTrace(ADJUSTMARMALADE, (attribution->creative));
    IwTrace(ADJUSTMARMALADE, (attribution->click_label));
}

// ...

int main()
{
    const char* app_token = "{YourAppToken}";
    const char* environment = "sandbox";
    const char* log_level = "verbose";

    adjust_config* config = new adjust_config(app_token, environment);
    config->set_log_level(log_level);
    config->set_attribution_callback(trace_attribution_data);
    
    adjust_Start(config);

    // ...
}
```

[해당 어트리뷰션 데이터 정책][attribution-data]을 고려하십시오.

### <a id="session-event-callbacks">8. 세션 및 이벤트 콜백

콜백을 등록하여 성공 또는 실패 트래킹 대상 이벤트 및/또는 세션에 대한 알림을 받을 수 있습니다.

동일한 단계에 따라 성공한 트래킹 이벤트에 대해 다음 콜백 함수를 구현하십시오.

```cpp
#include "AdjustMarmalade.h"

//...

static void trace_event_success_data(adjust_event_success_data* response) 
{
    IwTrace(ADJUSTMARMALADE, ("Event successfully tracked!"));
    IwTrace(ADJUSTMARMALADE, (response->message));
    IwTrace(ADJUSTMARMALADE, (response->timestamp));
    IwTrace(ADJUSTMARMALADE, (response->event_token));
    IwTrace(ADJUSTMARMALADE, (response->adid));
    IwTrace(ADJUSTMARMALADE, (response->json_response));
}

// ...

int main()
{
    const char* app_token = "{YourAppToken}";
    const char* environment = "sandbox";
    const char* log_level = "verbose";

    adjust_config* config = new adjust_config(app_token, environment);
    config->set_log_level(log_level);
    config->set_event_success_callback(trace_event_success_data);
    
    adjust_Start(config);

    // ...
}
```

실패한 트래킹 이벤트의 경우 다음 콜백 함수:

```cpp
#include "AdjustMarmalade.h"

//...

static void trace_event_failure_data(adjust_event_failure_data* response) 
{
    IwTrace(ADJUSTMARMALADE, ("Event tracking failed!"));
    IwTrace(ADJUSTMARMALADE, (response->message));
    IwTrace(ADJUSTMARMALADE, (response->timestamp));
    IwTrace(ADJUSTMARMALADE, (response->event_token));
    IwTrace(ADJUSTMARMALADE, (response->adid));
    IwTrace(ADJUSTMARMALADE, (response->will_retry));
    IwTrace(ADJUSTMARMALADE, (response->json_response));
}

// ...

int main()
{
    const char* app_token = "{YourAppToken}";
    const char* environment = "sandbox";
    const char* log_level = "verbose";

    adjust_config* config = new adjust_config(app_token, environment);
    config->set_log_level(log_level);
    config->set_event_failure_callback(trace_event_failure_data);
    
    adjust_Start(config);

    // ...
}
```

성공한 트래킹 세션의 경우:

```cpp
#include "AdjustMarmalade.h"

//...

static void trace_session_success_data(adjust_session_success_data* response) 
{
    IwTrace(ADJUSTMARMALADE, ("Session successfully tracked!"));
    IwTrace(ADJUSTMARMALADE, (response->message));
    IwTrace(ADJUSTMARMALADE, (response->timestamp));
    IwTrace(ADJUSTMARMALADE, (response->adid));
    IwTrace(ADJUSTMARMALADE, (response->json_response));
}

// ...

int main()
{
    const char* app_token = "{YourAppToken}";
    const char* environment = "sandbox";
    const char* log_level = "verbose";

    adjust_config* config = new adjust_config(app_token, environment);
    config->set_log_level(log_level);
    config->set_session_success_callback(trace_session_success_data);
    
    adjust_Start(config);

    // ...
}
```

그리고 실패한 트래킹 세션의 경우:

```cpp
#include "AdjustMarmalade.h"

//...

static void trace_session_failure_data(adjust_session_failure_data* response) 
{
    IwTrace(ADJUSTMARMALADE, ("Session tracking failed!"));
    IwTrace(ADJUSTMARMALADE, (response->message));
    IwTrace(ADJUSTMARMALADE, (response->timestamp));
    IwTrace(ADJUSTMARMALADE, (response->adid));
    IwTrace(ADJUSTMARMALADE, (response->will_retry));
    IwTrace(ADJUSTMARMALADE, (response->json_response));
}

// ...

int main()
{
    const char* app_token = "{YourAppToken}";
    const char* environment = "sandbox";
    const char* log_level = "verbose";

    adjust_config* config = new adjust_config(app_token, environment);
    config->set_log_level(log_level);
    config->set_session_failure_callback(trace_session_failure_data);
    
    adjust_Start(config);

    // ...
}
```

콜백 함수는 SDK에서 서버로 패키지를 보내려고 시도한 후에 호출됩니다. 콜백에서는 콜백 전용 응답 데이터 개체에 액세스할 수 있습니다. 세션 응답 데이터 속성에 대한 개요는 다음과 같습니다.

- `const char* message` 서버에서 전송된 메시지 또는 SDK에 의해 로깅된 오류.
- `const char* timestamp` 서버에서 전송된 타임스탬프.
- `const char* adid` adjust에 의해 제공된 고유 장치 식별자.
- `const char* jsonResponse` 서버에서 전송된 응답이 있는 JSON 개체.

두 개의 이벤트 응답 데이터 개체에는 다음 정보가 포함됩니다.

- `const char* eventToken` 트래킹 패키지가 이벤트인 경우 이벤트 토큰.

그리고 이벤트 및 세션 실패 개체에는 모두 다음이 포함됩니다.

- `const char* willRetry` 나중에 패키지를 다시 보내려는 시도가 있을 것임을 나타냅니다.

### <a id="disable-tracking">9. 트래킹 사용 중지

`false` 매개변수를 사용할 수 있는 `adjust_SetEnabled` 메서드를 호출하여 adjust SDK에서 트래킹을 사용할 수 없도록 설정할 수 있습니다. 이 설정은 **세션 간에 기억되지만**, 첫 번째 세션 후에만 활성화할 수 있습니다.

```cpp
adjust_SetEnabled(false);
```

`adjust_IsEnabled` 메서드를 사용하여 adjust SDK가 현재 활성화되어 있는지 확인할 수 있습니다. 매개변수가 `true`로 설정된 `adjust_SetEnabled`를 호출하여 adjust SDK를 언제든지 활성화할 수 있습니다.

### <a id="offline-mode">10. 오프라인 모드

adjust SDK를 오프라인 모드로 전환하여 adjust 서버로 전송하는 작업을 일시 중단하고 트래킹된 데이터를 보관하여 나중에 보낼 수 있습니다. 오프라인 모드일 때는 모든 정보가 파일에 저장되므로 오프라인 모드에서 너무 많은 이벤트가 트리거되지 않도록 주의하십시오.

`adjust_SetOfflineMode`를 `true`로 설정된 상태로 호출하면 오프라인 모드를 활성화할 수 있습니다.

```cpp
adjust_SetOfflineMode(true);
```

반대로 `adjust_SetOfflineMode`를 `false`로 설정한 상태로 호출하면 오프라인 모드를 비활성화할 수 있습니다. adjust SDK를 다시 온라인 모드로 전환하면 저장된 정보가 모두 올바른 시간 정보와 함께 adjust 서버로 전송됩니다.

트래킹 사용 중지와 달리 이 설정은 세션 간에 *기억되지 않습니다*. 따라서 앱을 오프라인 모드에서 종료한 경우에도 SDK는 항상 온라인 모드로 시작됩니다.

### <a id="device-ids">11. 장치 ID

Google Analytics와 같은 서비스를 사용하려면 중복 보고가 발생하지 않도록 장치 ID와 클라이언트 ID를 조정해야 합니다. 

#### Android

adjust SDK를 사용하면 앱이 실행 중인 Android 장치의 Google 광고 ID를 읽을 수 있습니다. 이렇게 하려면 `const char*` 매개변수를 수신하는 `adjust_config` 개체에서 콜백 메서드를 설정하십시오.
그런 다음 `adjust_GetGoogleAdId` 메서드를 호출하면 콜백 메서드에서 Google 광고 ID 값을 얻게 됩니다.

```cpp
#include "AdjustMarmalade.h"

//...

static void trace_google_ad_id_data(const char* response) 
{
    IwTrace(ADJUSTMARMALADE, ("Google Advertising Identifier received!"));
    IwTrace(ADJUSTMARMALADE, (response));
}

// ...

int main()
{
    const char* app_token = "{YourAppToken}";
    const char* environment = "sandbox";
    const char* log_level = "verbose";

    adjust_config* config = new adjust_config(app_token, environment);
    config->set_log_level(log_level);
    config->set_google_ad_id_callback(trace_google_ad_id_data);
    
    adjust_Start(config);

    // ...
    
    adjust_GetGoogleAdId();
    
    // ...
}
```

#### iOS

Android 장치의 Google 광고 ID와 마찬가지로, `adjust_config` 개체에서 적절한 콜백 메서드를 설정하고 `adjust_GetIdfa` 메서드를 호출하여 iOS 장치에서 IDFA 값에 액세스할 수 있습니다.

```cpp
#include "AdjustMarmalade.h"

//...

static void trace_idfa_data(const char* response) 
{
    IwTrace(ADJUSTMARMALADE, ("IDFA received!"));
    IwTrace(ADJUSTMARMALADE, (response));
}

// ...

int main()
{
    const char* app_token = "{YourAppToken}";
    const char* environment = "sandbox";
    const char* log_level = "verbose";

    adjust_config* config = new adjust_config(app_token, environment);
    config->set_log_level(log_level);
    config->set_idfa_callback(trace_idfa_data);
    
    adjust_Start(config);

    // ...
    
    adjust_GetIdfa();
    
    // ...
}
```

### <a id="deeplinking">12. 딥링크 연결

URL에서 딥링크를 통해 앱으로 연결하는 옵션과 함께 adjust 트래커 URL을 사용하는 경우 adjust SDK를 통해 딥링크 URL과 해당 내용에 대한 정보를 얻을 수 있습니다. 사용자가 이미 앱을 설치했거나(표준 딥링크 연결 시나리오) 앱이 사용자의 장치에 없는 경우(지연된 딥링크 연결 시나리오) URL에 연결될 수 있으므로, adjust SDK에는 딥링크 연결 시나리오에 따라 URL 콘텐츠를 가져오는 두 가지 방법이 있습니다.

#### <a id="deeplinking-standard">표준 딥링크 연결 시나리오

표준 딥링크 연결 시나리오에서 URL 콘텐츠에 대한 정보를 가져오려면 URL의 콘텐츠가 전달될 `const char*` 매개변수 1개를 수신할 `adjust_config` 개체에서 콜백 메서드를 설정해야 합니다. 이 메서드는 config 개체에서 `set_deeplink_callback` 메서드를 호출하여 설정해야 합니다.

```cpp
#include "AdjustMarmalade.h"

//...

static void trace_deeplink_data(const char* response) 
{
    IwTrace(ADJUSTMARMALADE, ("Deeplink received!"));
    IwTrace(ADJUSTMARMALADE, (response));
}

// ...

int main()
{
    const char* app_token = "{YourAppToken}";
    const char* environment = "sandbox";
    const char* log_level = "verbose";

    adjust_config* config = new adjust_config(app_token, environment);
    config->set_log_level(log_level);
    config->set_deeplink_callback(trace_deeplink_data);
    
    adjust_Start(config);

    // ...
}
```

#### <a id="deeplinking-deferred">지연된 딥링크 연결 시나리오

지연된 딥링크 연결 시나리오에서 URL 콘텐츠에 대한 정보를 가져오려면 URL의 콘텐츠가 전달될 `const char*` 매개변수 1개를 수신할 `adjust_config` 개체에서 콜백 메서드를 설정해야 합니다. 이 메서드는 config 개체에서 `set_deferred_deeplink_callback` 메서드를 호출하여 설정해야 합니다.

```cpp
#include "AdjustMarmalade.h"

//...

static void trace_deferred_deeplink_data(const char* response) 
{
    IwTrace(ADJUSTMARMALADE, ("Deferred deeplink received!"));
    IwTrace(ADJUSTMARMALADE, (response));
}

// ...

int main()
{
    const char* app_token = "{YourAppToken}";
    const char* environment = "sandbox";
    const char* log_level = "verbose";

    adjust_config* config = new adjust_config(app_token, environment);
    config->set_log_level(log_level);
    config->set_deferred_deeplink_callback(trace_deferred_deeplink_data);
    
    adjust_Start(config);

    // ...
}
```

지연된 딥링크 연결 시나리오에서는 `adjust_config` 개체에 설정할 수 있는 추가 설정이 1개 있습니다. adjust SDK에서 지연된 딥링크 정보를 가져온 후에는 adjust SDK가 이 URL을 열도록 할 것인지 선택할 수 있습니다. `set_should_deferred_deeplink_be_opened` 메서드를 config 개체에서 호출하여 이 옵션을 설정할 수 있습니다.

```cpp
#include "AdjustMarmalade.h"

//...

static void trace_deferred_deeplink_data(const char* response) 
{
    IwTrace(ADJUSTMARMALADE, ("Deferred deeplink received!"));
    IwTrace(ADJUSTMARMALADE, (response));
}

// ...

int main()
{
    const char* app_token = "{YourAppToken}";
    const char* environment = "sandbox";
    const char* log_level = "verbose";

    adjust_config* config = new adjust_config(app_token, environment);
    config->set_log_level(log_level);
    config->set_should_deferred_deeplink_be_opened(false);
    config->set_deferred_deeplink_callback(trace_deferred_deeplink_data);
    
    adjust_Start(config);

    // ...
}
```

아무것도 설정하지 않으면 adjust SDK에서 기본적으로 URL을 실행합니다.

앱에서 딥링크 연결을 지원할 수 있게 하려면 지원되는 각 플랫폼에 대해 구조를 설정해야 합니다.

#### <a id="scheme-android">Android 작업에 대해 구조 설정

Android 앱의 구조 이름을 설정하려면 딥링크 연결 후에 시작할 작업(일반적으로 기본 Marmalade Android 작업)에 다음 `<intent-filter>`를 추가해야 합니다.

```xml
<intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="schemeName" />
</intent-filter>
```

`schemeName`을 원하는 Android 앱의 구조 이름으로 변경해야 합니다.

이 intent 필터를 예제 앱에 추가한 후 `AndroidManifest.xml`의 작업 정의는 다음과 같습니다.

```xml
<activity android:name=".${CLASSNAME}"
          android:label="@string/app_name"
          android:configChanges="locale|keyboardHidden|orientation|screenSize"
          android:launchMode="singleTask"
          ${EXTRA_ACTIVITY_ATTRIBS}>
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="adjustExample" />
    </intent-filter>
</activity>
```

#### <a id="scheme-ios">iOS에서 사용자 지정 URL 구조 설정

iOS에서는 사용자 지정 URL 구조 이름을 설정해야 하지만, Android와 달리 앱의 `.mkb` 파일만 편집하면 됩니다. 다음 행을 앱의 `.mkb` 파일에 있는 `deployment` 부분에 추가해야 합니다.

```
iphone-bundle-url-name = com.your.bundle
iphone-bundle-url-schemes = schemeName
```

`com.your.bundle`를 앱의 번들 ID로 변경하고 `schemeName`을 원하는 iOS 앱 구조 이름으로 변경해야 합니다.

**중요**: iOS에서 이 딥링크 연결 지원 방법을 사용하면 **iOS 8 이하**가 설치된 장치에서 딥링크 연결 처리가 지원됩니다. **iOS 9**부터 Apple은 Universal Link를 사용했지만 현재 adjust SDK에서는 Universal Link가 기본적으로 지원되지 않습니다. Universal Link를 지원하려면 Xcode에서 기본적으로 생성된 iOS 프로젝트를 편집한 후 Universal Link 처리 지원을 추가해야 합니다. 이 작업을 네이티브 측에서 수행하는 방법에 대해 알아보려면 [기본 iOS Universal Link 설명서][universal-links-guide]를 참조하십시오.

[adjust.com]: http://adjust.com
[dashboard]: http://adjust.com
[example-app]: https://github.com/adjust/marmalade_sdk/tree/master/Example
[releases]: https://github.com/adjust/marmalade_sdk/releases
[custom-receiver]: https://github.com/adjust/android_sdk/blob/master/doc/referrer.md
[attribution-data]: https://github.com/adjust/sdks/blob/master/doc/attribution-data.md
[google_play_services]: http://developer.android.com/google/play-services/setup.html
[google_ad_id]: https://developer.android.com/google/play-services/id.html
[special-partners]: https://docs.adjust.com/en/special-partners
[universal-links-guide]: https://github.com/adjust/ios_sdk/#universal-links

## <a id="license">라이선스

adjust SDK는 MIT 라이선스에 따라 사용이 허가됩니다.

Copyright (c) 2012-2016 adjust GmbH,
http://www.adjust.com

이로써 본 소프트웨어와 관련 문서 파일(이하 "소프트웨어")의 복사본을 받는 사람에게는 아래 조건에 따라 소프트웨어를 제한 없이 다룰 수 있는 권한이 무료로 부여됩니다. 이 권한에는 소프트웨어를 사용, 복사, 수정, 병합, 출판, 배포 및/또는 판매하거나 2차 사용권을 부여할 권리와 소프트웨어를 제공 받은 사람이 소프트웨어를 사용, 복사, 수정, 병합, 출판, 배포 및/또는 판매하거나 2차 사용권을 부여하는 것을 허가할 수 있는 권리가 제한 없이 포함됩니다.

위 저작권 고지문과 본 권한 고지문은 소프트웨어의 모든 복사본이나 주요 부분에 포함되어야 합니다.

소프트웨어는 상품성, 특정 용도에 대한 적합성 및 비침해에 대한 보증 등을 비롯한 어떤 종류의 명시적이거나 암묵적인 보증 없이 "있는 그대로" 제공됩니다. 어떤 경우에도 저작자나 저작권 보유자는 소프트웨어와 소프트웨어의 사용 또는 기타 취급에서 비롯되거나 그에 기인하거나 그와 관련하여 발생하는 계약 이행 또는 불법 행위 등에 관한 배상 청구, 피해 또는 기타 채무에 대해 책임지지 않습니다.
--END--
