---
layout: post
title: "Otto 使ってみた"
date: 2013-04-15 21:25
comments: true
categories: Android
---

# [Otto - An event bus by Square](http://square.github.io/otto/)
画像のアップロード処理が終わったらActivityに通知するとか、
DBからデータを消したらViewに反映させるとかしたくなることってよくあると思います。

Interfaceでなんとかしようとするとコードが汚くなってやだなと思ってたので、前から気になってたevent busのライブラリ、Ottoを使ってみました。

## バッググラウンドで画像のアップロードが終わったらトーストを出す
Busですが、インスタンスごとにregisterとunregisterが出来るのですが、通常はSingletonで良いでしょう。
```java
public final class BusProvider {
  private static final Bus BUS = new Bus(ThreadEnforcer.ANY);

  public static Bus getInstance() {
    return BUS;
  }

  private BusProvider() {
    // No instances.
  }
}
```
Busのインスタンスの生成のときに実行するスレッドのチェックが出来るのですが、
event busを使うときってバックグラウンドスレッドからメインスレッドに通知するパターンが多そうなので、
ThreadEnforcer.ANYを指定しています。


イベントの発火。
```java
public class ImageUploaderService extends ProtonIntentService {

    private static final String TAG = ImageUploaderService.class.getSimpleName();
    public static final String EXTRA_UPLOAD_ENTITY = "extra_animation_entity";
    @Inject MyNotificationManager mNotificationManager;

    public ImageUploaderService() {
        super(TAG);
    }

    public ImageUploaderService(String name) {
        super(name);
    }

    @Override
    protected void onHandleIntent(Intent intent) {
        mNotificationManager.sendNotification();
        UploadEntity uploadEntity = intent.getParcelableExtra(EXTRA_UPLOAD_ENTITY);
        MyApiClient apiClient = new MyApiClient(this, uploadEntity);
        boolean result = apiClient.execute();
        Log.d(TAG, "upload result: " + result);
        mNotificationManager.cancelNotification();
        BusProvider.getInstance().post(new UploadFinishedEvent());
    }
}
```
bus#postに定義したイベントのインスタンスを渡します。


イベントはなんでもいいです。
```java
public class UploadFinishedEvent {
}
```
今回は特になにも渡すものがないので空です。


メソッドに@Subscribeを付けて定義したイベントを引数にすると受け取ることが出来ます。
```java
public class MyActivity extends Activity {

    ...

    @Override
    public void onResume() {
        super.onResume();
        BusProvider.getInstance().register(this);
    }

    @Override
    protected void onPause() {
        super.onPause();
        BusProvider.getInstance().unregister(this);
    }

    @Subscribe
    public void onUploadFinished(UploadFinishedEvent event) {
        mHandler.post(new Runnable() {
            @Override
            public void run() {
                ToastUtils.show(AnimationComposeActivity.this, R.string.upload_finished);
            }
        });
    }
}
```
ActivityではonResumeとonPauseでそれぞれregisterとunregisterをしてやる必要があります。


これだけでイベント通知のしくみが使えてとても便利。

{% oembed https://twitter.com/rejasupotaro/status/323451917235789824 %}
