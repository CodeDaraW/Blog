title: JavaScript原型学习笔记
date: 2016-01-24 17:30:05
categories: 前端
tags: [JavaScript,prototype]
---
对JS的原型一直不是很懂，虽然不少情况下也能似懂非懂的看懂和写出来，然而彻底理解原型对于学习JS是很有必要的。  
<!-- more -->
  
在阅读[理解JavaScript原型 - 博客 - 伯乐在线](http://blog.jobbole.com/9648/)这篇博文时，遇到了两个小问题，经过一阵思考才理解过来。  

## 问题1 ##  
```javascript
    var A = function(name){
        this.name = name;
    }
    var a = new A('alpha');
    a.name; //'alpha'

    A.__proto__.max = 123;

    a.max   //undefined    
```
```
    var A = function(name){
        this.name = name;
    }
    var a = new A('alpha');
    a.name; //'alpha'

    a.__proto__.max = 123;

    a.max   //123    
```
在我原来的想法中，若上面的调用`a.max`时会向上查找原型链，而A是a的constructor，所以第二段代码能正常运行，a能取到max的值，那为什么`A.__proto__`修改后却`a.max`却是未定义呢？  

在经过一番查找资料，使用`isPrototypeOf`方法测试一个对象是否在另一个对象的原型链上，于是对于第一段代码,在命令行里输入：
```javascript
    A.prototype.isPrototypeOf(a)  // true
    A.__proto__.isPrototypeOf(a)  // false
    a.__proto__.isPrototypeOf(a)  // true
```
很明显，第二段代码`a.max`为`123`是从`A.prototype`上获取的，而`A.constructor`并不在`a`的原型链上，故第一段代码`a.max`为未定义。  

然后我又好奇了，`A.constructor`不在`a`的原型链上那么它在谁的原型链上呢？  

我的疑惑被小秦一句话点破：  

> 因为a是一个对象,而不是一个function


看下面一段代码:
```javascript
    var A = function(){};
    var a = new A();
    var b = function(){};
    A.__proto__.max = 123;
    a.max // undefined
    b.max // 123
    a.constructor === A // true
    a.constructor === Function  // false
    b.constructor === Function  // true
    A.constructor === Function  // true
```
由上可以看出，`A.__proto__`指向了`Function.prototype`，所以`A.constructor`在所有`Function`的实例对象（例如`b`）的原型链上，`console.dir(Function.prototype)`可以看到`Function.prototype`其中有`max:123`。  


## 问题2 ##  

在看到文中的这段代码：
```javascript
    var a = {};


    //Firefox 3.6 and Chrome 5

    Object.getPrototypeOf(a);
    //[object Object]   


    //Firefox 3.6, Chrome 5 and Safari 4

    a.__proto__;
    //[object Object]   


    //all browsers

    a.constructor.prototype;
    //[object Object]
```
我下意识的认为`a.__proto__`和`a.constructor.prototype`是同一个玩意儿，接着在调试台的命令行里测试了一下：
```javascript
    a.__proto__ === a.constructor.prototype // true
```
哎呀，好像还真是的。然后分分钟被打脸：
这是原文中的代码：
```javascript
    //unusual case and does not work in IE

    var a = {};
    a.__proto__ = Array.prototype;
    a.length;   //0
```
于是我想，把`a.__proto__`换成`a.constructor.prototype`应该也能正常吧？于是在控制台里试了下：
```javascript
    //unusual case and does not work in IE

    var a = {};
    a.constructor.prototype = Array.prototype;
    a.length;   // undefined
```
咦？为啥是未定义？说好的一样的呢？
仔细分析，`a.constructor.prototype`就是`Object.prototype`，貌似记得这玩意儿是只读的啊，查了下MDN的文档[Object.prototype - JavaScript | MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/prototype)，果然是只读的。  

** 那么问题来了，为什么上面的代码用`a.__proto__`可以正常呢？ **   

在Stack Overflow上，搜到一个一样的问题，[`javascript - How does __proto__ differ from constructor.prototype? - Stack Overflow`](http://stackoverflow.com/questions/650764/how-does-proto-differ-from-constructor-prototype)，其中一个答案让我似懂非懂的明白了：

> ```
The Prototypal Inheritance in JavaScript is based on __proto__ property in a sense that each object is inheriting the contents of the object referenced by its __proto__ property.  
The prototype property is special only for Function objects and only when using new operator to call a Function as constructor. In this case, the created object's __proto__ will be set to constructor's Function.prototype.  
This means that adding to Function.prototype will automatically reflect on all objects whose __proto__ is referencing the Function.prototype.  
Replacing constructor's Function.prototype with another object will not update __proto__ property for any of the already existing objects.  
Note that __proto__ property should not be accessed directly, Object.getPrototypeOf(object) should be used instead.  
```

又查了一下MDN的文档，[`Object.prototype.__proto__ - JavaScript | MDN`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/proto) ：  
> ```
一个对象的__proto__ 属性和自己的内部属性[[Prototype]]指向一个相同的值 (通常称这个值为原型),原型的值可以是一个对象值也可以是null(比如说Object.prototype.__proto__的值就是null).该属性可能会引发一些错误,因为用户可能会不知道该属性的特殊性,而给它赋值,从而改变了这个对象的原型.
```

也就是说，`a.__proto__ = Array.prototype`改变了`a.__proto__`的指向，将其指向了`Array.prototype`，故`a`拥有了原型链上的方法`length`。  
而`a.__proto__`和`a.constructor.prototype`并非同一个东西，在做比较时相同是因为`a.__proto__`指向了`Object.prototype`。

经过这么一番折腾和总结，对JS的原型有了一个比以前清晰很多的认识，果然有问题时不止要多搜索，还要多查查文档啊！概念或者定义方面的问题，文档比很多人描述的要清晰很多。  

** 参考资料 **  
- [理解JavaScript原型 - 博客 - 伯乐在线](http://blog.jobbole.com/9648/)  
- [理解 JavaScript（四） - 太极客（Very Geek） - SegmentFault](http://segmentfault.com/a/1190000000400182)
- [js原型链原理看图说明 javascript技巧 脚本之家](http://www.jb51.net/article/30750.htm)
