---
layout: post
title: "エミュレータにGoogle Playにあるapkを入れる"
date: 2012-10-06 13:41
comments: true
categories: 
---
方法は2つ。  

1. エミュレータにGoogle Playを入れる
2. PCでapkをダウンロードしてadbでエミュレータに移す

1の方法は面倒な感じしたので2にすることにした。

## Google Playからapkを直接ダウンロードできるようにする
Chrome拡張を入れることでブラウザ版のGoogle Playからapkをダウンロードできるようになる。

### AndroidのデバイスIDを調べる
アプリを使うと簡単。
[Device ID](https://play.google.com/store/apps/details?id=com.redphx.deviceid)
を起動してDevice IDをメモしておく。

### APK Downloaderを入れる
[APK Downloader – Download APK files from Android Market to PC](http://codekiem.com/2012/02/24/apk-downloader/)

詳しい手順：
[PlayストアからAndroidのapkを直接ダウンロードできる「APK Downloader」](http://www.teradas.net/archives/3894/)

apkをダウンロードしたら、以下のオプションでchromeを起動する。
    open -a Google\ Chrome --args --ignore-certificate-errors
「ツール」→「拡張機能」の画面にドラッグ・アンド・ドロップ。

## エミュレータにインストールする
実際にapkをダウンロードしてエミュレータにインストールする。

### Google Playからapkをダウンロードする。
Google Playに行ってインストールしたいアプリのインストールページに行く。  
アドレスバーのところにAPK Downloaderのアイコンが出るので、クリックしてダウンロード。  

### エミュレータにインストールする
    $ emulator -avd device_name
    $ adb install ~/Downloads/HelloGooglePlay.apk

/data/data/パッケージ名 にダウンロードされる。
