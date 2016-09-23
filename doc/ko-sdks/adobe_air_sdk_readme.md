## 요약

이 항목에서는 adjust의 AIR SDK에 대해 설명합니다. adjust에 대한 자세한 내용은 [adjust.com]을 참조하십시오.

## 기본 설치

### 1. SDK 다운로드 및 설치

adjust의 [릴리스 페이지][releases]에서 최신 버전을 다운로드합니다. 압축 파일을 선택한 폴더에 풉니다.

### 2. 프로젝트에 추가

여기서는 IntelliJ IDEA를 IDE로 사용한다고 가정합니다. Flash Builder를 사용하는 경우 [여기][flash-builder]에 나와 있는 지침을 따르십시오.

IDEA의 Project Structure에 있는 Project Settings에서 모듈을 선택합니다. Dependencies 탭을 선택하고 더하기(+) 버튼을 클릭하여 모듈에 종속성을 추가합니다. `New Library...` 옵션을 선택합니다.

![][idea-new-library]

다운로드한 `Adjust-x.y.z.ane`를 찾은 후 `OK`를 클릭합니다.

![][idea-locate]

그런 다음 adjust SDK extension을 앱 설명자 파일에 추가합니다.

```xml
<extensions>
    <!-- ... --->
    <extensionID>com.adjust.sdk</extensionID>
    <!-- ... --->
</extensions>
```

### 3. adjust를 앱과 연동

adjust를 사용하여 트래킹하기 시작하려면 SDK를 먼저 초기화해야 합니다. 다음 코드를 주 Sprite에 추가합니다.

```actionscript
import com.adjust.sdk.Adjust;
import com.adjust.sdk.AdjustConfig;
import com.adjust.sdk.Environment;
import com.adjust.sdk.LogLevel;

public class Example extends Sprite {
    public function Example() {
    	var appToken:String = "{YourAppToken}";
    	var environment:String = Environment.SANDBOX;
    	
        var adjustConfig:AdjustConfig = new AdjustConfig(appToken, environment);
        adjustConfig.setLogLevel(LogLevel.VERBOSE);

        Adjust.start(adjustConfig);
    }
}
```

`{YourAppToken}`을 앱 토큰으로 변경합니다. 앱 토큰은 [대시보드]에서 찾을 수 있습니다.

앱을 테스트에 사용할지 아니면 프로덕션에 사용할 지에 따라 `environment`를 다음 값 중 하나로 설정해야 합니다.

```actionscript
var environment:String = Environment.SANDBOX;
var environment:String = Environment.PRODUCTION;
```

**중요:** 앱을 테스트하는 경우에만 이 값을 `Environment.SANDBOX`로 설정해야 합니다. 앱을 게시하기 직전에 environment를 `Environment.PRODUCTION`으로 설정하십시오. 개발 및 테스트를 다시 시작할 경우에는 environment를 `Environment.SANDBOX`로 다시 설정하십시오.

이 environment는 실제 트래픽과 테스트 장치의 테스트 트래픽을 구별하기 위해 사용합니다. 이 값을 항상 의미있게 유지해야 합니다! 이것은 특히 매출을 트래킹하는 경우에 중요합니다.

#### adjust 로깅

다음 매개변수 중 하나를 사용하여 `AdjustConfig`에서 `setLogLevel`을 호출하면 테스트에 표시되는 로그의 양을 늘리거나 줄일 수 있습니다.

```actionscript
adjustConfig.setLogLevel(LogLevel.VERBOSE); // enable all logging
adjustConfig.setLogLevel(LogLevel.DEBUG);   // enable more logging
adjustConfig.setLogLevel(LogLevel.INFO);    // the default
adjustConfig.setLogLevel(LogLevel.WARN);    // disable info logging
adjustConfig.setLogLevel(LogLevel.ERROR);   // disable warnings as well
adjustConfig.setLogLevel(LogLevel.ASSERT);  // disable errors as well
```

### 4. adjust Android 매니페스트

adjust SDK를 Android용 AIR 앱과 함께 사용하려면 Android 매니페스트 파일을 편집해야 합니다. AIR 앱의 Android 매니페스트 파일을 편집하려면:

1. 일반적으로 `src/{YourProjectName}-app.xml`에 있는 응용 프로그램 설명자 파일을 엽니다.
2. `<android>` 태그를 검색합니다.
3. `<manifest>` 태그를 편집합니다.

필요한 [권한][android-permissions]과 [브로드캐스트 수신기][brodcast-android]를 추가하는 방법은 adjust의 Android 설명서를 참조하십시오.

![][android-manifest]

### 5. Google Play 서비스 추가

2014년 8월 1일 이후로 Google Play Store의 모든 앱은 [Google 광고 ID][google_ad_id]를 사용하여 장치를 고유하게 식별해야 합니다. adjust SDK에서 Google 광고 ID를 사용할 수 있게 하려면 [Google Play 서비스][google_play_services]를 연동해야 합니다.

Google Play 서비스를 아직 (다른 ANE의 일부로, 또는 다른 방법을 사용하여) 앱에 추가하지 않은 경우 adjust에서 제공하고 adjust SDK에 적합하게 작성된 `Google Play Services ANE`를 사용할 수 있습니다. Google Play 서비스 ANE는 adjust의 [릴리스 페이지][releases]에서 릴리스의 일부로 제공됩니다.

다운로드한 ANE를 앱에 가져오면 adjust SDK에 필요한 Google Play 서비스가 추가됩니다.

![][idea-new-library-gps]

그런 다음 Google Play 서비스 extension을 앱 설명자 파일에 추가합니다.

```xml
<extensions>
    <!-- ... --->
    <extensionID>com.adjust.gps</extensionID>
    <!-- ... --->
</extensions>
```

Google Play 서비스를 앱과 연동한 후 다음 행을 앱의 Android 매니페스트 파일에 `<manifest>` 태그 본문의 일부로 추가합니다.

```xml
<meta-data
    android:name="com.google.android.gms.version"
    android:value="@integer/google_play_services_version"/>
```

## 장치에서 디버깅

### Android 장치

Android SDK와 함께 제공되는 다음 ```logcat``` 도구를 사용하십시오.

```
<path-to-android-sdk>/platform-tools/adb logcat -s "Adjust"
```

### iOS 장치

XCode의 Device Organizer에서 Console을 선택하여 AdjustIo 로그에 액세스합니다.

![][xcode-logs]


## 추가 기능

Adjust SDK를 프로젝트에 연동하면 다음 기능을 사용할 수 있습니다.

### 6. 사용자 지정 이벤트 트래킹 추가

원하는 모든 이벤트에 대해 adjust에 알릴 수 있습니다. 버튼의 모든 탭을 트래킹하려는 경우 [대시보드]에서 새 이벤트 토큰을 만드십시오. 이벤트 토큰이 `abc123`일 경우 다음 행을 버튼의 클릭 처리기 메서드에 추가하여 클릭을 트래킹할 수 있습니다.


```actionscript
var adjustEvent:AdjustEvent = new AdjustEvent("abc123");
Adjust.trackEvent(adjustEvent);
```

### 7. 매출 트래킹 추가

사용자가 광고를 누르거나 인앱 구매를 통해 매출을 발생시킬 수 있는 경우 이벤트를 사용하여 해당 매출을 트래킹할 수 있습니다. 한 번 누를 때 0.01 유로의 매출이 발생한다고 가정할 경우 매출 이벤트를 다음과 같이 트래킹할 수 있습니다.

```actionscript
var adjustEvent:AdjustEvent = new AdjustEvent("abc123");
adjustEvent.setRevenue(0.01, "EUR");
Adjust.trackEvent(adjustEvent);
```

#### iOS

##### <a id="deduplication"></a> 매출 중복 제거

중복되는 매출을 트래킹하지 않도록 트랜잭션 ID(선택 사항)를 추가할 수도 있습니다. 마지막 10개의 트랜잭션 ID가 저장되고 트랜잭션 ID가 중복되는 매출 이벤트는 건너뜁니다. 이 기능은 인앱 구매 트래킹에 특히 유용합니다. 아래 예를 참조하십시오.

인앱 구매를 트래킹하려면 트랜잭션이 완료되고 품목이 구매된 경우에만 `trackEvent`를 호출해야 합니다. 이렇게 해야 실제로 발생하지 않은 매출을 트래킹하는 오류를 방지할 수 있습니다.

```actionscript
var adjustEvent:AdjustEvent = new AdjustEvent("abc123");

adjustEvent.setRevenue(0.01, "EUR");
adjustEvent.setTransactionId("transactionId");

Adjust.trackEvent(adjustEvent);
```

##### 영수증 확인

인앱 구매를 트래킹하는 경우 트래킹하는 이벤트에 영수증도 첨부할 수 있습니다. adjust 서버에서는 해당 영수증을 Apple과 확인하고, 확인에 실패할 경우 이벤트를 무시합니다. 이 작업을 수행하려면 구매의 트랜잭션 ID도 adjust로 보내야 합니다. 트랜잭션 ID는 [위](#deduplication)에서 설명한 것처럼 SDK 측 중복 제거에도 사용됩니다.

```actionscript
var adjustEvent:AdjustEvent = new AdjustEvent("abc123");

adjustEvent.setRevenue(0.01, "EUR");
adjustEvent.setReceiptForTransactionId("receipt", "transactionId");

Adjust.trackEvent(adjustEvent);
```

### 8. 콜백 매개변수 추가

[대시보드]에서 해당 이벤트의 콜백 URL을 등록할 수도 있으며, 그러면 이벤트가 트래킹될 때마다 GET 요청이 해당 URL로 전송됩니다. 이 경우 몇 개의 키-값 쌍을 개체에 넣고 `trackEvent` 메서드에 전달할 수도 있습니다. 그러면 명명된 매개변수들이 콜백 URL에 추가됩니다.

예를 들어 이벤트 토큰 `abc123`을 사용하여 이벤트에 대해 `http://www.adjust.com/callback` URL을 등록한 후 다음 행을 실행합니다.

```actionscript
var adjustEvent:AdjustEvent = new AdjustEvent("abc123");

adjustEvent.addCallbackParameter("key", "value");
adjustEvent.addCallbackParameter("foo", "bar");

Adjust.trackEvent(adjustEvent);
```

이 경우에는 이벤트가 트래킹되고 요청이 다음 주소로 전송됩니다.

```
http://www.adjust.com/callback?key=value&foo=bar
```

iOS의 `{idfa}` 또는 Android의 `{android_id}`처럼 매개변수 값으로 사용할 수 있는 다양한 자리 표시자가 지원됩니다.  결과로 생성된 콜백에서는 iOS의 경우 `{idfa}` 자리 표시자가 현재 장치의 광고주용 ID로 변경되고 Android의 경우 `{android_id}`가 현재 장치의 AndroidID로 변경됩니다. 또한 사용자 지정 매개변수는 저장되지 않고 콜백에만 추가됩니다.  이벤트에 대한 콜백을 등록하지 않은 경우 해당 매개변수는 읽을 수 없습니다.

### 9. 파트너 매개변수

adjust 대시보드에서 네트워크 파트너로 전송할 수 있는 활성화된 연동에 대한 매개변수도 추가할 수 있습니다.

이 매개변수는 위에서 설명한 콜백 매개변수의 경우와 비슷하지만, `AdjustEvent` 인스턴스에서 `addPartnerParameter` 메서드를 호출해야 추가할 수 있습니다.

```actionscript
var adjustEvent:AdjustEvent = new AdjustEvent("abc123");

adjustEvent.addPartnerParameter("key", "value");
adjustEvent.addPartnerParameter("foo", "bar");

Adjust.trackEvent(adjustEvent);
```

특별 파트너와 해당 파트너와의 연동에 대한 자세한 내용은 [특별 파트너 설명서][special-partners]를 참조하십시오.

### 10. 어트리뷰션 변경 콜백 수신

트래커 어트리뷰션 변경에 대한 알림을 받기 위한 콜백을 등록할 수 있습니다. 어트리뷰션에 대해 다양한 소스가 고려되기 때문에 이 정보는 동시에 제공할 수 없습니다. 콜백(선택 사항)을 응용 프로그램에서 구현하려면 다음 단계를 수행하십시오.

1. 유형이 `AdjustAttribution`인 매개변수를 수신하는 void 메서드를 만듭니다.

2. `AdjustConfig` 개체의 인스턴스를 만든 후 이전에 만든 메서드를 사용하여 `adjustConfig.setAttributionCallbackDelegate`를 호출합니다.

SDK에서 최종 어트리뷰션 데이터를 수신하면 콜백 함수가 호출됩니다. 콜백 함수를 통해 `attribution` 매개변수에 액세스할 수 있습니다. 매개변수 속성에 대한 개요는 다음곽 같습니다.

- `var trackerToken:String` 현재 설치의 트래커 토큰
- `var trackerName:String` 현재 설치의 트래커 이름
- `var network:String` 현재 설치의 network 그룹화 기준
- `var campaign:String` 현재 설치의 campaign 그룹화 기준
- `var adgroup:String` 현재 설치의 ad group 그룹화 기준
- `var creative:String` 현재 설치의 creative 그룹화 기준
- `var clickLabel:String` 현재 설치의 클릭 레이블

```actionscript
import com.adjust.sdk.Adjust;
import com.adjust.sdk.AdjustConfig;
import com.adjust.sdk.Environment;
import com.adjust.sdk.LogLevel;
import com.adjust.sdk.AdjustAttribution;

public class Example extends Sprite {
    public function Example() {
    	var appToken:String = "{YourAppToken}";
    	var environment:String = Environment.SANDBOX;
    	
        var adjustConfig:AdjustConfig = new AdjustConfig(appToken, environment);
        adjustConfig.setLogLevel(LogLevel.VERBOSE);
        adjustConfig.setAttributionCallbackDelegate(attributionCallbackDelegate);

        Adjust.start(adjustConfig);
    }
    
    // ...
    
    private static function attributionCallbackDelegate(attribution:AdjustAttribution):void {
        trace("Tracker token = " + attribution.getTrackerToken());
        trace("Tracker name = " + attribution.getTrackerName());
        trace("Campaign = " + attribution.getCampaign());
        trace("Network = " + attribution.getNetwork());
        trace("Creative = " + attribution.getCreative());
        trace("Adgroup = " + attribution.getAdGroup());
        trace("Click label = " + attribution.getClickLabel());
    }
}
```

[해당 어트리뷰션 데이터 정책][attribution-data]을 고려하십시오.

### 11. 딥링크 리어트리뷰션 설정

앱을 열기 위해 사용하는 딥링크를 처리하도록 adjust SDK를 설정할 수 있습니다. adjust에서는 특정 adjust 관련 매개변수만 읽을 수 있습니다. 딥링크를 사용하여 리타게팅 또는 재참여 캠페인을 운영할 계획인 경우 이 설정은 필수입니다. 일반적으로 `src/{YourProjectName}-app.xml`에 있는 앱 설명자 파일에서 앱 스키마 이름을 올바르게 설정하기만 하면 됩니다. adjust SDK에서는 나중에 이 스키마 이름을 딥링크에 사용하여 앱 소스 코드에서 아무것도 설정할 필요 없이 딥링크를 자동으로 처리합니다.

#### iOS

iOS 앱에서 스키마 이름을 설정하려면 앱 설명자의 `<iPhone>` 섹션에서 다음 키-값 쌍을 `<InfoAdditions>` 섹션에 추가해야 합니다.

```xml
<iPhone>
    <!-- ... --->
    <InfoAdditions><![CDATA[
        <key>CFBundleURLTypes</key>
        <array>
            <dict>
              <key>CFBundleURLName</key>
              <string>com.adjust.example</string>
              <key>CFBundleURLSchemes</key>
              <array>
                <string>desired-scheme-name</string>
              </array>
            </dict>
        </array>
    ]]></InfoAdditions>
    <!-- ... -->
</iPhone>
```

#### Android

Android 앱에서 스키마 이름을 설정하려면 딥링크 연결 후 시작할 작업에 다음 `<intent-filter>`를 추가해야 합니다.

```xml
<!-- ... -->
    <activity>
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />
                <data android:scheme="desired-scheme-name" />
        </intent-filter>
    </activity>
<!-- ... -->
```

### 12. 이벤트 버퍼링

앱에서 이벤트 트래킹을 많이 사용하는 경우 일부 HTTP 요청을 지연하여 1분마다 하나의 배치로 보낼 수 있습니다. `true`로 설정된 `adjustConfig.setEventBufferingEnabled` 메서드를 호출하여 이벤트 버퍼링을 사용하도록 설정할 수 있습니다.

```actionscript
adjustConfig.setEventBufferingEnabled(true);
```

### 13. 트래킹 사용 중지

`false`로 설정된 `setEnabled` 메서드를 호출하여 adjust SDK에서 트래킹을 사용할 수 없도록 설정할 수 있습니다. 이 설정은 세션 간에 기억되지만, 첫 번째 세션 후에만 활성화할 수 있습니다.

```actionscript
Adjust.setEnabled(false);
```

`isEnabled` 메서드를 사용하여 adjust SDK가 현재 활성화되어 있는지 확인할 수 있습니다. `enabled` 매개변수가 `true`로 설정된 `setEnabled`를 호출하여 adjust SDK를 언제든지 활성화할 수 있습니다.

### 14. 오프라인 모드

adjust SDK를 오프라인 모드로 전환하여 adjust 서버로 전송하는 작업을 일시 중단하고 트래킹된 데이터를 보관하여 나중에 보낼 수 있습니다. 오프라인 모드일 때는 모든 정보가 파일에 저장되므로 오프라인 모드에서 너무 많은 이벤트가 트리거되지 않도록 주의하십시오.

`setOfflineMode` 매개변수를 `true`로 설정한 상태로 호출하면 오프라인 모드를 활성화할 수 있습니다.

```actionscript
Adjust.setOfflineMode(true);
```

반대로 `setOfflineMode` 매개변수를 `false`로 설정한 상태로 호출하면 오프라인 모드를 비활성화할 수 있습니다. adjust SDK를 다시 온라인 모드로 전환하면 저장된 정보가 모두 올바른 타임스탬프와 함께 adjust 서버로 전송됩니다.

트래킹 사용 중지와 달리 이 설정은 세션 간에 *기억되지 않습니다.* 따라서 앱을 오프라인 모드에서 종료한 경우에도 SDK는 항상 온라인 모드로 시작됩니다.

### 15. 장치 ID

[Google Analytics][google-analytics]와 같은 서비스를 사용하려면 중복 보고가 발생하지 않도록 장치 ID를 조정해야 합니다.

다음 메서드를 호출하여 adjust SDK에서 수집된 장치 ID를 검색할 수 있습니다.

#### Android

Google 광고 ID가 필요한 경우 제한 사항으로 인해 백그라운드 스레드에서만 해당 ID를 읽을 수 있습니다.  `String` 변수가 매개변수로 지정되는 함수를 전달하여 `getGoogleAdId` 함수를 호출할 경우 이 함수를 언제든지 사용할 수 있습니다.

```as
Adjust.getGoogleAdId(getGoogleAdIdCallback);

// ...

private static function getGoogleAdIdCallback(googleAdId:String):void {
    trace("Google Ad Id = " + googleAdId);
}
```

사용자 정의 메서드인 `getGoogleAdIdCallback`를 통해 Google 광고 ID에 `googleAdId` 변수로 액세스할 수 있습니다.

#### iOS

IDFA를 얻으려면 `getIdfa` 함수를 호출합니다.

```as
Adjust.getIdfa()
```

[adjust.com]: http://adjust.com
[dashboard]: http://adjust.com
[releases]: https://github.com/adjust/adjust_air_sdk/releases
[permissions]: https://raw.github.com/adjust/adjust_sdk/master/Resources/air/permissions.png
[xcode-logs]: https://raw.github.com/adjust/adjust_sdk/master/Resources/air/v4/xcode_logs.png
[google_play_services]: http://developer.android.com/google/play-services/setup.html
[google_ad_id]: https://developer.android.com/google/play-services/id.html
[attribution_data]: https://github.com/adjust/sdks/blob/master/doc/attribution-data.md
[special-partners]: https://docs.adjust.com/en/special-partners

[android-permissions]: https://github.com/adjust/android_sdk#5-add-permissions
[brodcast-android]: https://github.com/adjust/android_sdk#6-add-broadcast-receiver
[attribution-data]: https://github.com/adjust/sdks/blob/master/doc/attribution-data.md
[android-manifest]: https://raw.github.com/adjust/adjust_sdk/master/Resources/air/v4/android_manifest.png
[flash-builder]: https://github.com/adjust/adobe_air_sdk/blob/master/doc/flash_builder.md
[idea-locate]: https://raw.github.com/adjust/adjust_sdk/master/Resources/air/v4/idea_locate.png
[idea-new-library]: https://raw.github.com/adjust/adjust_sdk/master/Resources/air/v4/idea_new_library.png
[idea-new-library-gps]: https://raw.github.com/adjust/adjust_sdk/master/Resources/air/v4/idea_new_library_gps.png
[google-analytics]: https://docs.adjust.com/en/special-partners/google-analytics

## 라이선스

adjust SDK는 MIT 라이선스에 따라 사용이 허가되었습니다.

Copyright (c) 2012-2016 adjust GmbH, http://www.adjust.com

이로써 본 소프트웨어와 관련 문서 파일(이하 "소프트웨어")의 복사본을 받는 사람에게는 아래 조건에 따라 소프트웨어를 제한 없이 다룰 수 있는 권한이 무료로 부여됩니다. 이 권한에는 소프트웨어를 사용, 복사, 수정, 병합, 출판, 배포 및/또는 판매하거나 2차 사용권을 부여할 권리와 소프트웨어를 제공 받은 사람이 소프트웨어를 사용, 복사, 수정, 병합, 출판, 배포 및/또는 판매하거나 2차 사용권을 부여하는 것을 허가할 수 있는 권리가 제한 없이 포함됩니다.

위 저작권 고지문과 본 권한 고지문은 소프트웨어의 모든 복사본이나 주요 부분에 포함되어야 합니다.

소프트웨어는 상품성, 특정 용도에 대한 적합성 및 비침해에 대한 보증 등을 비롯한 어떤 종류의 명시적이거나 암묵적인 보증 없이 "있는 그대로" 제공됩니다. 어떤 경우에도 저작자나 저작권 보유자는 소프트웨어와 소프트웨어의 사용 또는 기타 취급에서 비롯되거나 그에 기인하거나 그와 관련하여 발생하는 계약 이행 또는 불법 행위 등에 관한 배상 청구, 피해 또는 기타 채무에 대해 책임지지 않습니다.
--END--
