title: 模仿百度新闻列表底部的“加载更多”
date: 2015-10-11
categories: 前端
tags: [ajax,js,json,jQuery]
---
## 前言  

&emsp;&emsp;自从上个月来到了学校的信息化中心实习后自由安排的时间越来越少，遂好久没来更新博客了。  
&emsp;&emsp;昨天在完成一个模仿手机端百度新闻列表底“点击加载更多”的功能时，由于第一次写ajax与后端交互，遇到了几个坑，现在逐一来分享。  
<!-- more -->
## 详情
1. 后端提供给我的一个用json传递内容的接口，接口地址类似于
```
http://xxxxxxxxx&category=xx&count=xx   
```
   &emsp;&emsp;category代表新闻的类型，一共有三种，不同的类型对应的列表不同，而count代表当前的页面上已有的新闻条数。  
   &emsp;&emsp;在与写后端的老师的交流中，得知了老师在新闻列表界面是通过将类型放入url来实现的，一共三种url，于是他的三种新闻的列表地址就是
```
http://xxxxxxxx/getlist/x   
```
   &emsp;&emsp;最后一个字符（1/2/3）代表类型。于是我想到了一个奇巧淫技，通过BOM获取当前的浏览器的url，然后正则获取url的最后一位，于是解决了分类的问题。  
   &emsp;&emsp; 当前页面的新闻的形式是一个无序列表，于是通过DOM获取无序列表里的<code>li</code>元素，得到的是一个数组，该数组的长度就是现在页面上的新闻的条数。  

2. 接下来的坑是ajax，jQuery将ajax封装了一遍，又将用json交互的ajax封装了一遍，即getJSON，感觉查到的手册里关于这一方法讲的并不是很好，于是在博客园中找到了这篇文章[Jquery getJSON方法分析（一） - 梅桦 - 博客园](http://www.cnblogs.com/jams742003/archive/2009/12/25/1632276.html)，看完以后豁然开朗。  
   &emsp;&emsp;我这次的案例中后端的json格式是这样的，
```json
[{"url":"url1","title":"title1","pub_date":"1"},{"url":"url2","title":"title2","pub_date":"2"}]
```
   于是我的核心部分代码如下：  
```javascript
        $.getJSON("http://xxxxxxx/getmorenews?category=" + category + "&count=" + count,
        function(news) {  
        if (news == "") { //判断是否有内容  
                $(".ui-refresh").html('已无更多');//提示没内容  
        } else {  
        $.each(news, function(k, v) {  
                var addHtml = ""; //每个循环都单独来一遍  
                $.each(v, function(kk, vv) {  
                                switch (kk) {  
                                case "url": addHtml += '&lt;li class="am-g am-list-item-dated"&gt;&lt;a href="' + vv +'" class="am-list-item-hd "&gt;';break;  
                                case "title": addHtml += vv +'&lt;/a&gt;&lt;span class="am-list-date"&gt;';break;  
                                case "pub_date": addHtml += vv + '&lt;/span&gt;&lt;/li&gt;';break;  
                                }  
                        });  
                $(".am-list").append(addHtml);  
                $(".ui-refresh").html('点击加载更多'); //按钮提示还没结束呢  
        });  
        }//判断部分end  
        });
```

3. 在chrome里调试时，还遇到了一个问题，ajax跨域请求被chrome拦截，在xss钩子遍地都是的网上，禁止ajax跨域确实能大幅提高安全，当然在开发和调试时会带来一些小问题。chrome报错信息如下：
```    
XMLHttpRequest cannot load http://xxxxxxxx/getmorenews?category=2&count=15.
No 'Access-Control-Allow-Origin' header is present on the requested resource.
Origin 'null' is therefore not allowed access.
```
解决方案也很简单，修改chrome的快捷方式的属性中的目标，假设原来是   
```
"C:\Program Files (x86)\Google\Chrome\Application\chrome.exe"   
```
在后面添加
```   
--disable-web-security
```
添加后是
```
"C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --disable-web-security
```
注意空格。然后重启chrome，chrome提醒命令行标记不受支持，安全性会下降就成功了。建议平时不要使用这个快加方式启动chrome，会降低chrome防止xss攻击的能力。  

## 尾声
&emsp;&emsp;唔，你可能已经发现了，我用了amaze ui，一个类似bootstrap的框架。在对bootstrap审美疲劳后，发现了amaze ui，还是挺喜欢它的ui的，等段时间来用amaze ui 做一个hexo的主题。
