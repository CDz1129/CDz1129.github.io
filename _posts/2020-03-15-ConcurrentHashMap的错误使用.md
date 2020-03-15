---
layout:     post
title:      ConcurrentHashMap的错误使用
subtitle:   线程安全类，不恰当使用也会线程不安全
date:       2020-03-15
author:     CDz
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - 并发
---


场景，多线程统计文本单词。

使用`ConcurrentHashMap`来做统计器。

其中问题在于，因为是多线程情况，所以累加统计的过程是并发的。

那么使用 `ConcurrentHashMap`，统计逻辑如果是。

`ConcurrentHashMap map  = new ConcurrentHashMap<String,Integer>();`

---- 
下面多线程运行，会出现BUG

```java
ConcurrentHashMap map  = new ConcurrentHashMap<String,Integer>();

//下面多线程运行，会出现BUG
Integer value= map.get(word);
if (value==null){
    map.put(word,1);
}else {
    map.put(word,value++);
}computeIfAbsent
```

虽然`ConcurrentHashMap`是线程安全的，但是原子只是保证了单个操作的原子性例如`map.get()`和`map.put()`方法。

但是并不能保证，两个一起就是原子性。

如何解决？
ConcurrentHashMap提供了两个方法`computeIfAbsent`和`computeIfPresent`

* `computeIfAbsent`计算如果不存在
* `computeIfPresent`计算如果存在

使用方法

```java
ConcurrentHashMap<String,Integer> map  = new ConcurrentHashMap<>();
Integer a1 = map.computeIfAbsent("a", (key) -> 1+1);
Integer a2 = map.computeIfPresent("a", (key,value) -> map.get(key)+value);
System.out.println(a1);
System.out.println(a2);
——
2
4
```

可以看出：

* `computeIfAbsent`是如果key不存在，存入一个值（第二个方法）
	* 并返回存入的值
* `computeIfPresent`是如果key存在，key value同时给出，然后自行计算公式，
	* 并返回，计算后的值

这样写，还是有问题。多线程统计的`a++`问题，这个解决办法也很简单，使用原子类来进行。

```java
ConcurrentHashMap<String,LongAdder> map  = new ConcurrentHashMap<>();
LongAdder a1 = map.computeIfAbsent("a", (key) ->new LongAdder());
a1.increment();
```

这样的方式才会，完全避免多线程，统计词汇出现并发问题。

