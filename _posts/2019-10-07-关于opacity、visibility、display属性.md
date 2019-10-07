--- 
layout:     post
title:      opacity,visibility,display属性
subtitle:   
date:       2019-10-07
author:     wells
header-img: img/c3.jpg
catalog: true
tags:
    - css 
--- 


## 问题

鼠标移入div，显示菜单ul，移出后隐藏菜单ul，只使用CSS，如何实现既有淡入淡出的效果，而又不影响其他元素，不产生回流？

```css
<div>
    <ul>
    </ul>
</div>
--------------------------
div{
	position:relative;
    width:400px;
    height:40px;
    background:red;
}
div ul{
    position:absolute;   
    top:0;
    left:0;
    width:200px;
    height:300px;
    background:yellow;
}
```

## 实现

这个问题，看上去似乎很简单，有些同学一定会想到，加透明度就能就解决，来看下是不是：

```css
<div>
    <ul>
    </ul>
</div>
--------------------------
div{
	position:relative;
    width:400px;
    height:40px;
    background:red;
}
 div ul{
     position:absolute;   
     top:0;
     left:0;
     width:200px;
     height:300px;
     background:yellow;
     opacity:0;           /* 开始透明度为0 */
     transition:opacity .5s;      /*  0.5s完成 透明度0-1的变化 */
}
div:hover ul{
    opacity:1;         /* 鼠标进入div，ul的透明度从0过渡到1 */
}
```

显然这样并没有达到目的，虽然`ul`隐藏了，但是只是改变透明度，并不是真的让这个元素消失。所以当鼠标移到隐藏的`ul`对应的位置时依然会触发div的`:hover`

说到这，你也许会想到用display属性，但是也是不行的

- display不支持过渡，用了他，淡入淡出的效果就没有了，

- 还会产生会回流和重绘，

所以这里，我们给用 visibility 属性就可以了，visibility 属性，支持过渡，而且不会产生回流，虽然 visibility=hidden; 会占据页面空间，但是并不影响其他元素的事件触发和显示。这样就达到了目的，代码如下：

```css
<div>
    <ul>
    </ul>
</div>
--------------------------
div{
	position:relative;
    width:400px;
    height:40px;
    background:red;
}
 div ul{
     position:absolute;   
     top:0;
     left:0;
     width:200px;
     height:300px;
     background:yellow;
     opacity:0;          
     transition:opacity .5s;     
     visibility:hidden;   /* 增加 */
}
div:hover ul{
    opacity:1;         
    visibility:visible;    /* 增加 */
}
```

## 分析

回流和重绘的概念：

- 回流( reflow )

  > 当页面中的一部分(或全部)因为元素的规模尺寸，布局，隐藏等改变而需要重新构建。这就称为回流(也有人会把回流叫做是**重布局或者重排** )。每个页面至少需要一次回流，就是在页面第一次加载的时候。

- 重绘 ( repaint )  

  > 当页面中的一些元素需要更新属性，而这些属性**只是影响元素的外观，风格，而不会影响布局**的时候，比如background-color。则称为重绘。

> 注意：**回流必将引起重绘，而重绘不一定会引起回流。**

|                          | opacity | display | visibility |
| ------------------------ | ------- | ------- | ---------- |
| 过渡transition           | yes     | no      | yes        |
| 重绘repaint              | no      | yes     | yes        |
| 回流feflow               | no      | yes     | no         |
| 元素隐藏可否可以触发事件 | yes     | no      | no         |
| 元素隐藏是否还占页面空间 | yes     | no      | yes        |

**visibility支持过渡**
visibility属性虽然支持过渡，但是，不是平滑的过渡，而是进行了一个延时，并且它只是 从 visible 过渡 到 hidden 有延迟，从 hidden 过渡到 visible 不延迟

**透明度（opacity）不会触发重绘:**

> 实际上透明度改变后，GPU在绘画时只是简单的降低之前已经画好的纹理的alpha值来达到效果，并不需要整体的重绘。不过这个前提是这个被修改 opacity 本身必须是一个图层，如果图层下还有其他节点，GPU也会将他们透明化

## 总结
最开始的问题，一般是会出现在做一些鼠标悬停特效的时候，鼠标悬停，出现一个img，而这些元素刚开始是看不见的，他们定位在页面上，如果他们只是透明度发什么变化，很有可能，影响到其他的元素不能触发事件,要解决问题，就要给要隐藏的元素用上visibility属性。
