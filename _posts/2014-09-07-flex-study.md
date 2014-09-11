---
layout: post
title: Flex 例子
description: ""
category: 
tags: []
---
{% include JB/setup %}
<script src="http://cdnjs.cloudflare.com/ajax/libs/holder/2.4.0/holder.js"></script>

### 第一个例子：图片垂直水品自适应居中

<iframe width="100%" height="300" src="http://jsfiddle.net/junyuecao/qwwxL1q4/embedded/result,html,css" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

### 第二个例子：flex-direction 伸缩方向

<iframe width="100%" height="400" src="http://jsfiddle.net/junyuecao/xzqhcntv/embedded/result,html,js,css" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

### 第三个例子：justify-content 主轴对齐方式
<iframe width="100%" height="200" src="http://jsfiddle.net/junyuecao/64k94n3L/embedded/result,html,js,css" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

### 第四个例子：align-items 侧轴对齐

<iframe width="100%" height="400" src="http://jsfiddle.net/junyuecao/rbysovjx/embedded/result,html,css,js" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

### 第五个例子

<iframe width="100%" height="500" src="http://jsfiddle.net/junyuecao/mqjhwgzu/embedded/result,html,css,js" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

### 属性列表：

#### 父：

- display:flex;
- align-content
- align-items
- justify-content
- flex-flow:
  - flex-direction: row ,row-reverse , colmn , column-reverse
  - flex-wrap: nowrap , wrap, wrap-reverse

#### 子：
 - order
 - align-self
 - flex:
   - flex-grow : 初始值0，在flex中被忽略时为1
   - flex-shrink : 初始值1，在flex中被忽略时为1
   - flex-basis : 初始值未auto，在flex中被忽略时为0%