---
title: threejs基础下：贴图资源加载与光照
date: 2018-10-11 21:16:01
tags: threejs
category: lesson
toc: true
comments: true
description: threejs加载器，贴图
---


### 加载器Loaders
加载器是threejs中很重要的一个步骤，可以用于加载纹理/图片/模型/音频等资源，不同的loader对应不同格式的文件，loaders通用流程如下：
```
var loader = new THREE.[Loader]();
/* 
 * 函数名：.load(url, onLoad, onProgress, onError)
 * url：资源地址
 * onLoad: 加载完成的回调，参数是已加载的资源文本
 * onProgress: 加载中的调用，参数是XmlHttpRequest实例
 * onError：加载出错时调用
*/
loader.load(url, onLoad, onProgress, onError)
```

#### 1. TextureLoader/ImageLoader
加载图片资源，可以作为贴图(map)覆盖在物体上或者直接绘制在canvas上。
```
var loader = new THREE.TextureLoader();
loader.load('texture/earth.jpg', function( texture ) {
    // 作为纹理，或直接使用TextureLoader
    // var geometry = new THREE.SphereGeometry(15, 10, 10);
    // var meterial = new THREE.MeshBasicMaterial({color: 0x739783, map:texture});
    // earth = new THREE.Mesh(geometry, material);
    // scene.add(earth);
    // 直接绘制在canvas上
    var canvas =document.createElement('canvas');
    var context = canvas.getContext('2d');
    context.drawImage(texture, 100, 100);    
})
```


#### 2. AudioLoader
加载音乐也是常见的需求，加载音乐并播放的实现方法如下：
```
// 初始化侦听器并添加到相机中
var listener = new THREE.AudioListener();
camera.add(listener);
// 实例化一个音频对象并添加到场景中
var audio = new THREE.audio( audioListener );
scene.add(audio);
// 加载资源
var loader = new THREE.AudioLoader();
loader.load( 'audio/amobient.ogg', function(audioBuffer) {
    audio.setBuffer(audioBuffer);
    audio.play();
},
function(xhr) {
    console.log( (xhr.loaded / xhr.total * 100) + "% is loaded" );
},
function(xhr) {
    console.log('An error happened');
})
```
音频可以做到与场景相关，比如说做出3D音效，或者是让音乐可视化。在这里暂时不讨论，在之后使用到的时候再进行具体研究。

#### 3. JSONLoader
实际项目中的物体可能十分复杂，不可以简单地用几何体实现，3D模型导出的JSON文件即可以存放物体的模型，也可以存放其材质和动画信息。解析一个 JSON 结构的数据并返回一个 object ，包含解析后的 geometry 和 materials。
```
var loader = new THREE.JSONLoader();
loader.load(
    './monster/monster.js',
    function (geometry, materials) {
        var material = new THREE.MultiMaterial(materials);
        var object = new THREE.Mesh(geometry, material);
        scene.add(object);
    }
)
```

**注意**：
- 使用加载的资源时，要适时重新渲染，否则无法生效。
- 存在跨域问题。

***

### 贴图详解
大部分贴图都是通过材质（Meterial）中的属性来应用到模型上，不同材质支持的贴图不同，需要具体问题具体分析，以下列出目前使用过的贴图介绍。      
#### 1. 凹凸贴图（Bump Map）
用于给模型增加立体感，实际上并没有改变模型的形状，而是通过模型表面的阴影来达到凹凸的效果，使用方法如下：   
```
var meterial = new THREE.MeshPhongMaterial({
    bumpMap: textureLoader.load('./img_bump.jpg')
})
```

#### 2. 漫反射贴图（Diffuse Map）
用于表现物体表面的反射和表面颜色，即表现出物体被光照射到而显现的颜色和强度。漫反射贴图可以反映出物体的固有色及纹理，还有贴图上的光影。对于由几个模型拼接成的模型来说，光影是不必要的，因为通过打光就可以实现光影效果了，比如由许多砖模型拼凑成的一道墙。而对于一个整体，比如说一道墙的模型，砖块是由贴图实现的，那么在砖缝上绘制投影就很有必要了。


#### 3. 高光贴图（Specular Map）
用来表现当光线照到模型表面时其表面属性，不同材质反射光的强度不同。越偏向RGB(0,0,0)的部分高光越弱，越偏向RGB(255,255,255)的部分高光越强。高光贴图需要与凹凸贴图和漫反射贴图配合使用，展现的材质才会趋近于真实世界。
```
var meterial = new THREE.MeshPhongMaterial({
    specular: 0x404040, // 高光颜色
    shininess: 5,              // 高光平滑度，默认30，值越高越强烈
    specularMap: textureLoader.load('./img_spec.jpg')
})
```

#### 4. 环境贴图（Cube Map）
通过一个虚拟的立方体包围住物体，通过上下左右前后6张图来模拟真实环境，threejs将这些图片渲染成无缝环境盒子。
```
// 立方体环境，顺序是前后上下右左
var cubeTexture = new THREE.CubeTextureLoader().setPath('../img/skyBox').load({
    'px.jpg', 'nx.jpg',
    'py.jpg', 'ny.jpg',
    'pz.jpg', 'nz.jpg'
})
scene.background = cubeTexture;
```

***

### 光源Lights
为了让图片看起来更加立体真实，通常需要增加一些光线。
**注意：**只有部分的材质会受到光照的影响，比如MeshPhongMaterial、MeshLambertMaterial等，如MeshBasicMaterial是不会受到光照影响的。 

#### 1. 环境光（AmbientLight）
这种光应用到全局范围内的所有对象，可用于提高全局亮度，弱化阴影，给全局添加一个基调色。  
```
var light = new THREE.AmbientLight(color, instensity); // 创建一个给定颜色和强度的环境光
scene.add(light);
```

#### 2. 平行光（DirectionalLight）
产生平行的光线，当材质为`MeshLambertMaterial`或`MeshPhongMaterial`时才会受到影响。
```
var light = new THREE.DirectionalLight(hex, instensity)
```


#### 3. 点光源（PointLight）
```
/*
* hex：颜色的RGB值，如0x333333
* intensity：光强，optional
* distance：光照为0处到光源的距离，0表示到无穷远处为0，默认值，optional
* decay：沿着光照距离的衰退量，为2时实现现实世界的光衰减，缺省为1，optional
*/
var light = new THREE.PointLight(hex, intensity, distance, decay)
```

#### PointLight和DirectionalLight常用属性
| 常用属性 | 默认值 | 说明|
| --- | --- | --- |
| target | - | 阴影相机定位的目标，必须为THREE.Object3D对象（如THREE.Mesh） |
| penumbra | 0.0 | 聚光锥的半影衰减百分比[0, 1] |
| shadow | - | 用于存储光照阴影的所有信息，具体属性可参考[光照阴影](http://techbrood.com/threejs/docs/#%E5%8F%82%E8%80%83%E6%89%8B%E5%86%8C/%E5%85%89%E7%85%A7(Lights)/%E5%85%89%E7%85%A7%E9%98%B4%E5%BD%B1(LightShadow)) |
| castShadow | false | 是否投射动态阴影，<font color="tomato">*很耗费计算资源*</font> |