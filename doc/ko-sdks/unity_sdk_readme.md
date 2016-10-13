## 요약

이 항목에서는 adjust의 Unity3d SDK에 대해 설명합니다. 이 SDK는 iOS, Android, Windows 8.1 및 Windows phone 8.1 대상을 지원합니다. adjust에 대한 자세한 내용은 [adjust.com]을 참조하십시오.

## 기본 설치

다음은 adjust SDK를 Unity3d 프로젝트와 연동하기 위해 최소한으로 수행해야 하는 절차입니다.

### 1. SDK 다운로드 및 설치

[릴리스 페이지][releases]에서 최신 버전을 다운로드합니다. Unity 패키지 압축 파일을 원하는 폴더에 풉니다.

### 2. 프로젝트에 추가

프로젝트를 Unity Editor에서 열고 `Assets a†’ Import Package a†’ Custom Package`로 이동하여 다운로드한 Unity 패키지 파일을 선택합니다.

![][import_package]

### 3. adjust를 앱과 연동

`Assets/Adjust/Adjust.prefab`에 있는 prefab을 첫 번째 화면에 추가합니다.

추가한 prefab의 Inspector 메뉴에서 Adjust 스크립트의 매개변수를 편집합니다.

![][adjust_editor]

`{YourAppToken}`을 앱 토큰으로 변경합니다. 앱 토큰은 [대시보드]에서 찾을 수 있습니다.

`Log Level` 값을 다음 중 하나로 변경하여 표시되는 로그 수량을 늘리거나 줄일 수 있습니다.

- `Verbose` - 모든 로깅 사용
- `Debug` - 추가 로깅 사용
- `Info` - 기본값
- `Warn` - 정보 로깅 사용 중지
- `Error` - 경고 사용 중지
- `Assert` - 오류 사용 중지

대상이 windows 기반인 경우 adjust 라이브러리에서 컴파일된 로그를 `released` 모드에서 보려면 로그 출력을 `debug` 모드에서 테스트하는 동안 앱으로 리디렉션해야 합니다.

SDK를 시작하기 전에 `setLogDelegate` 메서드를 `AdjustConfig` 인스턴스에서 호출합니다.

```cs
//...
adjustConfig.setLogDelegate(msg => Debug.Log(msg));
//...
Adjust.start (adjustConfig);
```

앱을 테스트에 사용할지 아니면 프로덕션에 사용할 지에 따라 `Environment`를 다음 값 중 하나로 변경해야 합니다.

```
    'Sandbox'
    'Production'
```

**중요:** 이 값은 앱을 테스트하는 경우에만 `Sandbox`로 설정해야 합니다. 앱을 게시하기 직전에 environment를 `Production`으로 설정해야 합니다. 테스트를 다시 시작할 때 `Sandbox`로 다시 설정하십시오.

이 environment는 실제 트래픽과 테스트 장치의 테스트 트래픽을 구별하기 위해 사용합니다. 이 값을 항상 의미 있게 유지해야 합니다! 이것은 특히 매출을 트래킹하는 경우에 중요합니다.

앱에서 이벤트 트래킹을 많이 사용하는 경우 일부 HTTP 요청을 지연하여 1분마다 하나의 배치로 보낼 수 있습니다. `Event Buffering` 상자를 선택하여 이벤트 버퍼링을 사용할 수 있습니다.

게임의 `Awake` 이벤트에서 adjust SDK를 시작하지 않으려면 `Start Manually` 상자를 선택합니다. 대신 `AdjustConfig` 개체를 매개변수로 사용하여 `Adjust.start` 메서드를 호출하면 adjust SDK를 시작할 수 있습니다.

이 옵션 등이 있는 버튼 메뉴 화면의 예를 보려면 `Assets/Adjust/ExampleGUI/ExampleGUI.unity`에 있는 화면 예제를 여십시오. 이 화면의 원본은 `Assets/Adjust/ExampleGUI/ExampleGUI.cs`에 있습니다.

### 4. Google Play 서비스 추가

2014년 8월 1일 이후로 Google Play Store의 앱은 [Google 광고 ID][google_ad_id]를 사용하여 장치를 고유하게 식별해야 합니다. adjust SDK에서 Google 광고 ID를 사용할 수 있게 하려면 [Google Play 서비스][google_play_services]를 연동해야 합니다. 아직 연동하지 않은 경우 `google-play-services_lib` 폴더를 Unity 프로젝트의 `Assets/Plugins/Android` 폴더로 복사해야 합니다. 그러면 앱을 작성한 후 Google Play 서비스가 연동됩니다.

`google-play-services_lib`는 이미 설치했을 수 있는 Android SDK의 일부분입니다.

Android SDK는 두 가지 방법으로 다운로드할 수 있습니다. `Android SDK Manager`가 있는 도구를 사용하는 경우 `Android SDK Tools`를 다운로드해야 합니다. 설치한 후에 `SDK_FOLDER/extras/google/google_play_services/libproject/` 폴더에서 라이브러리를 찾을 수 있습니다.

![][android_sdk_location]

Android SDK Manager가 있는 도구를 사용하지 않는 경우 [공식 페이지][android_sdk_download]에서 Android SDK의 단독 실행형 버전을 다운로드해야 합니다. 이 버전을 다운로드하면 Android SDK Tools가 포함되지 않은 기본 Android SDK 버전만 갖게 됩니다. 자세한 내용은 Google에서 제공하는 `SDK Readme.txt` 파일을 참조하십시오. 이 파일은 Android SDK 폴더에 있습니다.

### 5. 빌드 스크립트

빌드 프로세스를 쉽게 수행할 수 있도록 Android 및 iOS용 빌드 스크립트를 모두 연동했습니다. 각 빌드 후에 스크립트가 실행되고 `Assets/Editor/AdjustEditor.cs` 파일에 의해 호출됩니다. 스크립트를 사용하려면 `python 2.7` 이상이 설치되어 있어야 합니다.

`Assets a†’ Adjust a†’ Change post processing status` 메뉴를 클릭하여 사후 프로세싱을 사용하지 않도록 설정할 수 있습니다.
사후 프로세싱을 다시 사용하도록 설정하려면 같은 버튼을 누르십시오.

#### iOS

iOS 빌드 스크립트는 `Assets/Editor/PostprocessBuildPlayer_AdjustPostBuildiOS.py`에 있습니다. 이 스크립트는 Unity3d iOS 생성 프로젝트를 다음과 같이 변경합니다.

1. iAd 및 AdSupport 프레임워크를 프로젝트에 추가합니다. 이 작업은 adjust SDK에서 필수입니다. 자세한 내용은 adjust [iOS][ios] 페이지를 참조하십시오.

2. 다른 링커 플래그인 `-ObjC`를 추가합니다. 그러면 빌드 시 adjust Objective-C 카테고리가 인식될 수 있습니다.

Unity3d iOS 생성 프로젝트를 다른 위치에 저장하는 사용자 지정 빌드가 있는 경우, `Assets a†’ Adjust a†’ Set iOS build path` 메뉴를 클릭하고 iOS 프로젝트의 빌드 경로를 선택하여 스크립트에 알리십시오.

스크립트 실행 후에는 `AdjustPostBuildiOSLog.txt` 로그 파일이 Unity3d 프로젝트의 루트에 실행된 스크립트의 로그 메시지와 함께 기록됩니다.

#### Android

Android 빌드 스크립트는 `Assets/Editor/PostprocessBuildPlayer_AdjustPostBuildAndroid.py`에 있습니다. 이 스크립트는 `Assets/Plugins/Android/`에 있는 `AndroidManifest.xml` 파일을 변경합니다. 이 방법의 경우 Android 패키지에 빌드 프로세스가 끝나기 전의 매니페스트 파일이 사용된다는 문제가 있습니다.

이 문제를 방지하려면 이전 실행에서 만들었거나 변경된 매니페스트를 사용하여 빌드를 다시 실행하거나, `Assets a†’ Adjust a†’ Fix AndroidManifest.xml` 메뉴를 클릭하여 스크립트가 빌드 프로세스 전에 실행될 수 있게 하십시오. 이 단계는 매니페스트 파일과 adjust SDK의 호환성이 계속 유지되는 경우 한 번만 수행하면 됩니다.

![][menu_android]

`Assets/Plugins/Android/`에 `AndroidManifest.xml` 파일이 없으면 adjust의 호환 가능한 매니페스트 파일인 `AdjustAndroidManifest.xml`을 사용하여 복사본을 만드십시오. `AndroidManifest.xml` 파일이 이미 있는 경우 다음 사항을 확인하고 변경하십시오.

1. 브로드캐스트 수신기를 추가합니다. 자세한 내용은 adjust [Android][android] 페이지를 참조하십시오.

2. 인터넷에 연결할 수 있는 권한을 추가합니다.

3. Wi-Fi 네트워크에 대한 정보에 액세스할 수 있는 권한을 추가합니다.

스크립트 실행 후에는 `AdjustPostBuildAndroidLog.txt` 로그 파일이 Unity3d 프로젝트의 루트에 실행된 스크립트의 로그 메시지와 함께 기록됩니다.

## 추가 기능

adjust SDK를 프로젝트에 연동한 후에는 다음 기능을 사용할 수 있습니다.

### 6. 사용자 지정 이벤트 트래킹 추가

원하는 이벤트에 대해 adjust에 알릴 수 있습니다. 버튼의 모든 탭을 트래킹하려는 경우 [대시보드]에서 새 이벤트 토큰을 만들어야 합니다. 이벤트 토큰이 `abc123`일 경우 버튼의 클릭 핸들러 메서드에 다음 행을 추가하여 클릭을 트래킹할 수 있습니다.

```cs
AdjustEvent adjustEvent = new AdjustEvent ("abc123");
Adjust.trackEvent (adjustEvent);
```

### 7. 매출 트래킹 추가

사용자가 광고를 누르거나 인앱 구매를 하여 매출을 발생시킬 수 있는 경우 이벤트를 사용하여 해당 매출을 트래킹할 수 있습니다. 한 번 누를 때 0.01 유로의 매출이 발생한다고 가정할 경우 매출 이벤트를 다음과 같이 트래킹할 수 있습니다.

```cs
AdjustEvent adjustEvent = new AdjustEvent ("abc123");
adjustEvent.setRevenue (0.01, "EUR");
Adjust.trackEvent (adjustEvent);
```

##### 인앱 구매 유효성 검사

adjust의 서버 측 수신 확인 도구인 구매 유효성 검사를 사용하여 앱에서 이루어지는 인앱 구매의 유효성을 확인하려면 `Unity3d purchase SDK`를 확인하십시오. 자세한 내용은 [여기][unity-purchase-sdk]를 참조하십시오.

### 8. 콜백 매개변수 추가

[대시보드]에서 해당 이벤트의 콜백 URL을 등록할 수도 있으며, 그러면 이벤트가 트래킹될 때마다 GET 요청이 해당 URL로 전송됩니다. 이 경우 몇 개의 키-값 쌍을 개체에 포함하여 `trackEvent` 메서드에 전달할 수도 있습니다. 그러면 명명된 매개변수들이 콜백 URL에 추가됩니다.

예를 들어 이벤트 토큰 `abc123`을 사용하여 이벤트에 대해 `http://www.adjust.com/callback` URL을 등록하고 다음 행을 실행한다고 가정합니다.

```cs
AdjustEvent adjustEvent = new AdjustEvent ("abc123");

adjustEvent.addCallbackParameter ("key", "value");
adjustEvent.addCallbackParameter ("foo", "bar");

Adjust.trackEvent (adjustEvent);
```

이 경우에는 이벤트가 트래킹되고 요청이 다음 주소로 전송됩니다.

```
http://www.adjust.com/callback?key=value&foo=bar
```

iOS의 `{idfa}` 또는 Android의 `{android_id}`처럼 매개변수 값으로 사용할 수 있는 다양한 자리 표시자가 지원됩니다.  결과로 생성된 콜백에서는 iOS의 경우 `{idfa}` 자리 표시자가 현재 장치의 광고주용 ID로 변경되고 Android의 경우 `{android_id}`가 현재 장치의 AndroidID로 변경됩니다. 또한 사용자 지정 매개변수는 저장되지 않고 콜백에만 추가됩니다.  이벤트에 대한 콜백을 등록하지 않은 경우 해당 매개변수는 읽을 수 없습니다.

### 9. 파트너 매개변수

adjust 대시보드에서 활성화된 연동에 대해 네트워크 파트너로 전송할 매개변수도 추가할 수 있습니다.

이 매개변수는 위에서 설명한 콜백 매개변수의 경우와 비슷하지만, `AdjustEvent` 인스턴스에서 addPartnerParameter 메서드를 호출해야 추가할 수 있습니다.

```cs
AdjustEvent adjustEvent = new AdjustEvent ("abc123");

adjustEvent.addPartnerParameter ("key", "value");
adjustEvent.addPartnerParameter ("foo", "bar");

Adjust.trackEvent (adjustEvent);
```

특별 파트너와 해당 파트너와의 연동에 대한 자세한 내용은 [특별 파트너 설명서][special-partners]를 참조하십시오.

### 10. 어트리뷰션 변경 콜백 수신

트래커 어트리뷰션 변경에 대한 알림을 받기 위한 콜백을 등록할 수 있습니다. 어트리뷰션을 위해 고려되는 다양한 소스로 인해 이 정보는 동시에 제공될 수 없습니다. 콜백(선택 사항)을 응용 프로그램에서 구현하려면 다음 단계를 수행하십시오.

[해당 어트리뷰션 데이터 정책][attribution-data]을 고려하십시오.

1. `Action<AdjustAttribution>` 위임의 시그너처가 있는 메서드를 만듭니다.

2. `AdjustConfig` 개체를 만든 후 `adjustConfig.setAttributionChangedDelegate`를 이전에 만든 메서드를 사용하여 호출합니다. 시그너처가 같은 lambda도 사용할 수 있습니다.

3. `Adjust.prefab`을 사용하는 대신 `Adjust.cs` 스크립트가 다른 `GameObject`에 추가되었습니다.
이 `GameObject`의 이름을 `AdjustConfig.setAttributionChangedDelegate`의 두 번째 매개변수로 전달해야 합니다.

콜백은 AdjustConfig 인스턴스를 사용하여 구성되므로, `Adjust.start`를 호출하기 전에 `adjustConfig.setAttributionChangedDelegate`를 호출해야 합니다.

SDK에서 최종 어트리뷰션 데이터를 수신하면 콜백 함수가 호출됩니다. 콜백 함수를 통해 `attribution` 매개변수에 액세스할 수 있습니다. 매개변수 속성에 대한 개요는 다음과 같습니다.

- `string trackerToken` 현재 설치의 트래커 토큰.
- `string trackerName` 현재 설치의 트래커 이름.
- `string network` 현재 설치의 network 그룹화 기준.
- `string campaign` 현재 설치의 campaign 그룹화 기준.
- `string adgroup` 현재 설치의 ad group 그룹화 기준.
- `string creative` 현재 설치의 creative 그룹화 기준.

```cs
using com.adjust.sdk;

public class ExampleGUI : MonoBehaviour {
{
    void OnGUI () {
    {
        if (GUI.Button (new Rect (0, 0, Screen.width, Screen.height), "callback")) 
        {
            AdjustConfig adjustConfig = new AdjustConfig ("{Your App Token}", AdjustEnvironment.Sandbox);
            adjustConfig.setLogLevel (AdjustLogLevel.Verbose);
            adjustConfig.setAttributionChangedDelegate (this.attributionChangedDelegate);

            Adjust.start (adjustConfig);
        }
    }
    
    public void attributionChangedDelegate (AdjustAttribution attribution)
    {
        Debug.Log ("Attribution changed");
        
        // ...
    }
}
```

### 11. 트래킹하는 이벤트 및 세션에 대한 콜백 구현

콜백을 등록하여 성공 또는 실패 트래킹 대상 이벤트 및/또는 세션에 대한 알림을 받을 수 있습니다.

동일한 단계에 따라 성공한 트래킹 이벤트에 대해 다음 콜백 함수를 구현하십시오.

```cs
// ...

AdjustConfig adjustConfig = new AdjustConfig ("{Your App Token}", AdjustEnvironment.Sandbox);
adjustConfig.setLogLevel (AdjustLogLevel.Verbose);
adjustConfig.setEventSuccessDelegate (EventSuccessCallback);

Adjust.start (adjustConfig);

// ...

public void EventSuccessCallback (AdjustEventSuccess eventSuccessData)
{
    // ...
}
```

실패한 트래킹 이벤트의 경우 다음 콜백 함수:

```cs
// ...

AdjustConfig adjustConfig = new AdjustConfig ("{Your App Token}", AdjustEnvironment.Sandbox);
adjustConfig.setLogLevel (AdjustLogLevel.Verbose);
adjustConfig.setEventFailureDelegate (EventFailureCallback);

Adjust.start (adjustConfig);

// ...

public void EventFailureCallback (AdjustEventFailure eventFailureData)
{
    // ...
}
```

성공한 트래킹 세션의 경우:

```cs
// ...

AdjustConfig adjustConfig = new AdjustConfig ("{Your App Token}", AdjustEnvironment.Sandbox);
adjustConfig.setLogLevel (AdjustLogLevel.Verbose);
adjustConfig.setSessionSuccessDelegate (SessionSuccessCallback);

Adjust.start (adjustConfig);

// ...

public void SessionSuccessCallback (AdjustSessionSuccess sessionSuccessData)
{
    // ...
}
```

그리고 실패한 트래킹 세션의 경우:

```cs
// ...

AdjustConfig adjustConfig = new AdjustConfig ("{Your App Token}", AdjustEnvironment.Sandbox);
adjustConfig.setLogLevel (AdjustLogLevel.Verbose);
adjustConfig.setSessionFailureDelegate (SessionFailureCallback);

Adjust.start (adjustConfig);

// ...

public void SessionFailureCallback (AdjustSessionFailure sessionFailureData)
{
    // ...
}
```

콜백 함수는 SDK에서 서버로 패키지를 보내려고 시도한 후에 호출됩니다. 
콜백에서는 콜백 전용 응답 데이터 개체에 액세스할 수 있습니다. 
세션 응답 데이터 속성에 대한 개요는 다음과 같습니다.

- `string Message` 서버에서 전송된 메시지 또는 SDK에 의해 로깅된 오류.
- `string Timestamp` 서버에서 전송된 데이터의 타임스탬프.
- `string Adid` adjust에 의해 제공된 고유 장치 식별자.
- `Dictionary<string, object> JsonResponse` 서버에서 전송된 응답이 있는 JSON 개체.

두 개의 이벤트 응답 데이터 개체에는 다음 정보가 포함됩니다.

- `string EventToken` 트래킹 패키지가 이벤트인 경우 이벤트 토큰.

그리고 이벤트 및 세션 실패 개체에는 모두 다음이 포함됩니다.

- `bool WillRetry` 나중에 패키지를 다시 보내려는 시도가 있을 것임을 나타냅니다.

### 12. 트래킹 사용 중지

`false` 매개변수를 사용할 수 있는 `setEnabled` 메서드를 호출하여 adjust SDK에서 트래킹을 사용할 수 없도록 설정할 수 있습니다. 이 설정은 세션 간에 기억되지만, 첫 번째 세션 후에만 활성화할 수 있습니다.

```cs
Adjust.setEnabled(false);
```

`isEnabled` 메서드를 사용하여 adjust SDK가 현재 활성화되어 있는지 확인할 수 있습니다. `enabled` 매개변수가 `true`로 설정된 `setEnabled`를 호출하여 adjust SDK를 언제든지 활성화할 수 있습니다.

### 13. 오프라인 모드

adjust SDK를 오프라인 모드로 전환하여 adjust 서버로 전송하는 작업을 일시 중단하고 트래킹된 데이터를 보관하여 나중에 보낼 수 있습니다. 오프라인 모드일 때는 모든 정보가 파일에 저장되므로 오프라인 모드에서 너무 많은 이벤트가 트리거되지 않도록 주의하십시오.

`setOfflineMode`를 `true`로 설정된 상태로 호출하면 오프라인 모드를 활성화할 수 있습니다.

```cs
Adjust.setOfflineMode(true);
```

반대로 `setOfflineMode`를 `false`로 설정한 상태로 호출하면 오프라인 모드를 비활성화할 수 있습니다.
adjust SDK를 다시 온라인 모드로 전환하면 저장된 정보가 모두 올바른 시간 정보와 함께 adjust 서버로 전송됩니다.

트래킹 사용 중지와 달리 이 설정은 세션 간에 *기억되지 않습니다*. 따라서 앱을 오프라인 모드에서 종료한 경우에도 SDK는 항상 온라인 모드로 시작됩니다.

### 14. 장치 ID

Google Analytics와 같은 서비스를 사용하려면 중복 보고가 발생하지 않도록 장치 ID와 클라이언트 ID를 조정해야 합니다. 

#### Android

Google 광고 ID가 필요한 경우 제한 사항으로 인해 백그라운드 스레드에서만 해당 ID를 읽을 수
있습니다. 
`Action<string>` 위임이 있는 `getGoogleAdId` 함수를 호출하면 상황에 관계 없이 작동합니다.

```cs
Adjust.getGoogleAdId((string googleAdId) => {
    // ...
});
```

`OnDeviceIdsRead` 인스턴스의 `onGoogleAdIdRead` 메서드를 통해 Google 광고 ID에 `googleAdId` 변수로 액세스할 수 있습니다.

#### iOS

IDFA를 얻으려면 `getIdfa` 함수를 호출합니다.

```cs
Adjust.getIdfa ()
```

## 문제 해결

### iOS
사후 빌드 스크립트를 사용해도 프로젝트를 즉시 실행할 수 없는 경우가 있습니다.

필요한 경우 dSYM 파일을 사용하지 않도록 설정하십시오. `Project Navigator`에서 `Unity-iPhone` 프로젝트를 선택합니다. `Build Settings` 탭을 클릭하고 `debug information`을 검색합니다. `Debug Information Format` 또는 `DEBUG_INFORMATION_FORMAT` 옵션이 표시됩니다. 해당 옵션을 `DWARF with dSYM File`에서 `DWARF`로 변경합니다.

### 빌드 스크립트

사후 빌드 스크립트를 실행하려면 실행 권한이 필요합니다. 빌드 프로세스가 마지막에 중지되어 스크립트 파일 중 하나가 열리는 경우 스크립트가 기본적으로 실행될 수 없도록 시스템이 구성된 것이 원인일 수 있습니다. 이 경우 `Assets/Editor/PostprocessBuildPlayer_AdjustPostBuildiOS.py` 및 `Assets/Editor/PostprocessBuildPlayer_AdjustPostBuildAndroid.py`에 모두 있는 `chmod` 도구를 사용하여 실행 권한을 추가하십시오.


[adjust.com]:           http://adjust.com
[dashboard]:            http://adjust.com
[releases]:             https://github.com/adjust/adjust_unity_sdk/releases
[import_package]:       https://raw.github.com/adjust/adjust_sdk/master/Resources/unity/v4/import_package.png
[adjust_editor]:        https://raw.github.com/adjust/adjust_sdk/master/Resources/unity/v4/adjust_editor.png
[menu_android]:         https://raw.github.com/adjust/adjust_sdk/master/Resources/unity/v4/menu_android.png
[ios]:                  https://github.com/adjust/ios_sdk
[android]:              https://github.com/adjust/android_sdk
[attribution_data]:     https://github.com/adjust/sdks/blob/master/doc/attribution-data.md
[special-partners]:     https://docs.adjust.com/en/special-partners
[unity-purchase-sdk]:   https://github.com/adjust/unity_purchase_sdk
[google_ad_id]:         https://developer.android.com/google/play-services/id.html
[google_play_services]: http://developer.android.com/google/play-services/setup.html
[android_sdk_download]: https://developer.android.com/sdk/index.html#Other
[android_sdk_location]: https://raw.github.com/adjust/adjust_sdk/master/Resources/unity/v4/android_sdk_download.png

## 라이선스

mod_pbxproj.py 파일은 Apache License 2.0 버전(이하 "라이선스")에 따라 사용이 허가되며, 이 파일을 라이선스를 준수하지 않고 사용해서는 안 됩니다.
라이선스의 복사본은 http://www.apache.org/licenses/LICENSE-2.0에서 다운로드할 수 있습니다.

adjust SDK는 MIT 라이선스에 따라 사용이 허가됩니다.

Copyright (c) 2012-2015 adjust GmbH, http://www.adjust.com

이로써 본 소프트웨어와 관련 문서 파일(이하 "소프트웨어")의 복사본을 받는 사람에게는 아래 조건에 따라 소프트웨어를 제한 없이 다룰 수 있는 권한이 무료로 부여됩니다. 이 권한에는 소프트웨어를 사용, 복사, 수정, 병합, 출판, 배포 및/또는 판매하거나 2차 사용권을 부여할 권리와 소프트웨어를 제공 받은 사람이 소프트웨어를 사용, 복사, 수정, 병합, 출판, 배포 및/또는 판매하거나 2차 사용권을 부여하는 것을 허가할 수 있는 권리가 제한 없이 포함됩니다.

위 저작권 고지문과 본 권한 고지문은 소프트웨어의 모든 복사본이나 주요 부분에 포함되어야 합니다.

소프트웨어는 상품성, 특정 용도에 대한 적합성 및 비침해에 대한 보증 등을 비롯한 어떤 종류의 명시적이거나 암묵적인 보증 없이 "있는 그대로" 제공됩니다. 어떤 경우에도 저작자나 저작권 보유자는 소프트웨어와 소프트웨어의 사용 또는 기타 취급에서 비롯되거나 그에 기인하거나 그와 관련하여 발생하는 계약 이행 또는 불법 행위 등에 관한 배상 청구, 피해 또는 기타 채무에 대해 책임지지 않습니다.
--END--
