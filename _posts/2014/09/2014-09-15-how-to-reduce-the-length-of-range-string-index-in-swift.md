---
layout: post
title: "Swift 中如何缩短 Range<String.Index> 的长度"
description: 
headline: 
date:   2014-09-15 13:55
category: 机械境
tags: []
image: 
  feature: 
comments: true
mathjax: 
---

今天写一个小项目的时候遇到一个这样的需求，截取一个类似 `Michael: Hello, World!` 的字符串中的名字部分，当然，代码如下： 
 
<!-- more -->

{% highlight swift %}
// Note: You have to import Foundation in Playground  
let rawString = "Michael: Hello, World!"  
let range = rawString.rangeOfString("^\\w{0,10}:", options: .RegularExpressionSearch)  

if (range != nil) {  
    println(rawString.substringWithRange(range!))  
}  
// Output:  
// Michael:  
{% endhighlight %}

那么，这个 `range` 的长度如何减少1呢？  
这里的 `range` 的类型是 `Range<String.Index>` ，它无法直接减1，下面的做法是错误的：  
{% highlight swift %} 
range! - 1  
range!.endIndex - 1  
{% endhighlight %}
不过， `Range<SomeTypes>.startIndex` 或者 `Range<SomeTypes>.endIndex` 有 `predecessor()` 和 `successor()` 方法，分别为 向前 和 向后一位。这样，我们的需求就可以实现了：  
{% highlight swift %}
if (range != nil) {  
    let subRange = Range<String.Index>(start: range!.startIndex, end: range!.endIndex.predecessor())  
    // 这一行可以简写为：  
    let subRange2 = range!.startIndex ..< range!.endIndex.predecessor()  
    println(rawString!.substringWithRange(subRange))  
}  
// Output:  
// Michael  
{% endhighlight %}  
每次都需要调用 `predecessor()` 和 `successor()` 挺麻烦，但是这种需求可以通过重载操作符实现类似 `range!.endIndex - 1` 的效果，这个坑等以后再填吧。

附： [http://stackoverflow.com/questions/24044851/how-do-you-use-string-substringwithrange-or-how-do-ranges-work-in-swift](http://stackoverflow.com/questions/24044851/how-do-you-use-string-substringwithrange-or-how-do-ranges-work-in-swift)