---
title: 那些布局
date: 2018-05-27 23:01:02
tags:
- 布局
- css
---


工作中经常遇到要写布局，所以在这里总结一些常用布局。并且会不断补充。

其中，两列布局和三列布局中，思路主要是利用float， 然后，自适应的部分都是 在外层包裹一层div，并且设置width:100%; 然后定宽的元素反方向设置margin(即margin设为和宽度数值一样的负值)。


<!-- more -->


## 两列布局
### 两列右侧自适应布局
html:
```
<div class="container clearfix">
  <div class="left">
    左侧定宽
  </div>
  <div class="right-wrap">
    <div class="right">
      右侧自适应
    </div>
  </div>
</div>
```
css:

```
.left {
  width: 300px;
  background: lightgreen;
  float: left;
  margin-right:-300px;
}
.right-wrap {
  float: right;
  width: 100%;
}
.right {
  background: lightcoral;
  margin-left: 310px;
}
/* 清除浮动 */

.clearfix:after {
  content: "";
  clear: both;
  display: block;
}
```

### 两列左侧自适应布局
html:
```
<div class="container clerafix">
  <div class="left-wrap">
    <div class="left">
      左侧自适应
    </div>
  </div>
  <div class="right">
    右侧定宽
  </div>
</div>
```
css:
```
.left-wrap {
  float: left;
  width: 100%;
}
.left {
  background: lightcoral;
  margin-right: 310px;
}
.right {
  width: 300px;
  float: right;
  margin-left: -300px;
  background: lawngreen;
}
 /* 清除浮动 */
.clearfix:after {
  content: "";
  clear: both;
  display: block;
}
```

## 三列布局
### 三列中间自适应布局

html:
```
  <div class="container clearfix">
    <div class="left">左</div>
    <div class="center-wrap">
      <div class="center">中</div>
    </div>
    <div class="right">右</div>
  </div>
```

css:
```
.left,
.right {
  background: #edaeda;
  width: 200px;
  height: 300px;
}

.left {
  float: left;
  margin-right: -200px;
}

.right {
  float: right;
  margin-left: -200px;
}

.center-wrap {
  float: left;
  width: 100%;
}

.center {
  background: #3eabcd;
  margin-left: 210px;
  margin-right: 210px;
}

/* 清除浮动 */

.clearfix:after {
  content: "";
  clear: both;
  display: block;
}
```


---

未完待续，，，
