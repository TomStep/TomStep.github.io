---
title: RecyclerView坑点
date: 2016-11-28 16:11:49
tags: [Android]
---
RecyclerView用来代替ListView和GridView，它的优点很多，这里就不详细说明了。这里主要记录下我在使用RecyclerView时候的主要坑点。

### 卡顿！！

如果在item中使用了较为复杂的布局，如嵌套等。就会出现卡顿现象。（是特别卡）

**解决方法：**
网上查找的方案有以下几种思路：

1、如果有很多图片加载，可以使用图片加载框架。配合加载时机（停止滑动时加载图片）。
2、尽量不要使用太复杂的布局，可以自定义View,或者改UI
3、如果一定要使用嵌套的话，不要嵌套GridView，直接嵌套RecycleView好了。
4、可以用同一个RecycleViewPool

然而有时候这几种方案并不能完全解决我们的问题，卡顿还是会出现。

<!-- more -->  

----

### 从RecyclerView源码入手
>参考：http://www.jianshu.com/p/32c963b1ebc1（文／梨花满天便是雪）
可以先看这篇博客了解下RecyckerView大概缓存机制。

**RecycledViewPool**
>RecycledViewPool类是用来缓存Item用，是一个ViewHolder的缓存池，如果多个RecyclerView之间用setRecycledViewPool(RecycledViewPool)设置同一个RecycledViewPool，他们就可以共享Item。其实RecycledViewPool的内部维护了一个Map，里面以不同的viewType为Key存储了各自对应的ViewHolder集合。可以通过提供的方法来修改内部缓存的Viewholder。

所以，当不同的RecyclerView中使用相同类型的item时，利用共享一个RecycledViewPool，就可以提升滑动的流程程度。

这个类提供了四个公共方法：
- clear()  清空缓存池
- getRecycledView(int viewType)  得到一个viewType类型的Item
- putRecycledView(RecyclerView.ViewHolder scrap)  把viewType类型的Item放入缓存池
- setMaxRecycledViews(int viewType, int max)  设置对应viewType类型的Item的最大缓存数量

----

**ViewCacheExtension**
>ViewCacheExtension的代码一看什么都没有，没错这是一个需要开发者重写的类。上面Recycler里调用Recycler.getViewForPosition(int)方法获取View时，Recycler先检查自己内部的attached scrap和一级缓存，再检查ViewCacheExtension.getViewForPositionAndType(Recycler, int, int)，最后检查RecyclerViewPool，从上面三个任何一个只要拿到View就不会调用下一个方法。所以我们可以重写getViewForPositionAndType(Recycler recycler, int position, int type)，在方法里通过Recycler类控制View缓存。注意：如果你重写了这个类，Recycler不会在这个类中做缓存View的操作，是否缓存View完全由开发者控制。

```java
 public abstract static class ViewCacheExtension {

        abstract public View getViewForPositionAndType(Recycler recycler, int position, int type);
    }
```

这个类里面只有提供了一个方法，就是获取缓存View的方法。开发者要自己想办法把想要缓存的View放进去。

我的做法是：创建一个类来继承ViewCacheExtension，重写里面的方法。在onBindClickViewHolder（）方法创建实例，把对应holder的View加进去。

```java
    public class ViewHolderCache extends RecyclerView.ViewCacheExtension{
        private View mView;

        public ViewHolderCache(CateGoryHolder holder) {
            mView = holder.getItemView();
        }


        @Override
        public View getViewForPositionAndType(RecyclerView.Recycler recycler, int position, int type) {
            if(type == Type.CATEGORY.ordinal() && position == 1){
                return mView;
            }
            return null;
        }
    }


    ------
    @Override
    public void onBindClickViewHolder(RecyclerView.ViewHolder holder, int position) {
            ...
            ((CateGoryHolder) holder).setGridView(mContext,recycledViewPool, mCategoryList);

            ViewHolderCache viewHolderCache = new ViewHolderCache((CateGoryHolder)holder);
            ...
        }
    }

```


这里可以保存特定的View缓存。解决比较特殊的item Type卡顿现象。

**setItemViewCacheSize方法**

最后的最后，如果你实在解决不了了。可以使用RecyclerView.setItemViewCacheSize(data.size())。
虽然解决了燃眉之急，不过潜在的风险你懂得。



**RecyclerView嵌套RecyclerView**










