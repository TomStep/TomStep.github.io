---
title: 5、状态（State）
date: 2017-03-29 10:46:14
tags: [React-native]
---

### State的使用
>在RN中一般使用两种数据来控制一个组件。
 props,它是在父控件中指定，一经指定就不会在改变。
 State,它相当于一个动态的数据。是可以在控件中改变的

 **一般来说，你需要在constructor中初始化state，然后在需要修改时调用setState方法。**

<!--more-->

假如我们需要制作一段不停闪烁的文字。文字内容本身在组件创建时就已经指定好了，所以文字内容应该是一个prop。而文字的显示或隐藏的状态（快速的显隐切换就产生了闪烁的效果）则是随着时间变化的，因此这一状态应该写到state中。

``` 

    import React, {
      Component
    } from 'react';

    import {
      AppRegistry,
      View,
      Image,
      Text,
    } from 'react-native';


    class Blink extends Component {
      constructor(props) {
        super(props);
        this.state = {
          showText: true
        }
        setInterval(() => {
          this.setState({
            showText: !this.state.showText
          });
        }, 1000);
      }

      render() {
        // 根据当前showText的值决定是否显示text内容
        let display = this.state.showText ? this.props.text : ' ';
        return (
          <Text>{display}</Text>
        );
      }
    }


    class BlinkApp extends Component {
      render() {
        return (
          <View>
            <Blink text='I love to blink' />
            <Blink text='Yes blinking is so great' />
            <Blink text='Why did they ever take this out of HTML' />
            <Blink text='Look at me look at me look at me' />
          </View>
        );
      }
    }

    AppRegistry.registerComponent('AwesomeProject', () => BlinkApp);

```

---
#### setInterval()函数

>首先，认识一下setInterval（)函数
 1、setInterval() 方法可按照指定的周期（以毫秒计）来调用函数或计算表达式。
 2、setInterval() 方法会不停地调用函数，直到 clearInterval() 被调用或窗口被关闭。由 setInterval() 返回的 ID 值可用作 clearInterval() 方法的参数。

setInterval(code,millisec)

+ 参数1，必要的函数
+ 参数2，周期（以毫米计数）

使用clearInterval()函数来取消循环

``` 

    <script language=javascript>
    var int=self.setInterval("clock()",50) //self表示当前页面
    function clock()
      {
      var t=new Date()
      document.getElementById("clock").value=t
      }
    </script>
    </form>
    <button onclick="int=window.clearInterval(int)"> //在这个窗口下获取
    Stop interval</button>

```

---


State的核心使用方法：

1、在构造函数中初始化：this.state = ...
2、通过this.setState()函数来修改state的值。
3、使用定义的变量。

*每次state中的数发生变化时，会刷新一次UI界面。来修改数据*