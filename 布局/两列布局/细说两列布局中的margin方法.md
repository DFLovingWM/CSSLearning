# 细说两列布局中的margin方法

## 1. 基本原理

原理是先让left块占满一行，然后调整margin值，使得left、right两者的margin box的总宽度为一行的宽度，就能够在同一行之中展示了。

## 2. 基本代码

### 2.1 HTML结构

```html
<div class="container">
  <div class="left wrapper">
    <div class="left-content">左边：自适应</div>
  </div>
  <div class="right">右边：固定宽度</div>
</div>
```

至于为什么要left作为一个wrapper包住内容，原因稍后再说。

### 2.2 CSS书写步骤

1. 先让left占满一行，同时让right靠右浮动：

```css
.left {
  float: left;
  width: 100%;
}

.right {
  float: right;
  right: 250px; /* 假设定宽是250px */
}
```

2. 目前两者分行展示。可以调整right的margin，把right推上去：

```css
.right {
  /* 值等于自身宽度。这样使得margin box的宽度计算值为0，就可以与left待在同一行 */
  margin-left: -250px;
}
```

当然，也可以调整left的margin，把后面的right拉过来，效果上是等价的：

```css
.left {
  margin-right: -250px;
}
```

这涉及到负margin对float元素的行为影响，这里就不详述了。

3. 目前两者确实在同一行展示了，但left的右边被right覆盖了。这时候，wrapper就起作用了，现在要将内容往左边推(推出被被覆盖/重叠区域)。  
下面是两种等价的思路：

- 思路1：调整内容区域的宽度，即使用margin/border/padding，使content box的宽度缩小到最左边即可：

```css
.left-content {
  margin-right: 250px;
}

/* 或 */
.left-content {
  border-right: 250px solid transparent;
}

/* 或 */
.left-content {
  padding-right: 250px;
}
```

- 思路2：调整wrapper的宽度，其实跟上面差不多，只是可能需要改变box-sizing：

```css
.left {
  box-sizing: border-box;
  border-right: 250px solid transparent;
}

/* 或 */
.left {
  box-sizing: border-box; /* 理论上padding-box就可以了，但是很多浏览器不支持 */
  padding-right: 250px;
}
```

4. 目前已经基本实现了两列布局了。最后记得要解决高度塌陷的问题，可以通过触发父容器的BFC：

```css
.container {
  overflow: auto;
}
```

## 3. 思考

该方法虽然更难理解，但是如果掌握了，则将对盒子模型会有更多的理解。比起margin/BFC实现的两列布局，它还有一个优点，就是只使用同一个HTML布局便可以解决以下两种场景：

- 左边自适应，右边定宽
- 左边定宽，右边自适应

不像BFC/margin法，会受限于left、right的HTML顺序。  
上述CSS步骤写的是第一种情况，第二种情况可以用几乎相同的代码写出来：

```html
<div class="left" style="width: 250px;">左边：定宽</div>
<div class="right">
  <div class="right-content">右边：自适应</div>
</div>
```

```css
.left {
  float: left;
  margin-right: -250px;
  position: relative; /* 这里有个知识点，下面说 */
}

.right {
  float: right;
  width: 100%;
}

.right-content {
  margin-left: 250px;
}
```

CSS代码中，left的position设置了relative是因为：left设置了负margin使得right覆盖上去了，这时候如果left设置了relative定位，就反过来覆盖right了。  
关于这一点具体可以参考：[理解并运用 CSS 的负 margin 值](https://segmentfault.com/a/1190000007184954#articleHeader1)。
