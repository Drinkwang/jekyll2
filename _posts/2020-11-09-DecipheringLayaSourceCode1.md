---
layout:     post
title:      逆袭，从0开始解读Laya源代码（一）
subtitle:   Laya的H5开发
date:       2020-11-02
author:     BYDrinker
header-img: img/post-bg-js-version.jpg
catalog: true
tags:
    - 游戏开发，从0开始解读Laya源代码
---


# 逆袭，从0开始解读Laya源代码（1）



* 上一讲我们说道如何编译laya源代码，以及做了一些基础性的研究，那么这一期界面我们将进一步做更多考量

前面几期教程：

 https://drinker.site/2020/11/02/%E8%A7%A3%E8%AF%BBlaya%E6%BA%90%E4%BB%A3%E7%A0%81(0).html 

​						...

> laya官方案例:https://ldc2.layabox.com/doc/?language=zh&nav=zh-ts-0-3-4





## 1.研究Laya代码

### 1.了解框架的思维导图

![image]({{ "/assets/Laya/process2.png" | absolute_url }})

###  2.二番战，从配置文件出发，了解laya源码

首先我们打开这样一个配置文件Config.ts

```typescript
/**
	 *  Config 用于配置一些全局参数。如需更改，请在初始化引擎之前设置。
	 */
export class Config {
    /**
     * 动画 Animation 的默认播放时间间隔，单位为毫秒。
     */
    static animationInterval: number = 50;
    /**
     * 设置是否抗锯齿，只对2D(WebGL)、3D有效。
     */
    static isAntialias: boolean = false;
    /**
     * 设置画布是否透明，只对2D(WebGL)、3D有效。
     */
    static isAlpha: boolean = false;
    /**
     * 设置画布是否预乘，只对2D(WebGL)、3D有效。
     */
    static premultipliedAlpha: boolean = true;
    /**
     * 设置画布的是否开启模板缓冲，只对2D(WebGL)、3D有效。
     */
    static isStencil: boolean = true;
    /**
     * 是否保留渲染缓冲区。
     */
    static preserveDrawingBuffer: boolean = false;

    /**
     * 当使用webGL渲染2d的时候，每次创建vb是否直接分配足够64k个顶点的缓存。这样可以提高效率。
     */
    static webGL2D_MeshAllocMaxMem: boolean = true;

    /**
     * 是否强制使用像素采样。适用于像素风格游戏
     */
    static is2DPixelArtGame: boolean = false;

    /**
     * 是否使用webgl2
     */
    static useWebGL2: boolean = true;

    static useRetinalCanvas: boolean = false;
}
(window as any).Config = Config;


```
我们可以直接对Config这个类使用Shift+f12(Go to reference)，如这样

![image]({{ "/assets/Laya/layaSourceConfig.jpg" | absolute_url }})

要想读懂源代码，GotoReference不可少，这个config明显的在许多类中被调用过，如Laya.ts，Laya3D.ts，以及AnimationBase、Stage、Render、Mesh2D.ts，而实际上这些类就是Laya引擎的主干（至少目前我们学习时候可以这样看作）。上篇文章，我们介绍了Laya.ts和Laya3D.ts,那么我们这堂课就去讲讲有关动画的AnimationBase类的相关内容。

1.首先是Play方法

```  typescript
    /**
     * <p>开始播放动画。play(...)方法被设计为在创建实例后的任何时候都可以被调用，当相应的资源加载完毕、调用动画帧填充方法(set frames)或者将实例显示在舞台上时，会判断是否正在播放中，如果是，则进行播放。</p>
     * <p>配合wrapMode属性，可设置动画播放顺序类型。</p>
     * @param	start	（可选）指定动画播放开始的索引(int)或帧标签(String)。帧标签可以通过addLabel(...)和removeLabel(...)进行添加和删除。
     * @param	loop	（可选）是否循环播放。
     * @param	name	（可选）动画名称。
     */
	play(start: any = 0, loop: boolean = true, name: string = ""): void {
        this._isPlaying = true;
        this._actionName = name;
        this.index = (typeof (start) == 'string') ? this._getFrameByLabel(<string>start) : start;
        this.loop = loop;
        this._isReverse = this.wrapMode === AnimationBase.WRAP_REVERSE;
        if (this.index == 0 && this._isReverse) {
            this.index = this.count - 1;
        }
        if (this.interval > 0) this.timerLoop(this.interval, this, this._frameLoop, null, true, true);
    }
```

这块注释写的很明白，首先我们设置几个参数，如同`this._isPlaying`,`this._actionName`,最后通过获取start是否=string对象来获取对应的帧索引，这里调用`this._getFrameByLabel(<string>start)`

将string类型的start ，也就是开发者自己定义的 *帧标签* 转换成number对象，我们可以 <font color='red'> ctrl+鼠标左键</font> 点进`_getFrameByLabel`方法中，来查看具体代码

```  typescript
    /**@private */
    protected _getFrameByLabel(label: string): number {
        for (var i: number = 0; i < this._count; i++) {
            var item: any = this._labels[i];
            if (item && ((<any[]>item)).indexOf(label) > -1) return i;
        }
        return 0;
    }
```



这里逻辑很简单，基本上时使用二维数组（暂时可以这样理解）_labels来存放item，item再存放一系列的帧标签，通过for循环判断帧标签处于第几帧。

好的，我们看完继续回到原来的`play`方法，并且ctrl加左键点入被调用`interval`中,我们可以看到一个属性化的对象

```typescript
    /**
     * <p>动画播放的帧间隔时间(单位：毫秒)。默认值依赖于Config.animationInterval=50，通过Config.animationInterval可以修改默认帧间隔时间。</p>
     * <p>要想为某动画设置独立的帧间隔时间，可以使用set interval，注意：如果动画正在播放，设置后会重置帧循环定时器的起始时间为当前时间，也就是说，如果频繁设置interval，会导致动画帧更新的时间间隔会比预想的要慢，甚至不更新。</p>
     */
    get interval(): number {
        return this._interval;
    }

    set interval(value: number) {
        if (this._interval != value) {
            this._frameRateChanged = true;
            this._interval = value;
            if (this._isPlaying && value > 0) {
                this.timerLoop(value, this, this._frameLoop, null, true, true);
            }
        }
    }
```
我们...（待续）

## 2.接下来该怎么做

接下来干的事情就比较枯燥了..我们会更深入的开始理解laya里面各个模块，让大家可以自己能够进行修改，如果大家喜欢我这一期的教程的话，我还会继续努力的。

![image]({{ "/assets/Emoji/CanWeShot.jpg" | absolute_url }})