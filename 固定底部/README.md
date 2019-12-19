# 固定底部

探讨使元素固定底部的N种思路。这里固定底部通常应用于页脚，指的是：

1. 当页面内容超过一页时，元素自然地位于“内容区域”的底部。(所以一开始可能不能完整地看到该元素)
1. 当页面内容不足一页时，无论内容多少，元素都固定于“屏幕/viewport”的底部。

以下讨论共同布局：
```html
<body>
  <header>header</header>
  <section class="main">
    <!-- Contents here. -->
  </section>
  <footer>footer</footer>
</body>
```

下面几种思路其实是类似的，都是给主体部分一个相对固定的高度。

## 1. 主体设置最小高度

假设 footer 的高度是已知的(假设为100px)，那么可以给 header 和 main 整体占据至少 100vh - 100px 的高度，这种算法可以使用 `calc` 来轻易完成：

1. 首先修改下布局，增加一个 wrapper 层，包裹住 header 和 main ：

```html
<body>
  <div class="wrapper">
    <header>header</header>
    <section class="main"></section>
  </div>
  <footer>footer</footer>
</body>
```

2. 然后对 wrapper 使用calc，给一个最小(至少)高度：

```css
.wrapper {
  min-height: calc(100vh - 100px); /* 假设footer高度为100px */
}
```

该方法是最直观、简单。但缺点是只能在 footer 高度已知(不一定是定值)的场景下使用，并且有一定的兼容性问题。

## 2. 负margin

基本思路是：使 footer 外的部分占满屏幕，然后使用“负margin”把 footer 顶上去。CSS书写步骤：

1. 增加 wrapper 层。

2. 给 wrapper 一个100%的高度（注意：height设置百分比默认是无效的。所以为了使百分比高度生效，记得从 html、body 开始一层层设置高度）：

```css
.wrapper {
  min-height: 100vh;
}
```

3. 这时候 footer 被顶到一页之外：

“拉上来”可以使用：

```css
footer {
  margin-top: -100px;
}
/* 或 */
.wrapper {
  margin-bottom: -100px;
}
```

4. 拉上来之后发现footer会遮挡主体内容，设置`padding-bottom`作为“过渡区”：

```css
.wrapper {
  padding-bottom: 100px;
  box-sizing: border-box;
}
```

或者增加dummy结点，用来占据被遮挡的部分的高度：

```html
<body>
  <div class="wrapper">
    <header>header</header>
    <section class="main"></section>

    <!-- 空结点 -->
    <div class="dummy"></div>

  </div>
  <footer>footer</footer>
</body>
```

```css
.dummy {
  height: 100px; /* 与 footer 等高 */
}
```

该方法解决了`calc`的兼容性问题。

## 3. Flexbox

Flex思路是：将整个页面在竖直方向上进行划分，footer为定高部分，其余就是剩余区域（`flex-grow: 1`）。

```css
body {
  /* 至少一屏高度，即第2个条件 */
  min-height: 100vh;
  /* 对内部使用弹性盒子布局 */
  display: flex;
  flex-direction: column;
}

.main {
  /* 占满剩余空间 */
  flex-grow: 1;
}
```

优点：可以在footer高度不定的时候使用。
