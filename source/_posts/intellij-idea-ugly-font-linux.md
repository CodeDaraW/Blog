title: IntelliJ IDEA在Linux下字体不正常解决方案
date: 2016-11-24 20:34:15
categories: Linux
tags: [Intellij IDEA, Linux]
---
之前遇到了一件奇怪的事，WebStorm中字体正常，IDEA直接导入WebStorm的设置备份也还是不行，如图所示。
<!-- more -->
![](https://ww1.sinaimg.cn/large/005KE4htgw1f9hf9q1cy0j31hc0u0181.jpg)

本来这事也就放着不管了，昨天和Shaoxing聊天提到了这事，他提醒我正常情况下IDEA系列应该都是自带JDK的，于是我查了一下IDEA有带和不带JDK两个版本，自带JDK的会针对HiDPI和字体做一些优化。  

我打开了WS和IDEA进行对比，发现他们的About信息中的JVM版本果然不一样：
![](https://ww1.sinaimg.cn/large/005KE4htgw1fa3hgndesmj30hy0b6q6l.jpg)  
![](https://ww1.sinaimg.cn/mw690/005KE4htgw1fa3hgo3t08j30ia0bkae3.jpg)  

在IDEA的设置里手动切换了JVM版本后IDEA会自动重启，然而并没什么卵用，重启后JVM又回到了Oracle版本。  
我突然想起曾经在环境变量中配置过`IDEA_JDK`，于是删除了这个变量，在终端中输出这个变量已经不存在，然而还是不行。  

这时看到官网说可以在`idea.sh`中手动添加`IDEA_JDK`变量，于是我打开了`idea.sh`，其中60行往后为关键点：

```bash
# ---------------------------------------------------------------------
# Locate a JDK installation directory which will be used to run the IDE.
# Try (in order): IDEA_JDK, idea.jdk, ../jre, JDK_HOME, JAVA_HOME, "java" in PATH.
# ---------------------------------------------------------------------
if [ -n "$IDEA_JDK" -a -x "$IDEA_JDK/bin/java" ]; then
  JDK="$IDEA_JDK"
elif [ -s "$HOME/.IntelliJIdea2016.3/config/idea.jdk" ]; then
  JDK=`"$CAT" $HOME/.IntelliJIdea2016.3/config/idea.jdk`
  if [ ! -d "$JDK" ]; then
    JDK="$IDE_HOME/$JDK"
  fi
elif [ -x "$IDE_HOME/jre/jre/bin/java" ] && "$IDE_HOME/jre/jre/bin/java" -version > /dev/null 2>&1 ; then
  JDK="$IDE_HOME/jre"
elif [ -n "$JDK_HOME" -a -x "$JDK_HOME/bin/java" ]; then
  JDK="$JDK_HOME"
elif [ -n "$JAVA_HOME" -a -x "$JAVA_HOME/bin/java" ]; then
  JDK="$JAVA_HOME"
```

后面的代码省去，虽然没学过`shell`，但很明显查找JDK的流程为先查看环境变量有没有`IDEA_JDK`变量，如果没有再去看配置信息里有没有设置`idea.jdk`，如果没有再去找IDE的目录里自带的JDK。  
所以解决方法很简单了，把上面的流程注释掉，直接去IDE的目录下找自带的JDK：
```bash
# ---------------------------------------------------------------------
# Locate a JDK installation directory which will be used to run the IDE.
# Try (in order): IDEA_JDK, idea.jdk, ../jre, JDK_HOME, JAVA_HOME, "java" in PATH.
# ---------------------------------------------------------------------
# if [ -n "$IDEA_JDK" -a -x "$IDEA_JDK/bin/java" ]; then
#   JDK="$IDEA_JDK"
# elif [ -s "$HOME/.IntelliJIdea2016.3/config/idea.jdk" ]; then
#   JDK=`"$CAT" $HOME/.IntelliJIdea2016.3/config/idea.jdk`
#   if [ ! -d "$JDK" ]; then
#     JDK="$IDE_HOME/$JDK"
#   fi
# elif [ -x "$IDE_HOME/jre/jre/bin/java" ] && "$IDE_HOME/jre/jre/bin/java" -version > /dev/null 2>&1 ; then
if [ -x "$IDE_HOME/jre/jre/bin/java" ] && "$IDE_HOME/jre/jre/bin/java" -version > /dev/null 2>&1 ; then
  JDK="$IDE_HOME/jre"
elif [ -n "$JDK_HOME" -a -x "$JDK_HOME/bin/java" ]; then
  JDK="$JDK_HOME"
elif [ -n "$JAVA_HOME" -a -x "$JAVA_HOME/bin/java" ]; then
  JDK="$JAVA_HOME"
```

保存后打开IDEA，果然一切都正常了，About信息中也显示使用了自带的JDK。
