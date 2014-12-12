---
layout: post
title: Promise 使用心得
date: 2014-10-29 23:23
comments: true
author: Jero
tags: 
    - 前端 
    - js
---

#Promise 使用心得


Promise 是什么？到底能干嘛用？为什么要搞个 Promise 的概念？
*callback* 够用了，所以你不需要了解 *Promise* ？


## 曾经遇到的坑
### 场景一 callback hell
最近在使用 nodejs 做后台开发，涉及查询数据库的操作，代码逻辑如下：
```js
var a = get a from TableA; // 从 A 表拿到 a
var b = get b from TableB where id = a; // 根据 a 来从 B 表查 b;
var c = get c from TableB where id = b; // 根据 b 来从 C 表查 c;
....
```
ok，先别吵着数据库设计到底合不合理，这种场景实际开发中肯定能遇到，本来没事，但是用 nodejs 开发……数据查询是异步的，于是这样：
```js
getAFromTableA(function(a) {
    getBFromTableBByA(function(b) {
        getCFromTableBByB(function(c) {
            .....
        }); 
    });
});
```
这就是 **callback hell** 的感觉，实际开发中，你第二天完全不知道前一天干了什么！
### 场景二 并行之伤
上面的还只是串行的，也就是一步一步执行，我们再看看并行的。
想象个表单验证的场景。
普通的验证不多说，我说的是**异步验证**，假如 3 个字段同时进行异步验证，你会是什么逻辑？
先验证一个，请求完成再验证另外一个？*——这就是上面的情况。*
还是同时发出三个请求？这样的话我们又如何保证三个请求**都**完成了？更坑爹的是，假如其中一个请求出错了又怎么办？
好吧，有人说他不做这样复杂的逻辑，我们继续。
### 场景三 为毛事件不起作用！
如果你对 UI 开发足够熟悉的话，就应该了解**事件**的重要性，比如 `beforeClick` 和 `afterClick` 这类交互事件就非常常见。
假设 `created` 事件在 UI 渲染完成后立即执行。（有些组件会修改 DOM 结构，为了得到最新的 DOM ，这个事件其实很有必要。）代码如下：
```js
var c = new Component();
c.on('created', function() {
    // Todo...
});
```
你会发现上面绑定的函数多半不会执行，因为绑定事件之前， `created` 已经 `trigger` 过了。
相似的情况其实很多，比如给 `<img />` 绑定 `load` 事件，有时发现不执行，因为已经 `loaded` 了。

------
上面三个场景很能说明问题了：
* 能不疯狂的嵌套 `callback`  吗？
* 能简单优雅地处理并行的多个异步操作吗？
* 能不能给一个承诺（Promise），无论什么情况，承诺一定会兑现？

Promise 能给你答案。

## 什么是 Promise
Promise 是一个抽象，它的定义：

>promise是对异步编程的一种抽象。它是一个代理对象，代表一个必须进行异步处理的函数返回的值或抛出的异常。 – Kris Kowal on JSJ

Promise 有两个特点：
*   一个 Promise 只能成功或失败一次，并且状态无法改变（不能从成功变为失败，反之亦然）
*   如果一个 Promise 成功或者失败之后，你为其添加针对成功/失败的回调，则相应的回调函数会立即执行

注意第二点，如果用 promise ，就不会存在事件不执行的坑爹事了，你无需再担心事件发生的时间点，而只需对其做出响应。
回头看场景二，场景 created 是 promise 实现：
```js
var c = new Component();
c.created.then(function () {}); //再也不用担心不执行了
```


## callback 与 promise
其实上面的*场景一*已经足够说服大部分“回调党”，有时候回调函数不是那么好控制。
回调函数能编写简单的异步代码，但同时也丧失了控制流、异常处理和函数语义，尤其是当代码逻辑分散你在多个文件中，这个时候 callback 完全没办法，你只能引入额外的逻辑。
```js
// a.js
fs.readFile('path', function (err, content) {
    // todo something width content
});

// b.js 中也想要使用 content ，怎搞？把 content 存到全局变量？绑定事件？
```
如果用 Promise 则可以轻松搞定：
```js
// a.js
exports.promise = readFileReturnPromise('path').then(function (content) {
    // todo something width content
});
// b.js
require('/a').promise.then(function (content) {
    // todo something with content again
});
```
再看一种情况，假如 *a.js* 对 `content` 进行了加工，*b.js* 想使用加工后的 `content` ，又怎么办？
```js
// a.js
exports.promise = readFileReturnPromise('path').then(function (content) {
    // todo something width content
    return content; // 这里返回加工后的 content ，那么下一个 `then` 中的 content 就只加工后的数据了
});
// b.js
require('/a').promise.then(function (content) {
    // todo something with content again
});
```


## 值的处理

* 当回调返回非 promise 类型的值时，这个值会被传递到下一个 then ；
* 当回调返回一个 promise 时，那么下一个 then 会等到这个 promise 完成；

第一种情况上面已经说过了，第二种情况其实就是异步操作的串行。
我们再看*场景一*，假如是用 Promise 来编写：
```js
getAFromA().then(function (a) {
    return getBFromBWdithA(a);
}).then(function(b) {
    return getCFromCWithB(b);
}).then(function (c) {
    // todo something width c
};
```
怎样，是不是清晰很多？
## 结语
Promise 还有很多特性，比如错误处理，并行处理等等等等。
可以参考这篇文章 [JavaScript Promises](http://www.html5rocks.com/zh/tutorials/es6/promises/#toc-coding-with-promises) ，参照这篇文章做出 DEMO ，我相信你会对 Promise 爱不释手。
其他参考文章：
* [在Node.js 中用 Q 实现Promise – Callbacks之外的另一种选择](http://www.ituring.com.cn/article/54547)
* [指令式Callback，函数式Promise：对node.js的一声叹息](http://www.ituring.com.cn/article/50561)
