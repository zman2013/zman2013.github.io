---
title: A stream is just an array in time
tags:
  - backpressure
id: '335'
categories:
  - - Java
  - - pull-stream
date: 2019-11-11 23:27:36
---

`There is a deeper, platonic abstraction, where a streams is just an array in time, instead of in space. And all the various streaming "abstractions" are just crude implementations of this abstract idea.` _\-- pull-stream_   

如果Stream可以分版本，我理解目前是2.1。 
Stream 1.0 是一个有限集合，消费者通过foreach遍历集合内部元素。 
Stream 2.0 是一个无限集合，集合内部的元素随着时间不断产生，由生产者不断推向消费者，消费者被动接收。 
Stream 2.1 解决了2.0实际使用中出现的backpressure和error propagation问题，元素传递的决策权交由消费者进行控制。 Stream 2.1有两个组织分别提出了不同的解决思路：reactive-streams、pull-stream。RxJava采用的是reactive-streams，JDK9的Flow基于reactive-streams。IPFS采用了pull-stream。 reactive-streams提出了一套规范，并且声明主要目标就是解决backpressure问题，核心思路是Subscriber声明自己能承受的请求数量n，Publisher根据n决定推送元素的个数。 

**pull-stream实在是只有天才的大脑才能想出来，两个函数搞定~** 主要想介绍这两个函数，因为实在憋不住不分享出来。

```js
var n = 5;

// random is a source 5 of random numbers.
function random (end, cb) {
  if(end) return cb(end)
  // only read n times, then stop.
  if(0 > --n) return cb(true)
  cb(null, Math.random())
}

// logger reads a source and logs it.
function logger (read) {
  read(null, function next(end, data) {
    if(end === true) return
    if(end) throw end

    console.log(data)
    read(null, next)
  })
}

logger(random) //"pipe" the streams.
```

上面两个函数random为source（publisher/observer/producer），logger为sink（subscriber/consumer），实现了logger主动从random中读取一个随机数，并且”是否读取“、”何时终止“都由logger控制，例子中logger的逻辑会一直读取到random最后一个元素。

**翻译成Java**

```java
private int n = 5;
...
// 每次读取，提供一个随机数
public void read(boolean end, BiConsumer<Boolean, Integer> cb){
    if(end){  // 结束
        cb.accept(end, null);
        return;
    }

    if( n-- < 0 ){  // 读完所有元素
        cb.accept(true, null);
        return;
    }

    cb.accept(false, new Random().nextInt());  // 回调
}

// 从read中读取并打印所有数据
public void logger(BiConsumer<Boolean, BiConsumer<Boolean, Integer>> read){

    Holder<BiConsumer<Boolean, Integer>> holder = new Holder<>();
    holder.value  = (end, data) -> {
        if( end )   return;

        System.out.println(data);
        read.accept(false, holder.value);  // 递归
    };

    read.accept(false, holder.value);
}
...
logger(this::read);  // pipe
```

基于pull-stream思想，可以很简洁的写出rxJava和jdk-lambda的核心功能，比如.map

```java
// 所有元素\*3
public BiConsumer<Boolean, BiConsumer<Boolean, Integer>> triple(
            BiConsumer<Boolean, BiConsumer<Boolean, Integer>> readable) {

    return (end, f) ->
        readable.accept(end, (isEnd, data) ->
            f.accept(isEnd, isEnd ? null :3 \* data) // 所有元素\*3
        );
}
...
logger( this.triple(this::read) );
```

上面的调用可读性已经比较差了，优化一下：

```java
public class Pull{
    // 从source读取数据，经through转换后，由sink存储
    public static void pull(BiConsumer<Boolean, BiConsumer<Boolean, Integer>> source,
                     Function<BiConsumer<Boolean, BiConsumer<Boolean, Integer>>,
                             BiConsumer<Boolean, BiConsumer<Boolean, Integer>>> through,
                     Consumer<BiConsumer<Boolean, BiConsumer<Boolean, Integer>>> sink
                     ){
        sink.accept(through.apply(source));
    }
}
...
// 随机数 -> \* 3 -> 输出到stdio
Pull.pull(this::read, this::triple, this::logger);
```

这样调用起来就便捷多了，在接口设计上一定站在使用方的角度去思考。

  另外，你很可能遇到reactivex.io，reactivex目标是提供便捷的reactive API。reactive-streams != ReactiveX，rx内的工具主要是2.0规范，比如rxjs没有实现backpressure，而rxjava遵从reactive-streams实现了backpressure。 

**官方** 
https://pull-stream.github.io/ 
http://www.reactive-streams.org/   

