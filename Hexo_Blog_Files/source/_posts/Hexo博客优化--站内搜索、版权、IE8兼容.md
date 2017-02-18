---
title: Hexo博客优化——站内搜索、版权、IE8兼容
date: 2016-05-25 10:03:15
tags: hexo
categories: hexo博客折腾
---

### 1.添加Hexo的swiftype站内搜索
先去[swiftype官网](https://swiftype.com)注册一个账号,然后根据指引建立好自己网站对应的索引。
<!-- more --> 
步骤：
     `Create an engine` 
---> `Create a search engine >(standard web crawler)` 
---> `WEBSITE URL`下填写你的网站地址：如：[http://chaooo.github.io](http://chaooo.github.io)
---> `ENGINE NAME`自己取一个名字：如：chaooo
---> 然后他开始抓取你网站的数据。
---> 完成后，可以点击`Content`去看看抓了些什么数据，然后可以自己删除不想要的数据。（可选）
---> 点击`Install Search`复制里面的代码：

然后在`themes\landscape\layout\_partial\after-footer.ejs`在最后加上刚复制的代码：
``` js
<script type="text/javascript">
  (function(w,d,t,u,n,s,e){w['SwiftypeObject']=n;w[n]=w[n]||function(){
  (w[n].q=w[n].q||[]).push(arguments);};s=d.createElement(t);
  e=d.getElementsByTagName(t)[0];s.async=1;s.src=u;e.parentNode.insertBefore(s,e);
  })(window,document,'script','//s.swiftypecdn.com/install/v2/st.js','_st'); 

  _st('install','Hvy4-e-Ew4x8PR6Let84','2.0.0');
</script>
```
在`themes\landscape\_config.yml`末尾添加如下代码：
``` bash
  swift_search:
    enable: true
```

然后到`themes\landscape\layout\_partial\header.ejs`中找到：`<div id="search-form-wrap">...</div>`刪除里面的內容，插入如下代码：
``` html
<div id="search-form-wrap">
  <form action="" method="get" accept-charset="UTF-8" class="search-form">
    <input autocapitalize="off" autocorrect="off" autocomplete="off" name="q" results="0" id="search" maxlength="20" placeholder="Search" style="border:none;background:none;width:161px;height:30px;line-height:30px;padding:0px 11px 0px 28px;" class="st-default-search-input search-form-input" type="text">
    <button type="submit" class="search-form-submit"></button>
  </form>
</div>
```

然后到`themes\landscape\source\css\_partial\header.styl`找到`#search-form-wrap`对其样式微调，大概在118行，修改后的值：
``` css
#search-form-wrap
  position: absolute
  top: 14px
  width: 200px
  height: 30px
  right: 35px
  opacity: 0
  visibility: hidden
  transition: 0.2s ease-out
  transform: scale(.5) translate(94px, 0)
  &.on
    opacity: 1
    visibility: visible
    transform: scale(1) translate(0, 0)
  @media mq-mobile
    width: 80%
    right: -80%
```
然后到`themes\landscape\source\css\_partial\header.styl`找到`.nav-icon`，大概在81行，在其后面添加(z-index: 1)：
``` css
.nav-icon
  @extend $nav-link
  font-family: font-icon
  text-align: center
  font-size: font-size
  width: font-size
  height: font-size
  padding: 20px 15px
  position: relative
  cursor: pointer
  z-index: 1
```

注意：在使用中我发现swiftype搜索框在IE和火狐浏览器根本不能唤醒搜狗输入法的中文输入，必须要先输入一个英文字母才能输入中文，我在swiftype官网测试的swiftype搜索框也一样。（我分别测试了Chrome--v49，Firefox Developer Edition--v47，IE11/IE10/IE9,结果只有Chrome能唤起搜狗中文。）

### 2. 页尾版权信息修改

在`themes\landscape\layout\_partial\footer.ejs`中，第6行开始，修改其为居中对齐，添加网站地图、订阅、联系博主链接：
``` html
<div id="footer-info" class="inner" style="text-align:center;">
  Copyright &copy; <%= date(new Date(), 'YYYY') %> <%= config.author || config.title %><br>
  <%= __('powered_by') %> <a href="http://hexo.io/" target="_blank">Hexo</a><br>
    <a href="/baidusitemap.xml">网站地图</a>&nbsp; &nbsp;|&nbsp; &nbsp;<a href="/atom.xml">订阅本站</a>&nbsp; &nbsp;|&nbsp; &nbsp;<a href="mailto:zhenggchaoo@gmail.com" target="_blank">联系博主</a>           
</div>
```

### 3. 对于低版本IE布局微调
#### 3.1 加入低版本IE浏览器提醒

Hexo主题大多都只完美支持IE9及以上版本的浏览器，低版本IE会影响网站体验，所以要提示浏览者及时更换现代浏览器，在`themes\landscape\layout\_partial\header.ejs`中找到`<div id="header-outer" class="outer"></div>`以其为父元素添加：
``` html
<!--[if lt IE 9]>
  <style>    
    .hid-ltIE9{position:absolute;bottom:0;z-index:999;width:100%;line-height:46px;color:#7b1a00;font-size:14px;text-align:center;background:#fff3c8;border-radius:4px;border:1px solid #;text-shadow:0 1px #fff;-webkit-box-shadow:0 -1px 4px #ccc inset;box-shadow:0 -1px 4px #ccc inset;border:1px solid #ccc;}
    .hid-ltIE9 a{color:#258fb8;text-decoration:none;}
    .hid-ltIE9 a:hover{text-decoration:underline;}
    .hid-exclamation-triangle,.hid-chrome,.hid-firefox{font:normal normal normal 14px/1 FontAwesome;display:inline-block;width:30px;height:30px;font-size:14px;text-align:center;}
    .hid-exclamation-triangle:before{content:"\f071";}
    .hid-chrome:before{content:"\f268";}
    .hid-firefox:before{content:"\f269";}
  </style>
  <p class="hid-ltIE9">
    <i class="hid-exclamation-triangle" aria-hidden="true"></i>重要提示：您当前使用的浏览器版本过低，可能存在安全风险！想要更好的体验，建议升级浏览器：
    <a href="https://www.google.cn/intl/zh-CN/chrome/browser/desktop/" title="谷歌Chrome浏览器"><i class="hid-chrome" aria-hidden="true"></i>
  Chrome</a>、
    <a href="http://www.firefox.com.cn/download/"title="火狐Firefox浏览器"><i class="hid-firefox" aria-hidden="true"></i>Firefox</a>
  </p>
<![endif]-->
```
这样，现代浏览器都不会解析这段代码，直到IE8及其版本的浏览器才会显示。

#### 3.2 (对于旧IE)header与footer布局微调
发现博客在IE8及其版本的浏览器显示很多样式都乱掉了，特别是头部header，毕竟还有不少人用的低版本浏览器，平常工作中也要求做到兼容到IE8，所以这里只做稍微调。
在`themes\landscape\layout\_partial\header.ejs`中，把`<header id="header"></div>`用下面的代码包起来：
``` html
    <!--[if lt IE 9]><div id="header"><![endif]-->
    <header id="header">
       //....其他代码
    </header>
    <!--[if lt IE 9]></div><![endif]-->
```
在`themes\landscape\layout\_partial\footer.ejs`中，把`<header id="footer"></div>`用下面的代码包起来：
``` html
<!--[if lt IE 9]><div id="footer"><![endif]-->
<footer id="footer">
   //....其他代码
</footer>
<!--[if lt IE 9]></div><![endif]-->
```
虽然这样调整并不高明，但能使其在IE8下显示效果大体上还能接受。