### Adjust 브로드캐스트 수신기

`INSTALL_REFERRER` 인텐트로 다른 브로드캐스트 수신기를 사용하고 있지 않다면 `AndroidManifest.xml`에 있는 `application` 태그에 다음 `receiver` 태그를 추가합니다.

```xml
<receiver
    android:name="com.adjust.sdk.AdjustReferrerReceiver"
    android:exported="true" >
    <intent-filter>
        <action android:name="com.android.vending.INSTALL_REFERRER" />
    </intent-filter>
</receiver>
```

![](https://camo.githubusercontent.com/5604c6d59b8faca5c16d94d6a43f4199536ea07f/68747470733a2f2f7261772e6769746875622e636f6d2f61646a7573742f73646b732f6d61737465722f5265736f75726365732f616e64726f69642f76342f30395f72656365697665722e706e67)

이 브로드캐스트 수신기는 전환 트래킹을 개선하기 위해 설치 리퍼러 검색에 사용합니다.

다른 브로드캐스트 수신기를 `INSTALL_REFERRER` intent로 이미 사용 중이라면 [이 지침][referrer]에 따라 Adjust 수신기를
추가하십시오.
