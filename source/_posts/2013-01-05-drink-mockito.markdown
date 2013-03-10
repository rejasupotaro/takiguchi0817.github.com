---
layout: post
title: "モヒートの飲み方"
date: 2013-01-05 12:41
comments: true
categories: Android Mockito
---

# mojito
![](http://dl.dropbox.com/u/54255753/blog/201301/mojito.jpg)
モヒートは、キューバ・ハバナ発祥のカクテルの一つ。
ラムをベースにソーダ、ライム、砂糖、ミントを加えたもの。
ミントとソーダの清涼感が暑い夏にぴったりと、「夏と言えば」の定番カクテル。  
アーネスト・ヘミングウェイが好んで飲んでいた話は有名である。

# [mockito](http://code.google.com/p/mockito/)
![](http://dl.dropbox.com/u/54255753/blog/201301/mockito.jpg)
Mockito is a mocking framework that tastes really good.  

ジャバのモックライブラリ。
モックライブラリは他にもいろいろありますが[EasyMockと比べても](http://code.google.com/p/mockito/wiki/MockitoVSEasyMock)、mockitoの方が簡潔に書けそう。
というわけで、モヒートを飲んだあとのようにスカッとモックしたいので、どんなものか調べてみた。
　  
　  
## 導入

mockito本体と、Androidで動かすためにdexmakerとdexmaker-mockitoのjarをlibsに追加する。

- [mockitoのダウンロードはここから](http://code.google.com/p/mockito/downloads/list)
- [dexmakerのダウンロードはここから](http://code.google.com/p/dexmaker/downloads/list)

またEclipseの設定に追加しておくとContent Assistが効くようになって良い。
![](http://dl.dropbox.com/u/54255753/blog/201301/static_import.png)
　  
　  
## mockito使い方
- [Mockito API](http://docs.mockito.googlecode.com/hg/org/mockito/Mockito.html)

良く使いそうなものだけをピックアップした。

{% codeblock lang:java %}
public void testDrinkMockito() throws Exception {
    // mockでモックオブジェクトを作成する
    List<String> mockedList = mock(ArrayList.class);
    mockedList.add("foo");
    mockedList.clear();
 
    // whenで引数ごとの返り値を決められる
    when(mockedList.get(0)).thenReturn("first");
    when(mockedList.get(1)).thenThrow(new RuntimeException());
    assertEquals("first", mockedList.get(0));
    try {
        mockedList.get(1);
        fail("RuntimeExceptionがthrowされていない");
    } catch (RuntimeException e) {
    }
 
    // whenではanyInt()やanyString()やanyMap()のような指定の仕方もできる
    when(mockedList.get(anyInt())).thenReturn("element");
    assertEquals("element", mockedList.get(999));
 
    // verifyでモックオブジェクトが対象のメソッドを実行したか確認できる
    verify(mockedList).add("foo");
    verify(mockedList).clear();
 
    // verifyはメソッドの実行回数も確認することができる
    mockedList.add("bar");
    mockedList.add("bar");
    // mockedList.add("bar")が2回呼ばれたことを確認する
    verify(mockedList, times(2)).add("bar");
    // mockedList.add("bar")は1回も呼ばれなかったことを確認する
    verify(mockedList, never()).add("baz");
 
    // verifyはメソッドの実行順序も確認することができる
    mockedList.add("baz");
    mockedList.clear();
    InOrder inOrder = inOrder(mockedList);
    inOrder.verify(mockedList).add("baz");
    inOrder.verify(mockedList).clear();
 
    // spyで部分的にメソッドを置き換えることもできる
    List<String> spy = Mockito.spy(new ArrayList<String>());
    doReturn(100).when(spy).size();
    spy.add("foo"); // 実際のオブジェクトのメソッド呼び出し
    spy.size(); // => 100
}
{% endcodeblock %}

とりあえずmock、when、verify、spyだけ覚えておけば大丈夫そう。
テスト対象のオブジェクトを継承してモックオブジェクトを作るのに比べてるとだいぶ楽だ。
　  
　  
## Mockitoによるビヘイビア駆動開発

### [MockitoBDD API](http://docs.mockito.googlecode.com/hg/org/mockito/BDDMockito.html)

さらにMockitoBDDというもあって、それを使えばビヘイビアを先に記述してから開発するスペックファーストな実装ができる。

{% codeblock lang:java %}
public class Seller {
    // should implement
    public Bread askForBread() { return null; }
}

class Shop {
    private Seller mSeller;

    Shop(Seller seller) {
        mSeller = seller;
    }

    // should implement
    public Goods buyBread() { return null; }
}
{% endcodeblock %}

{% codeblock lang:java %}
public void testShouldBuyBread() throws Exception {
    Seller seller = mock(Seller.class);
    Shop shop = new Shop(seller);
    
    //given  
    given(seller.askForBread()).willReturn(new Bread());

    //when
    Goods goods = shop.buyBread();

    //then
    assertThat(goods, containBread());
}
{% endcodeblock %}
