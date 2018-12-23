# 实现(可滚动的)横向列表

写一个横向列表，无非要依次考虑以下几点：

1. 创建横向排列的items
2. 隐藏滚动条
3. 适当添加tab-line等元素及其动效

假设共同布局为：
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
基本的样式就不写了，可以自行发挥，方便理解即可。

下面来分解书写步骤：

## 1. 创建横向排列的items的N种方式

### 1.1 inline-block

将每个 item 设置 display: inline-block，这样就能横向排列。但是如果一行放不下，还是会换行了，这里我们对 list 设置 white-space：
```css
.item {
  display: inline-block; /* 水平排列 */
}

.list {
  overflow-x: auto; /* 超出就可滚动 */
  white-space: nowrap; /* 让item无论有多少个、都不会换行，全在一行展示 */
}
```

但是该方法有一个明显的缺点，就是 item 之间会有一定的空白间隙，那是因为当 item 变为 inline-block 时，HTML文档中的 item 与 item 之间的换行符会变成空格而存在。针对这个问题，参考[如何解决inline-block元素的空白间距](https://www.w3cplus.com/css/fighting-the-space-between-inline-block-elements)也有一些解决方案。这里不再展开，可以使用其中一种：

```css
.list {
  font-size: 0;
}
```

### 1.2 float

1. 思路类似，不过是将每个 item 设置 float: left，这样 item 就会从左到右排列。不过因为是浮动，要同时设置 BFC 来清除浮动。

```css
.item {
  float: left; /* 横向排列 */
}

.list {
  overflow: hidden; /* 这里指定一个overflow，不是为了ul可滚动，而是触发其BFC，使高度不塌陷 */
}
```

2. item 是横向排列了，但是当 item 数量很多时，会因为超出 list 宽度而换行。这里使用上述的 white-space 是没用的(知识点：float的块状化)，所以只能够给 list 设置一个固定的、足够大的宽度，来容纳所有的 item。比如 item 有 5 个、每个 200px，那么 list 就设置宽度为 1000px：

```css
.list {
  width: 1000px; /* 定宽 */
}
```

3. 最后因为 list 宽度超出了屏幕，所以需要在外层即 wrapper 层限制一下溢出就好了：
```css
.list-wrapper {
  overflow: auto; /* 这里指定的overflow，才是使列表在水平方向可滚动(div可滚动) */
}
```

其实这个方法还是不错的，至少没有像 inline-block 中的“空白间隙”这么棘手的问题，唯一问题只有对 list 的宽度设置。因为 item 的宽度和数量往往是不确定的，所以只能临时依靠 JS 的力量：

```js
// 计算所有 item 的宽度总和
const sumWidth = Array.from(document.querySelectorAll('.item')).reduce((sum, nextItem) => {
  return sum + nextItem.offsetWidth
}, 0)
// 设置为 list 的宽度
document.querySelector('.list').style.width = `${sumWidth}px`
```

### 1.3 (又见面了，感觉是万能的)Flex布局

1. 依靠 flex-wrap(或flex-flow) ，可以轻松实现这种布局方式：
```css
.list {
  display: flex;
  flex-flow: row nowrap; /* 横向排列、且不换行 */
}

.list-wrapper {
  overflow-x: auto; /* 设置滚动 */
}
```

2. 此刻发现，虽然 item 排列成一行了，但如果宽度总和大于屏幕宽度时，每个 item 会相应地减少自己的占据宽度，以让 list 的宽度等于屏幕。这是 flex-shrink 作用的结果。所以可以设置：
```css
.item {
  /* 意味不能缩小 item 的宽度 */
  flex-shrink: 0;
}
```
搞定。


## 2. 隐藏滚动条的N种方式

## 3. 下划线交互方式


## 参考
- [纯css隐藏移动端滚动条解决方案（ios上流畅滑动）](https://www.jianshu.com/p/01a85bf1e113)
- [不可思议的纯CSS导航栏下划线跟随效果](https://juejin.im/post/5ab9e9056fb9a028be360292)
