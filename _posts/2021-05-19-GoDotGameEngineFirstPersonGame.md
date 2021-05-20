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

首先我们二话不说运行实例

![image]({{ "/assets/godot/beganning.png" | absolute_url }})

游戏我这里用文字口诉下构成，大概是一个界面，以及点击界面中的play进入一个第一人称场景

![image]({{ "/assets/godot/FirstPersonA/began.png" | absolute_url }})

​	既然如此，我们就来了解下Ui，也就是`ui`有关的代码

![image]({{ "/assets/godot/FirstPersonA/playButton.png" | absolute_url }})

直接贴代码，它的按钮是通过Inspector旁边的Node面板里进行嵌入绑定，有点类似于Qt的开发方式

![image]({{ "/assets/godot/FirstPersonA/bind.png" | absolute_url }})

```python
extends Control

export var start_scene = "test"

func _play_game()://按钮点击进入的方法
	fader._fade_start(start_scene)//淡出并加载test场景

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

首先是第一人控制相关的内容

```python
extends KinematicBody

export var gravity = -30.0
export var walk_speed = 8.0
export var run_speed = 16.0
export var jump_speed = 10.0
export var mouse_sensitivity = 0.002
export var acceleration = 4.0
export var friction = 6.0
export var fall_limit = -1000.0

var pivot

var playable = true
var dir = Vector3.ZERO
var velocity = Vector3.ZERO

func _ready():
	pivot = $pivot
	Input.set_mouse_mode(Input.MOUSE_MODE_CAPTURED)

func _physics_process(delta)://循环调用的物理系统
	dir = Vector3.ZERO
	var basis = global_transform.basis
	if Input.is_action_pressed("move_forward")://当按下上键，前进
		dir -= basis.z
	if Input.is_action_pressed("move_back")://当按下下键，后退
		dir += basis.z
	if Input.is_action_pressed("move_left")://当按下左键，左边走
		dir -= basis.x
	if Input.is_action_pressed("move_right"):当按下右键，右边走
		dir += basis.x
	dir = dir.normalized()//将方向坐标归一化

	var speed = walk_speed
	if is_on_floor(): //如果在地面上
		#this prevents you from sliding without messing up the is_on_ground() check//防止滑行，不受重力影响
		velocity.y += gravity * delta / 100.0//重力
		if Input.is_action_pressed("run"):
			speed = run_speed
		if Input.is_action_just_pressed("jump")://跳跃
			velocity.y = jump_speed
	else:
		velocity.y += gravity * delta//重力

	var hvel = velocity
	hvel.y = 0.0

	var target = dir * speed//目标方向等于方向乘上一个速度
	var accel
	if dir.dot(hvel) > 0.0:
		accel = acceleration
	else:
		accel = friction
	hvel = hvel.linear_interpolate(target, accel * delta)线性穿插，accel * delta>=1后移动到目标点
	velocity.x = hvel.x
	velocity.z = hvel.z
	if playable:
		velocity = move_and_slide(velocity, Vector3.UP, true)	//第一个参数是移动方向，第二个参数是确定什么是墙，第三个参数是表示是否会被斜坡卡住

	#prevents infinite falling//跌落重新开始游戏
	if translation.y < fall_limit and playable:
		playable = false
		fader._reload_scene()

func _unhandled_input(event):
	if event is InputEventMouseMotion and playable://接受到鼠标移动事件，且游戏开始时
		rotate_y(-event.relative.x * mouse_sensitivity)
		pivot.rotate_x(-event.relative.y * mouse_sensitivity)//进行旋转操作
		pivot.rotation.x = clamp(pivot.rotation.x, -1.2, 1.2)
```

了解移动后，我们就可以了解下是如何触发物体的。
![image]({{ "/assets/godot/FirstPersonA/TriggerArea2.png" | absolute_url }})
这些是通过如同之前一样的嵌入方法，嵌入到body_entered，通过_body_entered方法发送`emit_signal("player_entered")`信号, 然后进入player_entered事件中，具体的话还是需要大家去自己学习，因为大同小异，我这里也不过多介绍了