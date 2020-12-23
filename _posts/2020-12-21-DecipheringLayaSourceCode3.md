---
layout:     post
title:      逆袭，从0开始解读Laya源代码（三）
subtitle:   Laya的H5开发
date:       2020-12-21
author:     BYDrinker
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - 游戏开发，从0开始解读Laya源代码
---


# 逆袭，从0开始解读Laya源代码（3）



* 上一讲我们讲解了Laya的Mesh的模块，我抛出了个观点，也就是Laya实际中3d内容，也就是在2d引擎的基础上额外封装一套Webgl。但是Mesh的源码分析不够直观(代码太多，太难了..555)，我们通过分析一个轻量点模块来进行学习，也就是SkyBox模块

前面的教程： [https://drinker.site/2020/11/02/DecipheringLayaSourceCode0.html](https://drinker.site/2020/11/02/DecipheringLayaSourceCode0.html)

 			[https://drinker.site/2020/11/09/DecipheringLayaSourceCode1.html](https://drinker.site/2020/11/09/DecipheringLayaSourceCode1.html)

 			[https://drinker.site/2020/11/09/DecipheringLayaSourceCode2.html](https://drinker.site/2020/11/09/DecipheringLayaSourceCode2.html) 


> laya官方案例:https://ldc2.layabox.com/doc/?language=zh&nav=zh-ts-0-3-4



## 1.研究 Skybox Laya源代码

### 1.了解SkyBox的导图

![image]({{ "/assets/Laya/skybox.png" | absolute_url }})

###  2.了解SkyBox，天空盒子是怎么实现的

这个类较为简单，整个模块就分成三个方法，__init__、constructor、_render,而且总行数不超过80行，最方便下手解析了，我们先来看看`__init__`这个方法

```typescript
	static instance: SkyBox;

	/**
	 * @internal
	 */
	static __init__(): void {
		SkyBox.instance = new SkyBox();//TODO:移植为标准Mesh后需要加锁
	}
```
最为基本的一个单例模式，没啥好说的，再来看看它的构造方法吧，不过为了便于理解，我们先放出用WebGl渲染四边形的代码


```javascript

onload = function(){
    // canvas对象获取
    var c = document.getElementById('canvas');
    c.width = 300;
    c.height = 300;
 
    // webgl的context获取
    var gl = c.getContext('webgl') || c.getContext('experimental-webgl');
    
    // 设定canvas初始化的颜色
    gl.clearColor(0.0, 0.0, 0.0, 1.0);
    
    // 设定canvas初始化时候的深度
    gl.clearDepth(1.0);
    
    // canvas的初始化
    gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT);
    
    // 顶点着色器和片段着色器的生成
    var v_shader = create_shader('vs');
    var f_shader = create_shader('fs');
    
    // 程序对象的生成和连接
    var prg = create_program(v_shader, f_shader);
    
    // attributeLocation的获取
    var attLocation = gl.getAttribLocation(prg, 'position');
    
    // attribute的元素数量(这次只使用 xyz ，所以是3)
    var attStride = 3;
    
    // 模型（顶点）数据
    var vertex_position = [
         0.0, 1.0, 0.0,
         1.0, 0.0, 0.0,
        -1.0, 0.0, 0.0
    ];
    
    // 生成VBO
    var vbo = create_vbo(vertex_position);
    
    // 绑定VBO
    gl.bindBuffer(gl.ARRAY_BUFFER, vbo);
    
    // 设定attribute属性有効
    gl.enableVertexAttribArray(attLocation);
    
    // 添加attribute属性
    gl.vertexAttribPointer(attLocation, attStride, gl.FLOAT, false, 0, 0);
    
    // 使用minMatrix.js对矩阵的相关处理
    // matIV对象生成
    var m = new matIV();
    
    // 各种矩阵的生成和初始化
    var mMatrix = m.identity(m.create());
    var vMatrix = m.identity(m.create());
    var pMatrix = m.identity(m.create());
    var mvpMatrix = m.identity(m.create());
    
    // 视图变换坐标矩阵
    m.lookAt([0.0, 1.0, 3.0], [0, 0, 0], [0, 1, 0], vMatrix);
    
    // 投影坐标变换矩阵
    m.perspective(90, c.width / c.height, 0.1, 100, pMatrix);
    
    // 各矩阵想成，得到最终的坐标变换矩阵
    m.multiply(pMatrix, vMatrix, mvpMatrix);
    m.multiply(mvpMatrix, mMatrix, mvpMatrix);
    
    // uniformLocation的获取
    var uniLocation = gl.getUniformLocation(prg, 'mvpMatrix');
    
    // 向uniformLocation中传入坐标变换矩阵
    gl.uniformMatrix4fv(uniLocation, false, mvpMatrix);
    
    // 绘制模型
    gl.drawArrays(gl.TRIANGLES, 0, 3);
    
    // context的刷新
    gl.flush();
    
  
```
然后就是Laya中天空盒的代码

```typescript
		super();
		var gl: WebGLRenderingContext = LayaGL.instance;
		var halfHeight: number = 1.0;
		var halfWidth: number = 1.0;
		var halfDepth: number = 1.0;
		var vertices: Float32Array = new Float32Array([-halfDepth, halfHeight, -halfWidth, halfDepth, halfHeight, -halfWidth, halfDepth, halfHeight, halfWidth, -halfDepth, halfHeight, halfWidth,//上
		-halfDepth, -halfHeight, -halfWidth, halfDepth, -halfHeight, -halfWidth, halfDepth, -halfHeight, halfWidth, -halfDepth, -halfHeight, halfWidth]);//下
		var indices: Uint8Array = new Uint8Array([
			0, 1, 2, 2, 3, 0, //上
			4, 7, 6, 6, 5, 4, //下
			0, 3, 7, 7, 4, 0, //左
			1, 5, 6, 6, 2, 1,//右
			3, 2, 6, 6, 7, 3, //前
			0, 4, 5, 5, 1, 0]);//后

		var verDec: VertexDeclaration = VertexMesh.getVertexDeclaration("POSITION");
		this._vertexBuffer = new VertexBuffer3D(verDec.vertexStride * 8, gl.STATIC_DRAW, false);
		this._vertexBuffer.vertexDeclaration = verDec;
		this._indexBuffer = new IndexBuffer3D(IndexFormat.UInt8, 36, gl.STATIC_DRAW, false);
		this._vertexBuffer.setData(vertices.buffer);
		this._indexBuffer.setData(indices);

		var bufferState: BufferState = new BufferState();
		bufferState.bind();
		bufferState.applyVertexBuffer(this._vertexBuffer);
		bufferState.applyIndexBuffer(this._indexBuffer);
		bufferState.unBind();
		this._bufferState = bufferState;
```
基本上除了变换空间，Laya中的天空盒代码跟WebGl渲染面片有很大的一致性，换言之，Laya是对Webgl进行了一层封装，然后再利用封装好的方法去实现具体功能，比如天空盒之类的。至于为什么没有变换空间的代码，那是因为这部分的代码被封装到相机的功能里去了。

![image]({{ "/assets/Laya/skyBoxCamera.png" | absolute_url }})

对比代码，WebGlcontext的获取`    // webgl的context获取
    var gl = c.getContext('webgl') || c.getContext('experimental-webgl');`

再laya里面则利用封装好的方法

`var gl: WebGLRenderingContext = LayaGL.instance;`

我们可以进行断点，查看里面的具体实现

![image]({{ "/assets/Laya/layaGL.png" | absolute_url }})

最后就是把该绑定的进行绑定，这些等同于生成vbo和绑定vbo的过程，然后一切准备就绪就是调用`_render`方法,我们可以查看下其中的引用
![image]({{ "/assets/Laya/skyboxMesh.png" | absolute_url }})
这块就与我们之前上一讲所说的Mesh是一样的，也就是Mesh和MeshRender调用具体WebGl的render，这里不多叙述，直接来看看_render的代码




```typescript
		var gl: WebGLRenderingContext = LayaGL.instance;
		gl.drawElements(gl.TRIANGLES, 36, gl.UNSIGNED_BYTE, 0);
		Stat.trianglesFaces += 12;
		Stat.renderBatches++;
```

这些代码和WebGl里渲染的代码如出一辙

```javascript
var uniLocation = gl.getUniformLocation(prg, 'mvpMatrix');

// 向uniformLocation中传入坐标变换矩阵
gl.uniformMatrix4fv(uniLocation, false, mvpMatrix);

// 绘制模型
gl.drawArrays(gl.TRIANGLES, 0, 3);
```


## 2.接下来该怎么做

待续
