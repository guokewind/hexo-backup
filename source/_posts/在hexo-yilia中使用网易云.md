---
title: " hexo中插入音乐"
date: 2017-02-14 01:24:49
categories:
tags: 
        hexo
---

### 前言
从用QQ空间开始就喜欢插入背景音乐，如今自己管理自己的博客，当然也要插入音乐，不用开绿钻哈哈哈哈。

### 关于主题
我用的是[yilia](https://github.com/litten/hexo-theme-yilia)。

### 添加音乐
添加的网易云音乐，很方便。
打开网易云音乐某首歌详情页，点击生成外链播放器，选择iframe插件。选择好尺寸。复制html代码。如：
```
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=4940455&auto=1&height=32"></iframe>
```

<!--more-->

这里手动修改width为"100%",再添上class。然后把下面这段代码复制到
`themes\yilia\layout\_partial\left-col.ejs`里，代码：

```CSS
<nav class="header-music">
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=4940455&auto=1&height=32"></iframe>
</nav>
```

如图:

![开启网易云](http://github-10029416.cos.myqcloud.com/%E5%BC%80%E5%90%AF%E7%BD%91%E6%98%93%E4%BA%91.png
)。
这样就可以生成播放器了，但是还需要调一下css样式。
在themes\yilia\source\main.2d7529.css(如果名字不同，也应为main.xx.css)末尾添加
```
.header-music {margin-top: 80px; }
```
接着hexo clean,hexo g,hexo s 搞定。
效果如下：

![网易云效果](http://gkwind-10029416.cos.myqcloud.com/hexo%E5%BC%80%E5%90%AF%E7%BD%91%E6%98%93%E4%BA%91%E4%BE%8B%E5%AD%90.png
)

之后会试着利用cplayer把列表插入到主题中。
