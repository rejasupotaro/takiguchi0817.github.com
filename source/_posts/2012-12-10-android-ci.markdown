---
layout: post
title: "JenkinsでAndroidをCIしたかった"
date: 2012-12-10 00:46
comments: false
categories: Jenkins Android
---

GitHubにあるAndroidプロジェクトにプッシュしたらJenkinsがビルドしてテストするというのを実現したかった。  

## 手順

- Jenkinsをインストールする
- Androidをantでビルドする
- JenkinsでAndroidをビルドする
- GitHubにプッシュしたらビルドを行う

ってする。

### Jenkinsをインストールする

{% codeblock %}
$ wget -q -O - http://pkg.jenkins-ci.org/debian/jenkins-ci.org.key | sudo apt-key add -
$ sudo sh -c 'echo deb http://pkg.jenkins-ci.org/debian binary/ > /etc/apt/sources.list.d/jenkins.list'
$ sudo aptitude update
$ sudo aptitude install jenkins
$ sudo service jenkins start
{% endcodeblock %}

http://yourhost:8080にアクセスするとJenkinsさん動いてるの確認できる。  
新規ジョブ作って定期実行するだけで、  

{% codeblock %}
Started by timer
Building in workspace /var/lib/jenkins/workspace/Test
[Test] $ /bin/sh -xe /tmp/hudson4214505998237671009.sh
+ echo Hello rejasupotaro
Hello rejasupotaro
Finished: SUCCESS
{% endcodeblock %}

1分毎にハローって言うかわいい。  
これだけでもカンタンにcronの管理が出来そうな感じ。  

#### Jenkinsにプラグインを入れる

Jenkinsの管理 -> プラグインの管理 -> 利用可能のタブから

- Android Emulator Plugin
- GitHub Plugin

入れる。  

### Androidをantでビルドする

まずant入れる。  

{% codeblock %}
$ sudo aptitude install ant
{% endcodeblock %}

bashrcに追記した。  

{% codeblock %}
ANDROID_HOME=/var/lib/jenkins/tools/android-sdk
PATH=$PATH:$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools
{% endcodeblock %}

この時点でAndroidのリストを見てみる。  

{% codeblock %}
$ android list target
id: 1 or "android-9"↲
     Name: Android 2.3.1
     Type: Platform
     API level: 9
     Revision: 2
     Skins: WVGA854, WQVGA400, HVGA, QVGA, WQVGA432, WVGA800 (default)
     ABIs : armeabi
{% endcodeblock %}

はい。  
プロジェクトはルートのディレクトリがあって、その下にアプリのプロジェクトとテストプロジェクトがあるっていう感じ。なので、  

{% codeblock %}
$ android update project -p ./MyAppTest --target android-9
$ android update test-project -p MyAppTest/ -m ~/develop/projects/MyProjet/MyApp
{% endcodeblock %}

設定ファイルとかが作られる。  
ant clean debugとかしてみて、モジュールが足りなかったりしたらlddとかしながら頑張って入れる。  
モジュールが足りないとadb: No such file or directoryとかなる。  

[Initializing a Build Environment](http://source.android.com/source/initializing.html)  
ここを見ればだいたい解決する(はず)。  

### JenkinsでAndroidをビルドする

Jenkinsで新規ジョブを作成して実行する。  

{% codeblock %}
emulator: ERROR: Could not load OpenGLES emulation library: libX11.so.6 #=> 試しにビルドしたらコケる

$ sudo aptitude install libx11-6:i386 #=> 試しに入れてみる

Could not load OpenGLES emulation library #=>コケる
Failed to load libEGL_translator.so #=> なんか読めてない
{% endcodeblock %}

でlibEGLを入れるとまたlibX11.soがないと言われ、libX11.soを入れると（ryの無限ループになった。  
うーん。わからん。  

### GitHubにプッシュしたらビルドを行う

GitHubに鍵を登録する。  

{% codeblock %}
$ sudo -u jenkins -H ssh-keygen -t rsa -C jenkins@rejasupotaro.com
[sudo] password for rejasupotaro:
Generating public/private rsa key pair.
Enter file in which to save the key (/var/lib/jenkins/.ssh/id_rsa):
Created directory '/var/lib/jenkins/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /var/lib/jenkins/.ssh/id_rsa.
Your public key has been saved in /var/lib/jenkins/.ssh/id_rsa.pub.
{% endcodeblock %}

鍵をcatしてコピペしてGitHubに貼る。  
　  
　  
　  
エミュレーターが動くようになったらHooksに登録してビルドするようにしたい。  
