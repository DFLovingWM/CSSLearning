# 包含块与百分比

## 包含块的定义

一个元素A的包含块（Containing block）取决于A的定位，如果A的`position`属性为：

- static/relative/sticky，则包含块为最近的块容器或上下文的`content box`：
  - 块容器（Block container）祖先，比如`display: inline-block/block/list-item`
  - 上下文（Formatting context）祖先，比如：table/flex/grid container
- absolute：最近的定位不为`static`的祖先元素的`padding box`
- fixed：viewport，即宽度百分比相当于`vw`、高度百分比相当于`vh`
- 特殊情况：当`position == absolute | fixed`时，包含块还可以是最近的满足下面条件之一的任意元素的`padding box`：
    - `transform/perspective`的值不为`none`，或者`will-change`的值为`transform/perspective`
    - （仅Firefox）`filter`的值不为`none`，或者`will-change`的值为`filter`
    - `contain: paint`

## 百分比与包含块的关系

许多表示尺寸的属性值除了能取定值，还能使用百分比。值为百分比时，依赖于包含块的属性有：

- width/height：包含块的宽/高
- margin/padding：包含块的宽（只有宽度没有高度，较为特殊。另外还需注意`border`不支持百分数）
- left/right：包含块的宽
- top/bottom：包含块的高

此外，还有依赖于自身的：

- transform中的`translate(x, y)`：自身`border box`的宽/高
- border-radius：自身`border-box`的宽/高

## 参考

- [MDN: 包含块](https://developer.mozilla.org/en-US/docs/Web/CSS/Containing_block)
