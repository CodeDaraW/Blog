title: vuex2中使用mapGetters/mapActions报错解决方法
date: 2016-09-22
categories: 前端
tags: [vuex2]
---

在尝鲜vuex2时，发现vuex2增加了`mapGetters`和`mapActions`的方法，借助stage2的`Object Rest Operator`特性,可以写出下面代码：
```javascript
methods: {
  marked,
  ...mapActions([
    'getArticles'
  ])
}
```
但是在借助babel编译转换时发生了报错：`BabelLoaderError: SyntaxError: Unexpected token`。
  
<!-- more -->
### 解决方案
在vuex的repo issues中有人提过这样的问题，后来是修改了eslint配置中对`Object Rest Operator`的支持解决了问题，然而我根本没有使用eslint。  
接着在babel的issues中搜索这样的报错，原来是我babel预置的转换器是`babel-preset-es2015`，并不能转换`Object Rest Operator`特性。  
解决方法很简单了，可以安装整个stage2的预置器或者安装`Object Rest Operator`的babel插件`babel-plugin-transform-object-rest-spread`。  
我选择了安装插件，接着在babel的配置文件`.babelrc`中应用插件：
```javascript
{
  "presets": [
    ["es2015", { "modules": false }]
  ],
  "plugins": ["transform-object-rest-spread"]
}
```
重启webpack，就不会再有报错了。