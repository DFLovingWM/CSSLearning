# 构造三列布局的N种方式

这里只讨论“中间自适应，左右定宽”这种情况。其它情况是类似的，可以参考[NEC：布局](http://nec.netease.com/library/category/#grid)。

三列布局总结起来只有两个步骤：
1. 将三部分置于同一行
2. 处理彼此重叠的部分

下面根据 HTML 结构来区分几种情景(最好在不改变 HTML 代码的情况下完成三列布局)。


## 1. 跟随两列布局思路，浮动在前(“后序”)

HTML：
```html
<div class="container">
  <div class="left">
    <div class="left-content"></div>
  </div>
  <div class="right">
    <div class="right-content"></div>
  </div>
  <div class="middle">
    <div class="middle-content"></div>
  </div>
</div>
```

浮动(同时也是定宽部分)在前，是为了让CSS代码写起来更简单，基本不用费心就能让三者在同一行展示，只剩下处理重叠的部分。

CSS：
```css
.left {
  float: left;
  width: 120px;
}

.right {
  float: right;
  width: 250px;
}
```

两侧浮动后，三者就已经在同一行了，所以接着就是处理重叠部分，有以下几种思路，因为与两列布局一致，这里就不详述啦：

- middle 使用 BFC ，不与左右 float 块重叠
- middle 使用 margin ，与左右区域分隔开(宽度耦合)


## 2. 主要内容在前(“前序”)

HTML：
```html
<div class="container">
  <div class="middle">
    <div class="middle-content"></div>
  </div>
  <div class="left">
    <div class="left-content"></div>
  </div>
  <div class="right">
    <div class="right-content"></div>
  </div>
</div>
```

主要内容(middle)在前，是为了可以让它先渲染，不重要的侧边内容在之后渲染。

但为此带来的代价是需要使用负 margin 等手法将三者置于同一行，代码更难理解一些。

举例：
- 圣杯
- 双飞翼

具体看同目录的另一篇。


## 3. 自然顺序(“中序”)

HTML：
```html
<div class="container">
  <div class="left">
    <div class="left-content"></div>
  </div>
  <div class="middle">
    <div class="middle-content"></div>
  </div>
  <div class="right">
    <div class="right-content"></div>
  </div>
</div>
```

自然顺序是指元素在 HTML 中的顺序符合直观思维，即“左中右”结构。

实现难度与“中序”差不多，因为都需要熟练运用“负margin”的作用：

```css
.left {
  float: left;
  width: 120px;

  /* 绝对值等于自身宽度 */
  /* 作用是通过使自身外部尺寸为0(或理解为允许被后继结点覆盖上来)，让middle挤上同一行 */
  margin-right: -120px;
}

.middle {
  float: left;
  width: 100%;
}

.right {
  float: left;
  width: 200px;

  /* 绝对值等于自身宽度 */
  /* 作用是通过使自身外部尺寸为0(或理解为主动覆盖前继结点)，让自己挤上middle那一行 */
  margin-left: -200px;
}
```

如果熟练使用“负margin”的话，还会知道从细节上而言，不止这一种写法：

- 对于互相作用的两者，“负margin”可以运用在任何一方(只要方向使用对)。比如 right 中的负 margin-left ，可以换成：
```css
.middle {
  margin-right: -200px;
}
```

- middle 一定是左浮动，而 right 向哪一边浮动都可以。那是因为 middle 让 right 覆盖上来的空间就只有那么大了，所以 right 是左还是右浮动并没有区别。

然后处理覆盖很简单，使用双飞翼的思路即可：

```css
.middle-content {
  margin: 0 205px 0 125px;
}
```


## 4. 其它

还有一些不知道怎么归类的、也挺简单的思路：

- 左右绝对定位 + 中间使用margin避免重叠
- 还有吗。。？


## N. 参考

- [NEC：布局](http://nec.netease.com/library/category/#grid)
