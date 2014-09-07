---
layout: post
title: Flex 布局
description: ""
category: 
tags: []
---
{% include JB/setup %}
<script src="http://cdnjs.cloudflare.com/ajax/libs/holder/2.4.0/holder.js"></script>

### 第一个例子：图片垂直水品自适应居中
<div class="clearfix">
<div class="flexTest">
    <img data-src="holder.js/300x200/auto/random"></img>
</div>

<div class="flexTest">
    <img data-src="holder.js/200x300/auto/random"></img>
</div>

<div class="flexTest">
    <img data-src="holder.js/200x200/auto/random"></img>
</div>

<div class="flexTest">
    <img data-src="holder.js/100x100/auto/random"></img>
</div>

<style>
.flexTest,.flexTest *{box-sizing:border-box;}
.flexTest{
    display:flex;
    width:200px;
    height:200px;
    padding:5px;
    float: left;
    margin-left:20px;
    border:1px solid #ddd;
    justify-content:center;
    align-items:center;
}
.flexTest img{
    max-width:100%;
    max-height:100%;
    width:auto;
    height:auto;
}
</style>
</div>

{% highlight css %}
.flexTest,.flexTest *{box-sizing:border-box;}
.flexTest{
    display:flex;
    width:200px;
    height:200px;
    padding:5px;
    float: left;
    margin-left:20px;
    border:1px solid #ddd;
    justify-content:center;
    align-items:center;
}
.flexTest img{
    max-width:100%;
    max-height:100%;
    width:auto;
    height:auto;
}
{% endhighlight %}

### 第二个例子：图片垂直水品自适应居中