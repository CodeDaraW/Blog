title: JavaScript作用域和闭包以及在ES6中的变化
date: 2016-02-02 18:47:08
categories: JavaScript
tags: [JavaScript, ES6]
---
JavaScript的闭包以及作用域，这是个老生常谈的话题了，随便Google就能搜到一大把相关的文章,每本和JS相关的书这也基本是必谈话题。本来是懒得写一篇关于它们的学习笔记的，最近学习ES6和Node.js的过程中又见到了一些新的花样，所以想想还是来啰嗦一下。  
本文中将只从JS语言的层面上来分析，并不仅仅针对前端JS。  
<!-- more -->
## 谈谈ES6以前的几个坑 ##
* 很多从其他语言转到JS的新手都写过这种代码：
```javascript
function foo() {
    var functionArray = [];
    for (var i = 0; i < 9; i++) {
        functionArray[i] = function() {
            console.log(i);
        }
    }

    functionArray[1]();
}

foo();
```
他们的初衷很美好，然而现实中输出的并不是`1`而是`9`。  
原因也很简单，`functionArray`数组中每个函数对象中`console.log`的作用域为`global`（浏览器中为`window`），每次循环都是在重写`global.i`的值，而在ES6之前，**JS中并不支持块级作用域**，因此在循环结束时，`i`是9，所以数组中每个函数对象执行后的输出结果都是9。  
**ES6之前JS只支持函数作用域**，但是JS是十分灵活的，这个栗子中常见的解决方案是借用闭包模拟出块级作用域：
```javascript
function foo() {
    var functionArray = [];
    for (var i = 0; i < 9; i++) {
        functionArray[i] = (function(i) {
            return function() {
                console.log(i);
            }
        })(i);
    }

    functionArray[1]();
}

foo();
```
当然还有使用`with`语句和`try...catch`语句等等构造闭包，想了解更多种方法的话推荐司徒正美的这个文章[javascript的闭包](http://www.cnblogs.com/rubylouvre/archive/2009/07/24/1530074.html)。  

* **变量提升**  
第一次知道变量提升是一个人的提问，贴了一段类似下面的代码：
```javascript
var x = 'outer';
(function(){
    console.log(x);
    var x = 'inner';
})();
```
他的问题是为什么输出的是`undefined`而不是想象中的`outer`,有人回答：
> 搜索“变量提升”  

接下来是另外一段相似的代码：
```javascript
var x = 'outer';
(function(){
    console.log(x);
})();
```
输出结果是意料之中的`outer`。  
那么问题来了，我知道学挖掘机找蓝翔，可这是为什么呢？  
答案是：JS中的**变量提升**。  
这涉及到了JS的变量作用域的问题，说实话对于JS的作用域我还是有点困惑的，虽然它和C系语言长的有点相似，很多地方却截然不同。  
在脚本开始运行时，通过`var`声明的变量会发生提升。
```javascript
console.log(x);
var x = 'outer';
```
上面的代码其实等效于下面的代码：
```javascript
var x;
console.log(x);
x = 'outer';
```
所以输出为`undefined`。  
回看上段出问题的代码，实际等效于下面的代码：
```javascript
var x = 'outer'; // 全局变量
(function(){
    var x; // 局部变量
    console.log(x);
    x = 'inner';
})();
```

在JS中，函数是“一等公民”，也是变量，但是要注意只有函数声明会被提升（注：这样说是有误的，关于函数的变量提升看我后来写的[JavaScript中函数的变量提升](/2016/03/13/javascript-hoisting-functions/) ）。对比下面的两段代码：
```javascript
foo(); // test

function foo() {
    console.log('test');
}
```

```javascript
foo(); // TypeError: foo is not a function

var foo = function() {
    console.log('test');
}
```

## ES6给我们带来的变化 ##
其实这里主要要谈的是ES6中最常用的`let`。  
我们先来看一段代码：
```javascript
'use strict';

function foo() {
    var functionArray = [];
    for (let i = 0; i < 9; i++) {
        functionArray[i] = function() {
            console.log(i);
        }
    }

    functionArray[1](); // 1
}

foo();
```
成功的按照我们的想法输出了`1`，怎么样，是不是感觉眼前一亮！  
ES6中`let`声明了一个块级域的本地变量，并且可以同时初始化该变量。`let` 允许把变量的作用域限制在块级域中。与 `var` 不同处是：`var` 申明变量要么是全局的，要么是函数级的，而无法是块级的。再看一段`var`和`let`的对比代码：
```javascript
'use strict';

function varTest() {
    var x = 31;
    if (true) {
        var x = 71;  // same variable!
        console.log(x);  // 71
    }
    console.log(x);  // 71
}

function letTest() {
    let x = 31;
    if (true) {
        let x = 71;  // different variable
        console.log(x);  // 71
    }
    console.log(x);  // 31
}
```
除了块级作用域，还要注意的是`let`没有变量提升，在`let`声明之前引用将引起`ReferenceError`错误：
```javascript
'use strict';

console.log(x); // ReferenceError: x is not defined
let x = 2;
```
同时，和上面不同的还有“暂时性死区”，在块级作用域内`let`声明的变量会绑定这个作用域，不受外部影响：
```javascript
'use strict';

var tmp = 123;

if (true) {
    tmp = 'abc';  // ReferenceError: tmp is not defined
    let tmp;
}
```
对了还有一点别忘了，`let`不允许在同一作用域内重复声明同一个变量：
```javascript
'use strict';

function foo1() {
    let a = 10;
    var a = 1; // SyntaxError: Identifier 'a' has already been declared
}

function foo2() {
    let a = 10;
    let a = 1; // SyntaxError: Identifier 'a' has already been declared
}
```
还要注意一个有fu趣ck的事情，由于ES6规定函数本身的作用域在其所在的块级作用域之内，下面这段代码在ES6之前输出为`inner`，ES6输出为`outer`：
```javascript
function foo() { console.log('outer'); }
if (true) {
    // 重复声明一次函数foo
    function foo() { console.log('inner!'); }
}
foo();
```


### **参考资料** ###
* [JavaScript中变量提升------Hoisting - 随风浪迹天涯 - 博客园](http://www.cnblogs.com/damonlan/archive/2012/07/01/2553425.html)
* [let - JavaScript | MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/let)
* [let和const命令 - ECMAScript 6入门](http://es6.ruanyifeng.com/#docs/let)



最后分享自己写这篇文章时循环播放的一首歌，不才的声音真的很好听啊：  

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="http://music.163.com/outchain/player?type=2&id=28912020&auto=0&height=66"></iframe>
