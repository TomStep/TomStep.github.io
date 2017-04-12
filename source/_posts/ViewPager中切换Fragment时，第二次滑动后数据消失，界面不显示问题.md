---
title: ViewPager中切换Fragment时，第二次滑动后数据消失，界面不显示问题
date: 2017-04-12 16:02:28
tags: [Android]
---

使用ViewPager+Fragment进行界面切换，当界面数量大于等于3时，出现两次滑动后的界面会消失。针对这问题，我们可以先看看Fragment在这过程做了什么。

### 分析3个fragment声明周期：

Fragment A，Fragment B，Fragment C

刚进入时，对于FragmentA来说，创建数据：
--> onCreateView
--> onStart

从A滑动到C时，对于FragmentA来说，数据被销毁：
--> onPause
--> onStop
--> onDestroyView
--> OnDestroy

再从C滑回A时，对于FragmentA来说，数据重建：
--> onCreateView
--> onStart

这时，就会发现FragmentA消失不见了。

<!-- more -->

### 分析原因：

主要原因是：onCreateView（给当前fragment绘制UI布局）被调用了两次，添加了多个View。

### 解决方案：

#### 方案一、

设置ViewPager的缓存界面数。

```java

    //表示缓存当前界面每一侧的界面数
    viewPager.setOffscreenPageLimit(2);

```

为了避免Fragment的数据被销毁，增加ViewPager的缓存数量，以避免Fragment被销毁。（界面数量固定，且数量较少时可以使用）


#### 方案二、

复用Fragmnet的RootView，

```java

    private View mRootView;

    @Override
    public void onDestroyView(){
        supe.onDestroyView();

        //移除mRootView，防止添加多个View
        if(null != mRootView){
            ((ViewGroup)mRootView.getParent()).removeView(mRootView);
        }
    }

    @Override
    public View onCreateView(LayoutInflater inflater,ViewGroup container,Bundle bundle){
        
        //若mRootView为null，则初始化view。否则复用mRootView
        if(null == mRootView){
            mRootView = inflater.inflater(...);
        }

        return mRootView;
    }

```




