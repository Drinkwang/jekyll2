---
layout:     post
title:      逆袭，从0开始解读Laya源代码（零）
subtitle:   Laya的H5开发
date:       2020-11-02
author:     BYDrinker
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - 游戏开发，从0开始解读Laya源代码（0）
---


# 逆袭，从0开始解读Laya源代码（0）



* 今天我们来做一件很了不起的事情，那就是从零开始解读Laya的源代码，事情的起因是这样，我一直从事laya相关的工作，有一天我无聊的点开了laya的官方案例，竟发现了源码地址（你早该发现的），于是我二话不说克隆了一份到自己的电脑，经过一番研究过后，我发现只要了解ts/js以及nodejs相关的知识，看懂它的源代码也不是什么难事，所以，让我们开始吧..

首先值得我们参考的教程是这个...



> laya官方案例:https://ldc2.layabox.com/doc/?language=zh&nav=zh-ts-0-3-4





## 1.如何下载laya的引擎源码（编译流程）

### 1.下载源代码，编译并跑起程序

首先，我默认大家已经学会了Git以及laya ts的基本操作，想要了解Laya源代码的不可能对这些没有基本的了解吧，不会吧，不会吧...

当然就算你真的不了解，那我也不能放弃你，这里放出二个我认为不错的了解渠径



>  [廖雪峰的官方Git教程](https://www.liaoxuefeng.com/)  https://www.liaoxuefeng.com/wiki/896043488029600
>
>  [Laya的官方教程案例]( https://ldc2.layabox.com/doc/?language=zh&nav=zh-ts-0-3-0  )  https://ldc2.layabox.com/doc/?language=zh&nav=zh-ts-0-3-0  

看完这些（不看完跟着操作也没什么问题），我们就可以继续我们的源代码之旅拉。首先，我们需要clone Laya 在Github上面的官方示例。



> git clone  https://github.com/layabox/LayaAir 

当然我强烈建议你Fork一下它的项目，然后再克隆下自己仓库的，一方面你修改后能够储存到自己的仓库里，万一哪一天你的电脑因不可抗拒的原因放弃工作且无法治疗的时候，那么你换了新电脑也不会失去原来的工作进度，并且假如哪一天你的开发水平厉害了，能找到Bug并且反馈给开发组也是一件美事。



![image]({{ "/assets/Laya/fork.png" | absolute_url }})

接下来，我们通过 VSCode 打开这个项目，大概可以得到一个这样的目录。如果你的电脑没有安装，建议去官网进行下载..

 https://code.visualstudio.com/



下载好后，我们就可以打开工程来一探究竟了，我们应该会得到一个这样的目录



![image]({{ "/assets/Laya/dictionary.png" | absolute_url }})



是不是看起来很多目录，很多源码，很恐怖？？？

![image]({{ "/assets/Emoji/horrible.jpg" | absolute_url }})





但是不要方，我们可以逐步来分析，首先我们逐一结构一下引擎源码。来画出这样的思维导图...

![image]({{ "/assets/Laya/process.png" | absolute_url }})

- 我们看出这个引擎实际上是被划分成了许许多多的模块，而我们主要关注的则是这个layaAir的文件夹，引擎的具体实现都在这里，并且我们之后的修改也在这个中执行，不过现在提修改引擎有点过早了，我们先来编译下案例看看效果吧。

而Laya 案例编译有二种方法执行，第一种使用rollUp执行，第二种使用f5进行编译，我们这里会把二种方法都介绍下，大家只需要使用其中一种就好。

首先我们来装一下rollup，你需要先安装它

`  npm i rollup -g `



然后 执行 `rollup -c` 

 要注意的是，值得一提的是，如果这个环节报错，那么你可能npm还没有安装相关依赖，不管是f5模式，还是rollup 都需要执行这样的操作

` npm install`

并且如果首次采用rollup编译，编译四五分钟，甚至十几分钟，也是正常的，因电脑性能差异会有所不同，卡住了就多等会，不要直接终止。首次编译缓存后，以后编译就会快起来。

编译结束后，我们看到bin目录下的rollup示例相关的结构如下图所示。

![图](https://official.layabox.com/laya_data/LayaAir2.0/Chinese/FAQ/useGithub/img/8-3.png)

`bundle.js`是整合了引擎与示例的js库，`indexRollUp.html`是rollup调试模式的示例入口文件。

编译之后的运行，推荐采用anywhere启动一个本地的web服务。命令 `anywhere 端口号`，端口号自己定义，不冲突就行，参照如下图所示。

![图](https://official.layabox.com/laya_data/LayaAir2.0/Chinese/FAQ/useGithub/img/8-5.png)

> 如果本地装没有anywhere，可以安装下，命令： `npm i anywhere -g`

web服务运行起来之后。按命令行中的网址在chrome浏览器中访问，点开tsc编译模式或者rollup编译模式对应的html入口文件即可。

另外一种F5就比较简单了，因为laya源文件已经配置好了vscode tsc相关流程，执行完npm安装依赖的` npm install`，直接f5就好



###  2.小试牛刀，初探源码

前面我们学会了编译，现在我们就直接开始看看源代码吧

首先我们打开Src -> layaAir ->Laya.ts,以及Laya3D.ts

前面一堆再Laya.ts里都是一些static配置，我们可以忽略，直接从init开始看起



```typescript
  static init(width: number, height: number, ...plugins): any {

  	if (Laya._isinit) return;

    Laya._isinit = true;

    ArrayBuffer.prototype.slice || (ArrayBuffer.prototype.slice = Laya._arrayBufferSlice);

   	Browser.__init__();



    // 创建主画布

   //这个其实在Render中感觉更合理，但是runtime要求第一个canvas是主画布，所以必须在下面的那个离线画布之前 
	 var mainCanv = Browser.mainCanvas = new HTMLCanvas(true);

    //Render._mainCanvas = mainCanv;

    var style: any = mainCanv.source.style;

    style.position = 'absolute';

    style.top = style.left = "0px";

    style.background = "#000000";

    if (!Browser.onKGMiniGame && !Browser.onAlipayMiniGame) {

      Browser.container.appendChild(mainCanv.source);//xiaosong add

    }



    // 创建离屏画布

    //创建离线画布

    Browser.canvas = new HTMLCanvas(true);

    Browser.context = <CanvasRenderingContext2D>(Browser.canvas.getContext('2d') as any);



    Browser.supportWebAudio = SoundManager.__init__();;

    Browser.supportLocalStorage = LocalStorage.__init__();



    //temp TODO 以后分包

    Laya.systemTimer = new Timer(false);

    systemTimer = Timer.gSysTimer = Laya.systemTimer;

    Laya.startTimer = new Timer(false);

    Laya.physicsTimer = new Timer(false);

    Laya.updateTimer = new Timer(false);

    Laya.lateTimer = new Timer(false);

    Laya.timer = new Timer(false);



    startTimer = ILaya.startTimer = Laya.startTimer;

    lateTimer = ILaya.lateTimer = Laya.lateTimer;

    updateTimer = ILaya.updateTimer = Laya.updateTimer;

    ILaya.systemTimer = Laya.systemTimer;

    timer = ILaya.timer = Laya.timer;

    physicsTimer = ILaya.physicsTimer = Laya.physicsTimer;



    Laya.loader = new LoaderManager();

    ILaya.Laya = Laya;

    loader = ILaya.loader = Laya.loader;



    WeakObject.__init__();

    SceneUtils.__init();

    Mouse.__init__();



    WebGL.inner_enable();

    if (plugins) {

      for (var i = 0, n = plugins.length; i < n; i++) {

        if (plugins[i] && plugins[i].enable) {

          plugins[i].enable();

        }

      }

    }

    if (ILaya.Render.isConchApp) {

      Laya.enableNative();

    }

    Laya.enableWebGLPlus();

    CacheManger.beginCheck();

    stage = Laya.stage = new Stage();

    ILaya.stage = Laya.stage;

    Utils.gStage = Laya.stage;

    URL.rootPath = URL._basePath = Laya._getUrlPath();

    MeshQuadTexture.__int__();

    MeshVG.__init__();

    MeshTexture.__init__();

    Laya.render = new Render(0, 0, Browser.mainCanvas);

    render = Laya.render;

    Laya.stage.size(width, height);

    ((<any>window)).stage = Laya.stage;



    WebGLContext.__init__();

    MeshParticle2D.__init__();

    ShaderCompile.__init__();

    RenderSprite.__init__();

    KeyBoardManager.__init__();

    MouseManager.instance.__init__(Laya.stage, Render.canvas);

    Input.__init__();

    SoundManager.autoStopMusic = true;

    Stat._StatRender = new StatUI();



    Value2D._initone(ShaderDefines2D.TEXTURE2D, TextureSV);

    Value2D._initone(ShaderDefines2D.TEXTURE2D | ShaderDefines2D.FILTERGLOW, TextureSV);

    Value2D._initone(ShaderDefines2D.PRIMITIVE, PrimitiveSV);

    Value2D._initone(ShaderDefines2D.SKINMESH, SkinSV);



    return Render.canvas;

  }
```

大致上就是创建画布，定义一堆Timer对象，然后把一堆模块进行启动，我们再来看看Laya3D.ts的init方法，就能看出更多的东西了

   

```typescript
	private static __init__(width: number, height: number, config: Config3D): void {
		Config.isAntialias = config.isAntialias;
		Config.isAlpha = config.isAlpha;
		Config.premultipliedAlpha = config.premultipliedAlpha;
		Config.isStencil = config.isStencil;

		if (!WebGL.enable()) {
			alert("Laya3D init error,must support webGL!");
			return;
		}

		RunDriver.changeWebGLSize = Laya3D._changeWebGLSize;
		Render.is3DMode = true;
		Laya.init(width, height);
		if (!Render.supportWebGLPlusRendering) {
			LayaGL.instance = WebGLContext.mainContext;
			(<any>LayaGL.instance).createCommandEncoder = function (reserveSize: number = 128, adjustSize: number = 64, isSyncToRenderThread: boolean = false): CommandEncoder {
				return new CommandEncoder(this, reserveSize, adjustSize, isSyncToRenderThread);
			}
		}
		config._multiLighting = config.enableMultiLight && SystemUtils.supportTextureFormat(TextureFormat.R32G32B32A32);

		ILaya3D.Shader3D = Shader3D;
		ILaya3D.Scene3D = Scene3D;
		ILaya3D.MeshRenderStaticBatchManager = MeshRenderStaticBatchManager;
		ILaya3D.MeshRenderDynamicBatchManager = MeshRenderDynamicBatchManager;
		ILaya3D.SubMeshDynamicBatch = SubMeshDynamicBatch;
		ILaya3D.Laya3D = Laya3D;
		ILaya3D.Matrix4x4 = Matrix4x4;

		//函数里面会有判断isConchApp
		Laya3D.enableNative3D();

		VertexElementFormat.__init__();
		VertexMesh.__init__();
		VertexShurikenParticleBillboard.__init__();
		VertexShurikenParticleMesh.__init__();
		VertexPositionTexture0.__init__();
		VertexTrail.__init__();
		VertexPositionTerrain.__init__();
		PixelLineVertex.__init__();
		SubMeshInstanceBatch.__init__();
		SubMeshDynamicBatch.__init__();

		Physics3D._bullet = (window as any).Physics3D;
		if (Physics3D._bullet) {
			StaticPlaneColliderShape.__init__();
			ColliderShape.__init__();
			CompoundColliderShape.__init__();
			PhysicsComponent.__init__();
			PhysicsSimulation.__init__();
			BoxColliderShape.__init__();
			CylinderColliderShape.__init__();
			CharacterController.__init__();
			Rigidbody3D.__init__();
		}

		ShaderInit3D.__init__();
		PBRMaterial.__init__();
		PBRStandardMaterial.__init__();
		PBRSpecularMaterial.__init__();
		SkyPanoramicMaterial.__init__();
		Mesh.__init__();
		PrimitiveMesh.__init__();
		Sprite3D.__init__();
		RenderableSprite3D.__init__();
		MeshSprite3D.__init__();
		SkinnedMeshSprite3D.__init__();
		ShuriKenParticle3D.__init__();
		TrailSprite3D.__init__();
		PostProcess.__init__();
		Scene3D.__init__();
		MeshRenderStaticBatchManager.__init__();

		Material.__initDefine__();
		BaseMaterial.__initDefine__();
		BlinnPhongMaterial.__initDefine__();
		// PBRStandardMaterial.__initDefine__();
		// PBRSpecularMaterial.__initDefine__();
		SkyProceduralMaterial.__initDefine__();
		UnlitMaterial.__initDefine__();
		TrailMaterial.__initDefine__();
		EffectMaterial.__initDefine__();
		WaterPrimaryMaterial.__initDefine__();
		ShurikenParticleMaterial.__initDefine__();
		ExtendTerrainMaterial.__initDefine__();
		PixelLineMaterial.__initDefine__();
		SkyBoxMaterial.__initDefine__();


		Command.__init__();

		//注册类命,解析的时候需要
		ClassUtils.regClass("Laya.SkyPanoramicMaterial", SkyPanoramicMaterial);
		ClassUtils.regClass("Laya.EffectMaterial", EffectMaterial);
		ClassUtils.regClass("Laya.UnlitMaterial", UnlitMaterial);
		ClassUtils.regClass("Laya.BlinnPhongMaterial", BlinnPhongMaterial);
		ClassUtils.regClass("Laya.SkyProceduralMaterial", SkyProceduralMaterial);
		ClassUtils.regClass("Laya.PBRStandardMaterial", PBRStandardMaterial);
		ClassUtils.regClass("Laya.PBRSpecularMaterial", PBRSpecularMaterial);
		ClassUtils.regClass("Laya.SkyBoxMaterial", SkyBoxMaterial);
		ClassUtils.regClass("Laya.WaterPrimaryMaterial", WaterPrimaryMaterial);
		ClassUtils.regClass("Laya.ExtendTerrainMaterial", ExtendTerrainMaterial);
		ClassUtils.regClass("Laya.ShurikenParticleMaterial", ShurikenParticleMaterial);
		ClassUtils.regClass("Laya.TrailMaterial", TrailMaterial);
		ClassUtils.regClass("Laya.PhysicsCollider", PhysicsCollider);
		ClassUtils.regClass("Laya.Rigidbody3D", Rigidbody3D);
		ClassUtils.regClass("Laya.CharacterController", CharacterController);
		ClassUtils.regClass("Laya.Animator", Animator);

		ClassUtils.regClass("PhysicsCollider", PhysicsCollider);
		ClassUtils.regClass("CharacterController", CharacterController);
		ClassUtils.regClass("Animator", Animator);
		ClassUtils.regClass("Rigidbody3D", Rigidbody3D);


		PixelLineMaterial.defaultMaterial = new PixelLineMaterial();
		BlinnPhongMaterial.defaultMaterial = new BlinnPhongMaterial();
		EffectMaterial.defaultMaterial = new EffectMaterial();
		// PBRStandardMaterial.defaultMaterial = new PBRStandardMaterial();
		// PBRSpecularMaterial.defaultMaterial = new PBRSpecularMaterial();
		UnlitMaterial.defaultMaterial = new UnlitMaterial();
		ShurikenParticleMaterial.defaultMaterial = new ShurikenParticleMaterial();
		TrailMaterial.defaultMaterial = new TrailMaterial();
		SkyProceduralMaterial.defaultMaterial = new SkyProceduralMaterial();
		SkyBoxMaterial.defaultMaterial = new SkyBoxMaterial();
		WaterPrimaryMaterial.defaultMaterial = new WaterPrimaryMaterial();

		PixelLineMaterial.defaultMaterial.lock = true;//todo:
		BlinnPhongMaterial.defaultMaterial.lock = true;
		EffectMaterial.defaultMaterial.lock = true;
		// PBRStandardMaterial.defaultMaterial.lock = true;
		// PBRSpecularMaterial.defaultMaterial.lock = true;
		UnlitMaterial.defaultMaterial.lock = true;
		ShurikenParticleMaterial.defaultMaterial.lock = true;
		TrailMaterial.defaultMaterial.lock = true;
		SkyProceduralMaterial.defaultMaterial.lock = true;
		SkyBoxMaterial.defaultMaterial.lock = true;
		WaterPrimaryMaterial.defaultMaterial.lock = true;
		Texture2D.__init__();
		TextureCube.__init__();
		SkyBox.__init__();
		SkyDome.__init__();
		ScreenQuad.__init__();
		ScreenTriangle.__init__();
		FrustumCulling.__init__();
		HalfFloatUtils.__init__();

		var createMap: any = LoaderManager.createMap;
		createMap["lh"] = [Laya3D.HIERARCHY, Scene3DUtils._parse];
		createMap["ls"] = [Laya3D.HIERARCHY, Scene3DUtils._parseScene];
		createMap["lm"] = [Laya3D.MESH, MeshReader._parse];
		createMap["lmat"] = [Laya3D.MATERIAL, Material._parse];
		createMap["jpg"] = [Laya3D.TEXTURE2D, Texture2D._parse];
		createMap["jpeg"] = [Laya3D.TEXTURE2D, Texture2D._parse];
		createMap["bmp"] = [Laya3D.TEXTURE2D, Texture2D._parse];
		createMap["gif"] = [Laya3D.TEXTURE2D, Texture2D._parse];
		createMap["png"] = [Laya3D.TEXTURE2D, Texture2D._parse];
		createMap["dds"] = [Laya3D.TEXTURE2D, Texture2D._parse];
		createMap["ktx"] = [Laya3D.TEXTURE2D, Texture2D._parse];
		createMap["pvr"] = [Laya3D.TEXTURE2D, Texture2D._parse];
		createMap["lani"] = [Laya3D.ANIMATIONCLIP, AnimationClip._parse];
		createMap["lav"] = [Laya3D.AVATAR, Avatar._parse];
		createMap["ltc"] = [Laya3D.TEXTURECUBE, TextureCube._parse];
		createMap["ltcb"] = [Laya3D.TEXTURECUBEBIN, TextureCube._parseBin];

		var parserMap: any = Loader.parserMap;
		parserMap[Laya3D.HIERARCHY] = Laya3D._loadHierarchy;
		parserMap[Laya3D.MESH] = Laya3D._loadMesh;
		parserMap[Laya3D.MATERIAL] = Laya3D._loadMaterial;
		parserMap[Laya3D.TEXTURECUBE] = Laya3D._loadTextureCube;
		parserMap[Laya3D.TEXTURECUBEBIN] = Laya3D._loadTextureCubeBin;
		parserMap[Laya3D.TEXTURE2D] = Laya3D._loadTexture2D;
		parserMap[Laya3D.ANIMATIONCLIP] = Laya3D._loadAnimationClip;
		parserMap[Laya3D.AVATAR] = Laya3D._loadAvatar;
		//parserMap[Laya3D.TERRAINRES] = _loadTerrain;
		//parserMap[Laya3D.TERRAINHEIGHTDATA] = _loadTerrain;

		Laya3D._innerFirstLevelLoaderManager.on(Event.ERROR, null, Laya3D._eventLoadManagerError);
		Laya3D._innerSecondLevelLoaderManager.on(Event.ERROR, null, Laya3D._eventLoadManagerError);
		Laya3D._innerThirdLevelLoaderManager.on(Event.ERROR, null, Laya3D._eventLoadManagerError);
		Laya3D._innerFourthLevelLoaderManager.on(Event.ERROR, null, Laya3D._eventLoadManagerError);
	}
```

同样是启动一堆东西，但与此同时，它会调用这个。

`		Laya.init(width, height);`

这个大概是laya2.0 与1.0的区别，2.0使用webGl来实现3d渲染，并且再这个Laya3D类中会初始化以前的Laya

（laya分为1.0和2.0引擎，2.0之后，引擎逐步支持3d功能）

## 2.接下来该怎么做

接下来干的事情就比较枯燥了..我们会逐步开始理解laya里面各个模块，并把其中的框架梳理清楚，如果大家喜欢我这一期的教程的话，我还会继续努力的。

![image]({{ "/assets/Emoji/CanWeShot.jpg" | absolute_url }})