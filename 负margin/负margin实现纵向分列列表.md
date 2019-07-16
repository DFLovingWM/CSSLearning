# 负margin实现纵向分列列表

## 1. 需求

一个纵向列表，数量太多而item很短时，分成几列展示。通常用于菜单列表。

比如5个菜单项分成三列，第一列显示1、2，第二列显示3、4，第三列显示5。



## 2. 基本布局

- HTML：top标记每一列的头，colX表示第几列。

```html
<ul>
  <li class="col1">首页</li>
  <li class="col1">归档</li>
  <li class="col2 top">分类</li>
  <li class="col2">联系方式</li>
  <li class="col3 top">关于我们</li>
</ul>
```

- CSS：利用负margin使多个block元素(li)占据同一行。

```css
ul {
    list-style: none;
    line-height: 2em; /* 使每个li高度为2em */
    width: 6em; /* 使每个li的(容错)宽度为6em */
}

.top {
    margin-top: -4em; /* 因为每列2个，所以向上偏移2个li的高度 */
}

.col2 {
    margin-left: 6em; /* 第2列与第1列水平分隔开 */
}

.col3 {
    margin-left: 12em; /* 第3列与第1列水平分隔开 */
}
```

可以这么理解：

1. 一开始每个块级元素 li 都在自己的那一行上。
2. col2往上偏移2个li高度即4em ，因为margin与relative不同，它会影响到后续元素的位置，即col2、col3都上升了4em；而因为col3的top也设置了“上升4em”，所以col3整体上升了8em。如此，3列都在同一水平线上了。
3. 但此时是重合的，需要设置col2、col3的margin-left，它相对的不是彼此、而是父元素的最左侧，所以分别为6em、12em。



## 3. 总结

负margin的应用场景：

- 使block元素在同一行展示，形成多列布局（本文）。
- 增加流体的“可用宽度”(border-box)。
- 结合float，构建多列布局。