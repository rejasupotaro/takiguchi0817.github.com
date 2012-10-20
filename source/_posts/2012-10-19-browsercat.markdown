---
layout: post
title: "Androidでconsole.log()を見る"
date: 2012-10-19 19:07
comments: false
categories: 
---
## おもにWindowsの方からつらいという声聞こえた
スマートフォン向けのウェブサイトのデバッグ、console.log()を見る方法いくつかある。  
- 標準ブラウザでabout:debug  
対応してないバージョンもある。それにPCにエラーログ出た方が嬉しい。  
- adb logcat  
Linuxとかなら
    adb logcat | grep browser
出来るけど、windowsだとgrep出来なくて悲しいとの声が聞こえました。

## BrowserCatを作ったニャン
[BrowserCat](https://github.com/takiguchi0817/BrowserCat)
Windowsでもadbが入ってれば動きます。  
機能まだ少ないです。  

Write once, run anywhere〜〜
