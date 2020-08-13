---
layout:     post
title:     论unity踩坑，Character的移动问题
subtitle:   用数学方法跟踪角色，以及其他
date:       2018-09-06
author:     BY俊客
header-img: img/linuxbg.png
catalog: true
tags:
    - unity
---

>How do you using unity control camera follow in character move？o

#如何用unity控制摄像跟随角色，在人物移动的时候

tobe honest ，it is sample question，是的，没错，它很简单，but，如果你想用直接把摄像机放到角色面就太lower了，而且如果有朝一日你要自己开发一个游戏引擎？要搞点大新闻呢？所以我们这里要用数学的方法，让这个简单的方法不简约！之后还能对镜头进行扩展，实现very impossible的效果！
```
'''first，你需要写一堆控制摄像机的方法，这个主要的作用是啥。
嗯，其实也还好，它的作用很easy，就是实现保持相机和人物的距离'''
具体代码：
using UnityEngine;
using System.Collections;
public Transform camera;//制定镜头的坐标
public Transform Player;//制定玩家的坐标
private Vector3 offset；
public class mycode{
void  OnStart()
{offset=camera.position-player.position;}
}
void Update()
{camera.position=player.position+offset;
}
 - [x]
```
###这里我们用代码实现了一个让镜头和玩家控制单位保持一定的功能，but，它不能实现角度的控制。也就是说，在某些第三人称视角游戏，镜头是始终保持在单位正前方视角的，这时我们就需要一个角度的控制
---
#But，wait one sconds，这里就不能简简单单控制相机的四元数角度旋转了。如果你只是控制相机的旋转，问题就是你会发现相机比校色旋转的更快。

![1]({{ "/assets/characterMove/1.png" | absolute_url }})



其实为什么出现这样的结果很简单，人物旋转时候并没有围绕相机做圆周运动，而应该是相机围绕人物做圆周运动，并且朝向和人物保持一致就好了

##那么问题来了

how to solve this question。it is very based，but waste my many many time！
解决这个问题的药典（import）便是，嗯，摄像机围绕人物做圆周运动，由之间圆上坐标，并算出旋转特定角度之后的坐标点。

![问题产生后的行为]({{ "/assets/characterMove/2.jpg" | absolute_url }})



当然这个问题实际上并没有那么好解决。因为你这个问题说难也难，说不难也不难，首先我将图片画了出来

![请无视渣画图，看不懂也没关系]({{ "/assets/characterMove/3.png" | absolute_url }})



大家不要被这图吓到，实际上我也看不懂。。。咳咳，首先我想到是连接旋转前和旋转后的点，知道了旋转角度，而且摄像机离玩家角色长度不变，所以通过角色的位置点连接旋转前的摄像机和旋转后的摄像机构筑一个等腰三角形，然后通过余弦定理求出各个边。。。但是，我不知道怎么表示出旋转后的坐标哇（一下子就哭了出来）
嗯，要是初中的我一定会知道的。一定！！！
后来我又采取二个公式，通过半径乘上cos的值来表示x坐标。乘上sin的值表示y的坐标。。but。这个需要知道与x轴水平夹角是多少，理论上如果回复到原点可能会知道的，但是。算了，我选择死亡！
---
最后解决方法就很坑了，采取矩阵的方式，接用了[陈z大大的文章的公式](https://www.zhihu.com/question/58468471)，大家可以从中详细了解其中原理。
大概采取矩阵变化的方式推导出二个公式x，y在旋转一定角度后等于
x=x*cos 角度-y*sin 角度
y=x*sin 角度+y*cos 角度
那么通过这二个公式既可以写出这样一个公式；
 ```
Vector3 nowc = camera.transform.position;
		Vector3 a = (player.transform.rotation.eulerAngles-angular);
//angular代表初始角度，这个需要通过 transform.rotation.emularAngles.y，来获取单位围绕y轴转动的角度
		//Debug.Log (Mathf.Cos(a.y*Mathf.PI/180));
//		Debug.Log(a);
		camera.transform.position=player.transform.position-new Vector3(
r*(offset.x*Mathf.Cos(a.y*Mathf.PI/180)-offset.z*Mathf.Sin(a.y*Mathf.PI/180)),
-2,
r*(offset.z*Mathf.Sin(a.y*Mathf.PI/180)+offset.x*Mathf.Cos(a.y*Mathf.PI/180))
);
		
 ```

因为半径是个差值，所以我们针对的（x，y）实际上是摄像机到玩家控制单位的一个向量。最后我们需要把camera的位置+算出来的插值，但因为玩家转动的角度相反，所以我们需要改成camera-算出来的插值

最后我们需要再加上最后一行代码，就大功搞成了
``camera.transform.lookat(player); ``

![人物旋转]({{ "/assets/characterMove/1.png" | absolute_url }})



我们需要将镜头旋转和人物旋转保持一致！

![result]({{ "/assets/characterMove/4.jpg" | absolute_url }})



如果能把offset的值乘上一个值n，能实现镜头远近的效果，再用个lerp便能实现镜头过度动画和人物对话拉近镜头的效果了！

![镜头在头顶，穿模]({{ "/assets/characterMove/5.jpg" | absolute_url }})



