---



layout:     post
title:      Unity'ECS Frame Learning
subtitle:   学习unity的ecs框架
date:       2021-06-07
author:     俊壳
header-img: img/tag-bg-o.jpg
catalog: true
tags:
    - Godot
---

# 学习unity的ecs框架

> ecs是基于流进行处理的框架，游戏内的每一个基本单元都是一个**实体**，每个**实体**又由一个或多个**组件**构成，每个**组件**仅仅包含代表其特性的数据（即在组件中没有任何方法），例如：移动相关的组件`MoveComponent`包含速度、位置、朝向等属性，一旦一个实体拥有了`MoveComponent`组件便可以认为它拥有了移动的能力，**系统**便是来处理拥有一个或多个相同**组件**的**实体**集合的工具，其只拥有行为（即在系统中没有任何数据），在这个例子中，处理移动的**系统**仅仅关心拥有移动能力的**实体**，它会遍历所有拥有`MoveComponent`**组件**的**实体**，并根据相关的数据（速度、位置、朝向等），更新实体的位置。如果有Spring相关经验就更好理解了，我这次介绍的ecs也存在依赖注入相关方面
>
> **实体**与**组件**是一个一对多的关系，**实体**拥有怎样的能力，完全是取决于其拥有哪些**组件**，通过动态添加或删除**组件**，可以在（游戏）运行时改变**实体**的行为。

## ecs介绍

ecs框架大纲图：

![img](https://pic3.zhimg.com/80/v2-04e15b14964c9b61bffdfad42e907ffc_hd.jpg)

我们这会不介绍框架的源码，而是通过一个弹球小游戏实例，由浅入深，来为大家讲解某个ecs框架用法

框架下载网址：[Drinkwang/ecs: LeoECS is a fast Entity Component System (ECS) Framework powered by C# with optional integration to Unity (github.com)](https://github.com/Drinkwang/ecs)

具体游戏下载地址在这里：[cadfoot/unity-ecs-bubble-shooter (github.com)](https://github.com/cadfoot/unity-ecs-bubble-shooter)

首先我们具体来看看Startup这个类，我们通过这个类来了解ecs的基本写法，上面我们提到过ecs是由多个系统 去处理实体，而实体的获取是通过组件进行获得的，具体获得方式就是autowire完成（依赖注入）

` 依赖注入:整个项目需要获取实体的变量会通过‘流’来自动获取，玩家只需要将变量放置在框架流中，就无需关注其中细节，直接可以使用`

## 框架理解

待续...



## 项目理解

我们来看看案例的代码，这样更好理解依赖注入的观念

```c#
        private void Start()
        {
            Application.targetFrameRate = 60;
            
            _world = new EcsWorld();
            _systems = new EcsSystems(_world);
#if UNITY_EDITOR
            //Leopotam.Ecs.UnityIntegration.EcsWorldObserver.Create(_world);
            //Leopotam.Ecs.UnityIntegration.EcsSystemsObserver.Create(_systems);
#endif
            _systems
                .Add(new BoardInitSystem())
                .Add(new CameraInitSystem())
                .Add(new BackgroundInitSystem())
                .Add(new BoardPhysicsBoundsInitSystem())

                .Add(new InputSystem())
                
                .Add(new TrajectorySystem())
                .Add(new NextBubbleSystem())
                
                .Add(new BubbleConnectionSystem())
                .Add(new BubbleFallSystem())
                
                .Add(new BubbleMergeSystem())
                .Add(new BubbleExplodeSystem())
                
                .Add(new ShootSystem())
                
                .Add(new BubbleFlowSystem())

                .Add(new NextBubbleViewSystem())
                .Add(new CreateBubbleViewSystem())
                .Add(new PredictionViewUpdateSystem())
                .Add(new TrajectoryViewUpdateSystem())
                .Add(new BubbleFallDeathSystem())
                
                .Add(new BubbleViewMergeSystem())
                .Add(new BubbleViewMoveSystem())
                .Add(new BubbleViewFlySystem())
                .Add(new BubbleViewFallSystem())
                .Add(new BubbleViewShakeSystem())
                
                .Add(new MergeTextSpawnSystem())
                .Add(new PerfectNotificationSystem())
                .Add(new ComboMergeNotificationSystem())
                
                .Add(new BubbleViewTweeningMarkSystem())
                .Add(new BubbleCompleteMergeSystem())

                .Add(new BubbleViewHangingDestroySystem())

                .Add(new InputClearSystem())
                .Add(new TrajectoryClearSystem())
                
                .OneFrame<Prediction>()
                .OneFrame<Connected>()
                .OneFrame<Created>()
                .OneFrame<WorldPosition>()
                .OneFrame<Destroyed>()
                .OneFrame<New>()

                .Inject(GetComponent<ISceneContext>())
                .Inject((IConfig)_config)
                .Inject((IRandomService)new RandomService(_useSeed ? _randomSeed : (int?) null))
                
                .Init();
        }

```

```c#
        Application.targetFrameRate = 60;
        
        _world = new EcsWorld();
        _systems = new EcsSystems(_world);
```
这三行代码主要用来实例化ecs，没什么好解释的，通过这个构建了ecs系统

待续...

