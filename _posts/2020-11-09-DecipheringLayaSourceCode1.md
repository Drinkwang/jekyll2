---
layout:     post
title:      逆袭，从0开始解读Laya源代码（一）
subtitle:   Laya的H5开发
date:       2020-11-09
author:     BYDrinker
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - 游戏开发，从0开始解读Laya源代码
---


# 逆袭，从0开始解读Laya源代码（1）



* 上一讲我们说道如何编译laya源代码，以及做了一些基础性的研究，那么这一期界面我们将进一步做更多考量

前面的教程： [https://drinker.site/2020/11/02/DecipheringLayaSourceCode0.html](https://drinker.site/2020/11/02/DecipheringLayaSourceCode0.html) 



> laya官方案例:[https://ldc2.layabox.com/doc/?language=zh&nav=zh-ts-0-3-4](https://ldc2.layabox.com/doc/?language=zh&nav=zh-ts-0-3-4) 



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
我们可以直接对Config这个类使用<font color='red'> Shift+f12</font>(Go to reference-跳转引用)，如这样

![image]({{ "/assets/Laya/layaSourceConfig.png" | absolute_url }})

要想读懂源代码，*跳转引用* 必不可少，这个config明显的在许多类中被调用过，如Laya.ts，Laya3D.ts，以及AnimationBase、Stage、Render、Mesh2D.ts，而这些类就是Laya引擎的主干（至少目前我们学习时候可以这样看作）。上篇文章，我们介绍了Laya.ts和Laya3D.ts,那么我们这堂课就去讲讲有关动画的`AnimationBase`类的相关内容。

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
如果大家现在还对Laya开发还有印象的话，Laya的动画类也是可以用Laya.interval来控制动画播放的帧间隔时间，这里引擎开发和具体使用上终于产生了微妙的联系...  而现在，在了解上述配置后，我们可以来看下`_frameLoop`这个放在定时器里的方法

```typescript
    /**@private */
    protected _frameLoop(): void {
        if (this._isReverse) {
            this._index--;
            if (this._index < 0) {
                if (this.loop) {
                    if (this.wrapMode == AnimationBase.WRAP_PINGPONG) {
                        this._index = this._count > 0 ? 1 : 0;
                        this._isReverse = false;
                    } else {
                        this._index = this._count - 1;
                    }
                    this.event(Event.COMPLETE);
                } else {
                    this._index = 0;
                    this.stop();
                    this.event(Event.COMPLETE);
                    return;
                }
            }
        } else {
            this._index++;
            if (this._index >= this._count) {
                if (this.loop) {
                    if (this.wrapMode == AnimationBase.WRAP_PINGPONG) {
                        this._index = this._count - 2 >= 0 ? this._count - 2 : 0;
                        this._isReverse = true;
                    } else {
                        this._index = 0;
                    }
                    this.event(Event.COMPLETE);
                } else {
                    this._index--;
                    this.stop();
                    this.event(Event.COMPLETE);
                    return;
                }
            }
        }
        this.index = this._index;
    }

```

​	首先`this._isReverse`判断是否逆序播放，如果是的话，每一次time都将index--，并且在`this.loop==true`的时候，判断`warpMode`是否等于`AnimationBase.WARP_PINGPONG`,如果等于Pingpong，那么就需要每次将`this._isReverse`进行取反，具体表现在代码里，就是如果如果是逆序就把this._isReverse=false,如果不是就等于true。

最后如果动画不是Loop的话，会在动画播放完毕时候，调用this.event，这个事件也会在其他地方进行注册。（例如`Skeleton`类）

我们看完这个类之后继续 <font color='red'> shift+f12</font> 打开`AnimationBase`查看相关的引用，继续强调，查看引用和观看堆栈，是读懂源代码和调试的绝对神器。

![image]({{ "/assets/Laya/animationBase.png" | absolute_url }})

我们看到有二个类，我们先来看看`Animation`这个类,它继承了`AnimationBase`,我们看看它重写的父类Play

```typescript
    play(start: any = 0, loop: boolean = true, name: string = ""): void {
        if (name) this._setFramesFromCache(name, true);
        super.play(start, loop, name);
    }

```

很基本的一个调用，不过它做了一些额外的工作，<font color='red'> if</font>判断中把该动画的名称从模版缓存池中拿出并解析。它是先从缓存池中提取的，而不是直接使用，这样能通过空间换取时间，提升性能

```typescript
    /**@private */
    protected _setFramesFromCache(name: string, showWarn: boolean = false): boolean {
        if (this._url) name = this._url + "#" + name;
        if (name && Animation.framesMap[name]) {
            var tAniO: any = Animation.framesMap[name];
            if (tAniO instanceof Array) {
                this._frames = Animation.framesMap[name];
                this._count = this._frames.length;
            } else {
                if (tAniO.nodeRoot) {
                    //如果动画数据未解析过,则先进行解析
                    Animation.framesMap[name] = GraphicAnimation.parseAnimationByData(tAniO);
                    tAniO = Animation.framesMap[name];
                }
                this._frames = tAniO.frames;
                this._count = this._frames.length;
                //如果读取的是动画配置信息，帧率按照动画设置的帧率播放
                if (!this._frameRateChanged) this._interval = tAniO.interval;
                this._labels = this._copyLabels(tAniO.labels);
            }
            return true;
        } else {
            if (showWarn) console.log("ani not found:", name);
        }
        return false;
    }

```

然后我们再来理解就是framesMap的调用逻辑了， 按下<font color='red'>shift+f12</font> ，我们找到首次赋值的逻辑

![image]({{ "/assets/Laya/framesMap.png" | absolute_url }})

找到这些逻辑后，我们来依次看看具体的实现，注意看二个方法分别是被命名为`_loadAnimationData`和`createFrames`,所以可以看出这二个都是实例化或者载入动画数据用的，也就是在此处进行`frameMap`的初始化

（理解思路就好，代码可以选择性观看）

```typescript
    private _loadAnimationData(url: string, loaded: Handler = null, atlas: string = null): void {
        if (atlas && !Loader.getAtlas(atlas)) {
            console.warn("atlas load fail:" + atlas);
            return;
        }
        var _this: Animation = this;

        function onLoaded(loadUrl: string): void {
            if (!Loader.getRes(loadUrl)) {
                // 如果getRes失败了，有可能是相同的文件已经被删掉了，因为下面在用完后会立即删除
                // 这时候可以取frameMap中去找，如果找到了，走正常流程。--王伟
                if (Animation.framesMap[url + "#"]) {
                    _this._setFramesFromCache(_this._actionName, true);
                    _this.index = 0;
                    _this._resumePlay();
                    if (loaded) loaded.run();
                }
                return;
            }
            if (url === loadUrl) {
                var tAniO: any;
                if (!Animation.framesMap[url + "#"]) {
                    //此次解析仅返回动画数据，并不真正解析动画graphic数据
                    var aniData: any = GraphicAnimation.parseAnimationData(Loader.getRes(url));
                    if (!aniData) return;
                    //缓存动画数据
                    var aniList: any[] = aniData.animationList;
                    var i: number, len: number = aniList.length;
                    var defaultO: any;
                    for (i = 0; i < len; i++) {
                        tAniO = aniList[i];
                        Animation.framesMap[url + "#" + tAniO.name] = tAniO;
                        if (!defaultO) defaultO = tAniO;
                    }
                    if (defaultO) {
                        Animation.framesMap[url + "#"] = defaultO;
                        _this._setFramesFromCache(_this._actionName, true);
                        _this.index = 0;
                    }
                    _this._resumePlay();
                } else {
                    _this._setFramesFromCache(_this._actionName, true);
                    _this.index = 0;
                    _this._resumePlay();
                }
                if (loaded) loaded.run();
            }
            //清理掉配置
            Loader.clearRes(url);
        }
        if (Loader.getRes(url)) onLoaded(url);
        else ILaya.loader.load(url, Handler.create(null, onLoaded, [url]), null, Loader.JSON);


    }
```

```typescript
    static createFrames(url: string | string[], name: string): any[] {
        var arr: any[];
        if (typeof (url) == 'string') {
            var atlas: any[] = Loader.getAtlas(<string>url);
            if (atlas && atlas.length) {
                arr = [];
                for (var i: number = 0, n: number = atlas.length; i < n; i++) {
                    var g: Graphics = new Graphics();
                    g.drawImage(Loader.getRes(atlas[i]), 0, 0);
                    arr.push(g);
                }
            }
        } else if (url instanceof Array) {
            arr = [];
            for (i = 0, n = url.length; i < n; i++) {
                g = new Graphics();
                g.loadImage(url[i], 0, 0);
                arr.push(g);
            }
        }
        if (name) Animation.framesMap[name] = arr;
        return arr;
    }
```

看到这里，你应该可以理清`Animation`这个类，我们接下来看看继承`AnimationBase`的第二个类，也就是`FrameAnimation`,帧动画具体对结点做操作，我们也把重点放在这个上面

``` 
    protected _displayToIndex(value: number): void {
        if (!this._animationData) return;
        if (value < 0) value = 0;
        if (value > this._count) value = this._count;
        var nodes: any[] = this._animationData.nodes, i: number, len: number = nodes.length;
        for (i = 0; i < len; i++) {
            this._displayNodeToFrame(nodes[i], value);
        }
    }
```

```
     * @private
     * 计算节点某个属性的帧数据
     */
    private _calculateNodePropFrames(keyframes: any[], frames: any[], key: string, target: number): void {
        var i: number, len: number = keyframes.length - 1;
        frames.length = keyframes[len].index + 1;
        for (i = 0; i < len; i++) {
            this._dealKeyFrame(keyframes[i]);
            this._calculateFrameValues(keyframes[i], keyframes[i + 1], frames);
        }
        if (len == 0) {
            frames[0] = keyframes[0].value;
            if (this._usedFrames) this._usedFrames[keyframes[0].index] = true;
        }
        this._dealKeyFrame(keyframes[i]);
    }

```

可以看出，这个基本上是对结点数组中进行计算和插值，我们就不做过多复述，如果你能够理解Animation和AnimationBase，按照我所说的方式分析其中的源代码也不难，这个就作为课外作业留给各位了...

## 2.接下来该怎么做

blog做到第二期，有点不敢相信..下一期我们将介绍下Mesh相关的逻辑，本人才疏浅学，很有可能一些内容并不够完善，也希望大家能给我一些意见，再次感谢大家捧场，多谢

![image]({{ "/assets/Emoji/CanWeShot.jpg" | absolute_url }})



