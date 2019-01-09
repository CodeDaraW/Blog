title: JavaScript中函数的变量提升
date: 2016-03-13 22:13:05
categories: 前端
tags: [JavaScript]
---
整理文件时偶然发现了一个讲JS预编译的PDF，仔细一看其实讲的是变量提升。文件中的一段代码有点无法理解：  
```JavaScript
var f = function () {
     console.log('1');
}

function f() {
     console.log('2');
}

f(); // 1
```
控制台输出的是1，按照道理JS没有重载，下一个函数应该会覆盖上一个同名函数，但是为什么输出的是`1`呢？  
<!-- more -->

这是一个讲变量提升的文章，那应该是变量提升在作祟。  
记得在博客园看过一篇讲JS变量提升的文章[JavaScript中变量提升](http://www.cnblogs.com/damonlan/archive/2012/07/01/2553425.html)，文中曾经说过：
> 函数提升是把整个函数都提到前面去。
在我们写js code 的时候，我们有2中写法，一种是函数表达式，另外一种是函数声明方式。我们需要重点注意的是，只有函数声明形式才能被提升。

下面还有代码演示：
```JavaScript
// 函数声明方式提升【成功】

function myTest(){
    foo();
    function foo(){
        console.log("我来自 foo");
    }
}
myTest(); // 我来自 foo
```

```JavaScript
// 函数表达式方式提升【失败】

function myTest(){
    foo();
    var foo =function foo(){
        console.log("我来自 foo");
    }
}
myTest(); // foo is not a function
```

其实这篇文章是**有问题**的，看下面两段代码：
```JavaScript
function myTest(){
    foo();
}
myTest(); // foo is not defined
```

```JavaScript
function myTest(){
    var foo;
    foo();
}
myTest(); // foo is not a function
```

注意两个报错信息的区别，`is not a function`和`is not defined`，`is not a function`事实上已经发生了变量提升！  

那么函数声明和函数表达式的变量提升有什么区别呢？  
《JavaScript秘密花园》中的一个栗子很清楚明了：

> JavaScript 会提升变量声明。这意味着 var 表达式和 function 声明都将会被提升到当前作用域的顶部。

```JavaScript
bar();
var bar = function() {};
var someValue = 42;

test();
function test(data) {
    if (false) {
        goo = 1;

    } else {
        var goo = 2;
    }
    for(var i = 0; i < 100; i++) {
        var e = data[i];
    }
}
```

> 上面代码在运行之前将会被转化。JavaScript 将会把 var 表达式和 function 声明提升到当前作用域的顶部。

```JavaScript
// var 表达式被移动到这里
var bar, someValue; // 缺省值是 'undefined'

// 函数声明也会提升
function test(data) {
    var goo, i, e; // 没有块级作用域，这些变量被移动到函数顶部
    if (false) {
        goo = 1;

    } else {
        goo = 2;
    }
    for(i = 0; i < 100; i++) {
        e = data[i];
    }
}

bar(); // 出错：TypeError，因为 bar 依然是 'undefined'
someValue = 42; // 赋值语句不会被提升规则（hoisting）影响
bar = function() {};

test();
```

那么一开始的栗子其实是和下面的代码等效的：
```JavaScript
var f;

function f() {
     console.log('2');
}

f = function () {
     console.log('1');
}

f(); // 1
```
一切也就不难理解了。
