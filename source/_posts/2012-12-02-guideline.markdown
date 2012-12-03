---
layout: post
title: "まさかこのままプッシュする気じゃないよね？"
date: 2012-12-02 16:17
comments: true
categories: Git
---

{% oembed https://twitter.com/r7kamura/status/275064946759454720 %}
　  
　  
　  
綺麗かどうかどころか、デバッグコードとかそのままプッシュしてしまって、後でレビューツールでdiff見ててアッてなるので書いた。  

{% codeblock .git/hooks/pre-commit lang:bash %}
#!/bin/sh

ax_code=$(find . -name *.java | xargs cat | grep -n -e TODO -e FIXME -e XXX -e DEBUG)

if [ -n "$ax_code" ]; then
  echo "$ax_code"
  echo "まさかこのままプッシュする気じゃないよね？"
fi
{% endcodeblock %}

コミットする。  

{% codeblock lang:bash %}
( *´ω｀*)  ~/develop/projects/Betterflow $ git commit
548:        // TODO: この名前はおかしいので後で変える
572:        // FIXME: 読み込む前にリサイズしないと余裕でOOMする
752:        // XXX: よくわからないけど動いてる
まさかこのままプッシュする気じゃないよね？
[master 98c0487] コメント書いた
 2 files changed, 3 insertions(+)
( *´ω｀*)  ~/develop/projects/Betterflow $
{% endcodeblock %}


でも人間は許せる生き物だから。  
