---
layout: post
title: "JavaScriptの名前空間"
date: 2012-11-16 01:28
comments: false
categories: JavaScript Namespace.js
---

## 大規模JavaScript案件

僕はあまりJavaScript書かないけど、大規模JavaScript案件すると変数とかかぶって滅ぶらしい。  
[Namespace.js](https://github.com/hirokidaichi/namespace-js)とかいうの使うとパッケージ的なもの作れるらしいと聞いた。  

## 使い方

{% codeblock lang:javascript %}
Namespace("sample.widget") // 名前空間を定義
.use("brook *")            // requireとかimport的ななにか
.use("brook.model *")
.use("brook.util *")
.define(function(ns) {
  // この中にメソッドとか書く
  ns.provide({
    // オブジェクト入れる
  })
});
{% endcodeblock %}

はーよくわからん。  

## [Namespace.js](https://github.com/hirokidaichi/namespace-js/blob/master/src/namespace.js)読んでみた

#### 一番最初のmerge関数  

{% codeblock lang:javascript %}
var merge = function(target, source) {
  for (var p in source) {
    if (source.hasOwnProperty(p)) target[p] = source[p];
  }
};
{% endcodeblock %}

ターゲットにソースのプロパティが追加される。  

#### 次にProcedureの定義  

{% codeblock lang:javascript %}
var Procedure = function _Private_Class_Of_Proc() {
  merge(this, {
    state: {},
    steps: [],
    _status: 'init'
  });
});
{% endcodeblock %}

windowオブジェクト + state + steps みたいな感じだ。そのあとnext、isRunning、enqueue、dequeue、call、_invokeが追加されてる。  
処理的なものsteps配列に追加されて、実行状態が init -> running -> finished と変化するようだ。  

#### 次にNamespaceObjectの定義  

{% codeblock lang:javascript %}
var NamespaceObject = function _Private_Class_Of_NamespaceObject() {
  merge(this, {
    stash: { CURRENT_NAMESPACE: fqn },
    fqn: fqn,
    proc: createProcedure()
  })
};
{% endcodeblock %}

windowオブジェクト + fqn + Procedure みたいな感じ。そのあとプロパティに色々追加されてる。  

#### 次、NamespaceObjectfactoryの定義  

{% codeblock lang:javascript %}
var NamespaceObjectFactory = (function() {
    var cache = {};
    return {
        create :function(fqn){
            _assertValidFQN(fqn);
            return (cache[fqn] || (cache[fqn] = new NamespaceObject(fqn)));
        }
    };
})();
{% endcodeblock %}

fqnを引数に取って、NamespaceObjectを生成してる。cacheがクロージャなので、同じfqnからは同じNamespaceObjectを返すようになってる。  
NamespaceObjectはファクトリーで生成するっぽい。  

#### 最後、NamespaceDefinition

{% codeblock lang:javascript %}
var NamespaceDefinition = function _Private_Class_Of_NamespaceDefinition(nsObj) {
    merge(this, {
        namespaceObject: nsObj,
        requires       : [],
        useList        : [],
        stash          : {},
        defineCallback : undefined
    });
    var _self = this;
    nsObj.enqueue(function($c){ _self.apply($c); });
};
{% endcodeblock %}

なんというか、NamespaceObjectを渡してuse的な、require的な機能を追加している。useするとファクトリーが適切なNamespaceObjectを取得して、requiresに貯める感じだ。  
あとdefine、こいつに関数を渡してやると色々マージされたnsを引数にコールバックしてくれる。  
渡してやる関数の中でns.provideにオブジェクト渡すとNamespaceObjectにマージされて、いわゆるパブリック状態になるっぽい。  

#### returnされるのは  

{% codeblock lang:javascript %}
var createNamespace = function(fqn){
    return new NamespaceDefinition(
        NamespaceObjectFactory.create(fqn || 'main')
    );
};
merge(createNamespace, {
    'Object'  : NamespaceObjectFactory,
    Definition: NamespaceDefinition,
    Proc      : createProcedure
});
return createNamespace;
{% endcodeblock %}

createNamespaceで、fqnを渡してやってNamespace受け取って、useしたりdefineしたりするみたい。  

眠たいので流し読みしたけど、なんとなく雰囲気だけわかった。  
