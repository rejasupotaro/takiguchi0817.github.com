---
layout: post
title: MySQL、文字化けた
date: 2012-11-23 19:21
comments: false
categories: MySQL Ubuntu
---

デプロイしておーエラー出なかったーって思ってブラウザで開いたら文字化けしてた。  

    mysql> STATUS

アーMacBookはutf8だったけどサーバーはlatin1のままだった。  
文字コード変えるためにmysql止めようとしたら、いつの間にか/etc/init.d/mysqlを使うんじゃなくてservice mysqlを使うことになってた。  

まあ、それはそれで、/etc/mysql/conf.d/charset.confに  

    [client]
    default-character-set=utf8
    
    [mysqld]
    character-set-server=utf8
    skip-character-set-client-handshake

    [mysqldump]
    default-character-set=utf8

って書いてオルターテーブルしたら文字化け直った、良かった。  
mysql5.5でmysqldのdefault-character-setなくなってたの知らなくてちょっと困ったりした。  
