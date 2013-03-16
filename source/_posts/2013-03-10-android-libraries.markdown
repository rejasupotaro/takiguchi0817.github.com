---
layout: post
title: "2013年 Androidアプリ開発で使いたいライブラリ"
date: 2013-03-10 22:35
comments: true
categories: Android
---

30億のデバイスのみなさん、こんにちは、ジャバです。

![迫り来るジャバモンスター](http://dl.dropbox.com/u/54255753/blog/201303/javamonster.gif)

何の前触れもなく、2013年 Androidアプリ開発で使いたいライブラリを発表します。

## JsonConverter => [gson](https://code.google.com/p/google-gson/)

JSONRPCでサーバーサイドと通信を行うときに、毎回パーサーを書くのは面倒なので、JsonConverterを使いたくなります。
そこで開発ではgoogleが開発しているgsonを使っていました。
NamingPolicyやDeserializerの使い方を覚えれば、手でjsonのパーサーを書くより圧倒的に楽で、

    Person person = gson.fromJson(json, Person.class);

このように一行でjsonからオブジェクトに変換できるようになり、

    String json = gson.toJson(person);

一行でオブジェクトからStringに変換することも出来ます。
なので、preferenceに保存 => 復旧もすごく便利になります。もうSerializableはやめましょう！

ただ、gsonは各々の型の変換のためにTypeAdapterを保持しているというのと、変換にはリフレクションを使っているので、パフォーマンスはあまり良くないです。
また、レスポンスのjsonの構造がクラスになってしまうので、変換したクラスをそのままモデルとして使おうとすると柔軟性が下がってしまいます。なので、使うのであれば

    json => api entity => model

ってしたいなと思います。

## ORM => [ActiveAndroid](https://www.activeandroid.com/) or [greenDAO](http://greendao-orm.com/)

SQL文をHelperに記述してるときは人間らしい心を失いそうになります。そして、人間はtypoする生き物なのでDBのバージョンアップで死んだりすることもあります。  
そこでORMということになるのですが、AndroidのORMでは、RailsのAndroid版であるActiveAndroidというのがあります。
ただし、Active Recordパターンの欠点もそのままなので、"構築が容易であり理解もしやすい"代償としてそのままでは複雑なロジックを扱いづらくなります。
ドメイン層とパーシステンス層が一緒になったのがActive Recordパターンなので、Rubyみたいにmixinができる言語は良いけど、ジャバだとテストが書きづらくなるので、あまりオススメしない、って某氏が言ってました。
データアクセスに関しては [データアクセスことはじめ](http://www.oracle.com/technetwork/jp/articles/index-087873-ja.html) が勉強になりました。

あまり規模が大きくないならActiveAndroidは記述量が減るし読みやすいしで、良い選択肢だと思います。
それ以上なら、ORMLiteを高速化したgreenDAOを使うのが良いかもしれません。
<iframe src="http://www.slideshare.net/slideshow/embed_code/12321475" width="427" height="356" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC;border-width:1px 1px 0;margin-bottom:5px" allowfullscreen webkitallowfullscreen mozallowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="http://www.slideshare.net/droidcon/green-dao" title="Green dao" target="_blank">Green dao</a> </strong> from <strong><a href="http://www.slideshare.net/droidcon" target="_blank">Droidcon Berlin</a></strong> </div>

## DI => [RoboGuice](https://github.com/roboguice/roboguice) or [Proton](https://github.com/hnakagawa/proton) or [Dagger](http://square.github.com/dagger/)

Fragmentのイベントを別のFragmentで作用させたいときに、interfaceを定義してActivity経由でlistenerを登録とかするとすぐにlistener地獄になってしまってつらいです。
そういうときにContextSingletonなヘルパーをインジェクトして使うとめちゃ便利で、コードの見通しが良くなって、仕様変更に強くなって心が豊かになってモテ始めたりすると思います。
他にもテスト実行時にインジェクトするオブジェクトを切り替えられるとか、記述が楽になる以外にもメリットはたくさんあると思います。  

有名なのはRoboGuiceです。これはサーバーサイドのDIフレームワークのGuiceを、Androidでも使えるようにラップしたものです。
サーバーサイドを想定して作られたものなので、無駄が多かったりするのですが、それなりにドキュメントがあって実績もあります。

そんなRoboGuiceを見て後述のTriainaフレームワークの開発者の人が、
「RoboGuice無駄に大きしTypeListenerとかいらんし、もっと早くて軽いのをフルスクラッチで書く」
と言って作られたのがProtonです。いらない機能を削ってAndroidに最適化した結果、サイズもメモリ使用量も圧倒的に少なくなったとのことです。
ただドキュメントがないので使うならRoboGuiceの知識が必須で、ソースコードを読みながらになると思います。

RoboActivityとかProtonActivityとか継承したくない、かつコンテキストシングルトンとかいらない！というケースであれば、単純にインジェクトだけをしたいのならDagger良いです。

# おわりに

もうちょっと書こうと思ったのですが意外と書くのがたいへんだったので突然ブログは終わります。他にも、  

- event busの[Otto](http://square.github.com/otto/)
- WebView Bridgeの[Triaina](https://github.com/mixi-inc/triaina)
- Code Dietが出来る[AndroidAnnotations](http://androidannotations.org/)
- jQueryのAndroid版の[android-query](https://code.google.com/p/android-query/)

とかあるので、また検証をしたら個別に記事を書こうと思います。
