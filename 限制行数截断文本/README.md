# 限制行数截断文本

参考：https://mp.weixin.qq.com/s/88qPznXjxPmGpX01fdznig

## 1. 单行

思路：把段落不换行展示，然后设置text-overflow：

```css
.text {
  /* 不换行。这时若内容足够多，会超出一行 */
  white-space: nowrap;
  /* 超出部分隐藏 */
  overflow-x: hidden;
  /* 可选：结尾用省略号表示省略部分 */
  text-overflow: ellipse;
}
```

## 2. 多行

### 2.1 webkit下常用做法：直接用webkit-line-clamp限制行数

使用前提，搭配：
- display: -webkit-box;
- -webkit-box-orient: vertical;

```css
.text {
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 2; /* 假如要保留2行 */
  overflow: hidden;
}
```

webkit-line-clamp不属于规范里指定的属性，而属于webkit的私有属性。所以缺点是只能在Chrome/Safari下生效。

### 2.2 伪元素模拟省略号

思路：将伪元素绝对定位于文本的右下内角处。

步骤：

1. 绝对定位于右下内角处
```css
.wrapper::after {
  content: "...";
  position: absolute;
  right: 0;
  bottom: 0;
}

.text {
  position: relative;
}
```

2. 根据行数计算高度，将多余部分直接隐藏掉：
比如这里要保留2行：
```css
.text {
  line-height: 20px;
  height: 40px; /* 20 * 2 = 40 */
  overflow-y: hidden;
}
```

3. 继续完善：

这时候发现省略号还是与文本重叠，可以给伪元素加个与同色背景，来遮挡住它下面的文字：
```css
.wrapper::after {
  background: white;
}
```

又发现文字和省略号之间有比较大的间距，可以使用word-break：
```css
.text {
  word-break: break-all;
}
```

该方法是一种模拟，缺点有：
- 需要手动计算height。
- 省略号是固定存在的，在文本比较少、不需要出现省略号的时候，它也会出现。所以只能用于文字一定溢出的场景。

### 2.3 借用float

该方法基于2.2所述方法，解决了它的第二个问题。

该方法完全参考于开头链接，这里直接引用其图片：

![float实现多行文本截断](https://mmbiz.qpic.cn/mmbiz_png/aVp1YC8UV0e9LkaNtpOdQWLC24do9CzozCfejP3bxfamvnayC3rcjVSSqvSMTcRQmMqoDoRUgFbCtNETwXAXoA/640?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

关键点在于：
- 当文本不足以溢出时，省略号block会自然地float在文本block的右下方。如果省略号block向右边偏移文本block的宽度后再往上一点，则在很远的地方，就表现为不显示。
- 当文本溢出时，省略号block会自然地float在文本block的左下方再往左偏移一点。如果省略号block向右偏移文本block的宽度后再往上一点，则刚好在文本block的右下内角处，就显示出来了。

首先HTML结构为：
```html
<div class="wrapper">
  ::before
  <div class="text">Long text...</div>
  ::after
</div>
```

其中涉及到角色有：
- text：文本区域
- after伪元素：省略号区域
- before伪元素：与限制高度一致，主要作用是(竖直方向上)将省略号区域顶到文本区域的下方

1. 设置基本结构与样式：
```css
/* 计算高度，并隐藏多余部分 */
.wrapper {
  line-height: 20px;
  height: 40px;
  overflow-x: hidden; /* 不溢出时，可隐藏远处的省略号(看图) */
  overflow-y: hidden; /* 溢出时，隐藏多余文本 */
}

.wrapper::before {
  content: '';
  float: left;
  /* 高度与限制高度一致，将after顶下去 */
  height: 40px;
  /* 随便一个大于0的宽度，比如1px */
  width: 2px;
}

.text {
  /* 右浮动，并重设宽度为100% */
  float: right;
  width: 100%;
}

.wrapper::after {
  content: '...';
  float: right;
  width: 3em;
}
```

2. 因为before的作用主要是将after顶下去，但副作用是占据了一定宽度(比如上面设置了2px)，这会把text顶到下一行，所以我们要把text和before放到同一行。设置margin为负数（使两者的margin~width所有参数加起来等于父容器content width，就会安排在同一行展示）即可：
```css
.text {
  /* 等于上述的随便给的宽度 */
  margin-left: -2px;
}
```

3. 此时，before和text在同一行了。下一步需要把after移动到最左边：
```css
.wrapper::after {
  /* 这一句让after的margin-box总宽度为0，和上面二者又在同一行展示 */
  margin-left: -3em;
}
```

4. 将after下移到text下方。只要在增加一点宽度(比如1px)就行了，这样就因为一行容不下而跑到text的下一行。
```css
.wrapper::after {
  padding-left: 1px;
}
```

另外，其实也可以将3、4步合并起来使用calc：
```css
.wrapper::after {
  margin-left: calc(-3em + 1px);
}
```

但是考虑到兼容性，还是不使用calc吧。

5. 此时，after已经在text的左下方了。剩下的步骤就是调整其位置，使用relative定位：
```css
.wrapper::after {
  position: relative;
  left: 100%; /* 往右移动 */
  top: -20px; /* 往上移动到text的最后一行 */
}
```

6. 这时候就完成了。最后加个背景啥的。
```css
.wrapper::after {
  background: white;
}
```
