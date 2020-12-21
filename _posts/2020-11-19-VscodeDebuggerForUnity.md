---
layout:     post
title:      用Vscode调试Unity踩的坑
subtitle:   从vs转成vscode经历的一系列问题
date:       2020-11-16
author:     BYDrinker
header-img: img/post-sample-image.jpg
catalog: true
tags:
    - unity,unity3d
---


# 用Vscode调试Unity踩的坑

### 从vs转成vscode经历的一系列问题

有一天，我在默默的拿vs community 2017 写代码，突然我发现我的ctrl+t 不能显示下拉框了，再修复工具修复无果后，毅然将自己的vs卸载，开始用vscode，首先是经管官方的教程，装一系列的插件

![image]({{ "/assets/vscode/unityForVscode.png" | absolute_url }})

这里可以参考一系列的教程或者官方教程，实在实太多，我这里只放出一个官方的示例

https://code.visualstudio.com/docs/other/unity

##### 当然很快我就遇到了第一个问题，也就是无法debug，按下debug按钮后生成不了launch.json文件。其实这里需要换个思路，你需要点击这里的按钮才能生成。

![image]({{ "/assets/vscode/howtCreatelaunch.png" | absolute_url }})

##### 创建后，我又遇到使用当中中第二个问题...(淦，问题超多的，正式项目坚决不要脑抽用vscode)

也就是无法进行断点，一断点就冻结住了，其实这个是工程没有迁徙的问题，删掉Unity项目里的library文件和.sln的解决方案文件就好，然后就是断点等待vscode帮你生成就好。

完成这些步骤，我就开始愉快的写代码，但是很快发现第三个问题，我的工程有点大，是自己开发的mvc架构。在一些使用`obsever` 调用模块的地方会异常卡死，比如todo方法调用view和controller的instance字段的地方，就会报错..

![image]({{ "/assets/vscode/unknowMember.png" | absolute_url }})

这个问题我找了二天终于找到了，也就是这个插件暂时无法评估static类型的变量

https://github.com/Unity-Technologies/vscode-unity-debug/issues/184

解决办法在这个网址里，维护人员提供了一个新的分支修复了这个问题，未来有可能正式版加入，目前根据分支重新编译一遍源码，然后替换插件就行。

分支地址：[https](https://github.com/Unity-Technologies/vscode-unity-debug/tree/fix-static-variable-evaluation) : [//github.com/Unity-Technologies/vscode-unity-debug/tree/fix-static-variable-evaluation](https://github.com/Unity-Technologies/vscode-unity-debug/tree/fix-static-variable-evaluation)



##### 还有个和上面并列出现的问题，应该解决上面的就没问题了，待补充...如果实在不行我就只能问下git的维护人员了，可惜我的英语不是很流利，最近时间有点紧张，下次一定...

![image]({{ "/assets/vscode/stackTrace.png" | absolute_url }})







