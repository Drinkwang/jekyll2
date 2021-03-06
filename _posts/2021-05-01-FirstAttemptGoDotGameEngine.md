---
layout:     post
title:      第一次尝试Godot引擎
subtitle:   业余学点东西
date:       2021-05-01
author:     BYDrinker
header-img: img/tag-bg-o.jpg
catalog: true
tags:
    - Godot
---



# 第一次尝试Godot引擎

> 今天来尝试下新引擎，从入门到入土


  很早就对Godot有所了解，一个用类似Python语言的开源游戏引擎(最近支持Mono)，不过我本身一直在使用U3d来写自己的项目，而在公司里则是用LayaBox完成自己所需要完成的工作(具体可以看我前几篇对Laya引擎源码解析的blog)，所以一直没有时间尝试这个引擎，今天手头逐渐空闲了下来，于是去下载尝试了一下。  

![image]({{ "/assets/godot/openScene.png" | absolute_url }})

  首先说结论，Godot是我接触过最简洁的游戏引擎，没有之一，引擎下载好后，可以看到只有一个可以运行的exe文件，引擎页面更是简洁，如果对unity有所了解一定能很快上手，并且里面的对象又是基于节点式的，对象都是基于一个节点产生的，不会有多余复杂的结构。

![image]({{ "/assets/godot/beganning.png" | absolute_url }})

2、场景和对象可以自由从属，哪怕如今的U3d已经出了在一个场景里进行场景并列，但是没有办法实现树状的从属，只能用预制体来代替。并且每一套东西都有自己额外的逻辑，又附带了多余的component。而Godot基于节点自由装配的形式就大大减轻了运行的消耗（尽管unity也准备推出类似自由装配的tiny，但是毕竟尾大甩不掉）

![image]({{ "/assets/godot/glass.png" | absolute_url }})

 3.开放，unity的成功是基于庞大的社区和用户的支持，所以这点在Godot有过之而无过及，插件通过类似git爬虫的形式搭建了个门户页面，基于Mit、Apache各式各样的协议..

![image]({{ "/assets/godot/openSource.png" | absolute_url }})

以上看上去，Godot确实是一个未来可期的游戏引擎，但我目前给出的评价也只是未来可期，缺点一样很明显

1.使用的脚本语言运行效率底下，gdscript比python3效率还低。

2.目前市场占有率比较少，能够得到支持也比较少。同理对于初学者，学习资料很少，不推荐初学者学习，除非确实是信仰党或者只准备从事2d相关方面开发

3.高度可装配意味着很多东西开发流程较多，类似python的语言让框架不好编写，虽然目前已经有mono支持c#了，但是出现的所有意外问题，在反馈社区没有解决前，都只能自己尝试解决

  最后还是希望Godot能越来越好，它是一个未来可期的引擎，相对于Blender一样，未来一定能在市场创出一片红海，另外每多出一个竞争者就能让这个行业共同进步，加油，所有人！

ps：我也会尝试用它来开发新项目或者写gamejam

![image]({{ "/assets/godot/sward.png" | absolute_url }})

...


​	







