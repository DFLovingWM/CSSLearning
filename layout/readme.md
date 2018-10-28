## CSS：圣杯布局、双飞翼布局

### 1. 圣杯布局

#### 1.1 HTML结构
- middle在最前，然后是left、right
- middle不需要wrapper，即与left、right在同一层
```html
<!-- 前提场景：left为100px，right为200px，middle需要自适应。 -->
<div class="container">
  <div class="middle"></div>
  <div class="left" style="width: 100px;"></div>
  <div class="right" style="width: 200px;"></div>
</div>
```

#### 1.2 CSS步骤
1. 全部左浮动。float具有包裹性，又因middle需要自适应，所以需要重新调整其宽度为100%：
```css
.middle,
.left,
.right {
  float: left;
}

.middle {
  width: 100%;
}
```

2. 此时left、right在第2行，需要将它们迁移到第1行，即与middle同一行。需要使用<u>负margin</u>：
```css
.left {
  /* margin百分数相对于父容器(的content box)的width */
  /* 使left覆盖在middle的左端 */
  margin-left: -100%;
}

.right {
  /* margin百分数相对于父容器(的content box)的width */
  /* 使right覆盖在middle的右端 */
  margin-left: -200px;
}
```

3. (圣杯布局的特点)设置container的padding，给左右两边留出空白：
```css
.container {
  padding-left: 100px; /* 等于left宽度 */
  padding-right: 200px; /* 等于right宽度 */
}
```

4. 此时，因为留出了空白，left和right跟着middle被收缩成一团，为了让left和right回到空白的位置，可以使用relative定位加left、right属性：
```css
.left {
  position: relative;
  left: -100px; /* 大小等于自身宽度 */
}

.right {
  position: relative;
  right: -200px; /* 大小等于自身宽度 */
}
```

5. 因为3个部分都是floating的，故需要解决塌陷问题，可以触发父容器BFC(BFC的特性：包裹住所有浮动子元素)：
```css
.container {
  overflow: auto; /* 触发container的BFC */
}
```

### 2. 双飞翼布局

#### 2.1 HTML结构
- 与圣杯一样，middle也在最前
- middle需要一个wrapper作为父容器
```html
<!-- 前提场景：left为100px，right为200px，middle需要自适应。 -->
<div class="container">
  <div class="wrapper">
    <div class="middle"></div>
  </div>
  <div class="left" style="width: 100px;"></div>
  <div class="right" style="width: 200px;"></div>
</div>
```

#### 2.2 CSS步骤

1. 同上：
```css
.wrapper,
.left,
.right {
  float: left;
}

.wrapper {
  width: 100%;
}
```

2. 同上：
```css
.left {
  margin-left: -100%;
}

.right {
  margin-left: -200px;
}
```

3. (双飞翼布局的特点)设置wrapper的padding(或设置middle的margin，效果一样)，使middle内容不被left、right遮挡住：
```css
.wrapper {
  padding-left: 100px;
  padding-right: 200px;
}

/* 或使用以下 */
.middle {
  margin-left: 100px;
  margin-right: 200px;
}
```

4. 同上面第5步：
```css
.container {
  overflow: auto;
}
```

### 3. 共同点

同时使用float、负margin所产生的作用：
- float的包裹性：设置了float的元素，即使是block元素，默认宽度也变为包裹住其内容(类似于inline-block)。
- 负margin的覆盖性：对float元素设置了负margin，方向相同时为主动向前覆盖，方向相反时为被后面的float的兄弟结点覆盖。


### 参考
[CSS布局 -- 圣杯布局 & 双飞翼布局](http://www.cnblogs.com/imwtr/p/4441741.html)
[负margin用法权威指南](https://www.w3cplus.com/css/the-definitive-guide-to-using-negative-margins.html)
[The Definitive Guide to Using Negative Margins](https://www.smashingmagazine.com/2009/07/the-definitive-guide-to-using-negative-margins/)
