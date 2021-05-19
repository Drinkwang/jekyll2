---


layout:     post
title:      Godot引擎第一人称控制示例学习
subtitle:   业余学点东西
date:       2021-05-19
author:     BYDrinker
header-img: img/tag-bg-o.jpg
catalog: true
tags:
    - Godot
---



# Godot引擎第一人称控制示例学习

> 这是我第二次写关于Godot有关的`Blog`，这次我们来学习下Godot里面的第一人称的案例

首先我们二话不说直接运行游戏

![image]({{ "/assets/godot/beganning.png" | absolute_url }})

...一段时间过来，我们就进入游戏的菜单的ui界面

![image]({{ "/assets/godot/FirstPersonA/began.png" | absolute_url }})

​	既然如此，我们就来了解下按键的一些设计，也就是`ui`有关的代码

![image]({{ "/assets/godot/FirstPersonA/playButton.png" | absolute_url }})

直接贴代码，它是通过Inspector旁边的Node面板里进行嵌入绑定，有点类似于Qt的开发方式

```python
extends Control

export var start_scene = "test"

func _play_game():
	fader._fade_start(start_scene)

```

这里放出fader类底下的`_fade_start`方法


```python
func _fade_start(var scene_path):
	path = "res://scenes/" + scene_path + ".tscn"
	var file = File.new()
	if file.file_exists(path):
		_fade_out()
		timer.start()
	else:
		_reload_scene()
```

总体而言是通过这个判断是否有场景文件，在加载场景文件时候播放FadeIn和FadeOut来过度，最终通过定义器嵌入一个结束方法进行切换场景

![image]({{ "/assets/godot/FirstPersonA/timeout.png" | absolute_url }})

另外退出游戏也是大同小异的过程

```python
extends Button

var timer//定义一个定时器

func _ready()://初始化方法，将类里的变量赋值成场景中的定时器
	timer = $timer

func _quit_pressed()://当按钮按下
	fader._fade_out()//淡出，定时器开始同步
	timer.start()

func _quit_game():
	get_tree().quit()//定时器时间到，游戏退出

```



接下来将讲讲游戏里的一些方法和设计

![image]({{ "/assets/godot/FirstPersonA/openScene.png" | absolute_url }})

待续...

