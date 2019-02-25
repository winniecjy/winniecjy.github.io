---
title: threejs基础上：初始化
date: 2018-10-11 21:15:23
tags: threejs
category: lesson
toc: true
comments: true
description: threejs入门第一步
---

Three.js是一个用于简化webGL编程的3D库，即使在不支持webGL的环境下也能做到优雅降级，以下教程将围绕构建这个地球DEMO来展开。   



ThreeJs中最主要的有三个对象场景（scene）、相机（camera）、渲染器（renderer）。scene是布景空间，camera是拍摄镜头，render是用来将scene和camera生成的场景渲染到屏幕上，有了这三个对象才能将场景渲染到网页上去。   



***



### 初始化对象

#### 1、初始化场景scene

场景相当于是一个容器来容纳所有的物体，创建场景如下：   

`var scene = new THREE.Scene();`   

设置背景scene.background，可以为纯色，也可以为图片资源：

`scene.background = new THREE.Color(0xffffff)`



#### 2、相机camera

THREE中的camera有三种，最常用的是远景相机，也就是人眼观察世界的模式，在相机拍摄的3D空间之外的物体不会被渲染。

`var camera = new THREE.PespectiveCamera(fov, aspect, near,far);`


| 参数 | 类型 | 默认值 | 说明|
| --- | --- | --- | --- |
| fov |number | 50 | 视野角度 |
| aspect | number | 1 | 实际窗口的宽高比 |
| near | number | 0.1 | 相机到近平面的距离 |
| far | number | 2000 | 相机到远平面的距离 |


#### 3、初始化渲染器renderer
创建一个WebGL渲染器，可以通过插件`Detector.js`检测canvas/webgl兼容性，并在页面添加不兼容信息。   
```JavaScript
var renderer = null;
if(Detector.webgl)
    renderer = new THREE.WebGLRenderer({antialias: true})
else if(Detector.canvas)
    renderer = new THREE.CanvasRenderer()      
else 
    console.log('不兼容')  
```
   
antialias属性开启用于抗锯齿。初始化渲染器有一些[其他参数](http://techbrood.com/threejs/docs/#%E5%8F%82%E8%80%83%E6%89%8B%E5%86%8C/%E6%B8%B2%E6%9F%93%E5%99%A8%28Renderers%29/WebGL%E6%B8%B2%E6%9F%93%E5%99%A8%28WebGLRenderer%29)，一般不需要设置使用默认值即可。   

      
##### setSize(width, height, updateStyle)

调整输出canvas尺寸（宽度，高度），通常设置为屏幕大小，若不是则需考虑设备像素比并设置视口（viewport）以匹配尺寸，updateStyle为true时，则显式添加像素输出canvas的样式中，默认为false。

##### setViewport(x, y, width, height)

设置视口，从(x, y)到(x+width, y+height)，***注意：**此处的(x, y)是该区域的左下角*

##### renderer.domElement

渲染器renderer的domElement元素表示渲染器中的画布，通常需要将domElement挂载到body下面，渲染的结果就能正确显示了。   

`document.body.appendChild(renderer.domElement)`;   

##### setPixelRatio(value)

设置设备像素比，用于HiDPI设备防止模糊输出canvas。

`renderer.setPixelRatio(window.devicePixelRatio)`



到这里，我们已经构建好了基本的“舞台”，就差“演员”上场了~



***



### 展示一个球体

#### 1. 创建物体

##### Mesh对象

可以通过类似以下方式创建：   
```JavaScript
var geometry = new THREE.SphereGeometry(15, 10, 10); /* 几何模型，用于定义结构 */
var meterial = new THREE.MeshNormalMeterial(); /* 材质，用于定义外观 */
var earth = new THREE.Mesh(geometry, meterial); 
```

一个普通的物体对象的构成需要两个参数。
- **Geometry**：在threejs中有两种几何体（基本几何体和buffer几何体）。   
***基本几何体*** 是通过类来管理自身信息，比如顶点位置、颜色等，便于操作，用于创建经常变化的物体；   
***buffer几何体*** 是用数组存储的，且保存在内存缓存区，减低对GPU的消耗，适用于一经创建就不需要修改的物体。   
不同的集合
- **Meterial**：不同材质的显示效果不同，可以在具体使用到的时候再进行查看，以下给出简单地描述，在需要使用的时候才去看具体的材质即可：   

| 材质 | 描述 |
| --- | --- |
| Material | 材料基类 |
| MeshBasicMaterial | 基于深度着色的材质，由物体和相机的距离决定颜色，可以做出逐渐消失的效果 |
| MeshDepthMaterial | 渲染成简单的平面多边形，不考虑光照的影响 |
| MeshNormalMaterial | 计算法向量颜色的材质 |
| MeshLambertMaterial | 对光源做出反应，可用于创建暗淡的材质 |
| MeshPhongMaterial | 对光源做出反应，可用于创建光亮的材质 |
| MeshFaceMaterial | 为几何体的每一面指定材质，更像是一种材质容器 |
| THREE.SceneUtil.createMultiMaterialObject | 同时应用多种材质 |
| ShaderMaterial | 自己创建着色程序，需要通过GLSL语言 |
| LineBasicMaterial | 只能应用于THREE.Line |
| LineDashedMaterial | 只能应用于THREE.Line |



 附：Meterial的[所有通用属性](http://techbrood.com/threejs/docs/#%E5%8F%82%E8%80%83%E6%89%8B%E5%86%8C/%E6%9D%90%E6%96%99%28Materials%29/%E6%9D%90%E6%96%99%28Material%29)，其中需要注意的有：   

- overdraw：解决绘制三角形时出现间隙，当设置为0.5时效果较好



#### 2. 添加到场景

将物体添加到场景中：   

`scene.add(earth);`

注意通过scene.add方法添加到场景的位置默认为原点(0,0,0)，此时跟相机位置是重叠的，需要移动一下相机的位置才能够看到物体。

`camera.position.z = 50;`

#### 3. 渲染页面

`renderer.render(scene, camera);`   

这样我们就简单地渲染出了一个带有静态球体的场景，由于此处使用的材质只是一个颜色，所以添加了经纬线使他看起来更像球体。    

***


### \**附表：对象的通用属性/函数*
以下给出3D对象的通用属性与函数，属性值可以直接通过console.log查看。

| 属性/函数 | 描述 |
| --- | --- |
| position | 决定对象相对于其父对象的位置，大部分情况下一个对象的父对象是THREE.Scene()对象 |
| rotation | 对象的局部旋转，单位为弧度 |
| scale | 控制对象的缩放 |
| up | 空间向上的方向，缺省是THREE.Vector3(0, 1, 0) |
| translateX/ranslateY/ranslateZ(distance) | 沿X/Y/Z轴平移对象 |
| rotateX/rotateY/rotateZ(rad) | 沿X/Y/Z轴平移对象 |
| lookAt(vector) | 一个世界向量观察点，用于旋转模型以面对观察点 |
| add(object, ...) | 添加object为该对象的子对象 |
| remove(object, ...) | 删除object子对象 |
| clone(recursive) | 克隆对象，当recursive为true时（默认为true），对象的后代也会被克隆 |

以上属性可以通过`obj.[attr].x/obj.[attr].y/obj.[attr].z`来设置，或者是一次性设置3个值`obj.[attr].set(x, y, z)` 