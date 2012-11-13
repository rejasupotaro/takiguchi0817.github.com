---
layout: post
title: "Androidでconsole.log()を見る"
date: 2012-10-19 19:07
comments: false
categories: Android
---
## おもにWindowsの方からつらいという声聞こえた
スマートフォン向けのウェブサイトのデバッグ、console.log()を見る方法いくつかある。  

- Eclipseのlogcatを見る  
Android開発者ならまだしも、JavaScript書いてるのにEclipse立ち上げたくない。  

- 標準ブラウザでabout:debug  
対応してないバージョンもある。それにPCにエラーログ出た方が嬉しい。  
![スクリーンショット](http://dl.dropbox.com/u/54255753/blog/201210/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202012-10-19%2019.29.40.png)  

- adb logcat  
Linuxとかなら adb logcat | grep browser 出来るけど、windowsだとgrep出来なくて悲しい。

- busyboxを入れて端末側でgrepする  
ビルドが面倒とか、すべての検証端末に入れるのが面倒とか。

## BrowserCatを作ったニャン
[BrowserCat](https://github.com/takiguchi0817/BrowserCat)  
![スクリーンショット](http://dl.dropbox.com/u/54255753/blog/201210/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202012-10-19%2020.22.30.png)  
windowsでもadbが入ってれば動きます。  
まだログが見れるくらいしか機能ないです。  
