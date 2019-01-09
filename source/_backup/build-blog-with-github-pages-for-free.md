title: 用GitHub Pages免费空间搭建Blog
date: 2015-08-25 14:12:30
categories: 生活
tags: [生活,Node.js, git]
---
## __前言__
&emsp;&emsp;其实之前就知道可以用GitHub Pages搭建静态博客，不过之前一直忙着爬手册撸代码==，昨天撸完了一些手册，于是把代码什么的放一放，来折腾静态博客吧！  

<!-- more --> 

## __准备__
&emsp;&emsp;GitHub Pages本来是给托管在GitHub的项目的介绍页面留的空间，由于其空间是免费的，拿来搭个博客真心不错。但是，一般的Blog CMS都是非静态的，而如果我们要用静态空间搭博客，那就只能自己动手切页面或者使用为这种静态空间设计的博客管理工具。   

&emsp;&emsp;这种博客管理工具用的比较多的应该算是Jekyll以及HEXO了，一开始在折腾Jekyll时，可能由于gem的兼容问题，出现了一个特别奇葩的依赖包明明安装好却找不到的问题，而且搜了很久也没找到解决方法，倒是在StackOverflow找到了一个同病相怜的Mac用户==不过今天在完成了hexo的配置后又试了了下Jekyll，居然又没问题了！但我不会再爱你了！！！    
&emsp;&emsp;我也建议使用Jekyll或者hexo，这两个用户比较多，教程和问题解决方案也比较多。接下来我要分享的是hexo的安装过程。
## __过程__
&emsp;&emsp;关于这方面的教程还是很多的，百度或者谷歌下一大把，但是在搜索教程时一定要注意教程的日期，由于版本差异，有些教程的部分步骤可能会导致乱七八糟的报错。另外一点就是遇到报错第一反应应该就是去搜索报错信息，一般情况下你遇到的问题已经有无数人遇到过了，搜索引擎可以帮助你找到他们的解决方案。最后，如果有耐心，还是建议看官方文档，官方文档才是最详细的！   
&emsp;&emsp;由于教程很多，我这里就不重复了，直接把我看过的觉得有用的教程贴出来吧。建议先看一篇完整的，理清头绪，然后再一步一步来，“遇到问题->解决问题->下一步” 的重复。   

参考资料：   
[Creating Project Pages manually - User Documentation](https://help.github.com/articles/creating-project-pages-manually/)    
[Hexo](https://hexo.io/zh-cn/)   
[Hexo 静态博客使用指南 - 简书](http://www.jianshu.com/p/73779eacb494)</br>
[hexo常用命令笔记 - SegmentFault](http://segmentfault.com/a/1190000002632530#articleHeader15)</br>
[如何搭建一个独立博客——简明Github Pages与Hexo教程 - poem_of_sunshine的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/poem_of_sunshine/article/details/29369785/)   
[搭建 hexo，在执行 hexo deploy 后,出现 error deployer not found:github 的错误 - V2EX](http://www.v2ex.com/t/175940)   

## __进阶__

*	__注册免费域名__：   
[新版Freenom免费域名TK,CF,ML,GA申请注册方法_百度经验](http://jingyan.baidu.com/article/e3c78d64688e913c4d85f559.html)</br>
[Freenom](http://www.freenom.com/zh/index.html?lang=zh)（不稳定，最好自备梯子。）

*	__域名解析__：   
[成功注册.ML免费顶级域名和添加DNS域名解析绑定空间方法 | 免费资源部落](http://www.freehao123.com/ml-dns/)</br>
[DNSpod](https://www.dnspod.cn/)   
[如何搭建一个独立博客——简明Github Pages与Hexo教程 - poem_of_sunshine的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/poem_of_sunshine/article/details/29369785/)</br>

*	__添加评论功能__：   
我用的是国内的“多说”    
[Hexo使用多说教程 - 多说开发者中心](http://dev.duoshuo.com/threads/541d3b2b40b5abcd2e4df0e9)</br>
[Hexo 主题Light修改教程 - 简书](http://www.jianshu.com/p/70343b7c2fd3)</br>

*	__添加访问统计__：   
由于大家都懂的操蛋的原因，谷歌统计在国内会大姨妈，我用的是百度站长统计</br>
[hexo 添加百度统计 | Just For Fun](http://blog.justforfun.top/2015/02/06/hexo-%E6%B7%BB%E5%8A%A0%E7%99%BE%E5%BA%A6%E7%BB%9F%E8%AE%A1/)</br>
[百度统计——最大的中文网站分析平台](http://tongji.baidu.com/web/welcome/login)</br>

*	__使用谷歌字体镜像站：__   
和上面一样的原因。使用镜像站可以提速很多。作为一名数字黑，不得不说数字家的字体镜像还是很好用的。</br>
[360网站卫士常用前端公共库CDN服务](http://libs.useso.com/)    
我用的是light主题，不清楚其他主题会不会是不同的地方，但是大体步骤是一样的，修改了两处：</br>
1.
```
	\themes\light\source\css\_base\variable.styl
```
的第14行改为
 ```css
 	@import url("//fonts.useso.com/css?family=Lato:400,400italic")
 ```

2.
```
	\themes\light\layout\_partial\after_footer.ejs
```
的第一行中js引用地址改为
```
	"//ajax.useso.com/ajax/libs/jquery/2.0.3/jquery.min.js"
```

其实总结下就是把谷歌的字体地址都改成数字的。配合浏览器的开发者模式看还有没有用到谷歌的，如果还有慢慢找出来改掉就行了。</br>

*	__修改网站ico__   
```
	\themes\light\layout\_partial\head.ejs
```
有这样一行代码：
```
	<link href="<%- config.root %>favicon.ico" rel="icon">;
```
所以很明显啦，只要在根目录下放一个favicon.ico图标就可以了，也就是放在<code>\source</code>文件夹下，hexo生成时会自动添加到网站根目录下。</br>
当然你也可以选择不修改文件名，把head.ejs中的favicon.ico改成你的图标的文件名也可。</br>
再送个工具：   
[制作ico图标 | 在线ico图标转换工具 方便制作favicon.ico - 比特虫 - Bitbug.net](http://www.bitbug.net/)

*	__添加导航__   
例如我的博客添加的About导航：   
[hexo怎么在菜单上添加页面和分类呢？ - SegmentFault](http://segmentfault.com/q/1010000000618915)</br>

*	__添加RSS订阅以及网站地图__   
[Hexo 入门指南（六） - sitemap、rss 和部署_百度经验](http://jingyan.baidu.com/article/e52e3615aac99740c60c5129.html)</br>
如果上述指令无效，在指令后添加<code> --save</code>，<code>npm install &lsaquo;name&rsaquo; --save</code>表示安装的同时，将信息写入package.json中。</br>
如果想和我一样把RSS放进导航，那么修改主题的<code>_config.yml</code>文件，在menu里添加<code>RSS: /atom.xml</code>。对，这时候你应该注意点了吧，其实<code>menu:</code>就是对应的导航==

## __总结__
我暂时就折腾了这么多，准备有时间来写个主题。    
有问题如果搜索后依然无解可以留言。
