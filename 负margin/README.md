# 负margin的特性

## 对于流体

对流体的一般理解是：非定宽块级元素。流体在默认情况下会填充父容器的`content box`，即有等式：

> content-box宽度 = 父元素content-box宽度 - ( border宽度 + padding宽度 + margin宽度 )

所以对流体使用负数的`margin-*`时，其`content-box`会增加。

通过一个例子来理解：

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

## 对于非流体、浮动元素

网上流传一个图：

![负margin对static元素的作用](https://cloud.netlifyusercontent.com/assets/344dbf88-fdf9-42bb-adb4-46f01eedd629/cbe79f7b-b5e0-4891-9322-aea13e2fe54e/margin-motion.gif)

转达一下结论：

- 负的`margin-top/left`，会让本元素往那个方向覆盖
- 负的`margin-bottom/right`，会被其后的兄弟元素覆盖

然而这个结论不完全准确，因为它基于一个前提：float为left。以下结论才具有普适性：

> 负margin方向与float方向：相同时会主动覆盖，不同时会被后面兄弟结点覆盖。

## 3. 参考

- [The Definitive Guide to Using Negative Margins](https://www.smashingmagazine.com/2009/07/the-definitive-guide-to-using-negative-margins/)
- [理解并运用 CSS 的负 margin 值](https://segmentfault.com/a/1190000007184954)
