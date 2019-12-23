---
title: vue组件库之popup弹窗组件
date: 2019-12-23 18:00:00
tags: [vue, 组件沉淀]
category: vue组件
toc: true
comments: true
---
## 需求背景
在做活动页面时经常需要实现各种各样的弹窗，有一些常见的问题需要处理，包含：   
- 滑动穿透问题：滑动上层元素导致底层元素滚动
- 多弹窗层级问题：当有多弹窗时，最新的弹窗永远在最上层    
- 出现/消失过渡动画

## 实现
已发布npm包欢迎使用反馈和star~   
```
npm install jdc-popup -S
```
github: https://github.com/jinglecjy/jdc-popup   
demo: https://jinglecjy.github.io/jdc-popup/demo/   
![](https://img11.360buyimg.com/imagetools/s200x200_jfs/t1/91021/5/7943/23039/5e00255bE34ea4ce4/da97ea669a8ccd64.png)

## 滑动穿透问题
### 初始的解决方案
打开浮层时fixed底部元素，同时为了保持body的位置与打开浮层前一致，设置top偏移为当前scrollTop；   
关闭浮层时恢复底部元素状态和滚动高度；    
```javascript
controlledBgScrolled() {
    let bgEle = document.getElementById('app');
    // 打开浮层
    if (this.showPopup) {
        let top = document.documentElement.scrollTop
          || window.pageYOffset
          || document.body.scrollTop;
        this.scrollTop = top;
        bgEle.style.position = 'fixed';
        bgEle.style.top = `-${top}px`;
        bgEle.style.height = '100%';
    } 
    else {
        bgEle.style.position = 'relative';
        bgEle.style.top = '';
        bgEle.style.height = '100%';
        document.documentElement.scrollTop = this.scrollTop; 
        window.pageYOffset = this.scrollTop;
        document.body.scrollTop = this.scrollTop;
    }
}
```

这个方案的缺点是：   
1. 打开/关闭弹层瞬间，可以看到有闪动
2. 由于是APP内嵌页面，当设置APP沉浸式状态栏有几率发生冲突

### 使用库实现
测试提出bug的时间比较紧急，所以直接引入了npm包[tua-scroll-body-lock](#ref1)解决了该问题，经过测试，ios8+，android4.4+下没有发现问题。   
库的源码比较清晰，有兴趣的同学可以看一下：https://github.com/tuateam/tua-body-scroll-lock/tree/master/src    
之前的个人的思路都是想要一套代码兼容各端，这个库是将问题细分，对不同端进行了不同的处理，更容易去兼容。据官方描述，这三种方案都无法完美兼容所有端，详细可以查看[[2]](#ref2)。             
#### PC端
PC端实现比较简单，通过在body设置`overflow: hidden`就OK了。    
```javascript
const $body = document.querySelector('body')
const bodyStyle = { ...$body.style }
const scrollBarWidth = window.innerWidth - document.body.clientWidth
// 打开浮层时
$body.style.overflow = 'hidden'
$body.style.boxSizing = 'border-box'
$body.style.paddingRight = `${scrollBarWidth}px`
// 关闭浮层时，恢复原始设置
['overflow', 'boxSizing', 'paddingRight']
.forEach((x: OverflowHiddenPcStyleType) => {
    $body.style[x] = bodyStyle[x] || ''
}
```

#### Android端
Android端的实现与个人的方案思路基本一致，打开弹层时将底部元素fixed并设置top偏移，关闭浮层时恢复现场，注意到底部元素必须同时设置html和body。    
```javascript
const scrollTop = $html.scrollTop || $body.scrollTop
const htmlStyle = { ...$html.style }
const bodyStyle = { ...$body.style }

// 打开浮层时，fixed底部
$html.style.height = '100%'
$html.style.overflow = 'hidden'
$body.style.top = `-${scrollTop}px`
$body.style.width = '100%'
$body.style.height = 'auto'
$body.style.position = 'fixed'
$body.style.overflow = 'hidden'

// 关闭浮层时，恢复现场
$html.style.height = htmlStyle.height || ''
$html.style.overflow = htmlStyle.overflow || ''
['top', 'width', 'height', 'overflow', 'position']
.forEach((x: OverflowHiddenMobileStyleType) => {
  $body.style[x] = bodyStyle[x] || ''
})

window.scrollTo(0, scrollTop)
```

#### iOS端    
iOS端在打开浮层时，禁用了底部的`touchmove`事件，如果弹窗内部元素需要可滚动，则通过另外的函数自行处理。关闭浮层时移除所有事件监听。经测试，该方案在Android下弹窗滚动到边界时，底部元素有几率出现滚动。该库在iPhone 6p下初次打开无法滚动。             
```javascript
/***  打开浮层时,处理浮层和底部元素的滚动事件 ***/
// 1. targetElement为需要滚动的元素容器，处理其滚动
if (targetElement && lockedElements.indexOf(targetElement) === -1) {
    targetElement.ontouchstart = (event) => {
        initialClientY = event.targetTouches[0].clientY
    }
    targetElement.ontouchmove = (event) => {
        if (event.targetTouches.length !== 1) return
        // 手动处理滚动
        handleScroll(event, targetElement)
    }
    // 记录可滚动元素
    lockedElements.push(targetElement)
}
const handleScroll = (event, targetElement) => {
    const clientY = event.targetTouches[0].clientY - initialClientY
    if (targetElement) {
        const { scrollTop, scrollHeight, clientHeight } = targetElement
        // 向上滚动时 且 已经到达顶部
        const isOnTop = clientY > 0 && scrollTop === 0
        // 当向下滚动 且 已经到达底部
        const isOnBottom = clientY < 0
                          && scrollTop + clientHeight + 1 >= scrollHeight
        if (isOnTop || isOnBottom) {
            return preventDefault(event)
        }
    }
    event.stopPropagation()
    return true
}

// 2. 禁止document的touchMove事件
if (!documentListenerAdded) {
    document.addEventListener(
      'touchmove',
      preventDefault,
      eventListenerOptions)
    documentListenerAdded = true
}
/***  关闭浮层时，移除时间监听 ***/
if (targetElement) {
    const index = lockedElements.indexOf(targetElement)
    if (index !== -1) {
        targetElement.ontouchmove = null
        targetElement.ontouchstart = null
        lockedElements.splice(index, 1)
    }
}
if (documentListenerAdded) {
    document.removeEventListener(
      'touchmove',
      preventDefault,
      eventListenerOptions)
    documentListenerAdded = false
}
```

### ui组件库都是怎么实现的？   
用库可以较好的完成这个需求，但是对于滑动穿透这样的小问题，每次都引入库解决有点小题大做。基于上述考虑，选取了京东的nutui/有赞的vant/饿了么的mintui对比其实现方案，对比如下：       

|组件库 | 实现思路 | 实现形式 |
| --- | --- | --- |
| [vant](https://github.com/youzan/vant/blob/dev/src/mixins/popup/index.js) | touch事件处理 | mixins，抽取了复杂复用逻辑 |
| [mint](https://github.com/ElemeFE/mint-ui/tree/master/src/utils/popup) | touch事件处理 | mixins，抽取了复杂逻辑，但实现方案上依赖组件结构，复用性不强 |
| [nut](https://github.com/jdf2e/nutui/blob/master/src/packages/dialog/dialog.vue) | fixed底层背景 | 组件内部函数，简洁易读，复用性不强 |

对比了一下三个方案，并实际测试（iOS8+/Android4+），还是无法兼容所有机型，最终的还是按照库的基本思路包装了一下组件。    
另外加入overscroll-behavior虽然兼容性不佳，但是安卓原生浏览器和chrome浏览器仍然有部分支持。   
```javascript
// 对于半透明蒙层阻止滚动
mask.addEventListener(
  'touchmove', 
  this.preventDefault,
  { capture: false, passive: false },
  false);
// 对可滚动元素容器手动处理滚动
onTouchMove(event, targetElement) {
    ...
    if (targetElement) {
        const {
            scrollTop,
            scrollHeight,
            clientHeight
        } = targetElement
        // 向上滚动时 且 已经到达顶部
        const isOnTop = this.deltaY > 0 && scrollTop === 0
        // 当向下滚动 且 已经到达底部
        const isOnBottom = this.deltaY < 0
                          && scrollTop + clientHeight + 1 >= scrollHeight
        if (isOnTop || isOnBottom) {
            this.preventDefault(event)
        }
    }
    event.stopPropagation()
    return true
}
// 关闭弹窗时，移除所有事件
...
```

## 多弹窗层级问题
当前后打开两个弹窗，用户的预期是按照打开的先后顺序，越后打开的弹窗在越上层，简而言之就是新弹窗永远在最上层。可以通过记录当前出现过的最大zIndex，新弹窗`zIndex = zIndex+1`。 另外滑动穿透问题在多弹窗情况下也需要处理，对于非当前最高层级弹窗，不应当收到滚动影响。        
```javascript
this.$el.style.zIndex = context.zIndex + 1; 
context.zIndex += 1;
```

## 扩展
业务上的弹窗样式一般比较复杂，如果只是简单通用的弹窗样式，想要解决滑动穿透问题，可以通过`Vue.extend`扩展，将弹窗组件直接挂载到document下（element-ui中就使用了类似的做法），不会和主体内容互相影响，调用起来也更加灵活。   

## 参考
<a href="ref1">[[1] tua-scroll-body-lock](https://tuateam.github.io/tua-body-scroll-lock/)</a>   
<a href="ref2">[[2] 滑动穿透(锁body)终极探索](https://juejin.im/post/5ca4816e5188250b251e34e9)</a>   
[[3] vant-popup](https://blog.csdn.net/riddle1981/article/details/102659958)   
[[4] NUTUI](https://github.com/jdf2e/nutui)   
   