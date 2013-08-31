---
layout: post
title: "Android Casual Talks #1"
date: 2013-08-31 17:44
comments: false
categories: Android
---

# はじめに

Androidの開発をしていて、

- WebとiOSとAndroidで足並み揃えるのどうするんだろう
- プラットフォーム間のUIの統一についてはどう考えたらいいんだろう
- 他のアプリではA/Bテストとかやってるのかな
- PCからスマホウェブでは全機能を移植するけど、その流れでアプリでも全機能使えるようにって会社やユーザから言われるけどどうしたらいいのか
- PCからネイティブアプリでは機能を削ってシンプルにってアプリ開発者は言うけど、削られた機能はどこにいくのか
- 多くのサービスでスマホやタブレットのUUが伸びてその分PCのPVが下がってると思うけど、スマホ時代の広告のうまい組み込み方とか売り方について話を聞きたい
- WebViewでアプリを組むとステートが複雑になったり標準的なユーザ体験を提供するのが難しい
- ブランチ管理でgit-flowを導入しようかと思ったけど、管理するコストと考えるとあれかなと思ったのでGitHub Flowでやっているけど、他社ではどうしてるのか
- 大規模でも破綻しない設計とは(「MVCを意識して書く」だと個人の技量に左右されるしスケールしない感じがする)
- 効果的なテストとメトリクス計測(取るだけじゃなくて改善するところも含めて)ってどうするのがいいんだろう
- 継続的にパフォーマンス計測をしたいけど、Jenkinsでどう実現したらいいのだろう
- レビューで叩かれるとへこむ
- デザイナーがみんなiPhoneユーザだ
- むしろエンジニアもMacとiPhoneを使っている人が多くて、Androidは現代のIEって言われる

など、悩むことが多いです。

こういう技術書にのっていない話は人に聞くのが一番早いかなと思っていますが、今までAndroid界隈では集まってこういう話をする場がなかったように思います。(僕が知らないだけであったのかもしれませんが)  
それでこの度、カジュアルに情報交換をしたいなと思って、[Android Casual Talks #1](http://atnd.org/events/41600)を開きました。

<img src="https://dl.dropboxusercontent.com/u/54255753/blog/201309/casual0.png" width="600px">

内容については「ぶっちゃけ過ぎてるんであんまりツイートしないでください」っていうのもあったので、さらっと概要だけまとめました。

# 1. クックパッドの開発環境について

<img src="https://dl.dropboxusercontent.com/u/54255753/blog/201309/casual4.jpg" width="600px">

僕はAndroid Studio + Gradleの導入とか、リポジトリ管理とか、ビルドの設定とか、人が増えてもスケールしそうな開発の話をしました。

# 2. 品質を保つための組織的な取り組みと人に依存しないテスト

メーカーの開発の品質は高いと伺っていたので、品質への取り組みについてお話いただきました。  
いわゆるウォーターフォールモデルだけど、1週間のイテレーションを回して目標値への達成度の確認と是正を行っていて、さすがにしっかり管理をしているなと思いました。  
メトリクスを細かく取っていたのも印象的でした。

# 3. グリーにJenkinsを導入して2年半でおこった事

Jenkinsの運用の話＆エモ枠としてお話いただきました。

<iframe src="http://www.slideshare.net/slideshow/embed_code/25716362" width="427" height="356" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC;border-width:1px 1px 0;margin-bottom:5px" allowfullscreen webkitallowfullscreen mozallowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="https://www.slideshare.net/okazaki/2013-0829-jenkins-for-cookpad-android" title="2013 08-29 jenkins for cookpad android" target="_blank">2013 08-29 jenkins for cookpad android</a> </strong> from <strong><a href="http://www.slideshare.net/okazaki" target="_blank">Takayuki Okazaki</a></strong> </div>

- 変えないことは大きなリスク
 - 変えないと、技術的負債激増
 - ある時点から技術的負債のために働くことになる

- 変えるのはツールではなくワークフロー
 - ツールの導入で解決 -> 仕組みの改善で解決
 - あるべき論、精神論 -> しくみ、制度化

いい話でした。詳しくはスライドを御覧ください。

# 4. injectionの基礎（android編）

イベント開催を知った師匠が話してくれました。

<img src="https://dl.dropboxusercontent.com/u/54255753/blog/201309/casual2.png" width="600px">

DIフレームワークの基礎ということで、依存性の注入や制御の反転について、あるチャットアプリの例を通して、仕様が変わっていく中でプログラムを綺麗な状態に保ちながらテストをしやすくする方法について話していただきました。

社内ではひっそりと[RoboGuice](https://github.com/roboguice/roboguice)を使っていたのですが、パフォーマンスとかで[Dagger](http://square.github.io/dagger/)の方が良いみたいな流れが最近あるので、そっちに移行したいなと思いました。

# 5. 意外と役立つ？Android Open Source Projectのすすめ

Androidアプリのデバッグ手法について話していただきました。

- アプリ開発で悩んだときはどうしますか？
 - ググる
 - Android Developersで調べる
 - 色々試す

- 上の方法で解決できないときはどうしますか？
 - 他のアプリを逆コンパイル
 - 明日考える
 - 仕様をドロップ

でも仕様を諦めるのはエンジニアとして負けた気分になる。そこで、[{OpenGrok](https://sites.google.com/site/devcollaboration/codesearch)

<img src="https://dl.dropboxusercontent.com/u/54255753/blog/201309/casual3.png" width="600px">

OpenGrokは、Androidのソースコードを簡単に見るために作られた検索エンジンで、Full Search、Definition、Symbol、File Path、Historyなど絞り込んで検索をすることができるみたいです。  
ちょっと調べ物をするのに便利そうでした。

# 6. アプリのリニューアルとその効果測定について

Android2系のデザインで作られた黒背景に白文字の「葬式UI」だったPixivアプリをリニューアルしたときの反響と効果測定についてお話いただきました。

<img src="https://dl.dropboxusercontent.com/u/54255753/blog/201309/casual5.png" width="600px">

リニューアルをしたときにはGoogle Playで☆1の嵐が吹き荒れて…

- 「最悪。アップデート前の方がいいです。」
- 「凄まじいまでの改悪、なぜこれでゴーが出たのか」

**開発者「もう許してくださいって思いました。」**

ただ定性的な意見だけではなく、定量的なデータからこのリニューアルはどうだったのかというと、操作性の向上を図ったことにより、

- ブックマークのイベント数：4.7倍
- 評価ボタンのイベント数：9.5倍

となり、順調にユーザ数も増加しているそうです。  
レビューと合わせて計測することの大切さを言っていました。

# おわりに

各社のいい話が聞けたのと、イベントを通して多くのAndroidエンジニアと知り合えたのはとても良かったです。
一方で、一通りトークが終わったあと時間が押してて「すいません、あと12分で交流してください」となってしまったのが残念でした。

このイベントの定員50人だったんですけど、150人以上の登録があって、思っていたより人が集まって驚きました。
それなりに需要があれば#2, #3...とやるかもしれないので、やりたいとか、会場を提供できるよとか、そういうのがあればまたやりたいですね。
次やるのなら交流をメインにしたいので、20分くらいのトークは2本にして、あとは5分のLTを募集してたくさんの人に発表してもらったあとに、さっき◯◯の話をしていたあの人と話そう、みたいにすると良いかなと思いました。

Androidエンジニアに幸あれ。
