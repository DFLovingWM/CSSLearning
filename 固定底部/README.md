# 固定底部

探讨使元素固定底部的N种思路。这里固定底部通常应用于页脚，指的是：
1. 当页面内容超过一页时，元素自然地位于“内容区域”的底部。(所以一开始可能不能完整地看到该元素)
2. 当页面内容不足一页时，无论内容多少，元素都固定于“屏幕/viewport”的底部。

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

## 1. calc算高度

假设 footer 的高度是已知的(假设为100px)，那么可以给 header 和 main 整体占据至少 100vh - 100px 的高度，这种算法可以使用 calc 来完成：

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

1. 同【calc算高度】中的第1步，增加 wrapper 层。这里略过。

2. 给 wrapper 一个100%的高度（注意：height设置百分比默认是无效的。所以为了使百分比高度生效，记得从 html、body 开始一层层设置高度）：
```css
html,
body {
  height: 100%; /* 个人疑问：为什么设置 min-height 为 100% 无效？ */
}

.wrapper {
  min-height: 100%; /* 注意，这里不能是 height，否则内容溢出时会出现非期望结果 */
}
```

3. 这时候 footer 被顶到一页之外，所以需要将它拉上来，这里有两种思路，本质上一样的：
```css
.wrapper {
  /* 意思是让 footer 覆盖上来 */
  margin-bottom: -100px;
}

/* 或 */
footer {
  /* 意思是主动往 wrapper 方向覆盖 */
  margin-top: -100px;
}
```

上面两个数值与 footer 等高。

4. 此时 footer 位于viewport底部了，但会遮挡住 main 的部分内容。于是可能这里又有两种差不多的思路可以解决(看到这里你应该联想到双飞翼布局)：

- wrapper 设置 padding-bottom(与 footer 等高)，将 main 往上顶：
```css
.wrapper {
  padding-bottom: 100px;
}
```

- main 设置 margin-bottom，将其 content box 往上减小：
```css
.main {
  margin-bottom: 100px;
}
```

实际上，这两种思路都只能解决“超出一屏”的情况；而在“不足一屏”时，因为两个方法都会使 wrapper 的 padding box 的高度增加了，所以一开始又会将 footer 给重新顶下去了。  
没有办法吗？不，还是有的，既然不能用 padding、margin，那我可以增加个空结点，用来占据被遮挡的部分的高度：

5. HTML布局中增加dummy结点，总体布局变为：
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
```scss
.wrapper {
  .dummy {
    height: 100px; /* 与 footer 等高 */
  }
}
```

完毕。

该方法解决了calc的兼容性问题，但本质上一样，只能在 footer 高度已知时使用。

## 3. Flexbox

上述两点用flex布局可以轻松做到：首先给 body 设置至少一页的高度，然后让 main 占满剩余空间，就可以将 footer 顶到最底了：

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

该方法突破了 footer 高度必须已知的限制，非常灵活。唯一问题可能就是浏览器兼容性。
