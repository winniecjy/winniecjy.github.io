---
title: threejs进阶下：交互
date: 2018-10-11 21:16:51
tags: threejs
category: lesson
---

### Raycaster

`THREE.Raycaster`是threejs中比较重要的一个类，可以用于物体选择和碰撞检测。实现原理是生成一条从显示的起点到重点的一条射线，检测与射线相交的物体集合。    

#### new Raycaster(origin, direction, near, far)

- origin：光线投射的起点向量
- direction：光线投射的方向向量，归一化
- near：投射近点，用于限制光线的起点，不能为负，缺省为0
- far：投射远点，用于限制光线的终点，不能小于near，缺省为无穷大

#### setFromCamera(coords, camera)

用一个新的原点和方向向量来更新射线，参数说明如下。

- coords：鼠标的归一化二维坐标，归一化方法如下：
    ```JavaScript
    mouse.x = (e.clientX / window.innerWidth) * 2 -1;
    mouse.y = - (e.clientY / window.innerHeight) * 2 + 1;
    ```
- camera：射线起点处的相机，即把射线起点设置在该相机位置处

#### intersectObject(object, recursive)
检查射线和物体之间的交叉点，交叉点按距离排序返回，最接近为第一个，返回值为一个对象数组。
- object：用于检测和射线相交的物体
- recursive：若为true，则检查所有后代，否则只检查本身，缺省值为false

#### 点击返回选中物体
```JavaScript
var raycaster = new THREE.Raycaster(origin, direction, near, far);
mouse.x = (e.clientX / window.innerWidth) * 2 -1;
mouse.y = - (e.clientY / window.innerHeight) * 2 + 1;
raycaster.setFromCamera(mouse, camera);
var intersects = raycaster.intersectObjects(locationGroup.children, true);    
```

#### 碰撞检测
原理是通过以物体中心为起点，向各个顶点发出射线，检测射线是否与其他物体相交，若相交则检查最近的一个交点与射线起点的距离，如果这个距离比物体中心到物体顶点的距离要小，则说明发生了碰撞。
```JavaScript
// 1. 获取物体中心点
var originP = obj.position.clone(); 
for(var i=0; i<obj.geometry.vertices.length; i++) {
    // 获取顶点坐标，将本地坐标乘以变换矩阵，得到了物体在世界坐标系中的值
    var vertex = obj.geometry.vertices[i].clone().applyMatrix4(obj.matrix);
    // 获取由中心指向顶点的向量
    var directV = vertex.sub(obj.position);
    // 方向向量归一化并发射射线
    var ray = new Three.Raycaster(originP, directV.clone().normalize());
    // 检测相交物体和距离
    var collisionRes = ray.intersectObjects(collideMeshList, true);
    if (collisionRes.length>0 && collisionRes[0].distance < directV.length() ) {
        console.log('发生碰撞');
    }
}
```

***

https://blog.csdn.net/birdflyto206/article/details/52414382
### 相机操作

模型移动常常通过相机的移动来实现，常用的相机控件如下：   

FirstPersonControls：第一人称控件，键盘移动，鼠标转动 
FlyControls：飞行器模拟控件，键盘和鼠标来控制相机的移动和转动 
RollControls：翻滚控件，FlyControls的简化版，可以绕z轴旋转 
TrackballControls：轨迹球控件，用鼠标来轻松移动、平移和缩放场景 
OrbitControls：轨道控件，在场景中绕某个对象旋转、平移
PathControls：路径控件，相机可以沿着预定义的路径移动。可以四处观看，但不能改变自身的位置
