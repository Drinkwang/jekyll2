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

#### 1.安装方法

##### 使用源码

这个仓库可以直接通过git 链接来安装，只需要在`Packages/manifest.json`加入下面这一行就好

```
"com.leopotam.ecs": "https://github.com/Leopotam/ecs.git",
```

默认情况下将会安装上一个发行版，如果你需要最新正在开发的版本（“ trunk/developing” ），将`#develp`增加到链接的后面

```
"com.leopotam.ecs": "https://github.com/Leopotam/ecs.git#develop",
```

##### 使用源码

如果你不想用这种这种方式安装，可以下载源码并使用

# 主要部分

## 组件-Component

包含数据和少量（没有）逻辑

```
struct WeaponComponent {
    public int Ammo;
    public string GunName;
}
```

> **注意!** 不要忘记手动初始化每个新组件的所有字段- 当回收到流时，它们将被重置为默认值。

## 入口-Entity

组件的容器，通过 实现`EcsEntity` 包装内部标识符

```

//在ecs框架的世界中(world字段)创建一个新的entity（入口）
EcsEntity entity = _world.NewEntity ();

// Get() 返回一个entity内的组件 ，如果组件不存在，则会新增一个
ref Component1 c1 = ref entity.Get<Component1> ();
ref Component2 c2 = ref entity.Get<Component2> ();

// Del()从ecs移除一个组件.如果它是最后一个组件，ecs也会自动将其移除
entity.Del<Component2> ();

//将组件替换成一个新的组件，如果组件不存在，则会新加入一个
var weapon = new WeaponComponent () { Ammo = 10, GunName = "Handgun" };
entity.Replace (weapon);

// 用 Replace() 方法你可以约束组件的创造：
var entity2 = world.NewEntity ();
entity2.Replace (new Component1 { Id = 10 }).Replace (new Component2 { Name = "Username" });

// 任意 entity 都可以被复制(包括其中所有组件):
var entity2Copy = entity2.Copy ();

// 任意 组件可以被合入/移动到另外一个entity(原始的将会被销毁) 
var newEntity = world.NewEntity ();
entity2Copy.MoveTo (newEntity); // 所有 entity2Copy  的组件将会移动到newEntity, entity2Copy 将会销毁(destroyed).

// 任意 entity 可以被销毁. 所有entity上的组件将会被先移除，然后entity再销毁
entity.Destroy ();
```

> **注意!** Entities 没有 components 将会自动删除，在上一个 `EcsEntity.Del()` 调用时.
>
> 

## System

Сontainer for logic for processing filtered entities. User class should implements `IEcsInitSystem`, `IEcsDestroySystem`, `IEcsRunSystem` (or other supported) interfaces:

```
class UserSystem : IEcsPreInitSystem, IEcsInitSystem, IEcsRunSystem, IEcsDestroySystem, IEcsPostDestroySystem {
    public void PreInit () {
        // Will be called once during EcsSystems.Init() call and before IEcsInitSystem.Init.
    }

    public void Init () {
        // Will be called once during EcsSystems.Init() call.
    }
    
    public void Run () {
        // Will be called on each EcsSystems.Run() call.
    }

    public void Destroy () {
        // Will be called once during EcsSystems.Destroy() call.
    }

    public void PostDestroy () {
        // Will be called once during EcsSystems.Destroy() call and after IEcsDestroySystem.Destroy.
    }
}
```

# Data injection

All compatible `EcsWorld` and `EcsFilter<T>` fields of ecs-system will be auto-initialized (auto-injected):

```
class HealthSystem : IEcsSystem {
    // auto-injected fields.
    EcsWorld _world = null;
    EcsFilter<WeaponComponent> _weaponFilter = null;
}
```

Instance of any custom type can be injected to all systems through `EcsSystems.Inject()` method:

```
class SharedData {
    public string PrefabsPath;
}
...
var sharedData = new SharedData { PrefabsPath = "Items/{0}" };
var systems = new EcsSystems (world);
systems
    .Add (new TestSystem1 ())
    .Inject (sharedData)
    .Init ();
```

Each system will be scanned for compatible fields (can contains all of them or no one) with proper initialization:

```
class TestSystem1 : IEcsInitSystem {
    // auto-injected fields.
    SharedData _sharedData;
    
    public void Init() {
        var prefabPath = string.Format (_sharedData.Prefabspath, 123);
        // prefabPath = "Items/123" here.
    } 
}
```

# Special classes

## EcsFilter

Container for keeping filtered entities with specified component list:

```
class WeaponSystem : IEcsInitSystem, IEcsRunSystem {
    // auto-injected fields: EcsWorld instance and EcsFilter.
    EcsWorld _world = null;
    // We wants to get entities with "WeaponComponent" and without "HealthComponent".
    EcsFilter<WeaponComponent>.Exclude<HealthComponent> _filter = null;

    public void Init () {
        _world.NewEntity ().Get<WeaponComponent> ();
    }

    public void Run () {
        foreach (var i in _filter) {
            // entity that contains WeaponComponent.
            ref var entity = ref _filter.GetEntity (i);

            // Get1 will return link to attached "WeaponComponent".
            ref var weapon = ref _filter.Get1 (i);
            weapon.Ammo = System.Math.Max (0, weapon.Ammo - 1);
        }
    }
}
```

> **Important!** You should not use `ref` modifier for any filter data outside of foreach-loop over this filter if you want to destroy part of this data (entity or component) - it will break memory integrity.

All components from filter `Include` constraint can be fast accessed through `EcsFilter.Get1()`, `EcsFilter.Get2()`, etc - in same order as they were used in filter type declaration.

If fast access not required (for example, for flag-based components without data), component can implements `IEcsIgnoreInFilter` interface for decrease memory usage and increase performance:

```
struct Component1 { }

struct Component2 : IEcsIgnoreInFilter { }

class TestSystem : IEcsRunSystem {
    EcsFilter<Component1, Component2> _filter = null;

    public void Run () {
        foreach (var i in _filter) {
            // its valid code.
            ref var component1 = ref _filter.Get1 (i);

            // its invalid code due to cache for _filter.Get2() is null for memory / performance reasons.
            ref var component2 = ref _filter.Get2 (i);
        }
    }
}
```

> Important: Any filter supports up to 6 component types as "include" constraints and up to 2 component types as "exclude" constraints. Shorter constraints - better performance.

> Important: If you will try to use 2 filters with same components but in different order - you will get exception with detailed info about conflicted types, but only in `DEBUG` mode. In `RELEASE` mode all checks will be skipped.

## EcsWorld

Root level container for all entities / components, works like isolated environment.

> Important: Do not forget to call `EcsWorld.Destroy()` method when instance will not be used anymore.

## EcsSystems

Group of systems to process `EcsWorld` instance:

```
class Startup : MonoBehaviour {
    EcsWorld _world;
    EcsSystems _systems;

    void Start () {
        // create ecs environment.
        _world = new EcsWorld ();
        _systems = new EcsSystems (_world)
            .Add (new WeaponSystem ());
        _systems.Init ();
    }
    
    void Update () {
        // process all dependent systems.
        _systems.Run ();
    }

    void OnDestroy () {
        // destroy systems logical group.
        _systems.Destroy ();
        // destroy world.
        _world.Destroy ();
    }
}
```

`EcsSystems` instance can be used as nested system (any types of `IEcsInitSystem`, `IEcsRunSystem`, ecs behaviours are supported):

```
// initialization.
var nestedSystems = new EcsSystems (_world).Add (new NestedSystem ());
// don't call nestedSystems.Init() here, rootSystems will do it automatically.

var rootSystems = new EcsSystems (_world).Add (nestedSystems);
rootSystems.Init ();

// update loop.
// don't call nestedSystems.Run() here, rootSystems will do it automatically.
rootSystems.Run ();

// destroying.
// don't call nestedSystems.Destroy() here, rootSystems will do it automatically.
rootSystems.Destroy ();
```

Any `IEcsRunSystem` or `EcsSystems` instance can be enabled or disabled from processing in runtime:

```
class TestSystem : IEcsRunSystem {
    public void Run () { }
}
var systems = new EcsSystems (_world);
systems.Add (new TestSystem (), "my special system");
systems.Init ();
var idx = systems.GetNamedRunSystem ("my special system");

// state will be true here, all systems are active by default.
var state = systems.GetRunSystemState (idx);

// disable system from execution.
systems.SetRunSystemState (idx, false);
```

# Engine integration

## Unity

> Tested on unity 2019.1 (not dependent on it) and contains assembly definition for compiling to separate assembly file for performance reason.

[Unity editor integration](https://github.com/Leopotam/ecs-unityintegration) contains code templates and world debug viewer.

## Custom engine

> C#7.3 or above required for this framework.

Code example - each part should be integrated in proper place of engine execution flow.

```
using Leopotam.Ecs;

class EcsStartup {
    EcsWorld _world;
    EcsSystems _systems;

    // Initialization of ecs world and systems.
    void Init () {        
        _world = new EcsWorld ();
        _systems = new EcsSystems (_world);
        _systems
            // register your systems here, for example:
            // .Add (new TestSystem1 ())
            // .Add (new TestSystem2 ())
            
            // register one-frame components (order is important), for example:
            // .OneFrame<TestComponent1> ()
            // .OneFrame<TestComponent2> ()
            
            // inject service instances here (order doesn't important), for example:
            // .Inject (new CameraService ())
            // .Inject (new NavMeshSupport ())
            .Init ();
    }

    // Engine update loop.
    void UpdateLoop () {
        _systems?.Run ();
    }

    // Cleanup.
    void Destroy () {
        if (_systems != null) {
            _systems.Destroy ();
            _systems = null;
            _world.Destroy ();
            _world = null;
        }
    }
}
```



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

