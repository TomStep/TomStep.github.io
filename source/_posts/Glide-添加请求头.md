---
title: Glide自定义请求头，加载图片无法显示的图片资源。
date: 2017-04-12 15:35:52
tags: [Android]
---

在使用Glide的时候一般都不会考虑添加请求头。
有些时候（抓图）会因为“防盗链”的原因，图片显示不出来。这时候就需要添加请求头，以获取图片资源。

### 什么是“防盗链”

> 防止别人通过一些技术手段绕过本站的资源展示页面，盗用本站的资源，让绕开本站资源展示页面的资源链接失效。

一般的防盗链原理也很简单，都是服务器验证请求头的**Referer**和**Host**的值，最多再加上**cookie**验证。如果你复制了这个图片地址，然后直接到 浏览器地址上面访问是无法显示的，因为请求里面没有验证的值。

<!-- more -->

### 在Gilde上添加请求头

实现Headers接口,重写getHeaders()方法：

```java
    
    public interface Headers {

        @Deprecated
        Headers NONE = new Headers() {
            @Override
            public Map<String, String> getHeaders() {
                return Collections.emptyMap();
            }
        };
        
        Headers DEFAULT = new LazyHeaders.Builder().build();

        Map<String, String> getHeaders();

    }

```

**实现方式如下：**

```

    @Override
    public Map<String, String> getHeaders() {
        Map<String, String> header = new HashMap<>();

        //不一定都要添加，具体看原站的请求信息
        header.put("Referer", "__查看下Referer中需要添加的值__");
        header.put("Host","__查看下Host中需要添加的值__")
        return header;
    }

```

**显示：**

```java
    
    /*
     * url : 请求地址
     * headers : 已实现的Headers实例
     */
    GlideUrl gliderUrl = new GlideUrl(url,headers)
    
    //显示图片
    Glide.with(context).load(gliderUrl).into(imageView);
```


