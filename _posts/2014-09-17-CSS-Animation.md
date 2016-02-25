---
layout: post
title: CSS Animation
description: "CSS 动画研究"
category: 
tags: ["CSS3"]
---


<!-- <div align="center">
<div class="first_circle"></div>
<div class="first_div">
  <div class="second_div">
    <div class="img_div">
      <img src="http://www.wifeo.com/image_design_v3/images_codes/i/img_welcome1410166189.png"></div>
    </div>
  </div>
<div class="txt_welcome">WELCOME</div>
<div class="txt_user">PRESS F5 TO REPLAY</div>
</div> -->

<style>
	.first_div
{
  background-color:#f28e84;
  width:220px; height:220px;
  padding: 10px;
  border-radius:50%;
  -webkit-animation: anim 0.7s 1 ease;
  -moz-animation: anim 0.7s 1 ease;
  -ms-animation: anim 0.7s 1 ease;
  animation: anim 0.7s 1 ease;
}
.second_div
{
  width:200px; height:200px;
  border:1px solid #ffffff;
  border-radius:50%;
  -webkit-animation:anim 1s 1 ease;
  -moz-animation:anim 1s 1 ease;
  -ms-animation:anim 1s 1 ease;
  animation:anim 1s 1 ease;
}
.img_div
{
  width:200px; height:200px;
  -webkit-animation:animuser 1s 1 ease;
  -moz-animation:animuser 1s 1 ease;
  -ms-animation:animuser 1s 1 ease;
  animation:animuser 1s 1 ease;
}
.txt_welcome
{
  font-size: 46px;
  font-weight: 300;
  color: #e84c3d;
  padding-top: 25px;
  -webkit-animation: animwelcome 1.7s 1 ease-in;
  -moz-animation: animwelcome 1.7s 1 ease-in;
  -ms-animation: animwelcome 1.7s 1 ease-in;
  animation: animwelcome 1.7s 1 ease-in;
}
.txt_user
{
  font-size: 22px;
  font-weight: 100;
  color: #e84c3d;
  -webkit-animation: animuser 1.9s 1 ease-in;
  -moz-animation: animuser 1.9s 1 ease-in;
  -ms-animation: animuser 1.9s 1 ease-in;
  animation: animuser 1.9s 1 ease-in;
}
.first_circle
{
  width: 244px; height: 244px; border-radius:50%;
  padding: 10px; margin-top: -12px;
  position: absolute; left: 50%; margin-left: -122px;
  border-top:2px solid #ffffff;
  border-right:2px solid #ffffff;
  border-bottom:2px solid #ffffff;
  border-left:2px solid #e84c3d;
  -webkit-animation:anim_wifeo 1.4s infinite linear;
  -moz-animation:anim_wifeo 1.4s infinite linear;
  -ms-animation:anim_wifeo 1.4s infinite linear;
  animation:anim_wifeo 1.4s infinite linear;
} 
@-webkit-keyframes anim
{
  0%{-webkit-transform:scale(0);}
  50%{-webkit-transform:scale(1.7);}
  100%{-webkit-transform:scale(1);}
}
@-moz-keyframes anim
{
  0%{-moz-transform:scale(0);}
  50%{-moz-transform:scale(1.7);}
  100%{-moz-transform:scale(1);}
}
@-ms-keyframes anim
{
  0%{-ms-transform:scale(0);}
  50%{-ms-transform:scale(1.7);}
  100%{-ms-transform:scale(1);}
}
@keyframes anim
{
  0%{transform:scale(0);}
  50%{transform:scale(1.7);}
  100%{transform:scale(1);}
}
@-webkit-keyframes animwelcome
{
  0%{-webkit-transform:scale(0);}
  50%{-webkit-transform:scale(0);}
  75%{-webkit-transform:scale(1.4);}
  100%{-webkit-transform:scale(1);}
}
@-moz-keyframes animwelcome
{
  0%{-moz-transform:scale(0);}
  50%{-moz-transform:scale(0);}
  75%{-moz-transform:scale(1.4);}
  100%{-moz-transform:scale(1);}
}
@-ms-keyframes animwelcome
{
  0%{-ms-transform:scale(0);}
  50%{-ms-transform:scale(0);}
  75%{-ms-transform:scale(1.4);}
  100%{-ms-transform:scale(1);}
}
@keyframes animwelcome
{
  0%{transform:scale(0);}
  50%{transform:scale(0);}
  75%{transform:scale(1.4);}
  100%{transform:scale(1);}
}
@-webkit-keyframes animuser
{
  0%{-webkit-transform:scale(0);}
  50%{-webkit-transform:scale(0);}
  75%{-webkit-transform:scale(1.4);}
  100%{-webkit-transform:scale(1);}
}
@-moz-keyframes animuser
{
  0%{-moz-transform:scale(0);}
  50%{-moz-transform:scale(0);}
  75%{-moz-transform:scale(1.4);}
  100%{-moz-transform:scale(1);}
}
@-ms-keyframes animuser
{
  0%{-ms-transform:scale(0);}
  50%{-ms-transform:scale(0);}
  75%{-ms-transform:scale(1.4);}
  100%{-ms-transform:scale(1);}
}
@keyframes animuser
{
  0%{transform:scale(0);}
  50%{transform:scale(0);}
  75%{transform:scale(1.4);}
  100%{transform:scale(1);}
}
@-webkit-keyframes anim_wifeo
{
  0%{-webkit-transform:rotate(0deg);} 
  50%{-webkit-transform:rotate(360deg);} 
  100%{-webkit-transform:rotate(720deg);}
}
@-moz-keyframes anim_wifeo
{
  0%{-moz-transform:rotate(0deg);} 
  50%{-moz-transform:rotate(360deg);} 
  100%{-moz-transform:rotate(720deg);}
}
@-ms-keyframes anim_wifeo
{
  0%{-ms-transform:rotate(0deg);} 
  50%{-ms-transform:rotate(360deg);} 
  100%{-ms-transform:rotate(720deg);}
}
@keyframes anim_wifeo
{
  0%{transform:rotate(0deg);} 
  50%{transform:rotate(360deg);} 
  100%{transform:rotate(720deg);}
}
</style>
<!-- 
<div class="contener_general">
      <div class="contener_mixte"><div class="ballcolor ball_1">&nbsp;</div></div>
      <div class="contener_mixte"><div class="ballcolor ball_2">&nbsp;</div></div>
      <div class="contener_mixte"><div class="ballcolor ball_3">&nbsp;</div></div>
      <div class="contener_mixte"><div class="ballcolor ball_4">&nbsp;</div></div>
  </div> -->

  <style>
  .contener_general
{
  -webkit-animation:animball_two 1s infinite;
  -moz-animation:animball_two 1s infinite;
  -ms-animation:animball_two 1s infinite;
  animation:animball_two 1s infinite;
  width:44px; height:44px;
}
.contener_mixte
{
  width:44px; height:44px; position:absolute;
}
.ballcolor
{
  width: 20px;
  height: 20px;
  border-radius: 50%;
}
.ball_1, .ball_2, .ball_3, .ball_4
{
  position: absolute;
  -webkit-animation:animball_one 1s infinite ease;
  -moz-animation:animball_one 1s infinite ease;
  -ms-animation:animball_one 1s infinite ease;
  animation:animball_one 1s infinite ease;
}
.ball_1
{
  background-color:#cb2025;
  top:0; left:0;
}
.ball_2
{
  background-color:#f8b334;
  top:0; left:24px;
}
.ball_3
{
  background-color:#00a096;
  top:24px; left:0;
}
.ball_4
{
  background-color:#97bf0d;
  top:24px; left:24px;
}
@-webkit-keyframes animball_one
{
  0%{ position: absolute;}
  50%{top:12px; left:12px; position: absolute;opacity:0.5;}
  100%{ position: absolute;}
}
@-moz-keyframes animball_one
{
  0%{ position: absolute;}
  50%{top:12px; left:12px; position: absolute;opacity:0.5;}
  100%{ position: absolute;}
}
@-ms-keyframes animball_one
{
  0%{ position: absolute;}
  50%{top:12px; left:12px; position: absolute;opacity:0.5;}
  100%{ position: absolute;}
}
@keyframes animball_one
{
  0%{ position: absolute;}
  50%{top:12px; left:12px; position: absolute;opacity:0.5;}
  100%{ position: absolute;}
}
@-webkit-keyframes animball_two
{
  0%{-webkit-transform:rotate(0deg) scale(1);}
  50%{-webkit-transform:rotate(360deg) scale(1.3);}
  100%{-webkit-transform:rotate(720deg) scale(1);}
}
@-moz-keyframes animball_two
{
  0%{-moz-transform:rotate(0deg) scale(1);}
  50%{-moz-transform:rotate(360deg) scale(1.3);}
  100%{-moz-transform:rotate(720deg) scale(1);}
}
@-ms-keyframes animball_two
{
  0%{-ms-transform:rotate(0deg) scale(1);}
  50%{-ms-transform:rotate(360deg) scale(1.3);}
  100%{-ms-transform:rotate(720deg) scale(1);}
}
@keyframes animball_two
{
  0%{transform:rotate(0deg) scale(1);}
  50%{transform:rotate(360deg) scale(1.3);}
  100%{transform:rotate(720deg) scale(1);}
}
</style>

<div class="dot-loading">
	<span class="dot"></span>	
	<span class="dot"></span>	
	<span class="dot"></span>	
	<span class="dot"></span>	
	<span class="dot"></span>	
</div>

<style>
	.dot-loading{
		text-align: center;
		height: 50px;
	}
	.dot-loading .dot{
		display: inline-block;
		width:10px;
		height: 10px;
		background-color: #e84c3d;
		border-radius: 50%;
		position: relative;
		top:30px;
	}
	.dot-loading .dot:first-child{
		-webkit-animation: dot_loading 2.5s 0.2s infinite ease-out;
	}
	.dot-loading .dot:nth-child(2){
		-webkit-animation: dot_loading 2.5s 0.4s infinite ease-out;
	}
	.dot-loading .dot:nth-child(3){
		-webkit-animation: dot_loading 2.5s 0.6s infinite ease-out;
	}
	.dot-loading .dot:nth-child(4){
		-webkit-animation: dot_loading 2.5s 0.8s infinite ease-out;
	}
	.dot-loading .dot:nth-child(5){
		-webkit-animation: dot_loading 2.5s 1s infinite ease-out;
	}
	@-webkit-keyframes dot_loading
	{
	  0%{ top:30px;}
	  3%{ top:28px;}
	  5%{ top:27px;}
	  7%{ top:24px;}
	  10%{ top:35px;}
	  12%{ top:33px;}
	  15%{ top:30px;}
	  
	  100%{ top: 30px;}
	}

.center{
    border: 1px solid #332;
    width: 60px;
    height: 60px;
    border-radius: 60px;
    background: rgba(0,0,0,0.5);
    -webkit-animation:ani 1s ease-in 1s 1 alternate forwards paused;
}
.center:hover{
	-webkit-animation-play-state:running;
}
@-webkit-keyframes ani{
    0%{-webkit-transform:translateX(-50px);}
    50%{-webkit-transform:translateX(40px);}
    70%{-webkit-transform:translateX(100px);}

}
</style>
<div class="center"></div>

#### 问题

 - CSS clip是什么？
 - animation-fill-mode

