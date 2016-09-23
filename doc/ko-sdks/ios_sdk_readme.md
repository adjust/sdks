## 요약

이 항목에서는 adjust의 iOS SDK에 대해 설명합니다. adjust에 대한 자세한 내용은 [adjust.com]을 참조하십시오.

## 목차

* [앱 예제](#example-apps)
* [기본 연동](#basic-integration)
    * [SDK 다운로드 및 설치](#step1)
    * [프로젝트에 추가](#step2)
    * [AdSupport 및 iAd 프레임워크 추가](#step3)
    * [앱에 Adjust 연동](#step4)
        * [Import 문](#import-statement)
        * [기본 설정](#basic-setup)
        * [adjust 로깅](#adjust-logging)
    * [앱 작성](#step5)
* [추가 기능](#additional-features)
    * [이벤트 트래킹 설정](#step6)
        * [매출 트래킹](#track-revenue)
            * [매출 중복 제거](#revenue-deduplication)
            * [인앱 구매 확인](#iap-verification)
        * [콜백 매개변수](#callback-parameters)
        * [파트너 매개변수](#partner-parameters)
    * [딥링크 리어트리뷰션 설정](#step7)
        * [Universal Link](#universal-links)
            * [대시보드에서 Universal Link 사용](#ulinks-dashboard)
            * [iOS 앱에서 Universal Link를 처리할 수 있게 설정](#ulinks-ios-app)
        * [adjust SDK에서 지원되는 모든 iOS 버전에 대한 딥링크 연결 지원](#ulinks-support-all)
    * [이벤트 버퍼링](#step8)
    * [백그라운드에서 보내기](#step9)
    * [어트리뷰션 콜백](#step10)
    * [트래킹되는 이벤트 및 세션에 대한 콜백](#step11)
    * [지연된 딥링크에 대한 콜백](#step12)
    * [트래킹 사용 중지](#step13)
    * [오프라인 모드](#step14)
    * [장치 ID](#step15)
    * [푸시 토큰](#step16)
* [문제 해결](#troubleshooting)
    * [지연된 SDK 초기화의 문제](#ts-delayed-init)
    * ["Adjust requires ARC" 오류가 나타납니다](#ts-arc)
    * ["\[UIDevice adjTrackingEnabled\]: unrecognized selector sent to instance" 오류가 나타납니다](#ts-categories)
    * ["Session failed (Ignoring too frequent session.)" 오류가 나타납니다](#ts-session-failed)
    * [로그에 "Install tracked"가 표시되지 않습니다](#ts-install-tracked)
    * ["Unattributable SDK click ignored" 메시지가 나타납니다](#ts-iad-sdk-click)
    * [adjust 대시보드에 잘못된 매출 데이터가 표시됩니다](#ts-wrong-revenue-amount)
* [라이선스](#license)

## <a id="example-apps"></a>앱 예제

[`examples` 디렉터리][examples]에
[`iOS (Objective-C)`][example-ios-objc], [`iOS (Swift)`][example-ios-swift], 
[`tvOS`][example-tvos] 및 [`Apple Watch`][example-iwatch] 앱 예제가 있습니다. Xcode 프로젝트를 열어 adjust SDK를 연동할 수 있는 방법의 예를 확인할 수 있습니다.

## <a id="basic-integration">기본 연동

이 항목에서는 adjust SDK를 iOS 프로젝트에 연동하는 절차에 대해 설명합니다.
여기서는 Xcode를 iOS 개발에 사용한다고 가정합니다.

[CocoaPods][cocoapods]를 사용하는 경우 다음 행을 `Podfile`에 추가한 후 [4단계](#step4)로 계속 진행합니다.

```ruby
pod 'Adjust', '~> 4.7.0'
```

또는:

```ruby
pod 'Adjust', :git => 'https://github.com/adjust/ios_sdk.git', :tag => 'v4.7.0'
```

[Carthage][carthage]를 사용하는 경우 다음 행을 `Cartfile`에 추가한 후 [3단계](#step3)로 계속 진행합니다.

```ruby
github "adjust/ios_sdk"
```

adjust SDK를 프로젝트에 프레임워크로 추가하여 프로젝트에 연동할 수도 있습니다.
[릴리스 페이지][releases]에 다음 압축 파일 3개가 있습니다.

* `AdjustSdkStatic.framework.zip`
* `AdjustSdkDynamic.framework.zip`
* `AdjustSdkStaticNoBitcode.framework.zip`

Apple은 iOS 8 출시 이후 동적 프레임워크(임베디드 프레임워크)를 도입했습니다. 
앱이 iOS 8 이상을 대상으로 하는 경우 adjust SDK 동적 프레임워크를 사용할 수 있습니다. 
사용할 프레임워크(정적 또는 동적)를 선택하고 프로젝트에 추가합니다.

Bitcode가 지원되지 않는 정적 adjust SDK 프레임워크를 사용하려는 경우 `AdjustSdkStaticNoBitcode.framework.zip` 파일을 선택하여 프로젝트에 추가할 수 있습니다.

adjust SDK를 연동할 방법을 선택한 후 [3단계](#step3)로 계속 진행할 수 있습니다. adjust SDK의 소스 파일을 프로젝트에 추가하여 adjust SDK를 추가하려면 [1단계](#step1)부터 진행합니다.

### <a id="step1"></a>1. SDK 다운로드 및 설치

[릴리스 페이지][releases]에서 최신 버전을 다운로드합니다. 압축 파일을 선택한 디렉터리에 풉니다.

### <a id="step2">2. 프로젝트에 추가

Xcode의 Project Navigator에서 `Supporting Files` 그룹(또는 기타 원하는 그룹)을 찾습니다. Finder에서 `Adjust` 하위 디렉터리를 Xcode의 `Supporting Files` 그룹으로 드래그합니다.

![][drag]

`Choose options for adding these files` 대화상자에서 `Copy items if needed` 확인란이 선택되었는지 확인하고 `Create
groups` 라디오 버튼을 선택합니다.

![][add]

### <a id="step3"></a>3. AdSupport 및 iAd 프레임워크 추가

Project Navigator에서 프로젝트를 선택합니다. 기본 뷰의 왼쪽에서 대상을 선택합니다. `Build Phases` 탭에서 `Link
Binary with Libraries` 그룹을 확장합니다. 해당 섹션의 하단에서 `+` 버튼을 클릭합니다.
`AdSupport.framework`를 선택하고 `Add` 버튼을 클릭합니다. tvOS를 사용하지 않는 경우 동일한 절차를 반복하여 `iAd.framework`를 추가합니다. 두 프레임워크에서 모두 `Status`를 `Optional`로 변경합니다.

![][framework]

### <a id="step4"></a>4. 앱에 Adjust 연동

#### <a id="import-statement">Import 문

adjust SDK를 소스에서 추가했거나 Pod 리포지토리를 통해 추가한 경우 다음 import 문 중 하나를 사용해야 합니다.

```objc
#import "Adjust.h"
```

또는

```objc
#import <Adjust/Adjust.h>
```

adjust SDK를 프레임워크로 추가했거나 Carthage를 통해 추가한 경우 다음 import 문을 사용해야 합니다.

```objc
#import <AdjustSdk/Adjust.h>
```

먼저 기본 세션 트래킹을 설정합니다.

#### <a id="basic-setup">기본 설정

Project Navigator에서 Application Delegate의 소스 파일을 엽니다.
`import` 문을 파일의 맨 위에 추가한 후 App Delegate의 `didFinishLaunching` 또는 `didFinishLaunchingWithOptions` 메서드에서 다음 호출을 `Adjust`에 추가합니다.

```objc
#import "Adjust.h"
// or #import <Adjust/Adjust.h>
// or #import <AdjustSdk/Adjust.h>

// ...

NSString *yourAppToken = @"{YourAppToken}";
NSString *environment = ADJEnvironmentSandbox;
ADJConfig *adjustConfig = [ADJConfig configWithAppToken:yourAppToken
                                            environment:environment];
[Adjust appDidLaunch:adjustConfig];
```
![][delegate]

**참고**: adjust SDK를 이런 방식으로 초기화하는 것은 매우 중요합니다. 그렇지 않으면 문제 해결 섹션에서 설명하는 여러 가지 [문제](#ts-delayed-init)가 발생할 수 있습니다.

`{YourAppToken}`을 앱 토큰으로 변경합니다. 앱 토큰은 [대시보드]에서 찾을 수 있습니다.

앱을 테스트에 사용할지 아니면 프로덕션에 사용할 지에 따라 `environment`을 다음 값 중 하나로 설정해야 합니다.

```objc
NSString *environment = ADJEnvironmentSandbox;
NSString *environment = ADJEnvironmentProduction;
```

**중요:** 이 값은 앱을 테스트하는 경우에만 `ADJEnvironmentSandbox`로 설정해야 합니다. 앱을 게시하기 직전에 environment를 `ADJEnvironmentProduction`으로 설정해야 합니다. 개발 및 테스트를 다시 시작하면 `ADJEnvironmentSandbox`로 다시 설정하십시오.

이 환경은 실제 트래픽과 테스트 장치의 테스트 트래픽을 구별하기 위해 사용합니다. 이 값을 항상 의미 있게 유지해야 합니다! 이것은 특히 매출을 트래킹하는 경우에 중요합니다.

#### <a id="adjust-logging">adjust 로깅

다음 매개변수 중 하나를 사용하여 `ADJConfig` 인스턴스에서 `setLogLevel:`을 호출하면 테스트에 표시되는 로그의 양을 늘리거나 줄일 수 있습니다.

```objc
[adjustConfig setLogLevel:ADJLogLevelVerbose]; // enable all logging
[adjustConfig setLogLevel:ADJLogLevelDebug];   // enable more logging
[adjustConfig setLogLevel:ADJLogLevelInfo];    // the default
[adjustConfig setLogLevel:ADJLogLevelWarn];    // disable info logging
[adjustConfig setLogLevel:ADJLogLevelError];   // disable warnings as well
[adjustConfig setLogLevel:ADJLogLevelAssert];  // disable errors as well
```

### <a id="step5">5. 앱 작성

앱을 작성하고 실행합니다. 앱을 작성한 경우 콘솔에서 SDK 로그를 주의깊게 읽어야 합니다. 앱을 처음 시작한 후에는 `Install tracked` 정보 로그를 확인해야 합니다.

![][run]

## <a id="additional-feature">추가 기능

adjust SDK를 프로젝트에 연동한 후에는 다음 기능을 사용할 수 있습니다.

### <a id="step6">6. 이벤트 트래킹 설정

adjust를 사용하여 이벤트를 트래킹할 수 있습니다. 특정 버튼의 모든 탭을 트래킹하려는 경우 `abc123`와 같은 관련 이벤트 토큰이 있는 새 이벤트 토큰을 [대시보드]에서 만듭니다. 그런 다음 버튼의 `buttonDown` 메서드에 다음 행을 추가하여 클릭을 트래킹할 수 있습니다.

```objc
ADJEvent *event = [ADJEvent eventWithEventToken:@"abc123"];
[Adjust trackEvent:event];
```

버튼을 누르면 `Event tracked`가 로그에 나타납니다.

이벤트 인스턴스를 사용하여 이벤트를 트래킹하기 전에 더 자세히 구성할 수 있습니다.

#### <a id="track-revenue">매출 트래킹

사용자가 광고를 누르거나 인앱 구매를 통해 매출을 발생시킬 수 있는 경우 이벤트를 사용하여 해당 매출을 트래킹할 수 있습니다. 한 번 누를 때 0.01 유로의 매출이 발생한다고 가정할 경우 매출 이벤트를 다음과 같이 트래킹할 수 있습니다.

```objc
ADJEvent *event = [ADJEvent eventWithEventToken:@"abc123"];
[event setRevenue:0.01 currency:@"EUR"];
[Adjust trackEvent:event];
```

이것을 콜백 매개변수와 결합할 수도 있습니다.

통화 토큰을 설정하면 adjust에 수신되는 매출이 선택한 보고 매출로 자동 변환됩니다. 통화 변환에 대한 자세한 내용은 [여기][currency-conversion]를 참조하십시오.

매출 및 이벤트 트래킹에 대한 자세한 내용은 [이벤트 트래킹 설명서][https://docs.adjust.com/en/event-tracking/#reference-tracking-purchases-and-revenues]를 참조하십시오.

##### <a id="revenue-deduplication"></a> 매출 중복 제거

중복되는 매출을 트래킹하지 않도록 트랜잭션 ID(선택 사항)를 추가할 수도 있습니다. 마지막 10개의 트랜잭션 ID가 저장되고, 트랜잭션 ID가 중복되는 매출 이벤트는 건너뜁니다. 이 기능은 인앱 구매 트래킹에 특히 유용합니다. 아래 예를 참조하십시오.

인앱 구매를 트래킹하려면 상태가 `SKPaymentTransactionStatePurchased`로 변경된 경우에만 `paymentQueue:updatedTransaction`에서 `finishTransaction` 후에 `trackEvent`를 호출해야 합니다. 이렇게 해야 실제로 발생하지 않은 매출을 트래킹하는 오류를 방지할 수 있습니다.

```objc
- (void)paymentQueue:(SKPaymentQueue *)queue updatedTransactions:(NSArray *)transactions {
    for (SKPaymentTransaction *transaction in transactions) {
        switch (transaction.transactionState) {
            case SKPaymentTransactionStatePurchased:
                [self finishTransaction:transaction];

                ADJEvent *event = [ADJEvent eventWithEventToken:...];
                [event setRevenue:... currency:...];
                [event setTransactionId:transaction.transactionIdentifier]; // avoid duplicates
                [Adjust trackEvent:event];

                break;
            // more cases
        }
    }
}
```

##### <a id="iap-verification">인앱 구매 유효성 검사

adjust의 서버 측 수신 확인 도구인 구매 유효성 검사를 사용하여 앱에서 수행되는 인앱 구매의 유효성을 검사하려면 iOS 구매 SDK를 확인하십시오. 자세한 내용은 [여기][ios-purchase-verification]을 참조하십시오.

#### <a id="callback-parameters">콜백 매개변수

[대시보드]에서 이벤트의 콜백 URL을 등록할 수 있습니다. 이벤트가 트래킹될 때마다 GET 요청이 해당 URL로 전송됩니다. 이 이벤트를 트래킹하기 전에 이벤트에서 `addCallbackParameter`를 호출하여 콜백 매개변수를 해당 이벤트에 추가할 수 있습니다. 그러면 해당 매개변수가 콜백 URL에 추가됩니다.

예를 들어 `http://www.adjust.com/callback` URL을 등록한 경우 이벤트를 다음과 같이 트래킹할 수 있습니다.

```objc
ADJEvent *event = [ADJEvent eventWithEventToken:@"abc123"];
[event addCallbackParameter:@"key" value:@"value"];
[event addCallbackParameter:@"foo" value:@"bar"];
[Adjust trackEvent:event];
```

#### <a id="partner-parameters">파트너 매개변수

adjust 대시보드에서 활성화된 연동에 대해 네트워크 파트너로 전송할 매개변수도 추가할 수 있습니다.

이 매개변수는 위에서 설명한 콜백 매개변수의 경우와 비슷하지만, `ADJEvent` 인스턴스에서 `addPartnerParameter` 메서드를 호출해야 추가할 수 있습니다.

```objc
ADJEvent *event = [ADJEvent eventWithEventToken:@"abc123"];
[event addPartnerParameter:@"key" value:@"value"];
[Adjust trackEvent:event];
```

특별 파트너와 해당 파트너와의 연동에 대한 자세한 내용은 [특별 파트너 설명서][special-partners]를 참조하십시오.

이 경우에는 이벤트가 트래킹되고 요청이 다음 주소로 전송됩니다.

    http://www.adjust.com/callback?key=value&foo=bar

매개변수 값으로 사용할 수 있는 `{idfa}`와 같은 다양한 자리 표시자가 지원됩니다. 결과로 생성된 콜백에서 이 자리 표시자는 현재 장치의 광고주 ID로 대체됩니다. 또한 사용자 지정 매개변수는 저장되지 않고 콜백에만 추가됩니다. 이벤트에 대한 콜백을 등록하지 않은 경우 해당 매개변수는 읽을 수 없습니다.

사용 가능한 값의 전체 목록을 포함한 URL 콜백 사용에 대한 자세한 내용은 [콜백 설명서][callbacks-guide]를 참조하십시오.

### <a id="step7">7. 딥링크 리어트리뷰션 설정

사용자 지정 URL 구조를 통해 앱을 여는 데 사용되는 딥링크를 처리하도록 adjust SDK를 설정할 수 있습니다. adjust에서는 특정 adjust 관련 매개변수만 읽습니다. 딥링크를 사용하여 리타게팅 또는 재참여 캠페인을 진행할 계획인 경우 이 설정은 필수입니다.

Project Navigator에서 Application Delegate의 소스 파일을 엽니다. `openURL` 메서드를 찾거나 추가하고 다음 호출을 adjust에 추가합니다.

```objc
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url
  sourceApplication:(NSString *)sourceApplication annotation:(id)annotation {
    [Adjust appWillOpenUrl:url];
    
    // Your code goes here
    BOOL canHandle = [self someLogic:url];
    return canHandle;
}
```

**중요**: `deep_link` 매개변수와 사용자 지정 URL 구조가 있는 트래커 URL은 `iOS 8 and higher`에서 더 이상 지원되지 않으며, 이 URL을 클릭해도 앱이 열리거나 `openURL` 메서드가 트리거되지 않습니다. Apple은 더 이상 앱에 딥링크로 연결하는 것을 지원하지 않고 `universal links`를 대신 사용합니다. 
그러나 이 방법은 iOS 7 이하 버전의 장치에서 사용됩니다.

#### <a id="universal-links">Universal Link

**참고**: Universal Link는 iOS 8 이하에서 지원됩니다.

[Universal Link][universal-links]를 지원하려면 다음 단계를 수행하십시오.

##### <a id="ulinks-dashboard">대시보드에서 Universal Link 사용

앱에 Universal Link를 사용하려면 adjust 대시보드로 이동하여 iOS `Platform Settings`에서 `Universal Linking` 옵션을 선택합니다.

![][universal-links-dashboard]

<a id="ulinks-setup">`iOS Bundle ID`와 `iOS Team ID`를 입력해야 합니다.

![][universal-links-dashboard-values]

iOS Team ID는 `Apple Developer Center`에 있습니다.

![][adc-ios-team-id]

이 두 개 값을 입력한 후에는 앱으로 연결되는 다음과 같은 Universal Link가 생성됩니다.

```
applinks:[hash].ulink.adjust.com
```

##### <a id="ulinks-ios-app">iOS 앱에서 Universal Link를 처리할 수 있게 설정

Apple Developer Center에서 앱에 대해 `Associated Domains`를 사용할 수 있게 설정해야 합니다.

![][adc-associated-domains]

**중요**: 일반적으로 `iOS Team ID`는 앱에 대한 `Prefix` 값(위 그림 참조)과 동일합니다. 그러나 경우에 따라 이 두 개 값이 다를 수 있습니다. 이런 경우 이 [단계](#ulink-setup)로 돌아가서 `Prefix` 값을 adjust 대시보드의 `iOS Team ID` 필드에 사용하십시오.

이 작업을 수행한 후에는 앱의 Xcode 프로젝트 설정에서 `Associated Domains`을 사용하도록 설정하고, 생성된 Universal Link를 대시보드에서 `Domains` 섹션으로 복사할 수 있습니다.

![][xcode-associated-domains]

그런 다음 `application:continueUserActivity:restorationHandler:` 메서드를 찾거나 Application Delegate에 추가합니다. 이 메서드에서 다음 호출을 adjust에 추가합니다.

``` objc
- (BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity 
 restorationHandler:(void (^)(NSArray *restorableObjects))restorationHandler {
    if ([[userActivity activityType] isEqualToString:NSUserActivityTypeBrowsingWeb]) {
        [Adjust appWillOpenUrl:[userActivity webpageURL]];
    }

    // Your code goes here
    BOOL canHandle = [self someLogic:[userActivity webpageURL]];
    return canHandle;
}
```

예를 들어 `deep_link` 매개변수를 다음과 같이 설정하는 경우

```
example://path/?key=foo&value=bar
```

adjust 백엔드에서 이 매개변수를 다음과 같은 Universal Link로 변환합니다.

```
https://[hash].ulink.adjust.com/ulink/path/?key=foo&value=bar
```

adjust에는 Universal Link를 딥링크 URL로 변환하는 데 사용할 수 있는 도우미 기능이 있습니다.

```objc
NSURL *deeplink = [Adjust convertUniversalLink:[userActivity webpageURL] scheme:@"example"];
```

Universal Link 구현에 대한 자세한 내용은 [Universal Link 설명서][universal-links-guide]를 참조하십시오.

#### <a id="ulinks-support-all">adjust SDK에서 지원하는 모든 iOS 버전에 대해 딥링크 연결 지원

앱을 iOS 8 이상에서 사용할 경우 앱에 딥링크 연결을 사용하도록 설정하려면 Universal Link만 있으면 됩니다. 그러나 iOS 6 및 iOS 7 장치에서도 딥링크 연결을 지원하려면 [여기서][adjust-universal-links] 설명하는 것과 같은 adjust 형식의 Universal Link를 작성해야 합니다.

또한 장치의 iOS 버전에 따라 `iOS 6 and iOS 7` 또는 `iOS 8 and higher` 메서드 중 하나가 트리거되기 때문에 두 메서드가 모두 `openURL` 및 `application:continueUserActivity:restorationHandler:`Application Delegate 클래스에 구현되고 파싱할 딥링크 콘텐츠가 앱에 전달되도록 하고, 사용자를 이동시킬 위치를 결정해야 합니다.

구현을 테스트하는 방법에 대한 설명은 이 [설명서][universal-links-testing]를 참조하십시오.

### <a id="step8">8. 이벤트 버퍼링

앱에서 이벤트 트래킹을 많이 사용하는 경우 일부 HTTP 요청을 지연하여 1분마다 하나의 배치로 보낼 수 있습니다. `ADJConfig` 인스턴스를 통해 이벤트 버퍼링을 사용하도록 설정할 수 있습니다.

```objc
[adjustConfig setEventBufferingEnabled:YES];
```

### <a id="step9">9. 백그라운드에서 보내기

adjust SDK는 기본적으로 앱이 백그라운드에서 실행되는 동안 HTTP 보내기를 일시정지합니다.
이 설정을 `AdjustConfig`에서 변경할 수 있습니다.

```objc
[adjustConfig setSendInBackground:YES];
```

### <a id="step10">10. 어트리뷰션 콜백

위임 콜백을 등록하여 트래커 어트리뷰션 변경에 대한 알림을 받을 수 있습니다. 어트리뷰션을 위해 고려되는 다양한 소스로 인해 이 정보는 동시에 제공될 수 없습니다. App Delegate에서 위임 프로토콜(선택 사항)을 구현하려면 다음 단계를 수행하십시오.

[해당 어트리뷰션 데이터 정책][attribution-data]을 고려하십시오.

1. `AppDelegate.h`를 열고 `AdjustDelegate` 선언을 추가합니다.

    ```objc
    #import "Adjust.h"
    // or #import <Adjust/Adjust.h>
    // or #import <AdjustSdk/Adjust.h>

    @interface AppDelegate : UIResponder <UIApplicationDelegate, AdjustDelegate>
    ```

2. `AppDelegate.m`을 열고 다음 위임 호출 함수를 앱 위임 구현에 추가합니다.

    ```objc
    - (void)adjustAttributionChanged:(ADJAttribution *)attribution {
    }
    ```

3. `ADJConfig` 인스턴스를 사용하여 위임을 설정합니다.

    ```objc
    [adjustConfig setDelegate:self];
    ```
    
위임 콜백은 `ADJConfig` 인스턴스를 사용하여 구성되므로, `[Adjust appDidLaunch:adjustConfig]`를 호출하기 전에 `setDelegate`를 호출해야 합니다.

SDK에 최종 어트리뷰션 데이터가 수신되면 위임 함수가 호출됩니다.
위임 함수를 통해 `attribution` 매개변수에 액세스할 수 있습니다.
매개변수 속성에 대한 개요는 다음과 같습니다.

- `NSString trackerToken` 현재 설치의 트래커 토큰.
- `NSString trackerName` 현재 설치의 트래커 이름.
- `NSString network` 현재 설치의 network 그룹화 기준.
- `NSString campaign` 현재 설치의 campaign 그룹화 기준.
- `NSString adgroup` 현재 설치의 ad group 그룹화 기준.
- `NSString creative` 현재 설치의 creative 그룹화 기준.
- `NSString clickLabel` 현재 설치의 클릭 레이블.

### <a id="step11">11. 트래킹되는 이벤트 및 세션에 대한 콜백

위임 콜백을 등록하여 성공 또는 실패 트래킹 대상 이벤트 및/또는 세션에 대한 알림을 받을 수 있습니다.

[어트리뷰션 변경 콜백](#step10)에 사용되는 것과 동일한 선택적 프로토콜인 `AdjustDelegate`가 사용됩니다.

동일한 단계에 따라 성공한 트래킹 이벤트에 대해 다음 위임 콜백 함수를 구현하십시오.

```objc
- (void)adjustEventTrackingSucceeded:(ADJEventSuccess *)eventSuccessResponseData {
}
```

실패한 트래킹 이벤트의 경우 다음 위임 콜백 함수:

```objc
- (void)adjustEventTrackingFailed:(ADJEventFailure *)eventFailureResponseData {
}
```

성공한 트래킹 세션의 경우:
```objc
- (void)adjustSessionTrackingSucceeded:(ADJSessionSuccess *)sessionSuccessResponseData {
}
```

그리고 실패한 트래킹 세션의 경우:

```objc
- (void)adjustSessionTrackingFailed:(ADJSessionFailure *)sessionFailureResponseData {
}
```

위임 함수는 SDK에서 서버로 패키지를 보내려고 시도한 후에 호출됩니다. 
위임 콜백에서는 위임 콜백 전용 응답 데이터 개체에 액세스할 수 있습니다. 세션 응답 데이터 속성에 대한 개요는 다음과 같습니다.

- `NSString message` 서버에서 전송된 메시지 또는 SDK에 의해 로깅된 오류.
- `NSString timeStamp` 서버에서 전송된 타임스탬프.
- `NSString adid` adjust에 의해 제공된 고유 장치 식별자.
- `NSDictionary jsonResponse` 서버에서 전송된 응답이 있는 JSON 개체.

두 이벤트 응답 데이터 개체에는 모두 다음이 포함됩니다.

- `NSString eventToken` 트래킹 패키지가 이벤트인 경우 이벤트 토큰.

그리고 이벤트 및 세션 실패 개체에는 모두 다음이 포함됩니다.

- `BOOL willRetry` 나중에 패키지를 다시 보내려는 시도가 있을 것임을 나타냅니다.

### <a id="step12">12. 지연된 딥링크에 대한 콜백

지연된 딥링크가 열리기 전에 알림을 받을 위임 콜백을 등록하고 adjust SDK에서 딥링크를 열도록 할 것인지 결정할 수 있습니다.

[어트리뷰션 변경 콜백](#step10) 및 [트래킹되는 이벤트 및 세션)(#step11)에 사용되는 것과 동일한 선택적 프로토콜인 `AdjustDelegate`가 사용됩니다

동일한 단계에 따라 지연된 딥링크에 대해 다음 위임 콜백 함수를 구현하십시오.

```objc
// evaluate deeplink to be launched
- (void)adjustDeeplinkResponse:(NSURL *)deeplink {
     // ...
     if ([self allowAdjustSDKToOpenDeeplink:deeplink]) {
         return YES;
     } else {
         return NO;
     }
}
```

콜백 함수는 SDK에서 지연된 딥링크를 서버로부터 수신한 후와 딥링크를 열기 전에 호출됩니다. 
콜백 함수에서 딥링크에 액세스할 수 있으며, 반환하는 부울식에 의해 SDK에서 딥링크를 실행할 것인지 결정됩니다.
예를 들어 딥링크를 SDK에서 지금 열지 않고 딥링크를 저장한 후 나중에 직접 열도록 할 수 있습니다.

### <a id="step13">13. 트래킹 사용 중지

`setEnabled`를 `NO`로 설정할 상태로 호출하면 adjust SDK에서 현재 장치의 모든 작업을 트래킹하지 않도록 할 수 있습니다. **이 설정은 세션 간에 기억되지만**, 첫 번째 세션 후에만 활성화할 수 있습니다.

```objc
[Adjust setEnabled:NO];
```

<a id="is-enabled">`isEnabled` 함수를 호출하여 adjust SDK가 현재 사용 가능한지 확인할 수 있습니다. 매개변수가 `YES`로 설정된 `setEnabled`를 호출하면 adjust SDK를 언제든지 활성화할 수 있습니다.

### <a id="step14">14. 오프라인 모드

adjust SDK를 오프라인 모드로 전환하여 adjust 서버로 전송하는 작업을 일시 중단하고 트래킹된 데이터를 보관하여 나중에 보낼 수 있습니다. 오프라인 모드일 때는 모든 정보가 파일에 저장되므로 오프라인 모드에서 너무 많은 이벤트가 트리거되지 않도록 주의하십시오.

`setOfflineMode`를 `YES`로 설정된 상태로 호출하면 오프라인 모드를 활성화할 수 있습니다.

```objc
[Adjust setOfflineMode:YES];
```

반대로 `setOfflineMode`를 `NO`로 설정한 상태로 호출하면 오프라인 모드를 비활성화할 수 있습니다.
adjust SDK를 다시 온라인 모드로 전환하면 저장된 정보가 모두 올바른 시간 정보와 함께 adjust 서버로 전송됩니다.

트래킹 사용 중지와 달리 이 설정은 세션 간에 *기억되지 않습니다.* 따라서 앱을 오프라인 모드에서 종료한 경우에도 SDK는 항상 온라인 모드로 시작됩니다.

### <a id="step15">15. 장치 ID

Google Analytics와 같은 서비스를 사용하려면 중복 보고가 발생하지 않도록 장치 ID와 클라이언트 ID를 조정해야 합니다. 

장치 식별자 IDFA를 얻으려면 `idfa` 함수를 호출합니다.

```objc
NSString *idfa = [Adjust idfa];
```

### <a id="step16">16. 푸시 토큰

adjust로 푸시 알림 토큰을 보내려면 app delegate의 `didRegisterForRemoteNotificationsWithDeviceToken`에서 다음 호출을 `Adjust`에 추가합니다.

```objc
- (void)application:(UIApplication *)app didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
    [Adjust setDeviceToken:deviceToken];
}
```

## <a id="troubleshooting">문제 해결

#### <a id="ts-delayed-init">지연된 SDK 초기화의 문제

[기본 설정 단계](#basic-setup)의 설명처럼 adjust SDK를 app delegate의 `didFinishLaunching` 또는 `didFinishLaunchingWithOptions` 메서드에서 초기화하는 것이 좋습니다. SDK의 모든 기능을 사용할 수 있도록 최대한 빨리 adjust SDK를 초기화하는 것이 중요합니다.

adjust SDK를 초기화하지 않기로 할 경우 앱 트래킹에 중대한 영향을 미칠 수 있으므로 다음 내용에 대해 잘 알고 있어야 합니다. **앱에서 모든 종류의 트래킹을 수행하려면 adjust SDK를 *초기화해야* 합니다.**

SDK를 초기화하기 전에 아래 작업 중 하나를 수행하기로 하면

* [이벤트 트래킹](#step6)
* [딥링크 리어트리뷰션](#step7)
* [트래킹 사용 중지](#step13)
* [오프라인 모드](#step14)

해당 작업은 수행되지 않습니다.

이 경우 앱에 `custom
actions queueing mechanism`을 만들어야 adjust SDK를 실제로 초기화하기 전에 수행하려고 했던 모든 작업을 수행할 수 있습니다.

오프라인 모드 상태는 변경되지 않고 트래킹 사용 가능/사용 중지 상태도 변경되지 않습니다. 딥링크 리어트리뷰션은 수행되지 않고 트래킹되는 이벤트는 모두 삭제됩니다.

세션 트래킹도 지연된 SDK의 영향을 받을 수 있습니다. adjust SDK를 실제로 초기화하기 전에는 세션 길이 정보를 수집할 수 없습니다.
이 경우 대시보드에서 DAU 수치가 올바르게 트래킹되지 않을 수 있습니다.

예를 들어, 특정 뷰나 뷰 컨트롤러가 로드되면 adjust SDK를 시작하고 사용자가 귀사 앱의 스플래시 화면 또는 처음 화면이 아닌 홈 화면에서 이 뷰로 이동해야 한다고 가정해 보십시오. 사용자가 앱을 다운로드하고 열면 홈 화면이 표시됩니다. 이 때 트래킹이 필요한 설치를 수행했는데, 이 사용자가 특정 캠페인에서 왔을 수 있고 앱을 시작했으며 자신의 장치에서 세션을 만들었으므로 해당 사용자는 실제로 앱의 일일 활성 사용자였습니다. adjust SDK는 이런 내용에 대해 전혀 모릅니다. 사용자가 이 SDK를 초기화하기로 결정한 화면으로 이동해야 하기 때문입니다. 사용자가 홈 화면을 본 후 바로 앱을 제거하기로 결정하면 위에서 언급한 모든 정보는 adjust SDK에 의해 트래킹되거나 대시보드에 표시되지 않습니다.

adjust SDK에서 수행할 모든 작업을 대기열로 보내고 SDK가 초기화되면 해당 작업을 수행해야 합니다.

##### 이벤트 트래킹

트래킹할 이벤트를 내부 대기열 메커니즘을 사용하여 대기열로 보내고 SDK가 초기화된 후 이벤트를 트래킹합니다. SDK를 초기화하기 전에 이벤트를 트래킹하면 이벤트가 영구 삭제되므로, SDK가 초기화되고 [사용 가능](#is-enabled)으로 설정된 후에 이벤트가 트래킹되는지 확인하십시오.

##### 오프라인 모드와 트래킹 사용/사용 중지

오프라인 모드는 SDK 초기화 후에도 계속 실행되는 기능이 아니므로, 기본적으로 `false`로 설정됩니다. SDK를 초기화하기 전에 오프라인 모드를 사용하도록 설정하면 나중에 SDK를 초기화해도 `false`로 설정됩니다.

트래킹 사용/사용 중지 설정은 SDK 초기화 후에도 유지됩니다.
SDK를 초기화하기 전에 이 값을 토글하려고 하면 토글 시도가 무시됩니다.
초기화된 SDK는 이 토글을 시도하기 전의 상태(사용 또는 사용 중지)로 유지됩니다.

##### 딥링크 리어트리뷰션

[7단계](#step7)에서 설명한 대로 딥링크 리어트리뷰션을 처리할 경우에는 사용하는 딥링크 연결 메커니즘(이전 방식 또는 Universal Link)에 따라 `NSURL` 개체를 얻게 되고 그런 다음 호출을 수행해야 합니다.

```objc
[Adjust appWillOpenUrl:url]
```

SDK가 초기화되기 전에 이 호출을 수행하면 사용자가 클릭하고 리어트리뷰션되었어야 할 URL의 딥링크에 대한 정보를 영구적으로 잃게 됩니다.
adjust SDK에서 사용자를 성공적으로 리어트리뷰션하려면 SDK가 초기화된 후 이 `NSURL` 개체 정보를 대기열로 보내고 `appWillOpenUrl` 메서드를 트리거해야 합니다.

##### 세션 트래킹

세션 트래킹은 adjust SDK에서 자동으로 수행하므로 앱 개발자가 제어할 수 없습니다. 올바른 세션 트래킹을 위해서는 이 추가 정보에서 권장하는 방법으로 adjust SDK를 초기화해야 합니다. 그렇지 않으면 올바른 세션 트래킹과 대시보드의 DAU 수치에 중대한 영향을 미칩니다. 예를 들어 다음과 같은 문제가 발생할 수 있습니다.

* SDK가 초기화되기도 전에 사용자가 앱을 삭제하여 설치와 세션이 트래킹되지 않고, 따라서 대시보드에서 보고되지 않음
* 자정 전에 사용자가 앱을 다운로드하고 연 다음 자정이 지난 후 adjust SDK가 초기화됨으로써 설치 및 세션이 다른 날에 보고됨
* 사용자가 같은 날에 앱을 사용하지 않고 자정 직후에 열고 자정이 지난 후에 SDK가 초기화되어 앱을 연 날이 아닌 날에 DAU가 보고됨

따라서 이 설명서의 내용을 준수하고 adjust SDK를 app delegate의 `didFinishLaunching` 또는 `didFinishLaunchingWithOptions` 메서드에서 초기화하십시오.

#### <a id="ts-arc">"Adjust requires ARC" 오류가 나타납니다

빌드 시 `Adjust requires ARC` 오류가 발생할 경우 프로젝트에서 [ARC][arc]를 사용하지 않은 것이 원인일 수 있습니다. 이 경우 [ARC를 사용하도록 프로젝트를 전환][transition]하는 것이 좋습니다. ARC를 사용하지 않으려면 대상의 빌드 단계에서 adjust의 모든 소스 파일에 ARC를 사용하도록 설정해야 합니다.

`Compile Sources` 그룹을 확장하고 모든 adjust 파일을 선택한 다음 `Compiler Flags`를 `-fobjc-arc`로 변경합니다(모두 선택하고 `Return` 키를 눌러 동시에 변경).

#### <a id="ts-categories">"[UIDevice adjTrackingEnabled]: unrecognized selector sent to instance" 오류가 나타납니다

이 오류는 adjust SDK 프레임워크를 앱에 추가하는 경우 발생할 수 있습니다. adjust SDK의 소스 파일에는 `categories`가 포함되어 있기 때문에 이 SDK 연동 방법을 선택한 경우 Xcode 프로젝트 설정에서 `-ObjC` 플래그를 `Other Linker Flags`에 추가해야 합니다. 이 플래그를 추가하면 이 오류가 해결됩니다.

#### "Session failed (Ignoring too frequent session.)" 오류가 나타납니다

이 오류는 일반적으로 설치를 테스트할 때 발생합니다. 앱을 제거하고 다시 설치해도 새 설치를 트리거할 수 없습니다. 서버에서는 SDK가 로컬에서 집계된 세션 데이터를 유실했다고 판단하며 서버에 제공된 장치 관련 정보에 따라 오류 메시지를 무시합니다.

이 동작은 테스트 중에 불편을 초래할 수도 있지만, sandbox 동작이 프로덕션 환경과 최대한 일치하도록 하기 위해 필요합니다.

장치의 세션 데이터를 adjust 서버에서 재설정할 수 있습니다. 로그에서 다음 오류 메시지를 확인합니다.

```
Session failed (Ignoring too frequent session. Last session: YYYY-MM-DDTHH:mm:ss, this session: YYYY-MM-DDTHH:mm:ss, interval: XXs, min interval: 20m) (app_token: {yourAppToken}, adid: {adidValue})
```

<a id="forget-device">아래에 `{yourAppToken}` 및 `{adidValue}` 값을 입력하고 다음 링크를 엽니다.

```
http://app.adjust.com/forget_device?app_token={yourAppToken}&adid={adidValue}
```

장치가 메모리에서 삭제되면 링크에서 `Forgot device`만 반환됩니다. 장치가 이미 메모리에서 삭제되었거나 값이 올바르지 않으면 `Device not found`가 반환됩니다.

#### <a id="ts-install-tracked">로그에 "Install tracked"가 표시되지 않습니다

테스트 장치에서 앱 설치 시나리오를 시뮬레이션하려는 경우 이미 앱이 설치되어 있는 테스트 장치의 Xcode에서 앱을 다시 실행하는 것만으로는 충분하지 않습니다. Xcode에서 앱을 다시 실행하면 앱 데이터가 모두 삭제되지 않고 adjust SDK가 앱에 보관하는 모든 내부 파일이 유지되므로, adjust SDK는 해당 파일을 확인한 후 앱이 이미 설치되어 있고 SDK가 앱에서 이미 시작되었지만 처음 열린 게 아니라 한 번 더 열렸을 뿐이라고 인식합니다.

앱 설치 시나리오를 실행하려면 다음 작업을 수행해야 합니다.

* 장치에서 앱을 제거합니다(완전 제거).
* [위](#forget-device) 문제에서 설명한 대로 테스트 장치를 adjust 백엔드에서 삭제합니다.
* 테스트 장치의 Xcode에서 앱을 실행하면 "Install tracked" 로그 메시지가 표시됩니다.

#### <a id="ts-iad-sdk-click">"Unattributable SDK click ignored" 메시지가 표시됩니다.

앱을 `sandbox` 환경에서 테스트하는 중에 이 메시지가 표시될 수 있습니다. 이 메시지는 Apple이 `iAd.framework` 버전 3에서 변경한 내용과 관련이 있습니다. 사용자가 iAd 배너를 클릭하면 앱으로 이동될 수 있으며, 이로 인해 adjust SDK에서 `sdk_click` 패키지를 adjust 백엔드로 보내 클릭된 URL의 내용에 대해 알릴 수 있습니다. Apple은 iAd 배너를 클릭하지 않았는데 앱이 열릴 경우 임의의 값을 사용하여 iAd 배너 URL 클릭을 인위적으로 생성하기로 결정했습니다. adjust SDK는 iAd 배너 클릭이 진짜인지 인위적으로 생성된 것인지 구별할 수 없으므로 모두 경우에 `sdk_click` 패키지를 adjust 백엔드로 보냅니다. 로그 레벨을 `verbose` 레벨로 설정한 경우 이 `sdk_click` 패키지는 다음과 같이 표시됩니다.

```
[Adjust]d: Added package 1 (click)
[Adjust]v: Path:      /sdk_click
[Adjust]v: ClientSdk: ios4.7.0
[Adjust]v: Parameters:
[Adjust]v:      app_token              {YourAppToken}
[Adjust]v:      created_at             2016-04-15T14:25:51.676Z+0200
[Adjust]v:      details                {"Version3.1":{"iad-lineitem-id":"1234567890","iad-org-name":"OrgName","iad-creative-name":"CreativeName","iad-click-date":"2016-04-15T12:25:51Z","iad-campaign-id":"1234567890","iad-attribution":"true","iad-lineitem-name":"LineName","iad-creative-id":"1234567890","iad-campaign-name":"CampaignName","iad-conversion-date":"2016-04-15T12:25:51Z"}}
[Adjust]v:      environment            sandbox
[Adjust]v:      idfa                   XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
[Adjust]v:      idfv                   YYYYYYYY-YYYY-YYYY-YYYY-YYYYYYYYYYYY
[Adjust]v:      needs_response_details 1
[Adjust]v:      source                 iad3
```

이 `sdk_click`을 고려할 경우 사용자가 다른 캠페인 URL을 클릭하거나 유기적 사용자로서 앱을 열 경우 존재하지 않는 iAd 소스에 어트리뷰션되는 상황이 발생할 수 있습니다.

그렇기 때문에 adjust 백엔드는 이를 무시하고 다음 메시지를 통해 알립니다.

```
[Adjust]v: Response: {"message":"Unattributable SDK click ignored."}
[Adjust]i: Unattributable SDK click ignored.
```

따라서 이 메시지는 SDK 연동에 문제가 있음을 의미하지 않으며, adjust 백엔드에서 사용자가 잘못 어트리뷰션/리어트리뷰션되는 결과를 초래했을 수 있는 인위적으로 생성된 `sdk_click`을 무시했음을 알려줄 뿐입니다.

#### <a id="ts-wrong-revenue-amount">adjust 대시보드에 잘못된 매출 데이터가 있습니다

adjust SDK는 지정된 대상만 트래킹합니다. 매출을 이벤트에 연결하는 경우, 금액으로 작성하는 숫자만 adjust 백엔드에 도달하고 대시보드에 표시되는 유일한 금액이 됩니다. adjust SDK는 금액 값을 조작하지 않으며 adjust 백엔드도 마찬가지입니다. 따라서 잘못된 금액이 트래킹되는 것이 보이면 adjust SDK에서 해당 금액을 트래킹하는 것입니다.

매출 이벤트 트래킹 사용자 코드는 일반적으로 다음과 같습니다.

```objc
// ...

- (double)someLogicForGettingRevenueAmount {
    // This method somehow handles how user determines 
    // what's the revenue value which should be tracked.
    
    // It is maybe making some calculations to determine it.
    
    // Or maybe extracting the info from In-App purchase which
    // was successfully finished.
    
    // Or maybe returns some predefined double value.
    
    double amount; // double amount = some double value
    
    return amount;
}

// ...

- (void)someRandomMethodInTheApp {
    double amount = [self someLogicForGettingRevenueAmount];
    
    ADJEvent *event = [ADJEvent eventWithEventToken:@"abc123"];
    [event setRevenue:amount currency:@"EUR"];
    [Adjust trackEvent:event];
}

```

트래킹하도록 지정한 값이 아닌 다른 값이 대시보드에 보일 경우 **금액 값을 결정하는 로직을 확인하십시오**.

[adjust.com]: http://adjust.com
[cocoapods]: http://cocoapods.org
[carthage]: https://github.com/Carthage/Carthage
[dashboard]: http://adjust.com
[examples]: http://github.com/adjust/ios_sdk/tree/master/examples
[example-ios-objc]: http://github.com/adjust/ios_sdk/tree/master/examples/AdjustExample-iOS
[example-ios-swift]: http://github.com/adjust/ios_sdk/tree/master/examples/AdjustExample-Swift
[example-tvos]: http://github.com/adjust/ios_sdk/tree/master/examples/AdjustExample-tvOS
[example-iwatch]: http://github.com/adjust/ios_sdk/tree/master/examples/AdjustExample-iWatch
[releases]: https://github.com/adjust/ios_sdk/releases
[arc]: http://en.wikipedia.org/wiki/Automatic_Reference_Counting
[transition]: http://developer.apple.com/library/mac/#releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html
[drag]: https://raw.github.com/adjust/sdks/master/Resources/ios/drag5.png
[universal-links-dashboard]: https://raw.github.com/adjust/sdks/master/Resources/ios/universal-links-dashboard5.png
[universal-links-dashboard-values]: https://raw.github.com/adjust/sdks/master/Resources/ios/universal-links-dashboard-values5.png
[adc-ios-team-id]: https://raw.github.com/adjust/sdks/master/Resources/ios/adc-ios-team-id5.png
[adc-associated-domains]: https://raw.github.com/adjust/sdks/master/Resources/ios/adc-associated-domains5.png
[xcode-associated-domains]: https://raw.github.com/adjust/sdks/master/Resources/ios/xcode-associated-domains5.png
[add]: https://raw.github.com/adjust/sdks/master/Resources/ios/add5.png
[framework]: https://raw.github.com/adjust/sdks/master/Resources/ios/framework5.png
[delegate]: https://raw.github.com/adjust/sdks/master/Resources/ios/delegate5.png
[run]: https://raw.github.com/adjust/sdks/master/Resources/ios/run5.png
[AEPriceMatrix]: https://github.com/adjust/AEPriceMatrix
[attribution-data]: https://github.com/adjust/sdks/blob/master/doc/attribution-data.md
[callbacks-guide]: https://docs.adjust.com/en/callbacks
[event-tracking]: https://docs.adjust.com/en/event-tracking
[special-partners]: https://docs.adjust.com/en/special-partners
[currency-conversion]: https://docs.adjust.com/en/event-tracking/#tracking-purchases-in-different-currencies
[universal-links-guide]: https://docs.adjust.com/en/universal-links/
[universal-links]: https://developer.apple.com/library/ios/documentation/General/Conceptual/AppSearch/UniversalLinks.html
[adjust-universal-links]: https://docs.adjust.com/en/universal-links/#redirecting-to-universal-links-directly
[universal-links-testing]: https://docs.adjust.com/en/universal-links/#testing-universal-link-implementations
[ios-purchase-verification]: https://github.com/adjust/ios_purchase_sdk

## <a id="license">라이선스

adjust SDK는 MIT 라이선스에 따라 사용이 허가됩니다.

Copyright (c) 2012-2016 adjust GmbH, http://www.adjust.com

이로써 본 소프트웨어와 관련 문서 파일(이하 "소프트웨어")의 복사본을 받는 사람에게는 아래 조건에 따라 소프트웨어를 제한 없이 다룰 수 있는 권한이 무료로 부여됩니다. 이 권한에는 소프트웨어를 사용, 복사, 수정, 병합, 출판, 배포 및/또는 판매하거나 2차 사용권을 부여할 권리와 소프트웨어를 제공 받은 사람이 소프트웨어를 사용, 복사, 수정, 병합, 출판, 배포 및/또는 판매하거나 2차 사용권을 부여하는 것을 허가할 수 있는 권리가 제한 없이 포함됩니다.

위 저작권 고지문과 본 권한 고지문은 소프트웨어의 모든 복사본이나 주요 부분에 포함되어야 합니다.

소프트웨어는 상품성, 특정 용도에 대한 적합성 및 비침해에 대한 보증 등을 비롯한 어떤 종류의 명시적이거나 암묵적인 보증 없이 "있는 그대로" 제공됩니다. 어떤 경우에도 저작자나 저작권 보유자는 소프트웨어와 소프트웨어의 사용 또는 기타 취급에서 비롯되거나 그에 기인하거나 그와 관련하여 발생하는 계약 이행 또는 불법 행위 등에 관한 배상 청구, 피해 또는 기타 채무에 대해 책임지지 않습니다.
--END--
