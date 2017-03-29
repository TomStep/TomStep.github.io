---
title: Gson解析泛型问题(更加灵活的处理方法)
date: 2016-11-30 17:33:19
tags: [Android]
---

### 出现的问题

后台服务器传回的json一般都是这样的类型：

```java
{
    "status": 0, 
    "msg" : null, 
    "data": {  
        "price": 4,  
        "name": "脉动",  
        "type": "饮料",  
        "amount": 50,  
        "summary": "随时随地，脉动回来",  
        "picture": "http://www.jpjy365.com/images/201402/source_img/29726_G_1393285888119.jpg",  
        "hot": true,  
        "sales": 100  
    },  
    "err": null  
}
```

<!-- more -->

外面都会包裹一层json,如msg这样的字段一般都不变，也很少用到,而只有里面的data数据会经常变化。我们一般都会这么做：

```java
public class BaseModle<T> {

    public boolean status;

    public String msg;

    public T data;

    public String err;
}
```

```java
public class DrinkModle {
    public int price;
    public String name;
    public String type;
    public int amount;
    public String summary;
    public String picture;
    public boolean hot;
    public int sales;
}
```

如此，在用Gson解析的时候很多人就会出现问题。

---

### 解决方案：
>利用反射获取泛型，

**直接上代码：**

```java
    //获取class类型
    private Class getTypeClassOfObject(Object obj) {
         return (Class) ((ParameterizedType) obj.getClass().getGenericSuperclass()).getActualTypeArguments()[0];
    }

    private class Element<T> implements ParameterizedType {

    private Class<T> cl;

    public Element(Class<T> cl) {
        this.cl = cl;
    }

    public Type[] getActualTypeArguments() {
        return new Type[] {cl};
    }

    //返回BaseModle类型
    public Type getRawType() {
        return BaseModle.class;
    }

    public Type getOwnerType() {
        return null;
    }
}
```


这里模拟网络请求，获取json格式的字符串。

```java
public class Requset<T> {
public <T> void getObject(String url, HashMap<String, String> paramsMap) {
    RequestParams params = convertParams(paramsMap);
    client.get(url, params, new TextHttpResponseHandler() {
        @Override
        public void onSuccess(int statusCode, Header[] headers, String responseBody) {
                //通过上述方法获取class,
                Class cl = getTypeClassOfObject(this);
                //Gson解析
                BaseModle<T> object  = new Gson().fromJson(content, new Element<T>(cl));

                getGsonData(object.data);
        }

        @Override
        public void onFailure(int statusCode, Header[] headers, String responseBody, Throwable error) {
            error.printStackTrace();
            callback.onFailure();
        }
    });
}

private RequestParams convertParams(HashMap<String, String> paramsMap) {
    RequestParams params = new RequestParams();
    if (paramsMap != null) {
        for (String key : paramsMap.keySet()) {
            params.put(key, paramsMap.get(key));
        }
    }
    return params;
}

public abstract  void getGsonData(T data);
}
```


使用：

```java
api.getObject(URL, paramsMap){
    @Override
    public void getGsonData(DrinkModle data){

        Log.d("TAG",data.toString);
    };
};
```

----


