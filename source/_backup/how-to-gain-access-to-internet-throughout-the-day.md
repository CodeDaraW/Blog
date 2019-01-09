title: 在学校里我是如何优雅地永不断网的
date: 2015-11-18 18:13:48
categories: 生活
tags: Shadowsocks
---
# 背景  
&emsp;&emsp;**夜间断网可以说是每个学生的心里的痛。**  
&emsp;&emsp;信息化中心有一个好，那就是不断网，而且网速比谁都快。有天闲聊得知这边的网和宿舍区教学区以及图书馆同属校园网，于是一个计划在心中形成了。  
<!-- more -->

# 过程  
&emsp;&emsp;首先我将一台安装了Ubuntu的电脑放在了信息化中心，将其联网。打开Ubuntu的终端，安装shadowsocks服务端：  
```
		sudo apt-get install python-gevent python-pip  
		sudo pip install shadowsocks  
		sudo apt-get install python-m2crypto  
```
&emsp;&emsp;这样，shadowsocks客户端就安装完了。  
&emsp;&emsp;接下来是配置，在任意位置新建一个json文件:  
```
		{
		"server":"Your ip",
		"server_port":8388,
		"local_port":1080,
		"password":"your password",
		"timeout":600,
		"method":"aes-256-cfb"
		}
```
&emsp;&emsp;将Your ip替换成该Ubuntu PC的IP，your password替换成设定的密码，其他参数可以不用改变。如想修改参数可以参照Github上的[Wiki](https://github.com/shadowsocks/shadowsocks/wiki)。  
&emsp;&emsp;接下来启动shadowsocks服务端：
```
		ssserver -c /hemo/daraw/config.json  
```
&emsp;&emsp;<code>-c</code> 后面带的参数即上面新建的json文件的位置。  
&emsp;&emsp;当然，一直开着终端是不太方便的，  
```
		nohup ssserver -c /etc/shadowsocks/config.json &  
```
&emsp;&emsp;<code>nohup</code> 命令可以将其一直在后台运行。  
&emsp;&emsp;就这样，服务器端配置完毕。  


&emsp;&emsp;学校宿舍区PPPoE可以用用户名free@88，密码888888来登陆校园网，24h不断，当然正常情况下只能访问校园网，登陆学校的一些网站例如官网还有信息门户等等啦。  
&emsp;&emsp;然而这就足够了，Android手机root后可以下载PPPoE拨号工具，当然部分手机系统是自带PPPoE拨号功能的（例如魅族的flyme）。登入校园网后，打开shadowsocks客户端，将配置文件和服务端设成一样，路由改为全局，然后运行，就可以享受永不断网的快乐了。  

# 后言  
&emsp;&emsp;前不久和学长ChiChou还有研究生GSX在吃饭时聊到这个问题，ChiChou说他在大一大二时也干过这种事，他说还可以直接http代理，免去shadowsocks，这样更加轻量。  

&emsp;&emsp;**其实，相比不断网带来的快乐，这种Geek成功的感觉更爽，geek for fun！**
