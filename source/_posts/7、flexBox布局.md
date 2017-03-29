---
title: 7、flexBox布局
date: 2017-03-29 10:48:15
tags: [React-native]
---

>参考阮一峰大神的博客:http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html

### 基本概念

>采用Flex布局的元素，称为Flex容器。它所有的子类元素自动成为容器成员(flex item)。

1、容器默认存在两个轴：水平的主轴（main axis）,垂直的交叉轴（cross axis)
2、主轴的开始位置（与边框的交叉点）叫做main start，结束位置叫做main end；（从左到右）
3、交叉轴的开始位置叫做cross start，结束位置叫做cross end。（从上到下）

<!--more-->

### 容器的属性

* flex-direction 决定主轴的方向
    - row （默认值）：主轴为水平方向，起点在左边。
    - row-reverse  ：主轴为水平方法，起点在右边。
    - column ：主轴为垂直方向，起点在上边。
    - column-reverse ：主轴为垂直方向，起点在下边。
    
* flex-wrap 决定子元素的排布方式
    - nowrap 沿轴线排布，不换行
    - wrap 沿轴线排布，换行
    - wrap-reverse;换行，最下面是第一行
    
* flex-flow 是flex-direction 和 flex-wrap的简写形式，默认为row nowrap
    - flex-flow: <flex-direction> || <flex-wrap>;


* justify-content 在主轴上的对齐方式
    - flex-start 主轴开始的方向对齐
    - flex-end 主轴结束的的方法对齐
    - center 居中
    - space-between 两端对齐，子元素之间的距离相等
    - space-around 每个项目两侧的间隔相等。所以，项目之间的间隔比项目与边框的间隔大一倍。

* align-items 在交叉轴的对齐方式
    - flex-start 
    - flex-end 
    - center 
    - baseline 项目的第一行文字的基线对齐。
    - stretch (默认值）：如果项目未设置高度或设为auto，将占满整个容器的高度。
    
* align-content

---


#### 子元素的属性

* order 属性定义项目的排列顺序。数值越小，排列越靠前，默认为0
    - order : -1;

* flex-grow 属性定义项目的放大比例，默认为0，即如果存在剩余空间，也不放大。
    - flex-grow: <number>; /* default 0 */
    - 貌似只有在direction为row时，才有用。是横向放大
    - 有点像权重

* flex-shrink 属性定义了项目的缩小比例，默认为1，即如果空间不足，该项目将缩小。
    - 如果所有项目的flex-shrink属性都为1，当空间不足时，都将等比例缩小。
* flex-basis
* flex

* align-self align-self属性允许单个项目有与其他项目不一样的对齐方式，可覆盖align-items属性。默认值为auto，表示继承父元素的align-items属性，如果没有父元素，则等同于stretch。