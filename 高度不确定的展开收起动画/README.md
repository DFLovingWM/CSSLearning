# 高度不确定(auto)的展开收起动画

CSS目前不能完成从0到auto的动画过渡(transition)，所以对于高度不确定的元素，对height单纯使用CSS是不能做到的。  
目前主要有以下解决方案：

- max-height取固定值
- height取scrollHeight

## 1. max-height取固定值

具体思路很简单，既然height不能用，那我可以用max-height，设置max-height为比最终的height要大的任意值就可以了：

```css
.target {
  /* 这个不用解释了 */
  overflow: hidden;
  /* 一开始是0 */
  max-height: 0;
  /* 在max-height上加transition */
  transition: max-height .5s;
}

/* 展开时可以加个“.active”类 */
.target.active {
  /* 该值需要事先确定，比最终的height要大就行 */
  max-height: 250px;
}
```

但因为max-height是固定的，只能用于展开元素高度相对比较确定的情况，并且如果应用到多个target时，需要保证每个target的展开高度差不多。所以其实比较少用。

## 2. height取scrollHeight

因为scrollheight代表了元素的可滚动高度，所以可以借助JS(DOM API)获取scrollHeight，将它设为height的展开值即可。

```css
.target {
  overflow: hidden;
  height: 0;
  transition: height .5s;
}
```

```js
$btn.addEventListener('click', event => {
  if ($target.classList.contains('opening')) { // 展开时，点击收起
    $target.style.height = 0
  } else { // 收起时，点击展开
    $target.style.height = `${$target.scrollHeight}px` // Clincher！设置height=scrollHeight
  }
  $target.classList.toggle('opening')
})
```

## 参考

- [Collapse组件(一) collapse过渡动画](https://segmentfault.com/a/1190000014075248)
- [知乎：css3怎么实现高度从固定到自动的过渡动画？](https://www.zhihu.com/question/35991373/answer/130256417)
- [Using CSS Transitions on Auto Dimensions](https://css-tricks.com/using-css-transitions-auto-dimensions/)
