---
title: 'Material Design UI组件学习笔记（1） '
date: 2016-11-10 11:43:02
tags: [Android,Material Design]
---
## Material Design UI组件

在Google I/O 2015大会上，谷歌发布了一个新的support library,里面包含了一些遵循Material Design's spec的UI组件。这些组件配合起来使用可以产生很好的效果。虽然Material Design风格没有在国内流行起来，但是这些组件还是很有学习的必要的。

<!-- more --> 

## 组件的使用
1、首先添加两个包：
>
    compile 'com.android.support:appcompat-v7:24.2.1'
    compile 'com.android.support:design:24.2.1'

2、控件使用说明：
<a href="#Snackbar">1、Snackbar和 FloatingActionButton的使用</a>
<a href="#Coordinator">2、CoordinatorLayout的使用</a>

---------

<a id="Snackbar"></a>
### Snackbar和 FloatingActionButton的使用
 * Snackbar有点类似于以前的Toast,但是它更加灵活一些，可以添加自定义可选操作等。
 * FloatingActionButton顾名思义，它是一个可浮动的button,配合CoordinatorLayout可以实现一些交互效果

### Snackbar
```java
//snackbar的用法（和Toast类似，多添加了setAction()方法）
Snackbar.make(mListView,strs[position], Snackbar.LENGTH_SHORT)
        .setAction("action", new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Toast.makeText(mContext, "点击了Snackbar", Toast.LENGTH_SHORT).show();
            }
        })
        .show();
```

### FloatingActionButton的简单使用：
>可以参考鸿大神的博客：[FloatingActionButton 完全解析][1]

### FloatingActionButton配合CoordinatorLayout使用：

1、配置xml文件
```html
    <!-- xml文件 -->
    <android.support.design.widget.CoordinatorLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <android.support.design.widget.FloatingActionButton
            android:id="@+id/fab"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="end|bottom"
            android:layout_margin="40dp"
            android:background="@drawable/fab"
            android:src="@drawable/btn_sousuo" />

    </android.support.design.widget.CoordinatorLayout>

```

2、设置activity代码
```java
//activity
public class FlyingButtonActivity extends BaseActivity{

    private FloatingActionButton mFloating;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        
        setContentView(R.layout.activity_flying_button);

        mFloating = (FloatingActionButton) findViewById(R.id.fab);
        mFloating.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
               Snackbar.make(mFloating,"点击浮动图标",Snackbar.LENGTH_SHORT)
                       .setAction("action", new View.OnClickListener() {
                           @Override
                           public void onClick(View v) {
                               Toast.makeText(FlyingButtonActivity.this, "点击了Action", Toast.LENGTH_SHORT).show();
                           }
                       }).show();
            }
        });
    }

}
```

**3、显示效果：**
>    floatingButton在snackbar出现时，往上移动。

**4、为什么会出现这样的效果？**
>   1、首先要了解CoordinatorLayout的作用。它是用来协调子类控件的行为。
    2、FloatingActionButton默认实现了一个FloatingActionButton.Behavior。它可以直接在CoordinatorLayout使用。


<a name="Coordinator"></a>
### CoordinatorLayout的使用：
>顾名思义，这个控件的目的就是协调它里面View的行为。
它需要于子类配合使用。这里我们看 CoordinatorLayout + AppBarLayout + NestedScrollView配合使用的效果。（一般CoordinatorLayout的直接子类都是这个两个）  

**1、首先重点介绍AppBarLayout：**
> AppBarLayout具有Linearlayout垂直布局的特性。

```html
     <android.support.design.widget.CoordinatorLayout
         xmlns:android="http://schemas.android.com/apk/res/android"
         xmlns:app="http://schemas.android.com/apk/res-auto"
         android:layout_width="match_parent"
         android:layout_height="match_parent">
    
     <!-- AppBarLayout -->
     <!--1、AppBarLayout最好是CoordinatorLayout的直接子view -->
     <android.support.design.widget.AppBarLayout
             android:layout_height="wrap_content"
             android:layout_width="match_parent">
        <!--2 、AppBarLayout的子view：需要设置app:layout_scrollFlags-->
         <android.support.v7.widget.Toolbar
                 ...
                 app:layout_scrollFlags="scroll|enterAlways"/>

         <android.support.design.widget.TabLayout
                 ...
                 app:layout_scrollFlags="scroll|enterAlways"/>

     </android.support.design.widget.AppBarLayout>

     <!-- NestedScrollView -->
     <!-- 3、指定behavior属性为AppBarLayout.ScrollingViewBehavior -->
     <android.support.v4.widget.NestedScrollView
             android:layout_width="match_parent"
             android:layout_height="match_parent"
             app:layout_behavior="@string/appbar_scrolling_view_behavior">

     </android.support.v4.widget.NestedScrollView>

 </android.support.design.widget.CoordinatorLayout>

```

* 1、AppBarLayout和NestedScrollView作为CoordinatorLayout直接继承子类。
* 2、AppBarLayout的子类需要设置layout_scrollFlags这个属性：
    - scroll 子类伴随着滚动事件而滚出或滚进屏幕
        + 注意点1:如果使用了其他值，必定要使用这个值才能起作用
        + 注意点2:如果在这个子类前面的任何其他子类没有设置这个值，那么这个子类的设置将失去作用。
    - enterAlways 快速返回
        + 注意点1:向上滑动时，总是最后一个滚出屏幕的。
        + 注意点2:只要向下滑动（不管在哪个位置），都会跟着出来。
    - enterAlwaysCollapsed enterAlways的附加值 比scroll先向下滚动一个附加值，之后等scroll滚动完成后，在向下滚。
        + 配合android:minHeight属性一起使用。
    - exitUntilCollapsed 于enterAlwaysCollapsed相同原理，但是它是针对向上滚动的情况。
    - snap 吸附效果，它不会出现停留在中间的情况。要不就完全隐藏，要不就完全出现。
* 3、需要给一个有滑动机制的控件添加app:layout_behavior="@string/appbar_scrolling_view_behavior"属性。这里我们使用的是NestedScrollView（它的作用也很强大，后面分析）

* 4、demo演示效果：
```html
<!-- xml文件 -->
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/main_content"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <!--1、AppBarLayout最好是CoordinatorLayout的直接子view -->
    <android.support.design.widget.AppBarLayout
        android:id="@+id/appbar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar">
        <!--2 、AppBarLayout的子view：需要设置app:layout_scrollFlags-->
        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="?attr/colorPrimary"
            android:title="Title"
            app:popupTheme="@style/ThemeOverlay.AppCompat.Light"
            app:layout_scrollFlags="scroll" />

        <ImageView
            android:id="@+id/mIv"
            app:layout_scrollFlags="scroll|snap"
            android:layout_width="match_parent"
            android:layout_height="100dp"
            android:scaleType="centerCrop"
            android:background="@android:color/holo_blue_bright" />

    </android.support.design.widget.AppBarLayout>

    <!-- 3、指定behavior属性为AppBarLayout.ScrollingViewBehavior -->
    <android.support.v4.widget.NestedScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical">
            <TextView
                android:layout_width="match_parent"
                android:layout_height="180dp"
                android:layout_margin="10dp"
                android:background="#6660c6df"
                android:gravity="center"
                android:text="折"
                android:textSize="26dp"
                android:transitionName="zi" />
            <TextView
                android:layout_width="match_parent"
                android:layout_height="180dp"
                android:layout_margin="10dp"
                android:background="#6660c6df"
                android:gravity="center"
                android:text="折叠"
                android:textSize="26dp"
                android:transitionName="zi" />
            <TextView
                android:layout_width="match_parent"
                android:layout_height="180dp"
                android:layout_margin="10dp"
                android:background="#6660c6df"
                android:gravity="center"
                android:text="折叠示"
                android:textSize="26dp"
                android:transitionName="zi" />
            <TextView
                android:layout_width="match_parent"
                android:layout_height="180dp"
                android:layout_margin="10dp"
                android:background="#6660c6df"
                android:gravity="center"
                android:text="折叠示例"
                android:textSize="26dp"
                android:transitionName="zi" />
        </LinearLayout>
    </android.support.v4.widget.NestedScrollView>
</android.support.design.widget.CoordinatorLayout>
```

```java
<!-- activity -->
public class AppBarActivity extends BaseActivity{

    private Toolbar toolbar;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_app_bar);
        toolbar = (Toolbar) findViewById(R.id.toolbar);
        toolbar.setTitle("Title");
        setSupportActionBar(toolbar);
    }
}
```

*注意：在style中要设置主题为NoActionBar*

**2、AppBarLayout + CollapsingToolbarLayout**
>好了，我们现在又多了一个组件。看看CollapsingToolbarLayout是怎么用的。
CollapsingToolbarLayout作用是提供了可以折叠的Toolbar（CollapsingToolbarLayout是一个实现了折叠工具栏效果Toolbar的包装器，它被设计为AppBarLayout的直接子View。）

实例说明：
```html
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
    android:clipToPadding="false"
    >

    <android.support.design.widget.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="300dp"
        android:fitsSystemWindows="true"
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar">

        <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/collapsing_toolbar"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:contentScrim="?attr/colorPrimary"
            app:expandedTitleMarginEnd="64dp"
            app:expandedTitleMarginStart="48dp"
            app:layout_scrollFlags="scroll|exitUntilCollapsed">

            <ImageView
                android:id="@+id/ivImage"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:fitsSystemWindows="true"
                android:transitionName="transition_book_img"
                android:scaleType="centerCrop"
                app:layout_collapseMode="parallax"
                app:layout_collapseParallaxMultiplier="0.7" />

            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:layout_collapseMode="pin"
                app:layout_scrollFlags="scroll|enterAlways"
                app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />
        </android.support.design.widget.CollapsingToolbarLayout>
    </android.support.design.widget.AppBarLayout>

    <android.support.v4.widget.NestedScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical">

            <TextView
                android:layout_width="match_parent"
                android:layout_height="180dp"
                android:layout_margin="10dp"
                android:background="#6660c6df"
                android:gravity="center"
                android:text="折"
                android:textSize="26dp"
                android:transitionName="zi" />

            <TextView
                android:layout_width="match_parent"
                android:layout_height="180dp"
                android:layout_margin="10dp"
                android:background="#6660c6df"
                android:gravity="center"
                android:text="折叠"
                android:textSize="26dp"
                android:transitionName="zi" />

            <TextView
                android:layout_width="match_parent"
                android:layout_height="180dp"
                android:layout_margin="10dp"
                android:background="#6660c6df"
                android:gravity="center"
                android:text="折叠示"
                android:textSize="26dp"
                android:transitionName="zi" />

            <TextView
                android:layout_width="match_parent"
                android:layout_height="180dp"
                android:layout_margin="10dp"
                android:background="#6660c6df"
                android:gravity="center"
                android:text="折叠示例"
                android:textSize="26dp"
                android:transitionName="zi" />


        </LinearLayout>

    </android.support.v4.widget.NestedScrollView>

</android.support.design.widget.CoordinatorLayout>
```

```java
public class ToolbarLayoutActivity extends BaseActivity{

    private CollapsingToolbarLayout mCollapsing;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        setContentView(R.layout.activity_tool_layout);

        initFindView();
        initView();
    }

    private void initView() {
        //设置标题的一些属性
        mCollapsing.setTitle("设置ToolBar");
        mCollapsing.setCollapsedTitleGravity(Gravity.CENTER); //标题收缩后的位置，居中
        mCollapsing.setExpandedTitleColor(Color.TRANSPARENT); // 标题展示时颜色
        mCollapsing.setCollapsedTitleTextColor(Color.WHITE); // 标题收缩时的颜色

        //沉浸式内容(改变收缩后toolbar背景，有渐变效果)
        //也可以通过app:contentScrim=”?attr/colorPrimary”来改变效果
        mCollapsing.setContentScrim(getResources().getDrawable(R.drawable.sy_2010091620583620405));

        //沉浸式状态栏
        collapsingToolbarLayout.setStatusBarScrim(getResources().getDrawable(R.drawable.sy_2010091620583620405));
    }

    private void initFindView() {
        mCollapsing = (CollapsingToolbarLayout) findViewById(R.id.collapsing_toolbar);
    }
}
```

说明：
    * 折叠标题 （如上述代码）。
    * 视差滚动，通过设置app:layout_collapseParallaxMultiplier属性获取。视差值介于0.0-1.0之间。
    * 顶部悬浮，通过设置app:layout_collapseMode设置折叠模式
        - “pin”：固定模式，在折叠的时候最后固定在顶端
        - “parallax”：视差模式，在折叠的时候会有个视差折叠的效果


[1]:http://blog.csdn.net/lmj623565791/article/details/46678867


