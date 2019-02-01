---
date: "2016-01-31"
layout: post
tags: ['Css,', 'tech']
title: "\u56FA\u5B9A\u5C3A\u5BF8\u7684\u54CD\u5E94\u5F0F\u56FE\u7247\u5360\u4F4D\u7B26"
---


对于图片列表，通常会使用懒加载的方式，好处就不多说了。通常实现是图片的 src 放一张占位符, data- 属性是原图地址，监听 滚动及触摸事件，如果在 viewport 内，加载原图。

<!--more-->

对于相同尺寸图片列表来说，如果图片存在 CDN，只是想单纯的给图片加个占位符，通过几行 Css 便可达到效果。

实现方式为图片外层包裹块级元素相对定位，通过 padding bottom 控制比例，图片绝对定位。

```css
...
figure {
    padding-bottom:100%;
    position: relative;
    background-color: #eee;
    img {
        position: absolute;
        max-width:100%;
    }
}
...
```

<p data-height="268" data-theme-id="0" data-slug-hash="yeEjpV" data-default-tab="result" data-user="lacuna" class='codepen'>See the Pen <a href='//codepen.io/lacuna/pen/yeEjpV/'>yeEjpV</a> by lacuna (<a href='//codepen.io/lacuna'>@lacuna</a>) on <a href='//codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

这种思路实际上在响应式视频上也能用到。前提是宽高比固定，宽度不定。 


```css
.img-holder\:1x1  { padding-bottom: 100%; }
.img-holder\:1x2  { padding-bottom: 200%; }
.img-holder\:2x1  { padding-bottom: 50%; }
.img-holder\:2x3  { padding-bottom: 150%; }
.img-holder\:3x2  { padding-bottom: 66.66%; }
.img-holder\:3x4  { padding-bottom: 133.333%; }
.img-holder\:4x3  { padding-bottom: 75%; }
.img-holder\:9x16 { padding-bottom: 177.777%; }
.img-holder\:16x9 { padding-bottom: 56.25%; }
.img-holder\:16x10 { padding-bottom: 62.5%; }
```

针对不同尺寸，计算不同 padding-bottom, 很简单的方式，便可实现响应式图片占位符。



