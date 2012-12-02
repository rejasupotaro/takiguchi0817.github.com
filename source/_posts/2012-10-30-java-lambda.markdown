---
layout: post
title: "ジャバ8のラムダでYコンビネータが書きたかった…"
date: 2012-10-30 04:50
comments: false
categories: Java Lambda
---

今日のSICP読書会でLazyKanがちょっとした話題になった。  
![LazyKan](http://dl.dropbox.com/u/54255753/blog/201210/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202012-10-30%205.19.46.png)  
画像から狂気を感じましたが、しかしYコンビネータ、よく耳にするけどどういうものか分かってなかった。  

ジャバ8でラムダ入るらしいので、書いてみたら何か分かるかもしれないと思って、  
amachanさんの[Yコンビネータってなに？](http://d.hatena.ne.jp/amachang/20080124/1201199469)  
を見ながらまずふつうに実装して、それからラムダで書いてみることにしました。  

まだ途中だけど、  

{% codeblock lang:java %}
System.out.println((new LambdaInterface() {
  public GInterface exec(FInterface f) {
    return new GInterface() {
      public int exec(int m) {
        return f.exec(this).fib(m);
      }
    };
  }
}).execute(new FInterface() {
  public FibInterface exec(GInterface g) {
    return new FibInterface() {
      public int fib(int n) {
        return (n <= 2) ? 1 : fib(n - 1) + fib(n - 2);
      }
    };
  }
}).exec(10));
{% endcodeblock %}

計算は出来ているが、この時点でかなりつらい。  
というか書いてて思ったけどこれ実装してもラムダにしようと思ったら、  

{% codeblock lang:java %}
GInterface g = (int m) -> return f.exec(this).fib(m);
{% endcodeblock %}

ってイコール使っちゃうからだめだし、でももしかしたらと思って  

{% codeblock lang:java %}
return (GInterface) ((int m) -> return f.exec(this).fib(m);
{% endcodeblock %}

って書いたけどやっぱり動かなかったし、無名関数の数だけ関数型インタフェースを用意しないといけないし、ジャバ8のラムダってふつうの糖衣構文だなって思いました(公式にもそう書いてあるのだけど)。  

あとジャバ9でプリミティブ型がなくなるのを今日知りました。
