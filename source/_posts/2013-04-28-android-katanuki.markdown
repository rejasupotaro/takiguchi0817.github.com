---
layout: post
title: "テンプレートアプリ \"katanuki\" を作った"
date: 2013-04-28 16:47
comments: true
categories: Android
---

# [Android Katanuki](https://github.com/takiguchi0817/katanuki)
A template application that includes tons of great open source tools and frameworks.

<img src="https://raw.github.com/takiguchi0817/katanuki/master/katanuki.png" width="640" height="480">

# 経緯
この前、[AndroidKickstartR](http://androidkickstartr.com/)というプロジェクトを見つけました。

<img src="https://dl.dropboxusercontent.com/u/54255753/blog/201304/androidkickstartr.png" width="640" height="480">

これは自分が使いたいライブラリを選んでポチポチ選んで "Download it!" を押したらAndroidアプリのテンプレートが落とせるというものです。
ハッカソンとかで便利そう。
雛形を提供するプロジェクトは他にも
[Android Bootstrap](android katanuk://github.com/donnfelker/android-bootstrap)
などもあります。

で、各々がサポートしているライブラリが以下になります。

### AndroidKickstartR
- android-maven-plugin
- AndroidAnnotations
- ActionBarSherlock
- Spring RESTTemplate
- Android support v4
- NineOldAndroid
- ACRA
- RoboGuice

### Android Bootstrap
- ActionBarSherlock
- Dagger
- Butterknife
- Otto
- Robotium
- android-maven-plugin
- http-request
- google-gson

新しいアプリを作るときのセットアップってほぼ作業だし、
毎回同じことをしてる気がしたので、自分用のアプリのテンプレートを作りました。
作ったというより設定を書いた、の方が近いですが。
（なぜわざわざ作ったかというと自分の使いたいライブラリや書き方が、上のテンプレートと微妙に合わなかったからです。）

# katanukiについて
このテンプレートは、プロトタイプ作成や一日でアプリを組むハッカソンを想定して作りました。
今から数時間でそれっぽいアプリを作るぞ！と思ったときに、何をすれば良いか考えて、

1. APIを叩く
2. データを保存する
3. かっこいいUIを作る

と作業を分解して、APIを叩くのにhttp-request、レスポンスのjsonをオブジェクトに変換するのにgson、それらをAsyncTaskLoaderで呼び出すようにしました。

次にデータの保存はActiveAndroidを使いました。エンタープライズでは使わないと思いますが、面倒なDB周りのコードを書く時間をバッサリカットできるので便利です。

UIについては、入れようか迷ったのですが、中のロジックは一緒でも見せ方はケースバイケースかなと思ったので入れませんでした。
プロジェクトには含めてませんが、このあたりが便利そうです。

- [ActionBarSherlock](https://github.com/JakeWharton/ActionBarSherlock) : ActionBarのcompatibility library
- [PullToRefresh](https://github.com/chrisbanes/Android-PullToRefresh) : 引っ張り更新
- [HoloEverywhere](https://github.com/Prototik/HoloEverywhere) : Holoを2系でも使えるようにする
- [MenuDrawer](https://github.com/SimonVT/android-menudrawer) : 横から出るメニューの実装

OpenSource便利。
