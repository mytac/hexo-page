---
title: 消化闭包
date: 2019-04-03 17:14:52
tags: 
    - js
---
## 前言
之前写过关于闭包的文章，本来以为自己懂了，后来面试时被问到怀疑人生。才明白自己只是觉得自己明白了而已，如果说要将一个东西理解的彻彻底底，就不能“抄书”（我之前就是抄书），而是死抠每一个知识点，一点含糊都会让整个系统崩塌。

ok，现在开始死抠。什么是闭包？
> 闭包就是能够读取其他函数内部变量的函数。例如在javascript中，只有函数内部的子函数才能读取局部变量，所以闭包可以理解成“定义在一个函数内部的函数“。在本质上，闭包是将函数内部和函数外部连接起来的桥梁 ——来自于百度百科

> 闭包是基于词法作用域书写代码时所产生的自然结果。当函数可以记住并访问所在的词法作用域时，就产生了闭包。 ————《你不知道的js（上）》

看不太懂，那就拆开看，什么是词法作用域？
## 词法作用域
如图，每个框框中都是一个作用域，引擎在执行`console.log()`时（黄色框中的语句），会从内向外逐个作用域查找变量。在baz中，我们找到了变量c，没有找到a,b，就会往上一层找，bar中有b,c,baz，找到了b，同名变量c被忽略，以此类推，直至所有执行语句都匹配了变量，否则引擎解析失败抛出错误。

![图示](https://wx3.sinaimg.cn/mw690/6f8e0013ly1g1l3a8vgw1j20ls0dg0ts.jpg)
### 除了词法作用域，还有啥？
其实作用域包括词法作用域和动态作用域，JavaScript中的作用域是词法作用域（大部分的编程语言也是基于词法作用域）。在上面的图中，我们能清晰地看出来，**每个函数的全部变量都可以在整个函数的范围中使用或复用（嵌套的函数可以使用外部函数的变量）**，这就是函数作用域。那么只有函数才能创建作用域“框框”吗？

我们看下面这几句代码：
```js
for(var b=0;b<3;b++){}

console.log('b',b) // 3
```
上面的代码中，没有声明任何函数，所以通过var声明的变量b被绑定到外部作用域上，也就是全局。(不了解变量提升的同学，可以看我的这篇文章=>[《详解ES6暂存死区TDZ》](https://www.jianshu.com/p/fe05129e8a4c))，所以上述代码相当于：
```js
var b;
for(b=0;b<3;b++){}
console.log('b',b) // 3
```
。。。是不是很奇葩，本来只想让变量b在for循环中使用，for循环之后销毁，为啥要让他污染到整个词法作用域嘞？幸运的是，由于人类的探索精神，和几个浏览器爹们对JavaScript这个不健全的儿子的扶持，ES6中有了let和const，作为块作用域的补充。（明明都9012了，我为啥还在写ES6的东西=.=）如下，b在for循环结束时就会被销毁，又由于词法作用域中不存在同名变量，所以这里会报错。
```js
for(let b=0;b<3;b++){}
console.log('b',b) // Uncaught ReferenceError: b is not defined
```
我们在理解块作用域的时候，可以将一个`{}`中看成一个块。

### 作用域和上下文到底是不是一个东西?
答案肯定是"NO!!"上文中我们已经明白了，作用域是在函数定义时决定的。上下文其实就是函数中`this`的指向，即当前函数运行时所挂载的对象。
```js
const a=1
function foo(){
    console.log(this.a)
}

const obj={a:2,foo}

foo() // undefined
obj.foo() // 2
```
这里有个小tips，为啥`const`声明的a，没有像var一样挂载到`window`上呢？其实秘密在这里，[《Javascript闭包：从理论到实现，[[Scopes]]的每一根毛都看得清清楚楚》](https://segmentfault.com/a/1190000015311755?utm_source=tag-newest) （写本章时我也没仔细读这篇文章），const 声明的a其实是在`[[scopes]]`上。
## 循环和闭包
### 一道经典面试题
以下代码为什么与预想的输出不符？
```js
// 代码块1
for (var i = 0; i < 5; i++) {
    setTimeout(() => {
        console.log(i) // 输出5次5
    }, 0)
}
```
假设A：因为`setTimeout`这块的任务直接进入了事件队列中，所以i循环之后i先变成了5，再执行`setTimeout`，`setTimeout`中的箭头函数会保存对i的引用，所以会打印5个5.

变体一：
```js
// 代码块2
for (let i = 0; i < 5; i++) {
    setTimeout(() => {
        console.log(i) // 输出 0,1,2,3,4
    }, 0)
}
``` 

假设结论A成立，那么上式应该也是输出5次5，但是很明显不是，所以结论A并不完全正确。

那我们去掉循环，先写成最简单的异步代码：
```js
function test(a){
    setTimeout(function timer(){
        console.log(a)
    })
}

test('hello')
```
执行`test`，`setTimeout`将`timer`函数放入了事件队列，`timer`保留着`test`函数的作用域（在函数定义时创建的），`test`执行完毕，主线程上没有其他任务了，`timer`从事件队列中出队，执行`timer`，执行`console.log(a)`，由于闭包的原因，a依然会保留着之前的引用，输出`'hello'`。

那我们在回到题目中，因为两段代码中的不同只有声明语句，所以我们提出假设B：因为在代码块1中，匿名函数保留着外部词法作用域，i都是在全局作用域上，代码块2中由于存在块作用域，所以它保留着每次循环时i的引用。

变体二：
```js
// 代码块3
for (var i = 0; i < 5; i++) {
    ((i) => {
        setTimeout(function timer() {
            console.log(i) // 输出 0,1,2,3,4
        }, 0)
    })(i)
}
```
使用IIFE传递了变量i给匿名函数，产生了一个新作用域，timer保留着这个作用域链，所以会依次输出。

变体三：
```js
// 代码块4
for (var i = 0; i < 5; i++) {
    (() => {
        setTimeout(function timer() {
            console.log(i) // 输出 5个5
        }, 0)
    })()
}
```
跟变体2的区别为IIFE没有给匿名函数传递i，`timer`保留的作用域链还是全局作用域。

## 完
希望看完的小伙伴可以彻底明白“闭包”，如果有任何错误请在下方评论区留言，欢迎指正。
## 推荐文章
1. [深入理解闭包之前置知识---作用域与词法作用域](https://www.jianshu.com/p/60ca27e185ec)