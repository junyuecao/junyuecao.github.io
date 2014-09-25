---
layout: post
title: CSS Transform
description: "CSS Transform Demo"
category: 
tags: ["CSS3"]
---

 > 本文翻译自：[http://desandro.github.io/3dtransforms/](http://desandro.github.io/3dtransforms/)

### 知识点

 - `perspective-origin`默认在50%，50%
 - transform不影响文档流
 - 多次transform函数作用时，后一次变换的坐标系为上一次变换完毕后的坐标系

### Perspective 视点
To activate 3D space, an element needs perspective. This can be applied in two ways: using the transform property, with the perspective as a functional notation.
首先为了激活3D空间，需要给元素加上视点（perspective），有两种形式：用transform属性，然后把perspective当做一个函数

```CSS
transform: perspective( 600px );
```

或者直接使用perspective属性：

```CSS
perspective: 600px;
```

<img src="http://desandro.github.io/3dtransforms/img/perspective01.png" alt="Perspective property at work">

两种形式都会激活一个3D空间,但是他们是有差别的.函数符号可以直接给单个元素应用3D变换。比如上面例子左侧的。但是当用在多个元素上时，他们并不会作用在一起，而是分别对每个元素应用，这样看起来就不是我们预期的，比如下图的例子。这种情况下，我们需要使用perspective到他们的父元素上,这样每个子元素就会共用这个视点。他们看起来就像是一个统一的3D系统。

<a href="http://desandro.github.io/3dtransforms/examples/perspective-02-children.html">
<img src="http://desandro.github.io/3dtransforms/img/perspective-children01.png" alt="Perspective differences when used with child elements"></a>

perspective的值决定3D效果的强度。你可以把它想象成与目标物体的距离，它的值越大,你离目标越远,反之亦然。`perspective: 2000px;`会产生一个微弱的3D效果，就想我们在很远的地方拿望远镜来看的，`perspective: 100px;`产生一个强烈的3D效果，就想近距离观察一个很大的物体。

3D空间的变换点(消失点)默认是在物体的中央，你可以通过`perspective-origin`来改变这个位置:

```
perspective-origin: 25% 75%;
```

<a href="http://desandro.github.io/3dtransforms/examples/perspective-03.html">
	<img src="http://desandro.github.io/3dtransforms/img/perspective02.png
" alt="Intense perspective value, with vanishing point modified">
</a>

### 3D变换函数

3D变换和2D用同一个属性：transform，如果你对2D变换比较熟的话，那3D你看起来也不会太陌生:

 - rotateX( angle )
 - rotateY( angle )
 - rotateZ( angle )
 - translateZ( tz )
 - scaleZ( sz )

与translateX在X轴上平移元素类似，translateZ在Z轴上平移元素，Z轴方向是这样的：屏幕背后是负的，屏幕前面是正的，距离屏幕越远，数值越大。

rotate函数可以使元素在相应的轴上旋转。这个和直觉有点不一样.用`rotateX( 45deg )`会使顶边向后移，底边向前移。

<a href="http://desandro.github.io/3dtransforms/examples/transforms-01-functions.html">
<img src="http://desandro.github.io/3dtransforms/img/transforms01.png" alt="CSS 3D transform functions"></a>

还有三个简写的方式：

 - translate3d( tx, ty, tz )
 - scale3d( sx, sy, sz )
 - rotate3d( rx, ry, rz, angle )

 > These foo3d() transform functions also have the benefit of triggering hardware acceleration in Safari. Dean Jackson, CSS 3D transform spec author and main WebKit dude, writes

 > 实质上，任意的有3d操作的的函数都会触发硬件加速，即使它只做了2d变换，或者啥也没干。注意这只是目前的行为，未来可能会改变（所以我们没有记录下来，也没有鼓励这么做）。但在某些情况下这是很有用的，可以显著提升渲染性能。

为了简单期间，这些demo只用了最基本的transform函数，但如果要用到生产环境下，确保用的是foo3d（）函数来获取最佳的性能。

### Matrix 简要解释

假设每一个列是一个点向量，前三行的最后一个数决定平移的量，最后一行的前三个元素不是用来做变换位置的，他们是用来做视点的投影用的。If (as your question suggests) the convention is that points are column vectors, and the last elements of the first 3 rows determine the translation, then the first 3 elements of the bottom row are not a positional offset: the 4th row of a transformation matrix is used for perspective projection, which is how the camera maps 3D points onto the 2D viewport. A sketch of how the 4x4 matrix is used to map points:


	result   =       4x4 matrix      *  point

	[ x' ]       [ Rxx Rxy Rxz Tx ]     [ x ]
	                                                    R** is the rotation/scaling matrix
	[ y' ]       [ Ryx Ryy Ryz Ty ]     [ y ]
	         =                       *             ->   T* is the translation vector
	[ z' ]       [ Rzx Rzy Rzz Tz ]     [ z ]
	                                                    P* fixes the camera projection plane
	[ w' ]       [ Px  Py  Pz  Pw ]     [ w ]

	-> x' = dot_product3([Rxx,Rxy,Rxz], [x,y,z])  + Tx * w
	-> y' = dot_product3([Ryx,Ryy,Ryz], [x,y,z])  + Ty * w
	-> z' = dot_product3([Rzx,Rzy,Rzz], [x,y,z])  + Tz * w
	-> w' = dot_product4([Px,Py,Pz,Pw], [x,y,z,w])




### Demo time

#### Card Flip

<section class="flip-container">
  <div id="card">
    <figure class="front">1</figure>
    <figure class="back">2</figure>
  </div>
</section>
<input class="btn btn-default" id="flipButton" type="button" value="Flip"/>

<style>
	.flip-container { 
	  width: 200px;
	  height: 260px;
	  position: relative;
	  perspective: 800px;
	}
	#card {
	  width: 100%;
	  height: 100%;
	  position: absolute;
	  transform-style: preserve-3d;
	  transition: transform 1s;
	}
	#card figure {
	  display: block;
	  height: 100%;
	  width: 100%;
	  line-height: 260px;
	  color: white;
	  text-align: center;
	  font-weight: bold;
	  font-size: 140px;
	  position: absolute;
	  -webkit-backface-visibility: hidden;
	  -moz-backface-visibility: hidden;
	  -o-backface-visibility: hidden;
	  backface-visibility: hidden;
	}
	#card .front {
	  background: red;
	}
	#card .back {
	  background: blue;
	  transform: rotateY( 180deg );
	}
	#card.flipped {
	  transform: rotateY( 180deg );
	}
</style>

<script>
	$("#flipButton").click(function(){
		$("#card").toggleClass("flipped");
	});
</script>

<div id="transformTabs">
	<ul class="navs">
		<li><span>tab1</span></li>
		<li><span>tab2</span></li>
		<li><span>tab3</span></li>
	</ul>
	<div class="panel-content">
		<div class="panel-div">This is a panel</div>
		<div class="panel-div">This is a panel</div>
		<div class="panel-div">This is a panel</div>
	</div>
</div>
<style>
	#transformTabs{
		perspective:4000px;
		display: flex;
		flex-direction:column;

	}
	#transformTabs ul{
		background-color: #00bcd4;
		margin-bottom: 0;
		padding: 0;
		display: flex;
		justify-content:space-around;
		list-style: none;
		color:#FFF;
		font-size: 14px;
	}
	#transformTabs li{
		flex:1;
		display: flex;
		justify-content:center;
		align-items:center;
		text-align: center;
		vertical-align: middle;
		height: 48px;
	}
	#transformTabs .active{
		/*border-bottom: 2px solid #FFFF8D;*/

	}
	.panel-content{
		height: 300px;
		position: relative;
		transform-style:preserve-3d;
		transform-origin:-150px center;
		transition:transform 400ms cubic-bezier(0.4, 0.0, 1, 1);	

	}

	.panel-content .panel-div{
		height: 300px;
		position: absolute;
		left:0;
		top: 0;
		width: 100%;
		height: 100%;
		transition:transform 400ms ease-in-out;
	}
	.panel-content .panel-div.show{
		opacity: 1;
	}
	.panel-content .panel-div:nth-child(1){
		background-color: rgb(190, 198, 248);
		transform:rotateY(0deg);
		transform-origin:-150px center;
	}
	.panel-content .panel-div:nth-child(2){
		transform-origin:-150px center;
		transform:rotateY(90deg);
		background-color: #291923;
	}
	.panel-content .panel-div:nth-child(3){
		transform-origin:-150px center;
		transform:rotateY(180deg);
		background-color: #921029;
	}
</style>
<script>
	var $aaaa = $("#transformTabs") 
	var $ani = $aaaa.find('.panel-content');
	var $lis = $aaaa.find('li');
	$lis.click(function(){
		var $li = $(this);
		$lis.removeClass('active');
		$li.addClass('active');
		switch($li.index()){
			case 0:
				$ani.css('transform','rotateY(0)') ;
				break;
			case 1:
				$ani.css('transform','rotateY(-90deg)');
				break;
			case 2:
				$ani.css('transform','rotateY(-180deg)');
				break;
		}
		$ani.find('.show').removeClass('show');
		$ani.find('.panel-div').eq($li.index()).addClass('show');
	});
</script>

<div style="perspective:2000px;width:400px;height:400px;">
<div class="father" style="transform: rotateY(280deg) translateZ(-200px);">
<div class="child"></div>
<div class="child"></div>
<div class="child"></div>
<div class="child"></div>
</div></div>
<style>

	.father{		
		position: relative;
		width: 400px;
		height: 400px;
		transform-origin: 50% 50% -200px;
		transform: translateZ(-200px);
		transform-style:preserve-3d;
	}
	.child{
		width: 400px;
		height:400px;
		position: absolute;
		left:0;
		top:0;
	}
	.child:nth-child(1){
		background-color: red;
		transform:translateZ(200px);
		opacity: 0.6;
		
	}
	.child:nth-child(2){
		transform:rotateY(90deg) translateZ(200px);
		background-color: #345251;
	}
	.child:nth-child(3){
		transform:rotateY(180deg) translateZ(200px);
		background-color: rgba(23,51,24,0.6);
	}
	.child:nth-child(4){
		transform:rotateY(-90deg) translateZ(200px);
		background-color: rgba(42,23,51,0.8);
	}
</style>
<!-- <div class="row demo-row">

<div class="col-md-3 demo-wrapper">
	<div id="rotate" class="demo">    
		I am rotating
	</div>
	<input type="range" id="rotatexHandler" min="0" max="360" for="rotateX" value="0">
	<input type="range" id="rotateyHandler" min="0" max="360" for="rotateY" value="0">
	<input type="range" id="rotatezHandler" min="0" max="360" for="rotateZ" value="0">
</div>
</div>

<style>
	.demo{
		    width: 100%;
		    height: 300px;
		    background: #FFA4A4;
		    transform-style:preserve-3d;
		    perspective:500px;
	}
	 .demo-wrapper{
	 	background-color: #ddd;
	 }
	 .demo-wrapper input{
	 	width: 100%;

	 }
</style>
<script>
	$('input').on('change', function(e){
	    var input = $(this);
	    var id = $(this).attr('id');
	    var x = $('#rotatexHandler').val();
	    var y = $('#rotateyHandler').val();
	    var z = $('#rotatezHandler').val();
	    $('#rotate').css('transform', 'rotateX(' + x + 'deg) rotateY(' + y + 'deg) rotateZ(' + z + 'deg)' );
	});

</script> -->