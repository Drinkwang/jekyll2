---
layout:     post
title:      逆袭，从0开始解读Laya源代码（二）
subtitle:   Laya的H5开发
date:       2020-11-16
author:     BYDrinker
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - 游戏开发，从0开始解读Laya源代码
---


# 逆袭，从0开始解读Laya源代码（2）



* 上一讲我们讲解了Laya的Animation的模块，这一期界面我们将对Mesh.ts做更多考量

前面的教程： https://drinker.site/2020/11/02/DecipheringLayaSourceCode0.html 

 					   https://drinker.site/2020/11/09/DecipheringLayaSourceCode1.html 



> laya官方案例:https://ldc2.layabox.com/doc/?language=zh&nav=zh-ts-0-3-4



## 1.研究Laya代码

### 1.了解框架的思维导图

![image]({{ "/assets/Laya/process2.png" | absolute_url }})

###  2.了解Mesh，以及laya渲染3d的流程

首先我们还是来看看定义的一些属性类

```typescript
  /**Mesh资源。*/

  static MESH: string = "MESH";



  /** @internal */

  private _tempVector30: Vector3 = new Vector3()

  /** @internal */

  private _tempVector31: Vector3 = new Vector3();

  /** @internal */

  private _tempVector32: Vector3 = new Vector3();

  /** @internal */

  private static _nativeTempVector30: number;

  /** @internal */

  private static _nativeTempVector31: number;

  /** @internal */

  private static _nativeTempVector32: number;、


```



前面是定义_tempVector30、_tempVector31、\_tempVector32、\_nativeTempVector30、\_nativeTempVector31、\_nativeTempVector32这么一些顶点类，后面nativeTempVector将在init中被使用上

```typescript
	/**
 	* @internal
 	*/
	static __init__(): void {
		var physics3D: any = Physics3D._bullet;
		if (physics3D) {
			Mesh._nativeTempVector30 = physics3D.btVector3_create(0, 0, 0);
			Mesh._nativeTempVector31 = physics3D.btVector3_create(0, 0, 0);
			Mesh._nativeTempVector32 = physics3D.btVector3_create(0, 0, 0);
		}
	}
```

这里是一系列的初始化，但是并没有具体实际意义的值，所以我们就是逐一查看引用，先查出_nativeTempVector30的引用方法，我们会发现

```typescript
	/**
	 * @internal
	 */
	_getPhysicMesh(): any {
		if (!this._btTriangleMesh) {
			var bt: any = Physics3D._bullet;
			var triangleMesh: number = bt.btTriangleMesh_create();//TODO:独立抽象btTriangleMesh,增加内存复用
			var nativePositio0: number = Mesh._nativeTempVector30;
			var nativePositio1: number = Mesh._nativeTempVector31;
			var nativePositio2: number = Mesh._nativeTempVector32;
			var position0: Vector3 = this._tempVector30;
			var position1: Vector3 = this._tempVector31;
			var position2: Vector3 = this._tempVector32;

			var vertexBuffer: VertexBuffer3D = this._vertexBuffer;
			var positionElement: VertexElement = this._getPositionElement(vertexBuffer);
			var verticesData: Float32Array = vertexBuffer.getFloat32Data();
			var floatCount: number = vertexBuffer.vertexDeclaration.vertexStride / 4;
			var posOffset: number = positionElement._offset / 4;

			var indices: Uint16Array = this._indexBuffer.getData();//TODO:API修改问题
			for (var i: number = 0, n: number = indices.length; i < n; i += 3) {
				var p0Index: number = indices[i] * floatCount + posOffset;
				var p1Index: number = indices[i + 1] * floatCount + posOffset;
				var p2Index: number = indices[i + 2] * floatCount + posOffset;
				position0.setValue(verticesData[p0Index], verticesData[p0Index + 1], verticesData[p0Index + 2]);
				position1.setValue(verticesData[p1Index], verticesData[p1Index + 1], verticesData[p1Index + 2]);
				position2.setValue(verticesData[p2Index], verticesData[p2Index + 1], verticesData[p2Index + 2]);

				Utils3D._convertToBulletVec3(position0, nativePositio0, true);
				Utils3D._convertToBulletVec3(position1, nativePositio1, true);
				Utils3D._convertToBulletVec3(position2, nativePositio2, true);
				bt.btTriangleMesh_addTriangle(triangleMesh, nativePositio0, nativePositio1, nativePositio2, true);
			}
			this._btTriangleMesh = triangleMesh;
		}
		return this._btTriangleMesh;
	}
```

这里的东西涉及到许多webGl的内容，当然，这里调用都是黑盒里面的类



待续

![image]({{ "/assets/Emoji/CanWeShot.jpg" | absolute_url }})



