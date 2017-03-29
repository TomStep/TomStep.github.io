---
title: java 反射机制
date: 2017-03-29 09:59:59
tags: [Java]
---

**反射解决的问题：**

> 反射主要解决动态编程,即使用反射时,所有的对象生成是动态的,因此调用的方法也是动态的。反射可以简化开发,但是代码的可读性很低。


<!-- more --> 

### Class对象

> java中所有类型都对应一个Class对象（java.lang.Class）
> 
> 虚拟机为每种类型管理一个独一无二的class对象。运行程序时，java
> 虚拟机首先检查是否所要加载的类相对应的class对象已经加载，如果没有加载，虚拟机会根类名查找.class文件，并将其class对象载入。
> 
> 一般某个类的class对象被载入内存，它就用来创建这个类的所有对象。

<img src='http://ofjn4kn3q.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-03-29%20%E4%B8%8A%E5%8D%8810.15.53.png'>


**如何获取class对象**

1、**对象.getClass()** 或者 **类名.Class** 可以获取对应对象的class。

2、对于基本类型：封装类.TYPE代表了对应的基本类型的Class对象 

    如： Integer.TYPE对应的是int的Class对象 

3、如果知道类名（全名)可以通过Class.forName(className)获取。这是动态的获取class对象。

---


### 反射获取对象。

>上面已经获得了class对象。我们真正想要的是类的实例。如何获得呢？


代码：
```

    //创建一个类student类
    public class Student {
        private String name;
        private int age;
        private String tag;

        public Student() {
            tag = "无参数";
        }

        public Student(String name, int age) {
            this.name = name;
            this.age = age;
            tag = "含参数";
        }

        @Override
        public String toString() {
            return tag;
        }
    }

```

测试代码：

```
    
    Class c = Class.forName("com.clazz.reflect.Student");

    //无参数的做法
    Object cObj = c.newInstance();//调用无参的构造方法
    System.out.prin(cObj);  //打印：无参数

    //有参数的做法
    Constructor conAll = c.getConstructor(String.class,int.class); //传入参数类型
    Object caobj = conAll.newInstance("李四",23);//调用含参的构造方法.
    System.out.prin(caobj);  //打印：含参数

```

1、创建无参数对象：.newInstance();

2、创建有参对象： 先通过getConstructor()指定参数类型，然后在调用newInstance()。

**注：如何student类本身没有无参构造函数，直接调用newInstance()，会报错**


---


### 关于反射的其他方法

>上面介绍了newInstance()方法和getConstructor()方法使用，下面详细介绍其他常用几个方法：


**1、获取构造方法：**

    Constructor<T> getConstructor(Class<?>... parameterTypes)  

        返回一个 Constructor 对象，它反映此 Class 对象所表示的类的指定公共构造方法。 

    Constructor<?>[] getConstructors() 

        返回一个包含某些 Constructor 对象的数组，这些对象反映此 Class 对象所表示的类的所有公共构造方法。 


**2、获取成员字段：**

     Field getField(String name)    //参数：属性名

          返回一个 Field 对象，它反映此 Class 对象所表示的类或接口的指定公共成员字段。 

     Field[] getFields() 

          返回一个包含某些 Field 对象的数组，这些对象反映此 Class 对象所表示的类或接口的所有可访问公共字段。

     Field getDeclaredField(String name)   //参数：属性名

          返回一个 Field 对象，该对象反映此 Class 对象所表示的类或接口的指定已声明字段。 

     Field[] getDeclaredFields() 

          返回 Field 对象的一个数组，这些对象反映此 Class 对象所表示的类或接口所声明的所有字段。 


**3、获取成员方法：**

     Method getMethod(String name, Class<?>... parameterTypes)  //参数：方法名，参数类型数组

      返回一个 Method 对象，它反映此 Class 对象所表示的类或接口的指定公共成员方法。 

    Method[] getMethods() 

      返回一个包含某些 Method 对象的数组，这些对象反映此 Class 对象所表示的类或接口（包括那些由该类或接口声明的以及从超类和超接口继承的那些的类或接口）的公共 member 方法。 

    Method getDeclaredMethod(String name, Class<?>... parameterTypes) //参数：方法名，参数类型数组

      返回一个 Method 对象，该对象反映此 Class 对象所表示的类或接口的指定已声明方法。

    Method[] getDeclaredMethods() 

      返回 Method 对象的一个数组，这些对象反映此 Class 对象表示的类或接口声明的所有方法，包括公共、保护、默认（包）访问和私有方法，但不包括继承的方法。 


**4、创建实例、执行方法、转换字符串**

    T newInstance() 

      创建此 Class 对象所表示的类的一个新实例。 <new Instance()可以动态的创建对象>

    String toString() 

      将对象转换为字符串。

    method.invoke(Object receiver, Object ... args)  //参数：对象实例，方法需要的参数

      执行方法


注意：如果拿到的是私有成员变量，首先要调用setAccessible(true)方法，表示允许访问修改。

    如：
        Field name = aClass.getDeclaredField("name");
        name.setAccessible(true);
        name.set(student1,"张三");


----

### 反射其他高级用法
>既然Class对象可以拿到类的所有信息，那当类有父类，有接口的时候，就可以通过反射获取它们。

```java

    //获取父类的class对象，拿到这个实例就可以对父类进行反射操作。
    Class superclass = aClass.getSuperclass();

    //获取接口集合
    Class[] interfaces = aClass.getInterfaces();


```


**在泛型上的运用：**

```java

    //获取返回值泛型类型
    getGenericReturnType()

    //获取方法参数泛型类型
    getGenericParameterTypes()
    
    //超类的泛型类型
    getGenericSuperclass()

```


```java

    public class MyClass<T> {
      protected List<String> stringList = ...;

      public List<String> getStringList(){
        return this.stringList;
      }

      public void setStringList(List<String> list){
        this.stringList = list;
      }
    }

    ...

    //获取返回值的泛型类型
    Method method = MyClass.class.getMethod("getStringList", null);
    Type returnType = method.getGenericReturnType();

    if(returnType instanceof ParameterizedType){
        ParameterizedType type = (ParameterizedType) returnType;
        Type[] typeArguments = type.getActualTypeArguments();
        for(Type typeArgument : typeArguments){
            Class typeArgClass = (Class) typeArgument;
            System.out.println("typeArgClass = " + typeArgClass);
        }
    }

    //获取参数上的泛型类型
    method = Myclass.class.getMethod("setStringList", List.class);
    Type[] genericParameterTypes = method.getGenericParameterTypes();

    for(Type genericParameterType : genericParameterTypes){
        if(genericParameterType instanceof ParameterizedType){
            ParameterizedType aType = (ParameterizedType) genericParameterType;
            Type[] parameterArgTypes = aType.getActualTypeArguments();
            for(Type parameterArgType : parameterArgTypes){
                Class parameterArgClass = (Class) parameterArgType;
                System.out.println("parameterArgClass = " + parameterArgClass);
            }
        }
    }

    //获取成员变量上的泛型类型
    field = Myclass.class.getDeclaredField("stringList");
    Type genericType = surper.getGenericType();

    if(genericType instanceof ParameterizedType){
        ParameterizedType aType = (ParameterizedType) genericType;
        Type[] parameterArgTypes = aType.getActualTypeArguments();
        for(Type parameterArgType:parameterArgTypes){
            Class fieldArgClass = (Class) parameterArgType;
            System.out.print("fieldArgClass = " + fieldArgClass);
        }
    }

    //获取类上的泛型(getGenericSuperclass方法使用时有规定)
    Class<?> clazz = getClass(); //获取实际运行的类的 Class
    Type type = clazz.getGenericSuperclass(); //获取实际运行的类的直接超类的泛型类型
    if(type instanceof ParameterizedType){ //如果该泛型类型是参数化类型
        Type[] parameterizedType = ((ParameterizedType)type).getActualTypeArguments();//获取泛型类型的实际类型参数集
        entityClass = (Class<T>) parameterizedType[0]; //取出第一个(下标为0)参数的值

        System.out.print("entityClass = " + entityClass);
    }

```

---


```java

    public class Base<T> {
      private Class<T> entityClass;

      public Base(){
        try {
            Class<?> clazz = getClass(); //获取实际运行的类的 Class
            Type type = clazz.getGenericSuperclass(); //获取实际运行的类的直接超类的泛型类型
            if(type instanceof ParameterizedType){ //如果该泛型类型是参数化类型
                Type[] parameterizedType = ((ParameterizedType)type).getActualTypeArguments();//获取泛型类型的实际类型参数集
                entityClass = (Class<T>) parameterizedType[0]; //取出第一个(下标为0)参数的值
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
      }
      
      //泛型的实际类型参数的类全名
      public String getEntityName(){
          return entityClass.getName();
      }
      
      //泛型的实际类型参数的类名
      public String getEntitySimpleName(){
          return entityClass.getSimpleName();
      }

      //泛型的实际类型参数的Class
      public Class<T> getEntityClass() {
          return entityClass;
      }
      

      ...

    public class Child extends Base<Child>{

    } 

    public static void main(String[] args){
        Child child = new Child();
        System.out.println(child.getEntityClass());
        System.out.println(child.getEntityName());
        System.out.println(child.getEntitySimpleName());
    }

    //输出：
      class test.Child
      test.Child
      Child
  }

```

**所以通过泛型反射，也只有在 超类（调用 getGenericSuperclass 方法） 或者成员变量（调用 getGenericType 方法）或者方法（调用 getGenericParameterTypes 方法）像这些有方法返回 ParameterizedType 类型的时候才能反射成功。
**
 