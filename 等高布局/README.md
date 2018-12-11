# 等高布局

## 1. 负margin+padding模拟

核心CSS如下，对所有列使用：
```scss
.container {
  overflow: hidden;

  // 对于所有列
  & > .column {
    padding-bottom: 9999px;
    margin-bottom: -9999px;
  }
}
```

它的原理是，给列的padding一个很大的值(保证大于最长的列的高度即可，比如9999px)，同时给margin一个绝对值相同的负值以抵消padding。这样，列的外在尺寸并没有改变，但因为padding增大了，列的“视觉区域”变得很长，同时外层用overflow: hidden来隐藏超出的部分，所以最终表现为列高度等于容器高度。  
也可以从负margin的效应上理解：对列增加padding-bottom后，列变得很长，这时候，对margin-bottom设置一个负值，根据负margin对浮动元素的影响，其后的元素会往上覆盖在列元素上，也就抵消了这个长度。

## 2. border模拟

用border作为短列的背景（因为border可以设置宽度、颜色）。
该方法也是一种模拟，因为短列的高度还是那么短，只是将长列的水平border赋予与列相同的颜色和宽度，作为短列的“背景”。  
缺点：
- border不支持百分数，所以短列只能是定宽的。
- border只有left与right，所以只能支持2～3列，顶多加上border-style: double又多几个对称的列。

关键代码：
```scss
.center {
  border-left: $left-width solid $left-bgColor;
  border-right: $right-width solid $right-bgColor;
}
```

## 3. flex实现

