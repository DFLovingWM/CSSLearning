# 负margin的作用

## 1 流体
这里“流体“是指：不指定width的块级元素。默认情况下它会填充父容器的可用宽度，即它的：
> content-box宽度 = 父元素content-box宽度 - ( border宽度 + padding宽度 + margin宽度 )
所以对流体应用(水平方向的)负margin时，其content width就会相应地增加。

举个例子：
```html
<div class="container" style="width: 100px;">
  <div class="thinner" style="margin-left: 10px;">流体1</div>
  <div class="fatter" style="margin-left: -10px;">流体2</div>
</div>
```
解释：
- container指定了width为100px，所以其content-box宽度为100px(因为box-sizing默认值为content-box)。
- thinner是流体，指定了正margin后，其(content-box的)宽度为：100px - 10px = 90px。
- 指定了负margin的fatter，其(content-box的)宽度为：100px - (-10px) = 110px。

而对于定宽元素(非流体)，margin只能移动其位置，而不能改变其宽度。

## 2 非流体或float元素
可以参考该图：

![负margin对static元素的作用](https://cloud.netlifyusercontent.com/assets/344dbf88-fdf9-42bb-adb4-46f01eedd629/cbe79f7b-b5e0-4891-9322-aea13e2fe54e/margin-motion.gif)

可以总结为：
- 负的margin-top、margin-left，会让本元素往那个方向覆盖
- 负的margin-bottom、margin-right，会被后面兄弟结点覆盖

然而这个结论是错的，它基于一个前提：float为left。但是下面的结论在float为left/right都适用：
> 负margin方向与float方向：相同时会主动覆盖，不同时会被后面兄弟结点覆盖。

## 3. 参考
- [The Definitive Guide to Using Negative Margins](https://www.smashingmagazine.com/2009/07/the-definitive-guide-to-using-negative-margins/)
