---
layout:     post
title:     unity发布项目到webgl 
subtitle:   发布游戏到4399踩的一些坑
date:       2018-09-06
author:     BY老王
header-img: img/home-bg-geek
catalog: true
tags:
    - unity
---
>How do you using unity deploy the WebGL game？o	
>#如何用unity部署webgl，发布game到4399踩的坑如下

![image]({{ "/assets/4399/43991.png" | absolute_url }})





首先我们要修改unity的webgl配置一栏，比如splash img等等a,具体来说要把分辨率改成800-600。
一些游戏网站对广告zero容忍，那么其次我们需要将一些代码给去掉,这部分就要修改下html的内容了。打开浏览器，按下f12，抓取到要和谐的unity自带广告，把某个div元素给删掉，这样就能屏蔽开头动画或者底部框栏

![image]({{ "/assets/4399/43992.png" | absolute_url }})



>这是需要删除的代码，可以用浏览器选取工具找到，其实也没几行，一些div元素，纯看代码也看得懂，删完之后代码就变成这样。


***
```
<!DOCTYPE html>
<html lang="en-us">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <title>Unity WebGL Player | Dreamday</title>
    <link rel="shortcut icon" href="TemplateData/favicon.ico">
    <link rel="stylesheet" href="TemplateData/style.css">
    <script src="TemplateData/UnityProgress.js"></script>  
    <script src="Build/UnityLoader.js"></script>
    <script>
      var gameInstance = UnityLoader.instantiate("gameContainer", "Build/dramday.json", {onProgress: UnityProgress});
    </script>
  </head>
  <body>
    <div class="webgl-content">
      <div id="gameContainer" style="width: 800px; height: 600px"></div>
     
    </div>
  </body>
</html>
```
这里是代码的粗糙介绍，从这篇代码我们也能看出端疑，从而来学习IL2CPP的技术，当然想要认真学习这块内容，就得了解虚拟机机制，以及mono和clr，这里就不多多复述了。
***
show you the best,but project will be wrong
#附上项目地址：[4399](http://www.4399.com/flash/200403.htm)
