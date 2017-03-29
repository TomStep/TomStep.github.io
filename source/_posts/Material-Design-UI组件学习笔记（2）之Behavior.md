---
title: 'Material Design UI组件学习笔记（2）之Behavior '
date: 2016-12-08 10:50:32
tags: [Android,Material Design]
---

###CoordinatorLayout与Behavior的使用：

首先定义一个可以捕捉滑动的view

**MoveView**

```java
    public class MoveView extends TextView {
    public MoveView(Context context) {
        super(context);
    }

    public MoveView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public MoveView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    private int lastX;
    private int lastY;
//    private int mWidth;
//    private int mHeight;


    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int startX = (int) event.getRawX();
        int startY = (int) event.getRawY();

        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                break;

            case MotionEvent.ACTION_MOVE:
                if(getLayoutParams() instanceof CoordinatorLayout.MarginLayoutParams) {
                    CoordinatorLayout.MarginLayoutParams layoutParams = (CoordinatorLayout.MarginLayoutParams) getLayoutParams();

                    int left = layoutParams.leftMargin + startX - lastX;
                    int top = layoutParams.topMargin + startY - lastY;

                    layoutParams.leftMargin = left;
                    layoutParams.topMargin = top;
                    setLayoutParams(layoutParams);

                    //请求
                    requestLayout();
                }
                break;

            case MotionEvent.ACTION_UP:
                break;
        }
        lastX = startX;
        lastY = startY;
        return true;
    }
}
```

**放置布局：**

```html
    <android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:id="@+id/activity_main"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context="com.test.coordinatorlayouttest.MainActivity">

        <Button
            android:layout_width="40dp"
            android:layout_height="80dp"
            android:text="button"
            app:layout_behavior="@string/DodoBehavior"/>

        <com.test.coordinatorlayouttest.childview.MoveView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Hello World!"
            android:background="@color/colorPrimary"
            android:padding="20dp"/>
    </android.support.design.widget.CoordinatorLayout>

```

在Button上添加layout_behaviors属性。
在strings文件中：<string name="DodoBehavior">com.test.coordinatorlayouttest.behavior.DodoBehavior</string>

这样就把Button添加了Behavior行为。

**Behavior**

```java
public class DodoBehavior extends CoordinatorLayout.Behavior<Button> {

    private final Context context;
    private final int width;
    private final int height;


    public DodoBehavior(Context context, AttributeSet attrs) {
        super(context, attrs);
        this.context = context;
        DisplayMetrics display =   context.getResources().getDisplayMetrics();
        width = display.widthPixels;
        height = display.heightPixels;
    }

    @Override
    public boolean layoutDependsOn(CoordinatorLayout parent, Button child, View dependency) {
        //建立依赖关系
        //如果dependency 是 DodoMoveView类型， 就依赖
        //当然，也可以通过自定义View传入id 比对，或者别的方式，来设置是否跟着触发
        return dependency instanceof MoveView;
    }

    @Override
    public boolean onDependentViewChanged(CoordinatorLayout parent, Button child, View dependency) {
        //这里可以设置对应view的大小和位置
        //自己返回true， 则改变， 返回false， 自己不改变 位置和大小

        //根据dependency的位置，设置Button的位置
        int top = dependency.getTop();
        int left = dependency.getLeft();

        int x = width - left - child.getWidth();
        int y = height - top - child.getHeight();

        setPosition(child, x, y);
        return true;
    }

    private void setPosition(View v, int x, int y) {
        CoordinatorLayout.MarginLayoutParams layoutParams = (CoordinatorLayout.MarginLayoutParams) v.getLayoutParams();
        layoutParams.leftMargin = x;
        layoutParams.topMargin = y;
        layoutParams.width = y/2;
        v.setLayoutParams(layoutParams);
    }
}
```


- layoutDependsOn方法，用于建立依赖关系，返回ture表示建立依赖关系。dependency instanceof MoveView;表示，如果是MoveView类型就建立依赖关系。也可以获取view的id来建立依赖关系。
- onDependentViewChanged方法，用于改变对应view的大小和位置，返回true表示改变。

### 实战






