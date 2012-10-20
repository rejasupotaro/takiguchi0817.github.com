---
layout: post
title: アプリのデータはどこに保存されるの
date: 2012-10-09 22:02
comments: true
categories: 
---
## 内部ストレージ /data/data/package.name
- cache
- databases
- files
- lib
- shared_prefs

### cache
    getCacheDir()
- com.android.renderscript.cache  
勝手に作られる。RenderScript使ったことないので良く知らない。
- webviewCacheChromium  
WebViewのキャッシュ

### files
    openFileOutput()
アプリによって使ったり。

### databases
    getWritableDatabase()
- webview.db  
メタデータとか認証情報とかパスワードとか。
- webviewCookiesChromium.db  
メタデータとかクッキーとか。

### lib
使ってたら入る。

### shared_prefs
    getSharedPreferences()
databasesに入れるほどでもないけど、filesに置きたくないデータを入れてる。  
最後にアプリを開いた時間とか、データをリフレッシュした時間とか、タブレットかとか。

## 外部ストレージ /mnt/sdcard/Android/data/package.name
    getExternalFilesDir()
見えても良いデータを置いてる。  
画像とか動画とかのキャッシュにされてることが多い気がする。

## その他
それ以外にもパス指定すればファイル作れる。  
各社、セキュアな情報はプリファレンスに入れてるか、暗号化してそこらに置いてる。  
プリファレンスの中は平文で保存されていることが多いけど、実際に端末を盗んでroot化しないの見れないので、良いと思う。  
アカウントマネージャを使っているアプリはSDカードに移動できないようにしている。再マウント時にアカウントを復帰させるのあんまりやらないのかな。 
