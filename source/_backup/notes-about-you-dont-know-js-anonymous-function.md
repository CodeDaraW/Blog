title: 《你不知道的JS上卷》阅读小记之匿名函数问题
date: 2017-02-20 19:48:52
categories: JavaScript
tags: [JavaScript]
---
## 匿名函数的问题
在第一部分`3.3.1匿名与具名`一节中，作者提到了匿名函数的三个缺点并给了一个最佳实践，三个缺点分别是调试时栈追踪看不到有意义的函数名带来麻烦，引用自身时只能使用过期的`arguments.callee`并举了两个例子回调和事件监听器触发后解绑自身，代码可读性；最佳实践是说始终传入具名函数，给函数表达式一个函数名。  

<!-- more -->
关于缺点三和最佳实践我觉得没什么问题；缺点一只要不是变态的多层嵌套，debug工具的栈追踪还是能比较清楚的理解的；问题出在了缺点二：  
Talk is cheap, 直接上代码：

```JavaScript
var y = g =>
    (f=>f(f))(
        self =>
            g( (...args)=>self(self)(...args) )
    )

var f = y(self =>
    n => n < 0 ? 0 : n + self(n-1))

f(100); // 5050
```
这段代码来自Winter的Gist([ycombinator in es2015
Raw
](https://gist.github.com/wintercn/7d06dd226d699c832b6a))，y函数有一个名字叫做Y Combinator。是不是有种黑魔法的感觉？  

关于Y组合子网上已经有一大堆科普文，如果对于它的推导过程感兴趣可以参考[Javascript推导Y-Combinator](http://blog.csdn.net/voidccc/article/details/39143733)。Y组合子的的精髓就是在于函数不动点：`g(f) = f`，高阶函数`g`传入一个函数`f`还会得到`f`。  

Y组合子是Lambda演算的一部分，也是函数式编程的理论基础，在没有赋值语句的前提下定义递归的匿名函数。在实际开发中其实并没有太大用处，敢在业务代码里写上面的YC的估计都被打死了吧hhhhh，实际开发中还是建议遵循书中的建议比较好一点。

## 参考
* [不动点组合子 - 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/%E4%B8%8D%E5%8A%A8%E7%82%B9%E7%BB%84%E5%90%88%E5%AD%90)
* [Javascript中匿名函数的递归调用](http://codedocker.com/anonymous-function-recursion-in-javascript/)
* [Javascript推导Y-Combinator](http://blog.csdn.net/voidccc/article/details/39143733)
