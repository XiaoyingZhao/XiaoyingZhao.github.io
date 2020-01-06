---
title: JS如何实现在IOS中video全屏播放
date: 2018-12-21 13:08:37
tags: 
  - js
  - ios
categories: js
---
现在开发人员不容易呀，产品、运营天天各种需求，哎这个顶导要在页面往上滑的时候缓缓的隐藏，下拉的时候在缓缓的显示~ 哎这个图片要自适应的呀，就是那种不够的话补黑边的呀~ 哎视频的进度条要自定义的呀，原生的那个多丑呀~ 哎。。。

唉，怎么办呢，没有办法呀，使劲磕呗，那今天的话题我们就来说说IOS中video全屏的问题，有需要的朋友可以参考下


### video的几个重要的属性
`controls`： 显示播放控件；controls属性规定浏览器应该为视频提供播放控件,换句话说，只有写了这个属性才会有播放控件，不写的话默认是没有的，所有如果要自定义播放控件就不需要写controls属性
写法：
```html
<video controls src='1.mp4'></video>      
```

`autoplay`：自动播放；指定后，视频会马上自动开始播放，不会停下来等着数据载入结束
写法：
```html
<video autoplay src='1.mp4'></video>
```

`playsinline`：取消全屏；只需在video上添加该属性，便可使视频播放时局域播放，不脱离文档流，但是这个属性需要APP支持，在APP开发中设置UIwebview 的allowsInlineMediaPlayback = YES   webview.allowsInlineMediaPlayback = YES才能生效
写法：
```html
<video webkit-playsinline="true" playsinline="playsinline" src='1.mp4'></video>
```

### js自定义video全屏
众所周知，IOS是不支持全屏API的，于是requesrFullScreen事件对于video全屏是不起作用的，但是IOS9+对于video标签有一个很重要的属性，那就是`playsinline`,这个属性去掉之后视频就可以全屏，但不是直接生效，而是下次播放的时候才可以，于是我们需要先把视频短暂的暂停一下，在播放就可以了，具体实现方法：
```javascript
video.pause()
video.removeAttribute('playsinline')
video.removeAttribute('webkit-playsinline')
video.play()
```
这个仅对IOS中的全屏有用，如果兼容Android需要使用`requestFullScreen`方法，具体实现方法：
```html
<button onclick="fullscreenFn()">全屏</button>
<video width="400" autoplay="" webkit-playsinline="true" playsinline="playsinline" src='1.mp4' id="video"></video>
```
```javascript
// 浏览器前缀判断
    var videoEle = document.getElementById('video')
    function browserPrefix (method) {
      let browserPrefixreturn
      switch (true) {
        // W3C
        case (!!(videoEle[method])):
          browserPrefixreturn = method
          break
        // FireFox
        case (!!(videoEle['moz' + method[0].toUpperCase() + method.slice(1)])):
          browserPrefixreturn = 'moz' + method[0].toUpperCase() + method.slice(1)
          break
        // Chrome等
        case (!!(videoEle['webkit' + method[0].toUpperCase() + method.slice(1)])):
          browserPrefixreturn = 'webkit' + method[0].toUpperCase() + method.slice(1)
          break
        // IE11
        case (!!(videoEle['ms' + method[0].toUpperCase() + method.slice(1)])):
          browserPrefixreturn = 'ms' + method[0].toUpperCase() + method.slice(1)
          break
      }
      return browserPrefixreturn
	}
    function fullscreenFn(){
      video.pause()
      video.removeAttribute('playsinline')
      video.removeAttribute('webkit-playsinline')
      video.play()
      video[browserPrefix('requestFullScreen')]()
      event.stopImmediatePropagation()
    }
```