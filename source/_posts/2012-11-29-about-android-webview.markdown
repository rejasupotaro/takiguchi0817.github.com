---
layout: post
title: "ハイブリッドの機運"
date: 2012-11-29 22:15
comments: false
categories: Android WebView
---

## [ネイティブでもHTML5でもない「ハイブリッドアプリ」の価値](http://el.jibun.atmarkit.co.jp/rails/2012/10/html5-d1ba.html)  

ハイブリッドの流れある。  
ネイティブのアプリが書ける人がまだそんなに多くない(ネイティブアプリ縛りの社内ハッカソンも人が集まらなくて大変だった)ので、Webアプリケーションのようにネイティブが書けるの良いと思う。  

弊社にもネイティブとブラウザを連携させるフレームワーク(一応オープンソース)はあって、  

- WebView Bridge (ネイティブとブラウザの相互呼び出し機能)
- Injector (DIコンテナ、A/Bテスト等に使用できる、動的な環境変数によってInjectするインスタンスを切り替える機能)
- Dynamic Loading (バイトコードを動的に読み込んで実行する機能)

の3つの機能がある。  
　  
　  
## [ハッカソンでアプリ作った](http://takiguchi0817.github.com/blog/2012/11/24/save-dailymotion/)

GIFを再生するためにメインのビューはWebViewにした。  
社内フレームワーク(一応オープンソース)は結構大きいので今回のハッカソンでは使わなかった(のでフレームワークの話は後日ブログに書くかもしれない)。  
それで、今回は簡易的なネイティブ・ブラウザ間の呼び出しのしくみを自分で書いた。  

### ジャバ側

{% codeblock lang:java %}
public class JavaScriptInterface {
    private static final String TAG = JavaScriptInterface.class.getSimpleName();

    private static final String INTERFACE_NAME = "Device";

    private WebView mWebView;
    private JavaScriptInterface.Receiver mReceiver;

    public JavaScriptInterface(WebView webView, Receiver receiver) {
        mWebView = webView;
        mReceiver = receiver;

        mWebView.addJavascriptInterface(this, INTERFACE_NAME);
    }

    public void call(String data) {
        if (mReceiver == null) return;

        try {
            mReceiver.receive(new JSONObject(data));
        } catch (JSONException e) {
            Log.e(TAG, data, e);
        }
    }

    public void callBrowserMethod(String jsMethodName) {
        mWebView.loadUrl("javascript:" + jsMethodName + "()");
    }

    public interface Receiver {
        public void receive(JSONObject jsonObject);
    }
}
{% endcodeblock %}

### JS側

{% codeblock lang:javascript %}
callDeviceMethod = function(json) {
  try {
    Device.call(JSON.stringify(json));
  } catch (e) {
    console.log("本来であればアプリ内ブラウザでみるもの: " + e);
  }
};
{% endcodeblock %}

### 呼び方

ネイティブからブラウザはメソッド名で呼び出す(今回はパラメータを渡す必要がなかったので)。  

{% codeblock lang:java %}
mJavaScriptInterface.callBrowserMethod("reload");
{% endcodeblock %}

ブラウザからネイティブはJSON渡して呼ぶ。  

{% codeblock lang:javascript %}
callDeviceMethod({
    method: "download.image",
    body: { image_url: image_url }
});
{% endcodeblock %}

ネイティブ側でメソッドをディスパッチするみたいな感じにした。  
JSONRPCみたいになってると良かった。コールバック？そんなものはない。  

![WebViewなのにタップするとトーストが出る](http://dl.dropbox.com/u/54255753/blog/201211/summer_hackathon.png)  

### セキュリティ的な話

WebViewヤバイ。  

{% codeblock lang:javascript %}
var classLoader = Device.getClass().getClassLoader();
var Runtime = classLoader.loadClass("java.lang.Runtime");
var getRuntimeMethod = Runtime.getMethod("getRuntime", {});
var runtime = getRuntimeMethod.invoke(null, {});
{% endcodeblock %}

クラスローダーが使えるし、リフレクション出来るし、Runtimeとってコマンドが実行出来る。  
JSからジャバのすべてが見えるし、プライベートもあったもんじゃない。  

だから実際に使うときには、

{% codeblock lang:java %}
webView.setWebViewClient(new WebViewClient(){
    @Override
    public void onPageStarted(WebView view, String url, Bitmap favicon) {
        Uri uri = Uri.parse(url);
        if (Constants.PRODUCTION && !UriUtils.compareDomain(uri, Constants.DOMAIN)) {
            throw new SecurityException();
        }
{% endcodeblock %}

みたいにする。  
危険だけどうまく使うとJSからJNI呼ぶみたいな、NaClみたいなこと出来て面白そう。  
　  
　  
## やってみたけど

ハッカソンではWebViewでユーザーインタフェースを書いてたけど、ビルドしなくても変更が反映されるの嬉しい。  
しかもWebViewは何もしなくても勝手にキャッシュされるのも嬉しい。  

しかし、僕はHTMLとCSSはあんまり書かないのでレイアウトうまく行かなくて深夜につらまってたし、端末ごとにCSS切り替えるのにMedia Queriesとやらを使ったけど全然言うこと聞かないし、サムスンの端末のキャッシュの挙動おかしいし(キャッシュ見にいくのに失敗してしかもサーバーからデータを取り直そうとしない)、サムスンの端末はGIFが動かなかったりするし、自分にWebViewのデバッグのノウハウがまったくないので、つらかった。  
　  
　  
## 感想

ハイブリッドは万能じゃない、社会は厳しい。負けるな、開発者。  
