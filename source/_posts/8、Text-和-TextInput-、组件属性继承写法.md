---
title: RN学习：8、Text 和 TextInput组件属性继承写法
date: 2017-03-29 10:50:16
tags: [React-native]
---

## Text
>一个用于显示文本的React组件，并且它也支持嵌套、样式，以及触摸处理
 1、嵌套的标题可以继承父元素的属性。也可以层叠掉父元素的属性。


<!--more-->

伪代码

```
    
    renderText: function() {
      return (
        <Text style={styles.baseText}>
          <Text style={styles.titleText} onPress={this.onPressTitle}>
            {this.state.titleText + '\n\n'}
          </Text>
          <Text numberOfLines={5}>
            {this.state.bodyText}
          </Text>
        </Text>
      );
    },
    ...
    var styles = StyleSheet.create({
      baseText: {
        fontFamily: 'Cochin',
      },
      titleText: {
        fontSize: 20,
        fontWeight: 'bold',
      },
    };

```

---

### 属性

>**numberOfLines={3}**
    用来当文本过长的时候裁剪文本。包括折叠产生的换行在内，总的行数不会超过这个属性的限制。
    类似于android的Maxline


>onLongPress={里面是调用的函数} 
    长按回调
    1、里面的函数一定要加this
    2、绑定函数最好使用箭头函数绑定

```html 

    //在类里面的定义函数
    onPressPhone(){
      console.log("长按电话");
    }

      <Text style={styles.TextStyle}
        onLongPress={()=>this.onPressPhone()}>
       手机号码
      </Text>

```

>onPress={里面是调用的函数}
    文本被点击时触发
    1、里面的函数一定要加this

---

## 样式（style）

> * 继承View的样式
  * color 
  * fontFamily
  * fontSize
  * fontStyle enum('normal', 'italic')
  * fontWeight enum('normal','bold')
  * lineHeight number 
  * textAlign enum('auto', 'left', 'right', 'center', 'justify')
      - 指定文本的对齐方式。其中'justify'值仅iOS支持，在Android上会变为left


### 嵌套文本
>把相同格式的文本嵌套包裹起来：

```html 

    <Text style={{fontWeight: 'bold'}}>
      I am bold
      <Text style={{color: 'red'}}>
        and red
      </Text>
    </Text>

```


## TextInput
>TextInput是一个允许用户在应用中通过键盘输入文本的基本组件。本组件的属性提供了多种特性的配置，譬如自动完成、自动大小写、占位文字，以及多种不同的键盘类型（如纯数字键盘）等等。


### 属性

> * autoCapitalize 控制TextInput是否要自动将特定字符切换为大写
>     - characters: 所有的字符。
>     - words: 每个单词的第一个字符。
>     - sentences: 每句话的第一个字符（默认）。
>     - none: 不自动切换任何字符为大写。
>  
> * defaultValue 提供一个文本框中的初始值。当用户开始输入的时候，值就可以改变。
> * placeholder 如果没有任何文字输入，会显示此字符串。
> * multiline 为true时可以输入多行文字。
> * maxLength 限制文本框中最多的字符数。使用这个属性而不用JS逻辑去实现，可以避免闪烁的现象。
> * keyboardType 决定弹出的何种软键盘的
>     - default
>     - numeric
>     - email-address
> * onChange 当文本框内容变化时调用此回调函数。
> * onChangeText 当文本框内容变化时调用此回调函数。改变后的文字内容会作为参数传递。
> * secureTextEntry 加密
> * underlineColorAndroid 去掉下划线 




## 组件继承写法

> {...this.props} 就可以继承了

```html 

    class MyTextInput extends Component {
      render(){
        return (
          <View>
            <TextInput style={styles.InputStyle}
                {...this.props}     //用于继承
                autoCapitalize='characters'
                underlineColorAndroid='transparent'
                onChangeText={
                  (input_Phone) => this.setState({input_Phone})
              }
            />

            <Text style={styles.TextPrefixPhone}>+86</Text>

             <View style={styles.verticalLine}/>
          </View>

        );
      }

    }


     <MyTextInput   //继承上面的属性
          placeholder="请输入账号"
         />


```