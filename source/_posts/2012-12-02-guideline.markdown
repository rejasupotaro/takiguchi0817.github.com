---
layout: post
title: "まさかこのままプッシュする気じゃないよね？"
date: 2012-12-02 16:17
comments: true
categories: Git
---

{% oembed https://twitter.com/r7kamura/status/275064946759454720 %}
　  
　  
　  
綺麗かどうかどころか、デバッグコードとかそのままプッシュしてしまって、後でレビューツールでdiff見ててアッてなるので書いた。  

{% codeblock lang:bash %}
#!/bin/sh

readonly MESSAGE="まさかこのままプッシュする気じゃないよね？"

has_ax_code=false
file_list=$(find . -name *.java)
for file in ${file_list[@]}; do
  ax_code=$(cat $file | grep -n -e TODO -e FIXME -e XXX -e DEBUG)
  if [ -n "$ax_code" ]; then
    echo $file
    echo "$ax_code"
    has_ax_code=true
  fi
done

if [ $has_ax_code ]; then
  echo $MESSAGE
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
