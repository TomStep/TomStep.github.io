---
title: RN学习：4、Hello World程序
date: 2017-03-29 10:44:15
tags: [React-native]
---

### 第一个hello world 代码

<!--more-->

```js
    
    //导入组件
    import React, {
      Component
    } from 'react';
    import {
      AppRegistry,
      StyleSheet,
      Text,
      View
    } from 'react-native';

    //定义根容器
    export default class Hellow extends Component {
      render() {
        return (
          <Text>Hello ,world!!</Text>
        );
      }
    }
    
    //注册
    AppRegistry.registerComponent('AwesomeProject', () => Hellow);

```

要点：
1、improt 用来导入所需要的组件（详细可以参考ES6）
2、定义根容器
    + 需要继承 Component对象。
    + 要在render（）方法内返回JSX语句。如（ <Text >Hello ,world!!</Text>）
3、AppRegistry模块则是用来告知React Native哪一个组件被注册为整个应用的根容器。只调用一次。



### 加入props(属性)

```html 

    import React, {
      Component
    } from 'react';

    import {
      AppRegistry,
      Image
    } from 'react-native';

    export default class Bananas extends Component {
      render() {
        let pic = {
          uri: 'https://upload.wikimedia.org/wikipedia/commons/d/de/Bananavarieties.jpg'
        };
        return (
          <Image
              style={{width : 193,height : 110}}
              source={pic} />
        );
      }
    }

    AppRegistry.registerComponent('AwesomeProject', () => Bananas);

```

要点：
1、如何指定props?
    + 在JSX中用{}的形式包住js变量。并且变量要使用对象类型

2、加入图片使用Image标签，图片的source属性对象为{uri:''};



### 自定义组件

```html 
  
    import React, {
      Component
    } from 'react';
    import {
      AppRegistry,
      View,
      Image,
      Text,
    } from 'react-native';


    class Greeting extends Component {
      render() {
        return (
          <Text>Hello {this.props.name}!!</Text>
        );
      }
    }
    class LotsOfGreetings extends Component {
      render() {
        return (
          <View style={{alignItems: 'center'}}>
            <Greeting name='Rexxar' />
            <Greeting name='Jaina' />
            <Greeting name='Valeera' />
          </View>
        );
      }
    }

    AppRegistry.registerComponent('AwesomeProject', () => LotsOfGreetings);


```


要点：
1、自定义组件很简单，通过基础的Text,View,Image 等组件，然后定义其中的props属性，如上述代码。之后调用代码时，属性就变成了标签属性。