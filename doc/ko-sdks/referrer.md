### 다중 브로드캐스트 수신기 지원

앱에 다중 SDK로 `INSTALL_REFERRER` intent를 등록해야 할 경우, `BroadcastReceiver`를 직접 실행하여 지원하려는 다른 수신기를 호출해야 합니다. 아래와 같이 보일 것입니다. [1]

```java
public class InstallReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        // Adjust
        new AdjustReferrerReceiver().onReceive(context, intent);

        // Google Analytics
        new CampaignTrackingReceiver().onReceive(context, intent);
    }
}
```

지원 수신기 목록을 조정하여 들어오는 정보를 반드시 수정하세요. `InstallReceiver`를 직접 사용하려면 `AndroidManifest.xml`을 업데이트해야 합니다.

```java
<receiver
    android:name="com.your.app.InstallReceiver"
    android:exported="true" >
    <intent-filter>
        <action android:name="com.android.vending.INSTALL_REFERRER" />
    </intent-filter>
</receiver>
```

패키지명을 바꾸는 걸 잊지 마세요.

------------------------------------

참조 및 관련 링크

[1] http://stackoverflow.com/questions/14158841/android-google-analytics-v2-broadcast-receiver-for-all-sdks

[2] http://stackoverflow.com/questions/4093150/get-referrer-after-installing-app-from-android-market

[3] http://docs.mdotm.com/index.php/Universal_Application_Tracking