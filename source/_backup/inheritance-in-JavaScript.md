title: JavaScript中的继承
date: 2016-07-14 16:21:19
categories: JavaScript
tags: [JavaScript, 继承]
---
最近在按照vue.js的历史commit依次阅读代码来学习和了解vue.js。  
在读到`commit 5ce3b82`时，发现这个时候尤大把`Seed`组件类抽象了出来，并提供了`extend`的接口可以自定义组件类，而自定义组件类继承于父类`Seed`。
<!-- more -->
`main.js`中这一块地方实现如下：
```JavaScript
Seed.extend = function (opts) {
    var Spore = function () {
        Seed.apply(this, arguments)
        for (var prop in this.extensions) {
            var ext = this.extensions[prop]
            this.scope[prop] = (typeof ext === 'function')
                ? ext.bind(this)
                : ext
        }
    }
    Spore.prototype = Object.create(Seed.prototype)
    Spore.prototype.extensions = {}
    for (var prop in opts) {
        Spore.prototype.extensions[prop] = opts[prop]
    }
    return Spore
}
```
顺便不得不提下尤大的代码写的非常优雅，虽然他是不加分号党，而我是分号党，但读他的代码并不会感觉什么不适。  
这儿可以看出来通过JS的原型实现了继承，顺便复习一下JS的继承吧。  

## ES3时代的继承
```JavaScript
function Father() {
    this.lang = "js";
}
Father.prototype = {
    say: function() {
      console.log("Hello world");
    }
};

function Son() {
  // 执行初始化
	Father.apply(this, arguments);
}

Son.prototype = new Father();
Son.prototype.constructor = Son;

Son.prototype.name = 'son';

var sonTest = new Son();
console.log(sonTest.name); // son
console.log(sonTest.lang); // js
sonTest.say(); // Hello World
console.log(sonTest instanceof Son); // true
console.log(sonTest instanceof Father); // true
console.log(sonTest.constructor === Son); // true
```

在上面的例子中，`sonTest`对象能够访问`Father`实例属性`lang`，并能同时访问`Son.prototype`和`Father.prototype`上的对象，
重复使用原型上的实例而不是新建一个实例，也就是实现了继承。  
原理也很简单，在查找一个对象的属性时，如果自身没有这个属性，会向上遍历原型链，一级一级向上查找，直到找到所要的属性，
而原型链的顶部则是`Object.prototype`，若依然没有查找，那么这个值为`undefined`。  

而通过将子类的`prototype`属性设为父类的一个实例对象，则能后让父类的`prototype`出现在子类的原型链中，也就是实现了原型继承。  

另外要注意性能方面的影响，继承的优点是通过共享原型链上的属性/函数来节省内存，但是属性在原型链的上方会消耗查找时间，
特别是属性根本不存在时，会遍历整个原型链。  

## ES5时代的继承
ES5中`Object`对象新增了一个方法，叫`create`，`Object.create`方法接受传入一个作为新创建对象的原型的对象，
创建一个拥有指定原型和若干个指定属性的对象。那么上面的例子可以改写为：
```JavaScript
function Father() {
    this.lang = "js";
}
Father.prototype = {
    say: function() {
      console.log("Hello world");
    }
};

function Son() {
	Father.apply(this, arguments);
}

Son.prototype = Object.create(Father.prototype);
Son.prototype.constructor = Son;

Son.prototype.name = 'son';

var sonTest = new Son();
console.log(sonTest.name); // son
console.log(sonTest.lang); // js
sonTest.say(); // Hello World
console.log(sonTest instanceof Son); // true
console.log(sonTest instanceof Father); // true
console.log(sonTest.constructor === Son); // true
```

好吧，其实也没简化什么。  
但是`Object.create`还是很强大的，它可以顺带传入若干个指定属性，借用MDN上的例子：
```JavaScript
var o;

// 创建一个原型为null的空对象
o = Object.create(null);


o = {};
// 以字面量方式创建的空对象就相当于:
o = Object.create(Object.prototype);


o = Object.create(Object.prototype, {
  // foo会成为所创建对象的数据属性
  foo: { writable:true, configurable:true, value: "hello" },
  // bar会成为所创建对象的访问器属性
  bar: {
    configurable: false,
    get: function() { return 10 },
    set: function(value) { console.log("Setting `o.bar` to", value) }
}})


function Constructor(){}
o = new Constructor();
// 上面的一句就相当于:
o = Object.create(Constructor.prototype);
// 当然,如果在Constructor函数中有一些初始化代码,Object.create不能执行那些代码


// 创建一个以另一个空对象为原型,且拥有一个属性p的对象
o = Object.create({}, { p: { value: 42 } })

// 省略了的属性特性默认为false,所以属性p是不可写,不可枚举,不可配置的:
o.p = 24
o.p
//42

o.q = 12
for (var prop in o) {
   console.log(prop)
}
//"q"

delete o.p
//false

//创建一个可写的,可枚举的,可配置的属性p
o2 = Object.create({}, { p: { value: 42, writable: true, enumerable: true, configurable: true } });
```


## ES6时代的继承
其实对于ES6我也并不熟悉，但是这已经是2016年下半年，说到继承不提一下ES6的`class`我都不好意思。虽然ES6的类是语法糖，但回顾ES6之前的说原型继承更先进的那些文章和书，真是分分钟打脸。    
上面的例子用ES6可以简写为：
```JavaScript
class Father {

	constructor() {
		this.lang = "js";
	}

	say() {
		console.log("Hello world");
	}
}

class Son extends Father {

	constructor() {
		super();
	}

}

Son.prototype.name = "son";

var sonTest = new Son();
console.log(sonTest.name); // son
console.log(sonTest.lang); // js
sonTest.say(); // Hello World
console.log(sonTest instanceof Son); // true
console.log(sonTest instanceof Father); // true
console.log(sonTest.constructor === Son); // true
```

确实简化了很多，不过要注意的是ES6的类只是语法糖，实现还是依赖原型，正如上面所示，`Son`类依然可以定义原型属性`name`，
让实例来共享原型属性。  
关于ES6的类，这里就不说更多了，推荐阮一峰老师的《ES6入门》。


# 参考资料
* [refactor · vuejs/vue@5ce3b82](https://github.com/vuejs/vue/commit/5ce3b82b91a134f57dd1dffe8553cec369a56c70)
* [JavaScript-Garden](http://bonsaiden.github.io/JavaScript-Garden/zh/)
* [Object.create() - JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create)
* [Class - ECMAScript 6入门](http://es6.ruanyifeng.com/#docs/class)
