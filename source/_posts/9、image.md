---
title: RN学习：9、image
date: 2017-03-29 10:51:30
tags: [React-native]
---

>一个用于显示多种不同类型图片的React组件，包括网络图片、静态资源、临时的本地图片、以及本地磁盘上的图片（如相册）等。详细用法参阅图片文档。


<!--more-->

目前暂时只能使用网络图片加载

``` 
    
      <Image
        style={styles.logo}
        source={{uri: 'http://facebook.github.io/react/img/logo_og.png'}}
      />

      //成为背景图片

        <Image      //成为背景图
           style={{
            flex:1,
           }}
           source={{uri:"http://ofjn4kn3q.bkt.clouddn.com/bg.png"}}
          >

          、、、内部其他代码
           
      </Image>

```

## 属性

三个回调方法：

* onLoad        //图片加载成功时加载
* onLoadStart   //加载开始时回调
* onLoadEnd     //加载结束后回调，不管成功与否

适配组件和图片
* resizeMode 
    - cover 在保持图片宽高比的前提下缩放图片，直到宽度和高度都大于等于容器视图的尺寸
    - contain 在保持图片宽高比的前提下缩放图片，直到宽度和高度都小于等于容器视图的尺寸
    - stretch 拉伸图片且不维持宽高比，直到宽高都刚好填满容器
    - repeat 重复平铺图片直到填满容器。图片会维持原始尺寸。仅iOS可用
    - center  居中不拉伸
