---
layout: post
title: "Androidの通知周り"
date: 2012-11-07 23:19
comments: false
categories: 
---

## ステータスバー

NotificationManagerを介してステータスバーに通知を出したり消したりする。  
[NotificationManagerドキュメント](http://developer.android.com/reference/android/app/NotificationManager.html)  

### 通知を出す・消す

    NotificationManager notificationManager =
        (NotificationManager) context.getSystemService(Context.NOTIFICATION_SERVICE);

- 通知を出す：notify(id, notification)  
- 通知を消す：cancel(id)  

イッツシンプル

### 表示するNotificationについて

Notification.Builderを使って生成する(〜API Level 11)  

![ステータスバーの対応表](http://developer.android.com/images/ui/notifications/normal_notification_callouts.png)  

    Notification notification = new Notification.Builder(context)
        .setContentTitle("通知のタイトル") 1に対応
        .setLargeIcon(R.drawable.large_icon) // 2に対応
        .setContentText("テキスト") // 3に対応
        .setContentInfo("ちょっとした情報") // 4に対応
        .setSmallIcon(R.drawable.small_icon) // 5に対応
        .setWhen(System.currentTimeMillis()) // 6に対応
        .setTicker("通知なう", remoteView) // 通知を最初に受け取ったときに表示するテキスト
        .setContentIntent(pendingIntent) // 通知をタップしたときに呼ばれるIntent
        .setSound(uri) // 音を鳴らす
        .setLights(argb, onMs, offMs) // ピカピカさせる
        .setVibrate(Notification.DEFAULT_VIBRATE) // バイブ設定
        .getNotification();

## プッシュ通知(C2DM)

なんでC2DMかって、2.1はGCMに対応していない。  

[Android C2DM - ソフトウェア技術ドキュメントを勝手に翻訳](http://www.techdoctranslator.com/code-google-com/android-c2dm)  

まず、アプリケーションの登録  

    Intent intent = new Intent("com.google.android.c2dm.intent.REGISTER");
    intent.putExtra("app", PendingIntent.getBroadcast(context, 0, new Intent(), 0));
    intent.putExtra("sender", emailOfSender);
    context.startService(intent);

アプリケーションの登録解除  

    Intent intent = new Intent("com.google.android.c2dm.intent.UNREGISTER");
    Intent.putExtra("app", PendingIntent.getBroadcast(this, 0, new Intent(), 0));
    startService(intent);

BroadcastReceiverを継承したPushNotificationReceiverクラスを作る。  

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

C2DMからRegistration IDを受信したときの処理は  

- SyncManagerを動かす
- サーバーに登録IDを送る
- SharedPreferenceにIDを保存する

とか。その他なんらかのメッセージを受信したときの処理は定型なのでリンク先を参照されたし。  

Manifestには以下のように記述しておく。  

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

だいたいそんな感じ。  
