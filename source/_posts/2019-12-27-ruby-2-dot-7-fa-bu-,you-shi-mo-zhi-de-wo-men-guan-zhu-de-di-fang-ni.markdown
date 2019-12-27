---
layout: post
title: "ruby 2.7 新發布，新功能說明"
date: 2019-12-27 13:37:22 +0800
comments: true
categories: ["ruby", "release", "new feature", "future"]
---

就如往常，Matz 選在聖誕節發布 Ruby 2.7 作為聖誕禮物，那麼這個新版本有什麼特別的呢？又有什麼是未來整個 Ruby 社群的方向呢？

Ruby 2.7 的新功能列表

1. [IRB 多行模式與即時文件查閱](#new-irb)
1. [Array#intersection](#array-intersection)
1. [Enumerable#tally](#tally)
1. [Enumerable#filter_map](#filter-map)
1. [Enumerable#produce](#produce)
1. [Block 預設參數名稱(實驗性)](#default-param)
1. [Pattern Matching(實驗性)](#pattern-matching)
1. [Compaction GC](#compaction-gc)
1. [Keyword Argument](#keyword-argument)


<!-- more -->

根據 Matz 的說法，本次的版本為最後的 2.x 版本，下個版本將會直接跳到從 2016 開始大家就在引頸期盼的 [Ruby 3.0](https://blog.heroku.com/ruby-3-by-3)！

下面是 Matz 在 RubyConf 2019 上針對 Ruby 2.7 的說明。
<iframe width="560" height="315" src="https://www.youtube.com/embed/2g9R7PUCEXo" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

下面，我們開始逐一說明本次 Ruby 2.7 的新功能。

## <a name="new-irb">IRB 多行模式與即時文件查閱</a>

<video autoplay="autoplay" controls="controls" muted="muted" poster="https://cache.ruby-lang.org/pub/media/irb-reline-screenshot.png" width="576" height="259">
  <source src="https://cache.ruby-lang.org/pub/media/irb_improved_with_key_take3.mp4" type="video/mp4">
</video>

新版 irb 支援了多行編輯的功能，只要按 alt-enter 即可換行繼續編輯，我們可以從上面的影片知道，按方向鍵往上可以將整個 block 重新叫回來編輯，另外，按一次 tab 會提示可用的關鍵字，連續按多次 tab 可以進入文件模式，在互動模式下快速查找 rdoc，之後開發者可以在不用離開互動模式的狀況下進行文件的查詢，也可以鼓勵開發者在開發程式的時候編寫 rdoc 。

##<a name="array-intersection">Array#intersection</a>

這並不完全算是一個新功能，這是一個簡寫，在 Ruby 2.6 引進一個兩個新功能：[`union`](https://ruby-doc.org/core-2.6/Array.html#union-method) 與 [`difference`](https://ruby-doc.org/core-2.6/Array.html#method-i-difference)，在 Ruby 2.7 中更進一步將其簡化了，使用方法如下：

```ruby
a = [1, 2, 3]
b = [1, 2]
p a & b
# [1, 2]
p a | b
# [1, 2, 3]
p a - b
# [3]
p a.intersection(b)
# [1, 2]
p a.union(b)
# [1, 2, 3]
p a.difference(b)
# [3]
```

##<a name="tally">Enumerable#tally</a>

回傳一個 Hash ，其中為給定之 Array 中內容物與其個數。


```ruby
[a, a, a, b, b].tally
```

會回傳

```ruby
{"a"=>3, "b"=>2}
```

## <a name="filter-map">Enumerable#filter_map</a>

`filter_map` method 想要將 `select` 與 `map` 結合，從字面上，我們可以知道，這個 method 會回傳一個經過條件選擇的新 Array。

在過去我們如果要做到相同的效果的話，一般都需要使用 chain method 的方式來達到：

```ruby
(1..8).select{ |x| x % 2  == 0 }.map{ |x| x ** 2 }
```

在 Ruby 2.7 中我們可以這樣做

```ruby
(1..8).filter_map{ |x| x ** 2 if x % 2 == 0 }
```

## <a name="produce">Enumerable#produce</a>

這個功能需要一點點想想力才有辦法理解，讓我們先來看一下此功能的 [proposal](https://bugs.ruby-lang.org/issues/14781):

> “This method produces an infinite sequence where each next element is calculated by applying the block to the previous element.” 

沒錯，這個 method 理論上會回傳一個無限長的 Array，讓我看看下面的範例：

```ruby
Enumerable.produce(1){ |x| x * 2 }.take(5)
# [1, 2, 4, 8, 16]
```

這個 method 長這樣的：`Enumerator.produce(initial, &block)`，預設上 initial 即為 1 ，我們亦可不傳入值，注意到我們後面的 block 為需要產生這個 Enumerable 的條件，而最後不管我們有沒有給定 `take(5)` 在記憶體裡面都是一個無限長的大小，只是最後回傳的時候給出最前面五筆，這邊需要特別注意使用，可能在大型產品上會有記憶體管理方面的問題產生。

## <a name="default-param">Block 預設參數名稱(實驗性)</a>

Ruby 從 Scala, Closure, Groovy 借鏡了 Block 的預設參數名稱，規則很簡單：第一個參數的預設名稱即為 `_1`、第二個參數即為 `_2` ⋯以此類推，下面是一個簡單的範例：

```ruby
[1, 2, 3].map { _1 * 2 }
# [2, 4, 6]
# same as [1, 2, 3].map { |x| x * 2 }
```
你仍舊可以在一般的區域變數使用「底線加上數字」做為變數名稱，但是 compiler 會跳出警告，告訴你這樣是不好的行為，請盡量不要這樣做。

## <a name="pattern-matching">Pattern Matching(實驗性)</a>

在演講中，Matz 提到，像是 Pattern Matching 這樣的 Functional Programming 行為，對於 Ruby 來說需要一個全局的重新思考，或許目前的行為在之後會與目前通通不同。在那之前，讓我們看看目前 Ruby 2.7 所提供的 Pattern Matching 使用方法。

Pattern Matching 會回傳一個布林值，當 Pattern 相同時，則會回傳 true 反之亦然。下面是幾個簡單的範例：

```ruby
# Array
[1,2,3] in [a,b,c] # true
[1,2,3] in [a]     # false

# Hash
{ a: 1, b: 2, c: [] } in { a: a, b: b, c: [] }
# true
p a
# 1

# 合併
case JSON.parse(request.body.read)
in {name: "Alice", age: age }
  puts age
in
  puts "no Alice"
end
```

## <a name="compaction-gc">Compaction GC</a>

在 Ruby 2.7 中引入了記憶體壓縮垃圾蒐集演算法，意即將碎片化的記憶體重組的垃圾蒐集演算法，在一些多執行緒程式執行的過程中，可能會造成記憶體使用上的碎片化，長久下來會造成高記憶體使用與效能的下降。

新的 GC method `GC.compact` 期望能夠將 heap 的碎片化降到最低。

詳細內容請見原 [proposal](https://bugs.ruby-lang.org/issues/15626)

## <a name="keyword-argument">Keyword Argument</a>

**這是 Ruby 2.7 與 Ruby 2.6 以前最重要的不同！**<br/>
**未來 Ruby 3 之後將不向下相容 Ruby 2.6 的 Keyword Argument**

Ruby 2.6 以前的 keyword argument 行為如下

```ruby
def m(a, key:10)
  p [a, key]
end
m(key: 5) #=> [{:key => 5}, 10]

def foo(opt={}, key:10)
  p [opt, key]
end
m(key: 42) #=> [{}, 42]
```
這真的是非常地令人崩潰的行為，上面的 function 呼叫行為應該是以下這幾種問題的綜合：

1. Wrong Number of Arguments
1. Combination of Optional/ Rest Argument
1. Combination with Hash Argument

但是 Ruby VM 卻沒有吐出錯誤訊息，所以在 Ruby 2.7 開始，這樣的行為將會被拋棄，在 Ruby 2.7 時只會吐出警告訊息，但是在 Ruby 3 之後就不能執行了，所以 Library 的開發者，如果有使用到這樣的行為，請儘速更新您的軟體，以符合未來的 Ruby 版本。

## Reference

1. Ruby 3x3: Matz, Koichi, and Tenderlove on the future of Ruby Performance, https://blog.heroku.com/ruby-3-by-3
1. Ruby 2.7.0 Released, https://www.ruby-lang.org/en/news/2019/12/25/ruby-2-7-0-released/
1. Ruby Progress Report by Yukihiro Matsumoto(Matz), https://www.youtube.com/watch?v=2g9R7PUCEXo
1. New Features, Methods & Improvements in Ruby 2.7, https://www.rubyguides.com/2019/12/ruby-2-7-new-features
1. Enumerator.generate proposal, https://bugs.ruby-lang.org/issues/14781
1. Ruby 2.7 adds Enumerator#produce, https://blog.saeloun.com/2019/11/27/ruby-2-7-enumerator-produce.html
1. Manual Compaction for MRI's GC (`GC.compact`), https://bugs.ruby-lang.org/issues/15626
