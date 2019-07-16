# 负margin解决横向列表流的间距问题

## 1. 需求

一个列表，100x100的item横向排列，间距为10px，每行排列3个。



## 2. 基本布局

因为是横向、换行排列，类似于流，可以使用float布局。具体看本目录的html文件，这里只列出重要的代码。

- HTML

```html
<div class="container">
  <ul class="list">
    <li class="item red">1</li>
    <li class="item green">2</li>
    <li class="item blue">3</li>
    <li class="item red">4</li>
    <li class="item green">5</li>
  </ul>
</div>
```

- CSS：

每个 item 只设置 margin-right ，作为每两个 item 之间的间距：

```css
.list {
    overflow: hidden; /* BFC避免列表高度塌陷 */
    width: 320px; /* 100 * 3 + 10 * 2 = 320 */
}

.item {
    width: 100px;
    height: 100px;
    margin-right: 10px; /* 作为水平间距 */
    margin-bottom: 10px;
}
```



## 3. 问题与解决方案

目前的问题是，一行容不下3个，因为一行的3个item的外部尺寸总和：

> (100 + 10) * 3 = 330 > 320

已经超出了列表的 320px 的宽度，表现为每行只能容下2个item。

以下讨论几个解决方案。

### 3.1 使用nth-child伪类

每一行的最后一个的“下标”是3的倍数，重置其 margin-right 为 0。

```css
.item:nth-child(3n) {
    margin-right: 0;
}
```

然而，这种方法中的“3n”写死了，不好扩展。如果需求变成了每行放4个，需要同时改动两个地方：

- list/item的宽度
- 将3n改为4n

感觉不是那么DRY。

### 3.2 使用负margin

虽然使用的是负 margin ，但起作用的其实是“流”的特性：一个“流体”(可以理解为 width 为 auto 的块级元素)设置负 margin(left/right) ，则其 border-box 尺寸变大（其实理解起来很简单）。

回顾上面的布局。这次 320px 的宽度不再设置在 list 上，而需要设置在 list 的父元素 container 上：

```css
.container {
    width: 320px;
}
```

这一步很重要，是为了让 list 变回“流体”。接着负 margin 上场，设置在 list 上：

```css
.list {
    margin-right: -10px; /* 数值与间距相等 */
}
```

此时就可以使每3个item在同一行了。因为设置了负 margin 后， list 的 border-box 为：

> 320 - (-10) = 330px

又因为 list 没有设置 border、padding 的宽度，所以其 content-box 即一行的宽度就变成了 330px ，于是能够容下3个item以及其右边距。

同时，它的代码很DRY，如果需求改成了一行4个，我可以只需要改一个地方：

```css
.container {
    width: 430px; /* 4 * 100 + 3 * 10 = 430 */
}
```

需求又要改成一行5个，那就：

```css
.container {
    width: 540px; /* 5 * 100 + 4 * 10 = 540 */
}
```

