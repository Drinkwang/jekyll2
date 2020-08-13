---
layout:     post
title:      H5开发恐怖事情
subtitle:   Laya的H5开发
date:       2020-01-06
author:     BYDrinker
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - 游戏开发
---


# # H5开发恐怖事情-Laya运行出问题总结

# window和linux差距

* 首先我们需要知道Window和linux还是有很多不同的，比如window的线程和linux的线程不一样。以及window不支持检索大小写不同的文件夹，也就是在*window*里面AA文件夹和aa文件夹是同个文件夹

所以当一个引擎具备打包图集能力，如果需要统一资源命名格式，而公司的新人不太清楚这项规时，就容易出现修改前名字和修改后名字除了大小写完全一致情况，然而默认生成图集不会删除，就会生成2个图集，而引擎默认使用最先的一个，此时就需要手动删除，重新导入，需要删除三个地方。不然又会重新生成不匹配的图集

![image](https://s2.ax1x.com/2020/01/06/lrTuFS.png)



## List中可怕的事情

### 1.如果引擎设计多个客户端模块，通过中间件进行进行调用

在laya里，如果涉及List赋值数据，则可能涉及不同的段名，导致list.datasource无法赋值，此时换成list.array就可以解决。

`List.DataSource=procel.xxx; `

改为List.Array=procel.xxx;

### 2.Laya List如果赋值有些没有赋值。

比如数据如下

* [null,null,data1,data2] 

当为list赋值data为这个数组时，则会看到效果如这样

![img](https://s2.ax1x.com/2020/01/06/lrT56A.png)

​				[data1,data1,data1,data2] 

List会自动取用相邻字段，这应该时一个小Bug，在laya1.0可以复现，2.0还未测试。

# 使用as进行开发恐怖的事情

如果用As进行开发，Laya支持As，公司一些老旧项目会用action script进行开发，这个时候如果你按照官方的教程进行，就有可能报一堆错误。因为官方编译器导入Laya的包编译条件更为严谨，而实际as语言是没那么严谨的，而碰巧公司项目“没那么严谨的话”，就会出现出现一堆错误，此时其实没必要管它，最好就是删掉官方的编译引用，换成adobe官方的，这样错误就不见啦。

具体可以参照这篇Blog：[Actionscript坑点](https://mp.csdn.net/postedit/102863722)
