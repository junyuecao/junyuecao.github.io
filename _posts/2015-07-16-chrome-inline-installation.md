---
layout: post
title: Chrome插件的inline installation
description: "Chrome插件现在支持在自己的网页里插入安装按钮"
category: 
tags: ["JavaScript"]
---



<button onclick="ins()" id="install-button">Add to Chrome</button>

<div id="installed">你已经安装了我的插件了 </div>

<script>
function ins() {
    chrome.webstore.install("https://chrome.google.com/webstore/detail/dcdlpgfpelckoblffdeppjjlcnmppjom",
        function(data) {
            alert(data);
        },
        function(data) {
            alert(data);
        });
}
if (chrome.app.isInstalled) {
  document.getElementById('install-button').style.display = 'none';
  document.getElementById('installed').style.display = 'block';
}
</script>

如果你能看到上面的按钮， 点击就会弹出安装的对话框。

需要四步：

1. Verify the site in webmaster (we need different sites for different app if needed) 

2. Associated every single app to the site in dashboard of webstore 

3. Add chrome web store <link> tag in the <head> of the page that we want to place the install button ( https://developer.chrome.com/webstore/inline_installation?hl=en-US ). 

4. Use chrome.webstore.install(url, successCallback,failureCallback) to install the app without go to webstore