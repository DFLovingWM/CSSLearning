# 实现(可滚动的)横向列表

写一个横向列表，无非要依次考虑以下几点：

1. 创建横向排列的items
2. 隐藏滚动条
3. （可选）适当添加tab-line等元素及其动效

以下讨论基于以下布局代码：
```html
<div class="list-wrapper">
  <ul class="list">
    <li class="item">1</li>
    <li class="item">2</li>
    <li class="item">3</li>
    <li class="item">4</li>
    <li class="item">5</li>
  </ul>
</div>
```
基本的CSS就不写了，可以自行发挥，方便理解即可。

下面来分解书写步骤。



## 1. 创建横向排列的items的N种方式

### 1.1 inline-block

将每个 item 设置`display: inline-block`，这样就能横向排列。但是如果一行放不下，还是会换行了，这里我们对 list 设置`white-space`：
```css
.item {
  display: inline-block; /* 水平排列 */
}

.list {
  overflow-x: auto; /* 超出就可滚动 */
  white-space: nowrap; /* 让item无论有多少个、都不会换行，全在一行展示 */
}
```

但是该方法有一个明显的缺点，就是 item 之间会有一定的空白间隙，那是因为当 item 变为` inline-block `时，HTML文档中的 item 与 item 之间的换行符会变成空格而存在。针对这个问题，参考[如何解决inline-block元素的空白间距](https://www.w3cplus.com/css/fighting-the-space-between-inline-block-elements)也有一些解决方案。这里不再展开，可以使用其中一种：

```css
.list {
  font-size: 0;
}
```

### 1.2 float

1. 思路类似，将每个 item 设置`float: left`，这样 item 就会从左到右排列。不过因为是浮动，要同时设置 BFC 来清除浮动、重置父容器高度。

```css
.item {
  float: left; /* 横向排列 */
}

.list {
  overflow: hidden; /* 这里指定一个overflow，不是为了ul可滚动，而是触发其BFC，使高度不塌陷 */
}
```

2. item 是横向排列了，但是当 item 数量很多时，会因为超出 list 宽度而换行。这里使用上述的 white-space 是没用的（float使item块状化，考试重点！），所以只能够给 list 设置一个固定的、足够大的宽度，来容纳所有的 item。比如 item 有 5 个、每个 200px，那么 list 就设置宽度为 1000px：

```css
.list {
  width: 1000px; /* 定宽 */
}
```

3. 因为 list 宽度超出了屏幕，所以需要在外层即 list-wrapper 层限制一下溢出就好了：
```css
.list-wrapper {
  overflow: auto; /* 这里指定的overflow，才是使列表在水平方向可滚动(div可滚动) */
}
```

其实这个方法还是不错的，至少不会存在像 inline-block 方法中的`空白间隙`这么棘手的问题，唯一问题是list的宽度是不定的。<u>item虽然会左浮动排列，但是因为list是块级元素，最多被撑开到屏幕100%宽度，所以需要我们手动设置list的宽度</u>。上面第2步固定死宽度的做法不可取，因为item的宽度和数量往往是不确定的，于是临时借助JS来实现：

```js
// 计算所有 item 的宽度总和
const sumWidth = Array.from(document.querySelectorAll('.item')).reduce((sum, nextItem) => {
  return sum + nextItem.offsetWidth
}, 0)

// 设置为 list 的宽度
document.querySelector('.list').style.width = `${sumWidth}px`
```

### 1.3 flex

1. 因为flex容器的宽度天然包裹住所有flex-item元素，并且`flex-wrap`默认为`nowrap`，所以设置`flex-direction: row`就能够创建一个横向列表。这时list很长，需要在list-wrapper这里设置横向滚动。
```css
.list {
  display: flex;
  flex-direction: row;
}

.list-wrapper {
  overflow-x: auto; /* 设置x方向滚动 */
}
```

2. 然后发现，虽然 item 排列成一行了，但如果"所有item的总宽度"大于"list"时，每个 item 会相应地减少自己的占据宽度，以让 list 的宽度等于屏幕。这是 flex-shrink 作用的结果。所以可以设置：
```css
.item {
  flex-shrink: 0; /* 禁止缩小item */
}
```
### 1.4 总结

| 方法         | 缺点               | 不定宽者 | 定宽者(限制区域、产生滚动)                 | 优点                                     |
| ------------ | ------------------ | -------- | ------------------------------------------ | ---------------------------------------- |
| inline-block | 需要消除间隙       | -        | list（故不需要list-wrapper层，有层级优势） | 不需要wrapper层                          |
| float        | 需要JS算容器总宽度 | list     | list-wrapper                               | -                                        |
| flex         | -                  | list     | list-wrapper                               | 能根据数量，同时满足按比例分配、横向排列 |



## 2. 隐藏滚动条的多种思路

横向列表在可滚动时，默认会在限制区域的底部产生水平滚动条。在PC、移动两端的表现稍有区别：

- PC端：滚动条出现在区域之下，独自占用17px高（整个区域高度多出17px）
- 移动端：滚动条出现在区域内下方（整个区域高度不变）

### 2.1 -webkit-scrollbar

`-webkit-scrollbar`伪元素可以简单地完成需求：

```css
::-webkit-scrollbar {
  display: none;
}

/* 或者 */
list::-webkit-scrollbar {
  display: none;
}
```

兼容性是它的唯一缺点。所以要寻找新的思路。

### 2.2 外层 + overflow:hidden

首先要明确，产生滚动条的是那个设置了`overflow-x: auto`的元素，对应上面表格的最后一列`定宽者`。

由于两端的滚动条表现形式不同，接下来分开讨论。

#### PC端

PC端的滚动条独自占用17px（有误差）高，列表整体高度便增加了。于是有以下几种思路：

- 外层定高

外层的高度写死、并且等于列表高度，多出的高度就可以被忽略了：

```css
.wrapper {
  height: 100px; /* 高度为列表高度 */
  overflow-y: hidden;
}
```

- 负margin

对列表使用负的`margin-bottom`，使得后面的元素可以覆盖上来：

```css
.list {
  margin-bottom: -17px; /* 高度为滚动条高度，让后面元素刚好覆盖住滚动条 */
}
```

#### 移动端

移动端，因为滚动条悬浮于列表内部下方位置，所以整体高度没有增加。这里的主要思路是：利用`padding-bottom`将滚动条移出可视区域之外。然后剩余的步骤其实与PC端无异。

- 外层定高（+ padding-bottom）

```css
.list {
  padding-bottom: 20px; /* 偏移量足够大，看不到滚动条即可 */
}

.wrapper {
  height: 100px; /* 高度为列表高度 */
  overflow-y: hidden;
}
```

- 负margin（+ padding-bottom）

先使用`padding-bottom`撑开一段距离，然后使用等值的负`margin-bottom`来使后来者覆盖上来，将滚动条遮挡住。

这种方式也有另一种理解（详见"负margin"章节）：正的`padding-bottom`和负的`margin-bottom`互相抵消，ul的`外部尺寸/margin box`没有发生改变，却让`可视区域/border-box`增加了，然后又因为滚动条是在`padding-box`的底部，所以被隐藏掉了。

```css
.list {
  padding-bottom: 888px;
  margin-bottom: -888px;
}
```

#### 总结

PC端和移动端的思路异曲同工。另外，`外层定高法`与`负margin`的优劣：

|          | 优点                     | 缺点                                                 |
| -------- | ------------------------ | ---------------------------------------------------- |
| 外层定高 | 适用于任意场合           | 需要增加wrapper层                                    |
| 负margin | 简单，HTML结构不需要变动 | 后面有兄弟节点时才能用。比如"悬浮header"就不适用了。 |



## 3. 下划线交互方式





## 参考

- [纯css隐藏移动端滚动条解决方案（ios上流畅滑动）](https://www.jianshu.com/p/01a85bf1e113)
- [不可思议的纯CSS导航栏下划线跟随效果](https://juejin.im/post/5ab9e9056fb9a028be360292)
