---
layout:     post
title:      论kindle电纸书黑客可以怎么玩（1）
subtitle:   kindle折腾相关
date:       2019-12-17
author:     BY俊客
header-img: img/hackEbook.jpg
catalog: true
tags:
    - kindle
---

>so,i buy one kindle,after seen,i think of this is linux device ,so it is funny， let do it !

在我买到kindle时，在我一番探视下，我发现这个是一个linux设备，所以有趣的事情来了，让我们开干吧！
首先我来介绍一个论坛[!国外最大电纸书](https://www.mobileread.com/forums/newreply.php?do=newreply&noquote=1&p=2764521)
在其次我们要理解一个概念，也就是hacking（黑客），但我们不要狭义理解它，这个词本身也就是挥砍的意思。在英文后续渐渐的有喜欢折腾计算机的人的意思，所以我们这里理解的这个词应该要作为一个褒义词，折腾自己的计算机和设备本身并没有错，或者说破解这件事也没有错，但错的是利用这个破解来出售卖钱。如果但从技术角度上来看，许多国家包括那些产品公司本身也支持粉丝多折腾折腾自己设备，所以如果在尊重知识产权的用时进行非商业技术研究破解是应该被支持的。这一点在这个论坛上一些贴纸深有体会，在美国，对盗版如此厌恶的国家，但你也可以破解下载软件的权力以及一些扩展功能这没事，但是相反如果是破解了并且还用此牟利，就要受到口诛笔伐，甚至牢狱之灾。这个。咳咳，this is question，不过某一方面给技术无罪提供一个说法吧，虽然是有限程度的，这一方面类似于国家把无线电合法授权给ham爱好者一样，但是你得有呼号！
首先我们要使用的设备是k3.我们着重往以下几个方面介绍

***我们打开我们的设备，我的kindle3并没有刷入多看系统，虽然对于爱看书的人，多看确实是一个不错的玩意，但是多看系统虽然被发烧的小米收购了，但是并没有发出让我们这些死爱折腾折腾的途径，大多数都是静静的书友们。所以我这种俗人自然没有安装，等到有一天不再想折腾了，或许也会去使用吧，这个各有所需，不过多评价***

![我的kindle设备](https://upload-images.jianshu.io/upload_images/13871785-6cbb0b6ae459f294.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
``然后打开我们的kual，官方解释是一个kindle Unified Application Launcher ，这个替代了早先k3 的lauch pad和klite，在所有可越狱的设备上均可以安装``
![P80921-155340.jpg](https://upload-images.jianshu.io/upload_images/13871785-8a51db622e3bec1a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
``打开后我们就可以看到这样一个界面了，这个里面布局是可以通过调整json和一些玩意来实现的，我们来介绍下我所安装的一些软件``
![kpdf阅读器](https://upload-images.jianshu.io/upload_images/13871785-f3a47aabcf357550.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
``kpdf这个是调整原先kindle pdf的运行状态的，在原生pdf阅读器无法阅读扫描文件，文字放大比只有100，150，200，但是100看不清，150又放太大的问题层出不穷，所以125甚至130很有必要，这就需要我们有这个软件了，在多看系统下，貌似实现了个类似的功能，所以多看的pdf兼容性比kindle原生是要好的，当然kindle也有办法来实现很好的阅读pdf，就是通过亚马逊转码服务。来转成合适的格式。这个就不细说``
>对了，这个软件应该算得上hacking软件典范的例子，他属于合法，首先代码属于自制，而且准从gplv3开源协议，后续推出koread的k5，k6触摸屏版本陆续被移植到安卓和ubuntu touch等多平台，所以它本身只属于对一个设备推出的软件，而并不存在对收费软件的破解，我想就算是收费软件，也不会触犯到法律吧，当然它是开源的，不过开源讲道理也可以用作商业，呃，这个就不得而知了。。

![usbnetwork](https://upload-images.jianshu.io/upload_images/13871785-048be1142b5270b6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这个软件应该算得上kindle破解技术宅必备，通过这，既可以搭建一个内网应用环境，用电脑usb连接到kindle，或者用wifi连接ssh。用来登入putty，telnet，等等网络协议，我见过有大神在上面交叉编译实现node.js在上面部署服务器代码以及用c语言实现黑苹果，用了它kindle将无所不能。因为它本身是一个linux系统设备，当作树莓派某些功能来用问题不大，只要肯折腾，说它是一生产工具也是完全不过分，不过当然。。这样的设备估计只有在末日才能发挥它应有的生产力的作用，让处在和平时期看惯图形和网络的便利“娇生惯养”的我通过它来码代码，这不是找罪受么，恕臣妾实在是做不到啊！
>剩下就是通过elink load game 来模拟gb。通过水墨屏什么来实现gb，呃有点。。。不过体验一下和弄清楚源码也很不错拉，虽然我水平不是很够，但是慢慢开始看github上面各个模块的代码，也是一种淡淡的享受！
![kindle玩gb俄罗斯方块.jpg](https://upload-images.jianshu.io/upload_images/13871785-b67d24ab2c1a8e0b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![kindle跑fbgnuboy gb模拟器](https://upload-images.jianshu.io/upload_images/13871785-a490234e08074a76.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![kindle进入游戏.jpg](https://upload-images.jianshu.io/upload_images/13871785-70f1cad227b7da72.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

以上就是论kindle可以怎么玩的初体验，上诉案例都可以在论坛找到源代码以及实现方法，还有很多没有介绍给大家，之后会一一介绍给大家，包括一些不正经的内容。比如kindle实现假注册用来模拟一个注册过的设备。当然，这些内容在论坛里也被提及到，这样会加大盗窃设备分险而口诛笔伐，这也说明外国人的版权意识形态。以及对合法破解和不合法破解的态度，当然我也会处于更理性的文字去跟大家分析这些破解hacking的点点滴滴。以及其中的技术。
之后，由于我还在学习过程中，如果大家有什么好玩的想法，可以跟我分享噢，这是我的博客地址
[myblog](https://drinkwang.github.io/),欢迎大家参观。。虽然现在貌似没有什么内容拉。。
