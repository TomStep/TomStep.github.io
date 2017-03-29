---
title: DataBinding笔记
date: 2017-03-29 10:25:54
tags: [Android]
---

>参考：

  1、https://developer.android.com/topic/libraries/data-binding/index.html?hl=zh-cn#data_binding_layout_files

  2、http://yanghui.name/blog/2016/02/17/data-binding-guide/

这篇笔记主要用于个人梳理用。

### DataBinding的作用：

减少应用中逻辑以及布局所需要的“胶水代码”,适用于MVVM模式。


<!-- more -->  

---

### 编译环境：

注意：支持版本在Android Studio 1.3 以及之后

在 build.gradle配置：

```

    android {
        ....
        dataBinding {
            enabled = true
        }
    }

```

开启后就可以使用啦～

---

### DataBinding布局：

```

    //main_activity.xml
    <!-- layout标签包裹 -->
    <layout xmlns:android="http://schemas.android.com/apk/res/android">
        <!-- data -->
        <data>
            <variable name="user" type="com.example.User"/>
        </data>

        <!-- view -->
        <LinearLayout
           android:orientation="vertical"
           android:layout_width="match_parent"
           android:layout_height="match_parent">

           <TextView android:layout_width="wrap_content"
               android:layout_height="wrap_content"
               android:text="@{user.firstName}"/>

           <TextView android:layout_width="wrap_content"
               android:layout_height="wrap_content"
               android:text="@{user.lastName}"/>

       </LinearLayout>

    </layout>

```

与传统的布局文件不同，DataBinding布局是用**layout标签**来包裹，标签内分为**data**，**view**两块。

#### view部分

view的部分和之前xml原本文件差不多，所以布局方式和以前是一样的。


```
    
    <TextView android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{user.lastName}"/>

```

在布局中有个陌生的表达式：**“@{}”**，这里的意思是TextView 的文本被设置为 user中的 firstName 属性。（记住这种表达方式。）


#### data部分

data的部分是比较陌生的

```
    
    <data>
        <variable name="user" type="com.example.User"/>
    </data>

```

单看这种写法，很容易可以理解：定义一个变量user，类型是com.example.User。

没错，这样就相当于拿到了一个User对象，然后就可以在布局中使用它的方法和成员变量啦。

看看User类里面是什么样子的：

```

    //(POJO)
    public class User {

       public final String firstName;

       public final String lastName;

       public User(String firstName, String lastName) {
           this.firstName = firstName;
           this.lastName = lastName;
       }
    }

```

结合起前面的代码我们就能理解，**android:text=@{user.lastName}**的意思就是把user中的lastName值赋到text（TextView显示的文字）中。

**注意：**
    从 data binding 的角度看，pojo类和javaBeans类是一样的，所以User类也也可写成：

```
    
    //javaBean
    public class User {
       private final String firstName;

       private final String lastName;

       public User(String firstName, String lastName) {
           this.firstName = firstName;
           this.lastName = lastName;
       }

       public String getFirstName() {
           return this.firstName;
       }

       public String getLastName() {
           return this.lastName;
       }
    }

```

***所以不用担心dataBinding的@{user.lastName}获取不到私有的lastName，它会从getLastName()方法中获取。***

---


### 绑定数据和布局

在项目中完上面的代码。默认情况下，会给予布局文件生成一个Binding类，将它转换成帕斯卡命名并在名字后面接上”Binding”。（如果没有，可以按rebuild，重建项目），上面的那个布局文件叫 main_activity.xml，所以会生成一个 MainActivityBinding 类。

在MainActivityBinding中会有一个setUser()方法，用于绑定user对象。

```
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
       super.onCreate(savedInstanceState);

       //可以用DataBindingUtil.setContentView()代替原本的setContentView()
       MainActivityBinding binding = DataBindingUtil.setContentView(this, R.layout.main_activity);

       User user = new User("Test", "User");

       //绑定数据
       binding.setUser(user);  
    }

```

**注意：**

    1、DataBindingUtil.setContentView()代替原本的setContentView()方法中获取。同时获取binding对象。

    2、记住一定要绑定数据，不然显示不出来。

这时候运行项目就可以看见测试数据显示在界面上了。

#### Binding类
>我们这里说的Binding类是指MainActivityBinding

Binding类中除了可以绑定数据，可以获取整个布局View，如果控件上设置了Id，还可以直接通过Binding类获取该控件View。



```

     <TextView 
        android:id="+id/text_view"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@{user.lastName}"/>

    ...

    <!-- 在activity中 -->
    
    //获取整个布局：
    View view = binding.getRootView();  //可以用于需要注入布局的地方，如fragment的onCreatView()

    //获取TextView
    textView = binding.textView;    //取名会去掉下横线，使用驼峰命名

```

这样就不用在出现findviewById()了。

**注意一个坑点，通过上面得到的binding获取控件，会得到一个Null值。我也不知道为啥**
    
```

    MainActivityBinding binding = DataBindingUtil.setContentView(this, R.layout.main_activity);
    
    textView = binding.textView //textView = null


```

我们可以用其他的方式获取binding:

```

    //调用MainActivityBinding静态方法inflate获取
    MainActivityBinding binding = MainActivityBinding.inflate(getLayoutInflater());

```

如果要在ListView和RecyclerView的adapter中使用databinding的话，可以用下面方法获取：

```
    
    ListItemBinding binding = ListItemBinding.inflate(layoutInflater, viewGroup, false);
    //or
    ListItemBinding binding = DataBindingUtil.inflate(layoutInflater, R.layout.list_item, viewGroup, false);

```

---

### 绑定事件
>事件可以直接与函数绑定,事件属性的命名由 listener 的函数命名决定。举个例子，View.OnLongClickListener 中有一个 onLongClick() 函数，所以这个事件的对应属性就是 android:onLongClick。

```

    public class Handlers {
      public void onClickFriend(View view) { ... }
      public void onClickEnemy(View view) { ... }
      public void onClickChanged(View view,String firstName, boolean completed){...}
    }

```

```

    <layout xmlns:android="http://schemas.android.com/apk/res/android">
       <data>
           <variable name="handlers" type="com.example.Handlers"/>
           <variable name="user" type="com.example.User"/>
       </data>

       <LinearLayout
           android:orientation="vertical"
           android:layout_width="match_parent"
           android:layout_height="match_parent">

           <TextView android:layout_width="wrap_content"
               android:layout_height="wrap_content"
               android:text="@{user.firstName}"
               android:onClick="@{handlers::onClickFriend}"/>

           <TextView android:layout_width="wrap_content"
               android:layout_height="wrap_content"
               android:text="@{user.lastName}"
               android:onClick="@{(view)-> handlers.onClickEnemy}"/>

           <TextView android:layout_width="wrap_content"
               android:layout_height="wrap_content"
               android:text="@{user.lastName}"
               android:onClick="@{(view)-> handlers.onClickChanged(view,user.firstName,true)}"/>
       </LinearLayout>
    </layout>

```

**绑定事件有两种方式：**

如上：
  
```

    1、方法引用
    android:onClick="@{handlers::onClickFriend}"
    //or
    android:onClick="@{handlers.onClickFriend}"

    2、监听，这种方式可以用于多个参数的情况
    android:onClick="@{(view)-> handlers.onClickEnemy}"
    
    //多个参数
    android:onClick="@{(view)-> handlers.onClickChanged(view,user.firstName,true)}"

```

上面绑定事件的方式，有时候会有局限性（可能没完全掌握吧）。有时间会利用自定义setters来做事件的绑定。（下文详细说明）

---


### 布局细节

**导入方式**

data标签内可以有多个 import 标签。你可以在布局文件中像使用 Java 一样导入引用。

比如直接引用公共的常量。或者静态方法

```

    <data>
        <import type="android.view.View"/>
        <import type="com.example.MyStringUtils"/>  

        //导入变量的两种方式
        //利用import
        <import type="com.example.User"/>
        <variable name="user"  type="User"/>
        //or
        <variable name="user"  type="com.example.User"/>


    </data>

    <TextView
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"
       android:visibility="@{user.isAdult ? View.VISIBLE : View.GONE}" //常量
       android:text="@{MyStringUtils.capitalize(user.lastName)}"       //静态方法
       /> 


```

1、使用静态方法或者静态常量时，不需要加variable标签，可以直接调用。
2、变量的声明需要variable标签声明，声明后会在Binding类中生成get/set方法来调用它。
3、一些基本类型，如：string，int，等不需要写全包名。

如：

```

    <variable name="note"  type="String"/>

```

**当类名发生冲突时，可以使用 alias：
**
```
  
    <import type="android.view.View"/>
    <import type="com.example.real.estate.View"
            alias="Vista"/>

```

---

**Includes**

在使用应用命名空间的布局中，变量可以传递到任何 include 布局中。include布局内也需要先声明变量。

```

    <layout xmlns:android="http://schemas.android.com/apk/res/android"
            xmlns:bind="http://schemas.android.com/apk/res-auto">
       <data>
           <variable name="user" type="com.example.User"/>
       </data>
       <LinearLayout
           android:orientation="vertical"
           android:layout_width="match_parent"
           android:layout_height="match_parent">

           <include layout="@layout/name"
               bind:user="@{user}"/>

           <include layout="@layout/contact"
               bind:user="@{user}"/>

       </LinearLayout>
    </layout>

```

**注意：Data binding 不支持直接包含 merge 节点。
**

---


**数据的双向绑定**

>任何 POJO 都能用在 data binding 中，但是更改 POJO 并不会同步更新 UI。data binding 的强大之处就在于它可以让你的数据拥有更新通知的能力。这里有三种不同的数据变动通知机制，Observable 对象，observable 域，与 observable 容器类。
当以上的 observable 对象绑定在 UI 上，数据发生变化时，UI 就会同步更新。

```

    private static class User extends BaseObservable {
       private String firstName;
       private String lastName;

       @Bindable
       public String getFirstName() {
           return this.firstName;
       }

       @Bindable
       public String getLastName() {
           return this.lastName;
       }

       public void setFirstName(String firstName) {
           this.firstName = firstName;
           notifyPropertyChanged(BR.firstName);
       }

       public void setLastName(String lastName) {
           this.lastName = lastName;
           notifyPropertyChanged(BR.lastName);
       }
    }

```

只需要继承BaseObservable类，然后在getXX()方法上使用Bindable 注解，并在setXXX()方法里面实现通知notifyPropertyChanged()。就可以实现双向绑定。

只需要修改数据，UI自动更新。

如果数据类不多，如上，也可以用更简单的方式实现双向绑定。

```
  
    private static class User {
     public final ObservableField<String> firstName =
         new ObservableField<>();
     public final ObservableField<String> lastName =
         new ObservableField<>();
     public final ObservableInt age = new ObservableInt();
  }

```

直接创建一个ObservableField域，然后只需要使用get set 方法，就可以修改数据和UI。

*注意：*
  当变量或者 observable 发生变动时，会在下一帧触发 binding。有时候 binding 需要马上执行，这时候可以使用 executePendingBindings())。

```
  
    user.firstName.set("Google");
    int age = user.age.get();

```

---

**Observable 容器类
**

>Observable 容器类允许使用 key 来获取这类数据。

如：

```

    ObservableArrayMap<String, Object> user = new ObservableArrayMap<>();
    user.put("firstName", "Google");
    user.put("lastName", "Inc.");
    user.put("age", 17);

    ...
    <!-- 布局中 -->

    <data>
        <import type="android.databinding.ObservableMap"/>
        <variable name="user" type="ObservableMap&lt;String, Object>"/> //注意这个类型的声明。
    </data>
    …
    <TextView
       android:text='@{user["lastName"]}'
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"/>
    <TextView
       android:text='@{String.valueOf(1 + (Integer)user["age"])}'
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"/>

```

**注：这里的声明。**

当然也可以用List

```
  
    ObservableArrayList<Object> user = new ObservableArrayList<>();
    user.add("Google");
    user.add("Inc.");
    user.add(17);

    ...
    <!-- 布局文件 -->

    <data>
      <import type="android.databinding.ObservableList"/>
      <import type="com.example.my.app.Fields"/>
      <variable name="user" type="ObservableList&lt;Object>"/>
    </data>
  
      <TextView
         android:text='@{user[Fields.LAST_NAME]}'
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"/>

      <TextView
         android:text='@{String.valueOf(1 + (Integer)user[Fields.AGE])}'
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"/>

```

---

**表达式语言**

>从上面代码也可以看到，databinding允许使用一些表达式语言。

* 数学计算： + - ／ * %
* 字符串连接： +
* 逻辑符号： && ||
* 二进制：& | ！
* 比较：== > < >= <=
* instanceof
* 组：()
* null
* 数字
* 类型转换
* 函数调用
* 域存取
* 三元运算符 ？：

```

    android:text="@{String.valueOf(index + 1)}"
    android:visibility="@{age < 13 ? View.GONE : View.VISIBLE}"
    android:transitionName='@{"image_" + id}'

```
 
*null合并运算符：*
  Null合并运算符(??)会在非 null 的时候选择左边的操作，反之选择右边。
  可以用于避免NullPointerException。


```

  android:text="@{user.displayName ?? user.lastName}"
  
  //等同于
  android:text="@{user.displayName != null ? user.displayName : user.lastName}"

```

*通用的容器类：*
  数组，lists，sparse lists，和 map，可以用 [] 操作符来存取

```

    <data>
        <import type="android.util.SparseArray"/>
        <import type="java.util.Map"/>
        <import type="java.util.List"/>
        <variable name="list" type="List&lt;String>"/>
        <variable name="sparse" type="SparseArray&lt;String>"/>
        <variable name="map" type="Map&lt;String, String>"/>
        <variable name="index" type="int"/>
        <variable name="key" type="String"/>
    </data>
      …
      android:text="@{list[index]}"
      …
      android:text="@{sparse[index]}"
      …
      android:text="@{map[key]}"
      …
      android:text='@{map["firstName"]}'  //这里用单引号包裹

```

*注：*
  1、注意这个里的声明
  2、当@{}内部需要用字符串时，外面可以用单引号包裹。

---

**引用资源**
>也可以在表达式中使用普通的语法来引用资源：

```

    android:padding="@{large? @dimen/largePadding : @dimen/smallPadding}"

```

*字符串格式化可以这样实现：*

```

    android:text="@{@string/nameFormat(firstName, lastName)}"

    ...
    <!-- 资源文件 -->

    <string name="nameFormat">%s, %s</string>

```


|Class | Listener Setter Attribute|
| --------   | ----- |
|String[] | @array  @stringArray|
|int[]| @array  @intArray|
|TypedArray  |@array  @typedArray|
|Animator | @animator @animator|
|StateListAnimator | @animator @stateListAnimator|
|color int| @color  @color|
|ColorStateList | @color  @colorStateList|


----


### 高级Binding

#### 属性Setter
>当绑定数据发生变动时，生成的 binding 类必须根据 binding 表达式调用 View 的 setter 函数。Data binding 框架内置了几种自定义赋值的方法。

**自动Setter**
>对一个 attribute 来说，data binding 会尝试寻找对应的 setAttribute 函数。属性的命名空间不会对这个过程产生影响，只有属性的命名才是决定因素。

举个例子，针对一个与 TextView 的 android:text 绑定的表达式，data binding会自动寻找 setText(String) 函数。如果表达式返回值为 int 类型， data binding则会寻找 setText(int) 函数。所以需要小心处理函数的返回值类型，必要的时候使用强制类型转换。需要注意的是，data binding 在对应名称的属性不存在的时候也能继续工作。你可以轻而易举地使用 data binding 为任何 setter “创建” 属性。举个例子，support 库中的 DrawerLayout 并没有任何属性，但是有很多 setter，所以你可以使用自动 setter 的特性来调用这些函数。

```

    <android.support.v4.widget.DrawerLayout
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      app:scrimColor="@{@color/scrim}"
      app:drawerListener="@{fragment.drawerListener}"/>

```

**自定义Setter**
>一些属性需要自定义 setter 逻辑。例如，目前没有与 android:paddingLeft 对应的 setter，只有一个 setPadding(left, top, right, bottom) 函数。结合静态 binding adapter 函数与 BindingAdapter 注解可以让开发者自定义属性 setter。

***这个特别有用！！***

例如，这是一个 paddingLeft 的自定义 setter：

```

    @BindingAdapter("android:paddingLeft")
    public static void setPaddingLeft(View view, int padding) {
       view.setPadding(padding,
                       view.getPaddingTop(),
                       view.getPaddingRight(),
                       view.getPaddingBottom());
    }

```

一个 loader 可以在非主线程加载图片:

```

    @BindingAdapter({"bind:imageUrl", "bind:error"})
    public static void loadImage(ImageView view, String url, Drawable error) {
       Picasso.with(view.getContext()).load(url).error(error).into(view);
    }

    <!-- 布局文件 -->
    <ImageView app:imageUrl=“@{venue.imageUrl}”
      app:error=“@{@drawable/venueError}”/>
```

也可以用于事件的传递：

```

    @BindingAdapter("android:onLayoutChange")
    public static void setOnLayoutChangeListener(View view, View.OnLayoutChangeListener oldValue,
           View.OnLayoutChangeListener newValue) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
            if (oldValue != null) {
                view.removeOnLayoutChangeListener(oldValue);
            }
            if (newValue != null) {
                view.addOnLayoutChangeListener(newValue);
            }
        }
    }

```


当 listener 内置多个函数时，必须分割成多个 listener。例如，View.OnAttachStateChangeListener 内置两个函数：onViewAttachedToWindow()) 与 onViewDetachedFromWindow())。在这里必须为两个不同的属性创建不同的接口。

```
    //创建接口，注解上了HONEYCOMB_MR1以上可用
    @TargetApi(VERSION_CODES.HONEYCOMB_MR1)
    public interface OnViewDetachedFromWindow {
        void onViewDetachedFromWindow(View v);
    }

    @TargetApi(VERSION_CODES.HONEYCOMB_MR1)
    public interface OnViewAttachedToWindow {
        void onViewAttachedToWindow(View v);
    }

```

设置不同的使用方式

```

    @BindingAdapter("android:onViewAttachedToWindow")
    public static void setListener(View view, OnViewAttachedToWindow attached) {
        setListener(view, null, attached);
    }

    @BindingAdapter("android:onViewDetachedFromWindow")
    public static void setListener(View view, OnViewDetachedFromWindow detached) {
        setListener(view, detached, null);
    }

    @BindingAdapter({"android:onViewDetachedFromWindow", "android:onViewAttachedToWindow"})
    public static void setListener(View view, final OnViewDetachedFromWindow detach,
            final OnViewAttachedToWindow attach) {
        if (VERSION.SDK_INT >= VERSION_CODES.HONEYCOMB_MR1) {
            final OnAttachStateChangeListener newListener;
            if (detach == null && attach == null) {
                newListener = null;
            } else {
                newListener = new OnAttachStateChangeListener() {
                    @Override
                    public void onViewAttachedToWindow(View v) {
                        if (attach != null) {
                            attach.onViewAttachedToWindow(v);
                        }
                    }

                    @Override
                    public void onViewDetachedFromWindow(View v) {
                        if (detach != null) {
                            detach.onViewDetachedFromWindow(v);
                        }
                    }
                };
            }

            //获取旧的监听器
            final OnAttachStateChangeListener oldListener = ListenerUtil.trackListener(view,
                    newListener, R.id.onAttachStateChangeListener);

            //移除旧的监听器
            if (oldListener != null) {
                view.removeOnAttachStateChangeListener(oldListener);
            }

            //重新设置监听器
            if (newListener != null) {
                view.addOnAttachStateChangeListener(newListener);
            }
        }
    }

```

**注意：对于listener 内置多个函数时，可以用上面官方给的方案解决。
**







