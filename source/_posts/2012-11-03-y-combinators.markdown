---
layout: post
title: "文化の日"
date: 2012-11-03 14:15
comments: false
categories: 
---

## わかりやすかった  
[おとうさんにもわかるYコンビネータ！(絵解き解説編)](http://d.hatena.ne.jp/r-west/20090417/1239972722)  

## かわいみコート買った
![かわいみコート買った](http://dl.dropbox.com/u/54255753/blog/201211/1351951130405-1.jpg)  

## Yコンビネータみたいなの書いてた  

### case of ruby
    puts lambda { |y|
      y.call(lambda { |f|
        lambda { |n|
          n <= 2 ? 1 : f.call(n - 1) + f.call(n - 2)
        }
      }).call(10)
    }.call(lambda { |f|
      lambda { |g|
        lambda { |m|
          f.call(g.call(g)).call(m)
        }
      }.call(lambda { |g|
        lambda { |m|
          f.call(g.call(g)).call(m)
        }
      })
    })

### case of coffeescript
    console.log ((Y) -> Y((f) -> (n) -> if (n <= 2) then 1 else f(n - 1) + f(n - 2)
    )(10)) (f) -> ((g) -> (m) -> f(g g) m) (g) -> (m) -> f(g g) m

## 夕食
![鶏肉焼いてパスタ茹でた](http://dl.dropbox.com/u/54255753/blog/201211/1351934736127.jpg)  
