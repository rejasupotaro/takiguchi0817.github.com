---
layout: post
title: "Androidアプリを作るときに気にすること(セキュリティ)"
date: 2013-01-15 23:28
comments: false
categories: Android Security
---

### Androidセキュリティあるあるをまとめてみました。
　  
#### <input type="checkbox">アプリケーションデータディレクトリの外に個人情報を置いていないか？
基本的に個人情報はSDカードや他のディレクトリに置いてはいけません。
どうしても置きたい場合は[暗号化](http://takiguchi0817.github.com/blog/2012/12/31/java-aes/)します。

#### <input type="checkbox">WebViewで許可されたドメイン以外にJavaScriptInterfaceが公開されていないか？
JavaScriptInterfaceからクラスローダーが取得できたりJNIが呼べたりしてしまうので[たいへん危険](https://www.google.co.jp/#hl=ja&q=Android+WebView+%E5%8D%B1%E9%99%BA&fp=1)です。

#### <input type="checkbox">ContentProviderのパーミッションは適切か？
コンテントプロバイダはデフォルトで公開されているので注意が必要です。
また2.2以前だとexportedをfalseにしていても外部アプリから情報が読めてしまいます。

#### <input type="checkbox">BroadCastのパーミッションは適切か？
パーミッションを付け忘れると外に情報が出てしまいます。
またStickyブロードキャストはパーミッションが指定できないので個人情報を入れてはいけません。

#### <input type="checkbox">端末固有の識別子をサーバ側でIDとして使っていないか？
[怖い人](http://news.mynavi.jp/articles/2012/03/28/abc2012_06/index.html)が飛んでくるおそれがあります。
　  
　  
### まとめ
脆弱性のあるアプリを一度世に出してしまったら、いくらセキュリティパッチをあてたところで、ユーザにアップデートをしてもらえなければそれまでです。
ちょっとした注意で防げるものもあるので、外部からの攻撃ならまだしも、こちら側で防げるものは確実に防いでおきたいですね。
