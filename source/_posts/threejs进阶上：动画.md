---
title: threejs进阶上：动画
date: 2018-10-11 21:16:18
tags: threejs
category: 教程
toc: true
comments: true
description: threejs动画
---

### 简单动画
对于一些简单的动画，比如旋转/位置变换等等，可以直接使用`requestAnimationFrame`来进行重绘，示例：   
```JavaScript
function render() {
    earth.rotation.y += 0.005;
    cloud.rotation.y += 0.003;

    renderer.render(scene, camera);
    var id = requestAnimationFrame(render);
}
render();
```

`requestAnimationFrame`方法设置的动画，停止的方式如`cancelRequestFrame(id)`。
通过`requestAnimationFrame`实现的动画是匀速的，如果希望有缓动效果，可以结合补间动画库[tween.js](http://www.createjs.cc/tweenjs/)来实现。


### 模型骨骼动画
*注意：复杂的骨骼动画容易出问题而且定位较难，建议**谨慎使用**。*
three.js提供了各种各样的模型加载器，但是这些加载器的完善程度有待商榷，容易出现问题。官方目前推荐使用的是`glTF`格式的模型数据，支持较好。   
使用官方示例的模型测试没什么问题，实际应用估计问题还比较多，考虑到使用模型导入动画的资源大小问题，实际应用可能比较困难，待之后使用到再进行补充。

### 性能插件stats
Three.js辅助库，显示性能帧数，每次刷新所用时间，占用内存。   
```JavaScript
// 1. 引入插件
<script src="../lib/stats.min.js"></script>

// 2. 实例化组件并添加到dom中
var stats = new Stats();
container.appendChild(stats.dom);

// 3. 在requestAnimationFrame()函数调用里面更新stats
function render() {
    // ...
    stats.update();
    requestAnimationFrame(render);
}
render();
```