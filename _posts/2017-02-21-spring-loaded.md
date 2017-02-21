---
layout: post
title:  "使用spring-loaded实现java web的热部署"
date:   2015-09-12 18:20:19
categories: Java
---
做过java web的同学应该都知道jvm 的hotswap。例如你在使用tomcat作为web容器的时候，在debug模式下启动tomcat，你这个时候修改java代码，会立即生效。但是，这个“修改”是有条件限制的：就是你只能够修改方法体中的内容。这就苦了我们的工程师，如果修改方法名，修改方法参数类型、数量，新增变量、方法，这些都是不可以奏效的。这时，eclipse会弹出一个窗口提示。

![](https://github.com/licaomeng/licaomeng.github.io/blob/master/images/spring-loaded/1.jpg)

这个时候如果点击continue的话，你的修改不会起到任何作用，代码实际上执行的还是之前的逻辑。如果想要你修改的代码生效就只能重启服务，但是重启服务是需要花时间的。怎样才能不重启服务器也可以及时运行我们修改的代码呢？
经过一番查找，找到JRebel这个东西，据说非常强大，但可惜的是JRebel是收费的，而且还不便宜呢，$365/年。那有没有免费开源的，同样可以实现我们热部署？有的，就在github上面：https://github.com/spring-projects/spring-loaded

下面就介绍一下如何利用spring-loaded实现热部署。

**STEP1：**
首先需要得到spring-loaded的jar包，github上面有链接，这里我使用springloaded-1.2.5.RELEASE.jar，将它放到本地c:/springloaded-1.2.5.RELEASE.jar。

**STEP2：**
Window -> Preferences -> Java -> Installed JREs
![](https://github.com/licaomeng/licaomeng.github.io/blob/master/images/spring-loaded/2.jpg)
然后Edit Installed JREs, Default VM arguments 添加参数：
-javaagent:c:/springloaded-1.2.5.RELEASE.jar –noverify
至此spring-loaded的配置完成了，即使在tomcat非debug模式下也可以实现官方所说的：

>add/modify/delete methods/fields/constructors. The annotations on types/methods/fields/constructors can also be modified and it is possible to add/remove/change values in enum types.

但是对于第三方像是Spring注解这些的修改，spring-loaded就无能为力了，必须求助于前面所说的收费的JRebel了。
