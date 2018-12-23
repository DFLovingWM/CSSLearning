# 深入flex-basis

## 1. 场景与问题

需求：现有一个竖直的商品列表(list)。每个商品(item)中，左边(thumb)是定宽的图片，右边(detail)是商品名称(name)等文字信息。其中，商品名称需要在一行内显示，但可能会很长，而因为屏幕宽度的限制，我们需要在它过长的时候截断、并显示省略号。

基础布局为：
```html
<ul class="product-list">
  <li class="product-item">
    <div class="thumb"></div>
    <div class="detail">
      <div class="name">这个商品的名称很长很长实在难以在一行之内说完</div>
    </div>
  </li>
  <!-- More items here... -->
</ul>
```

基础CSS，使用 flex 布局：
```css
.thumb {
  /* 因为是定宽图片，不能使其压缩宽度 */
  flex-shrink: 0;
}

.detail {
  /* B部分占满剩余宽度 */
  flex-grow: 1;
}
```

然后像平常一样对 b 进行溢出文本的截断(标准的“三句话”)：
```css
.name {
  white-space: nowrap;
  overflow-x: hidden;
  text-overflow: ellipsis;
}
```

结果发现并没有达到期望效果，name 的文字过多时，还是会超出 item 的边界而显示，甚至超出了屏幕(如不设置overflow，页面可向左滚动)。

## 2. 分析原因

首先，上面的“三句话”生效的前提是：name 是定宽、且宽度小于文本总长。在这个布局中，flex 容器(product-item)是流体、所以是满宽的，然后左右两个 flex-item 的情况：左边的 thumb 是定宽，右边的 detail 占据剩余宽度，按道理这个 detail 也算是定宽的了。

但是，在浏览器的“开发者工具”中审查元素时发现，detail 的并不是定宽的！然后进一步发现：当子元素宽度较小时，它的确是占满容器中剩余的宽度；但当子元素超出了它的宽度时，它的宽度就为子元素的最长宽度了。

所以有了本博客的疑问：当一个 flex-item 元素的子元素宽度超出时，为什么它的宽度不再被 flex container 限制、而随着子元素而增长，从而导致了一系列的问题？这个是BUG吗？

## 3. 深入 flex-basis 属性

其实这涉及到 flex-basis 这个属性的特点。它表示的是一个元素在放入 flexbox 之前的初始、假设(hypothetical)宽度。为什么说是“hypothetical”，因为最终宽度可能不是所声明的宽度，可能会因为 flex-grow、flex-shrink 等属性的参与而有偏移量。

flex-basis 的取值有以下几种：

- 长度值
- 百分数，相对于 flex container 的宽/高
- auto(默认值)：退一步，让width/height来决定。
- content：取决于最大子元素宽度(即包裹性)。但这个取值还没有被广泛实现，在笔者当下使用的 Chrome(70.0.3538.102) 中会提示“Invalid property value”。

在决定 flex-item 的假设宽度时，我们会发现设置 width 也是有效的。其实，它们之间存在一个优先级的问题：

1. 如果指定了 flex-basis，则 width 的值在这里被忽略，即 flex-basis 直接决定了假设宽度。但同时，它会受到 min-width 和 max-width 的限制。
2. 没有指定 flex-basis 即 flex-basis 为 auto 时，取 width/height 的值。
3. 没有指定 width/height 即 width/height 为 auto 时，由其内容/子元素决定，相当于使 flex-basis 取 “content”。

一句话概括：

> flex-basis(受限于min-width与max-width) > width/height > 内容("content")

理解了之后，回到上述问题中继续分析：对于 detail 这个 flex-item，它并没有设置 flex-basis，也并没有设置宽度，所以它的宽度其实是 “content” 的。当子元素内容较少时，它的宽度其实也很小，但因为设置了 flex-grow 的缘故，就被撑大到占满剩余宽度；当子元素内容过多时，它的宽度也变得一样大，超越了 flex container 的界限。

## 4. 解决

如果理解了，解决方案也差不多能想到了——<u>关键是要让 flex-item 有相对固定的宽度、并且不取决于内部，这样size规则就反过来了：内部是div元素，所以占满 flex-item 的宽度，最终使得那个 div 元素内容对“三句话”生效</u>。

- 方法1：设置 flex-basis 为 0，然后利用 flex-grow: 1 来撑大到占据容器内所有剩余宽度：

```css
.detail {
  flex-basis: 0;

  /* 单单设置flex-basis是无效的，需要同时设置min-width */
  min-width: 0;
}
```

这里有一个延伸(不是本篇重点，可以略过)，即为什么要同时设置 min-width 才能生效？这里要理解当 min-width 为 auto 时的意义，看规范[Automatic Minimum Size of Flex Items](https://drafts.csswg.org/css-flexbox-1/#min-size-auto)：

> To provide a more reasonable default minimum size for flex items, the used value of a main axis automatic minimum size on a flex item that is not a scroll container is a content-based minimum size...

可以看到对于 flex-item 来说，min-width: auto 对应于 “content-based minimum size”。再看看它的解释：

> In general, the content-based minimum size of a flex item is the smaller of its content size suggestion and its specified size suggestion...

这里涉及到太多约定的术语。根据我的理解，对于 flex-item 来说，min-width: auto 对应于刚好包裹子元素内容的宽度。所以上面的 flex-basis 受到 min-width 的限制，至少也是子元素的宽度，所以上面需要重置 min-width 为 0 。

- 方法2：思路差不多，设置 width 为 0 也可以

```css
.detail {
  width: 0;
}
```

这两种思路都是为了触发第1、2个优先级规则，让 defail 这个 flex-item 主动设置为某个宽度，这样依赖关系就反过来了：detail 中的块级元素满宽即宽度一定，于是上面的“三句话”就生效了。

除此之外，[这篇博客](https://blog.csdn.net/zgh0711/article/details/78270555)提供了另一种方法：

```css
.detail {
  overflow: hidden;
}
```

至于确切的原因，还有待探究……

## 5. 其它问题

类似地，还有一些比如[flex-item 子元素的 word-break 失效](https://github.com/philipwalton/flexbugs/issues/39#issuecomment-111578174)的问题，也是类似的原因、以及解决方案。

## 6. 参考
- [flex布局中，保持内容不超出容器的解决办法](https://blog.csdn.net/zgh0711/article/details/78270555)（这篇博客提出了解决方案、没有分析原因，但启发了我，感谢作者）
- [CSS规范：flex-basis](https://www.w3.org/TR/2018/CR-css-flexbox-1-20181119/#propdef-flex-basis)
- [Firefox doesn't apply flex-basis if children have larger image](https://github.com/philipwalton/flexbugs/issues/41)
- [The Difference Between Width and Flex Basis](https://gedd.ski/post/the-difference-between-width-and-flex-basis/)
- [Firefox: break-word doesn't work inside flex items](https://github.com/philipwalton/flexbugs/issues/39#issuecomment-111578174)
