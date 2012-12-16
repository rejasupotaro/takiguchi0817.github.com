---
layout: post
title: "RoboGuice 2.0 で Dependency を Injection しました"
date: 2012-12-01 16:07
comments: false
categories: Android RoboGuice
---

{% oembed https://twitter.com/mizchi/status/274790220988297216 %}
{% oembed https://twitter.com/mizchi/status/274790200465555456 %}

人間はだるくない開発がしたい。  
　  
　  
そこで、DIフレームワークを使うことにした。  
[Deep dive into RoboGuice beyond "Hello World apps"](http://www.blog.project13.pl/wp-content/uploads/2011/12/presentation.html#slide1)  

![微妙に可愛くないキャラクター](http://dl.dropbox.com/u/54255753/blog/201212/roboguice.png)

["RoboGuice 2 smoothes out some of the wrinkles in your Android development experience and makes things simple and fun!"](http://code.google.com/p/roboguice/)  
ということらしいので、RoboGuice入れてみた。  
　  
　  
## インストール

maven使ってないので [Installation for Non-Maven Projects](http://code.google.com/p/roboguice/wiki/InstallationNonMaven)  

- [RoboGuice 2.0](http://repo1.maven.org/maven2/org/roboguice/roboguice/)
- [Guice 3.0-no_aop](http://repo1.maven.org/maven2/com/google/inject/guice/)
- [jsr305 (optional support for @Nullable annotation, if desired)](http://repo1.maven.org/maven2/com/google/code/findbugs/jsr305/)

これらを入れろって言ってるけど、入れても動かなかった。  

{% codeblock lang:java %}
12-01 14:26:49.250: E/AndroidRuntime(15131): FATAL EXCEPTION: main
12-01 14:26:49.250: E/AndroidRuntime(15131): java.lang.NoClassDefFoundError: javax.inject.Provider
12-01 14:26:49.250: E/AndroidRuntime(15131):  at com.google.inject.internal.MoreTypes.canonicalizeForKey(MoreTypes.java:81)
12-01 14:26:49.250: E/AndroidRuntime(15131):  at com.google.inject.Key.<init>(Key.java:119)
12-01 14:26:49.250: E/AndroidRuntime(15131):  at com.google.inject.Key.get(Key.java:212)
12-01 14:26:49.250: E/AndroidRuntime(15131):  at roboguice.inject.ContextScope.enter(ContextScope.java:87)
12-01 14:26:49.250: E/AndroidRuntime(15131):  at roboguice.inject.ContextScope.<init>(ContextScope.java:61)
12-01 14:26:49.250: E/AndroidRuntime(15131):  at roboguice.RoboGuice.newDefaultRoboModule(RoboGuice.java:163)
12-01 14:26:49.250: E/AndroidRuntime(15131):  at roboguice.RoboGuice.setBaseApplicationInjector(RoboGuice.java:120)
12-01 14:26:49.250: E/AndroidRuntime(15131):  at roboguice.RoboGuice.getBaseApplicationInjector(RoboGuice.java:59)
12-01 14:26:49.250: E/AndroidRuntime(15131):  at roboguice.RoboGuice.getInjector(RoboGuice.java:149)
12-01 14:26:49.250: E/AndroidRuntime(15131):  at roboguice.activity.RoboActivity.onCreate(RoboActivity.java:76)
{% endcodeblock %}

javaxのinjectがないって言ってる。  
調べたらRoboGuiceを使うのにGoogle Guiceが必要で、Google Guiceは[javax.inject](http://code.google.com/p/atinject/downloads/list)が必要とのこと。  
ダウンロードしてlibsに入れたら動くようになった。  

ググっても全然ブログ出てこないのでちょっと不安になってきた。  
　  
　  
## 使い方

公式の[チュートリアル](http://code.google.com/p/roboguice/wiki/InjectView)のやる気スイッチ押してあげたい。  

ActivityとかFragmentActivityは、RoboActivityとかRoboFragmentActivityとかを継承するようにする。  
RoboGuice 2.0からはRoboApplicationは継承しなくてよくなった。  

### @ContentView

{% codeblock lang:java %}
public class MainActivity extends Activity {

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
{% endcodeblock %}

みたいにレイアウトをセットしてたのが、  

{% codeblock lang:java %}
@ContentView(R.layout.activity_main)
public class MainActivity extends RoboActivity {
    
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // setContentView(R.layout.activity_main); <= いらない
{% endcodeblock %}

ってなる。  

### @InjectView

{% codeblock lang:java %}
@ContentView(R.layout.activity_main)
public class MainActivity extends RoboActivity {
    
    private Button mPostButton;
    private Button mCloseButton;
    
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        
        mPostButton = (Button) findViewById(R.id.button_post);
        mCloseButton = (Button) findViewById(R.id.button_close);
        
        mPostButton.setOnClickListener(new View.OnClickListener() {
            public void onClick(View v) {
                // アーアアー
            }
        });
{% endcodeblock %}

みたいにIDからビューを取得してキャストしてたのが、  

{% codeblock lang:java %}
@ContentView(R.layout.activity_main)
public class MainActivity extends RoboActivity {
    
    @InjectView(R.id.button_post)
    private Button mPostButton;
    
    @InjectView(R.id.button_close)
    private Button mCloseButton;
    
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        
        mPostButton.setOnClickListener(new View.OnClickListener() {
            public void onClick(View v) {
                // アーアアー
            }
        });
{% endcodeblock %}

いきなりイベントをセットしたりできるようになる。キャストも不要でかっこいい。  

### @Injection

いろいろ使えるけど、ヘルパーをインジェクトするのはすごく具合が良かった。  

モジュールを定義する  

{% codeblock res/values/roboguice.xml lang:xml %}
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string-array name="roboguice_modules">
        <item>com.rejasupotaro.roboguicesample.RoboGuiceTestModule</item>
    </string-array>
</resources>
{% endcodeblock %}

{% codeblock lang:java %}
public class RoboGuiceSampleModule extends AbstractModule {

    @Override
    protected void configure() {
        bind(MainActivityHelper.class).in(ContextSingleton.class);
    }

}
{% endcodeblock %}

ヘルパーを定義する  

{% codeblock lang:java %}
public class AbstractActivityHelper {

    @Inject
    private Context mContext;

    protected Activity getActivity() {
        return (Activity) mContext;
    }

    protected Context getContext() {
        return mContext;
    }
}
{% endcodeblock %}

こんな抽象クラスを作っておくと、

{% codeblock lang:java %}
public class MainActivityHelper extends AbstractActivityHelper {

    public void launchGallarey() {
        Intent intent = new Intent();
        intent.setType("image/*");
        intent.setAction(Intent.ACTION_GET_CONTENT);
        getActivity().startActivityForResult(intent, REQUEST_GALLERY);
    }

    public void launchActivity(Class targetClass) {
        getActivity().startActivity(new Intent(getActivity(), targetClass));
    }
}
{% endcodeblock %}

こんな感じでコンストラクタなしでどんどんメソッド書ける。  

{% codeblock lang:java %}
public class MainActivity extends RoboActivity {
    
    @Inject
    private MainActivityHelper mActivityHelper;
    
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        mMainActivityHelper.launchGallarey();
{% endcodeblock %}

みたいにいきなりメソッドが呼べてかっこいい。  
　  
　  
何も考えずにAndroidアプリ書いてるとコントローラがどんどん太るから、コントローラにはなるべく処理は書かずにジュースを使ってどんどんヘルパーにデリゲートしよう。  
