---
layout: post
title: "Androidの通知周り"
date: 2012-11-07 23:19
comments: false
categories: Android
---

## ステータスバー

NotificationManagerを介してステータスバーに通知を出したり消したりする。  
[NotificationManagerドキュメント](http://developer.android.com/reference/android/app/NotificationManager.html)  

### 通知を出す・消す

{% codeblock lang:java %}
NotificationManager notificationManager =
    (NotificationManager) context.getSystemService(Context.NOTIFICATION_SERVICE);
{% endcodeblock %}

- 通知を出す：notify(id, notification)  
- 通知を消す：cancel(id)  

イッツシンプル

### 表示するNotificationについて

Notification.Builderを使って生成する(〜API Level 11)  

![ステータスバーの対応表](http://developer.android.com/images/ui/notifications/normal_notification_callouts.png)  

{% codeblock lang:java %}
Notification notification = new Notification.Builder(context)
    .setContentTitle("通知のタイトル") 1に対応
    .setLargeIcon(R.drawable.large_icon) // 2に対応
    .setContentText("テキスト") // 3に対応
    .setContentInfo("ちょっとした情報") // 4に対応
    .setSmallIcon(R.drawable.small_icon) // 5に対応
    .setWhen(System.currentTimeMillis()) // 6に対応
    .setTicker("通知なう", remoteView) // 通知を最初に受け取ったときに表示テキスト
    .setContentIntent(pendingIntent) // 通知をタップしたときに呼ばれるIntent
    .setSound(uri) // 音を鳴らす
    .setLights(argb, onMs, offMs) // ピカピカさせる
    .setVibrate(Notification.DEFAULT_VIBRATE) // バイブ設定
    .getNotification();
{% endcodeblock %}

FLAGがたくさんあると思うけど、なんとなくの理解で、  

|フラグの種類            |内容                                   |
|------------------------|---------------------------------------|
|FLAG_AUTO_CANCEL        |タップしたら消える                     
|FLAG_FOREGROUND_SERVICE |現在実行中のサービスを表す？？？       
|FLAG_HIGH_PRIORITY      |ステータスバーが非表示でも通知が出せる 
|FLAG_INSISTENT          |通知が消されるまで音が繰り返しなる     
|FLAG_NO_CLEAR           |タップとかで消せないようにする         
|FLAG_ONGOING_EVENT      |現在進行形のイベントを表す？？？       
|FLAG_ONLY_ALERT_ONCE    |通知が来る度に音とか振動とか出す       
|FLAG_SHOW_LIGHTS        |ピカピカさせる                         

こんな感じな気がする。  

## プッシュ通知(C2DM)

なんでC2DMかって、2.1はGCMに対応していない。  

[Android C2DM - ソフトウェア技術ドキュメントを勝手に翻訳](http://www.techdoctranslator.com/code-google-com/android-c2dm)  

まず、アプリケーションの登録  

{% codeblock lang:java %}
Intent intent = new Intent("com.google.android.c2dm.intent.REGISTER");
intent.putExtra("app", PendingIntent.getBroadcast(context, 0, new Intent(), 0));
intent.putExtra("sender", emailOfSender);
context.startService(intent);
{% endcodeblock %}

アプリケーションの登録解除  

{% codeblock lang:java %}
Intent intent = new Intent("com.google.android.c2dm.intent.UNREGISTER");
Intent.putExtra("app", PendingIntent.getBroadcast(this, 0, new Intent(), 0));
startService(intent);
{% endcodeblock %}

BroadcastReceiverを継承したPushNotificationReceiverクラスを作る。  

{% codeblock lang:java %}
@Override
public void onReceive(Context context, Intent intent) {
    if (intent != null) {
        String action = intent.getAction();
        if (ACTION_REGISTER.equals(action)) {
            // アーアアー
        } else if (ACTION_RECEIVE.equals(action)) {
            // アーアアー
        }
    }
}
{% endcodeblock %}

C2DMからRegistration IDを受信したときの処理は  

- SyncManagerを動かす
- サーバーに登録IDを送る
- SharedPreferenceにIDを保存する

とか。その他なんらかのメッセージを受信したときの処理は定型なのでリンク先を参照されたし。  

Manifestには以下のように記述しておく。  

{% codeblock lang:xml %}
<!-- このアプリケーションだけがメッセージの受信と登録の結果の受信が可能 -->
<permission android:name="com.rejasupotaro.permission.C2D_MESSAGE" android:protectionLevel="signature" />
<uses-permission android:name="com.rejasupotaro.permission.C2D_MESSAGE" />

<!-- このアプリケーションは登録とメッセージ受信の許可がある -->
<uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />

<!-- 登録 ID をサーバに送信する -->
<uses-permission android:name="android.permission.INTERNET" />

<!-- C2DM サーバのみがアプリにメッセージを送信できる。許可がセットされないと、他のどのアプリでもこれを生成できてしまう -->
<receiver android:permission="com.google.android.c2dm.permission.SEND"
    android:name=".PushNotificationReceiver" >

    <!-- 実際のメッセージの受信 -->
    <intent-filter >
        <action android:name="com.google.android.c2dm.intent.RECEIVE" />
        <category android:name="com.rejasupotaro" />
    </intent-filter>

    <!-- 登録 ID の受信 -->
    <intent-filter >
         <action android:name="com.google.android.c2dm.intent.REGISTRATION" />
         <category android:name="com.rejasupotaro" />
     </intent-filter>
</receiver>
{% endcodeblock %}

だいたいそんな感じ。  
