---
title: threejs问题集锦
date: 2018-10-11 21:45:32
tags: threejs
category: lesson
---

- [贴图](#map)
- [官方插件](#official-plugin)

### <div id="map">贴图</div>
1. 贴图大小限制
WebGL对于texture的支持是有大小限制的，使用CanvasRenderer也会有同样的限制，只要手机支持WebGL，可以保证允许2048*2048的贴图，因此贴图最好不超过该限制。

#### <div id="official-plugin">官方插件</div>
1. 使用最新版`OrbitControls.js`(2018/10/11)后，点击事件无法生效
最新版的代码在所有涉及到的事件中都添加了`event.preventDefault()`，注释掉对应的就可以了。