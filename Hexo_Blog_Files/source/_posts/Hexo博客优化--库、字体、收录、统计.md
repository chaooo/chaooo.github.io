---
title: Hexo博客优化——库、字体、收录、统计
date: 2016-05-24 11:22:56
tags: hexo
categories: hexo博客折腾
---

### 1. jQuery 库的优化
landscape默认是使用Google jQuery 库，但在国内速度不是很理想，这里把它换成新浪的，在`themes\landscape\layout\_partial\after-footer.ejs`17行：
<!-- more --> 
``` bash
<script src="//ajax.googleapis.com/ajax/libs/jquery/2.0.3/jquery.min.js"></script>
```
替换为如下代码：
``` bash
<script src="http://lib.sinaapp.com/js/jquery/2.0.3/jquery-2.0.3.min.js"></script>
<script type="text/javascript">
//<![CDATA[
if (typeof jQuery == 'undefined') {
  document.write(unescape("%3Cscript src='/js/jquery-2.0.3.min.js' type='text/javascript'%3E%3C/script%3E"));
}
// ]]>
</script>
```
这里不但将 Google 的 jQuery 替换成了 SAE 的，随后还进行了一个判断，如果获取新浪的 jQuery 失败，则使用本网站自己的 jQuery。为了让这段代码有效，我们要去 jQuery 官方下载合适版本的 jQuery 并将其放到 `themes/landscape/source/js/`目录下，命名为 `jquery-2.0.3.min.js`。
还有一点需要特别注意，那就是 jQuery 这个文件在 hexo 生成博客时会被解析，因此一定要将 jQuery 文件开头处的 //@ sourceMappingURL=jquery-2.0.3.min.map 这一行代码删去，否则会导致博客无法生成。

### 2. 字体优化
#### 2.1跨平台字体优化
为了能在各个平台上都显示令人满意的字体，我们要修改CSS文件中的字体设置，列出多个备选的字体，操作系统会依次尝试，使用系统中已安装的字体。我们要修改的是`themes/landscape/source/css/_variables.styl`这一文件，将其中第22行:
``` css
font-sans = "Helvetica Neue", Helvetica, Arial, sans-serif
```
改成如下内容：
``` bash
font-sans = Tahoma, "Helvetica Neue", Helvetica, "Hiragino Sans GB", "Microsoft YaHei Light", "Microsoft YaHei", "Source Han Sans CN", "WenQuanYi Micro Hei", Arial, sans-serif
```
其中海维提卡（Helvetica）、Arial是英文字体，前者一般存在于苹果电脑和移动设备上，后者一般存在于Windows系统中。冬青黑体（Hiragino Sans GB）、思源黑体（Source Han Sans CN）、文泉驿米黑（WenQuanYi Micro Hei）是中文字体，冬青黑体从OS X 10.6开始集成在苹果系统中，文泉驿米黑在Linux的各大发行版中均较为常见，而思源黑体是近期Google和Adobe合作推出的一款开源字体，很多电脑上也安装了这一字体。这样一来，在绝大部分操作系统中就可以显示美观的字体了。

#### 2.2代码等宽字体优化

Hexo默认的等宽字体是Google的Source Code Pro，这里把它换成360的，在`themes/landscape/layout\_partial\head.ejs` 第31行:
``` html
<link href="//fonts.googleapis.com/css?family=Source+Code+Pro" rel="stylesheet" type="text/css">
```
改成如下内容：
``` html
<link href="http://fonts.useso.com/css?family=Source+Code+Pro" rel="stylesheet" type="text/css">
```

### 3. hexo提交搜索引擎（百度+谷歌）

#### 3.1 确认博客是否被收录
在百度或者谷歌上面输入下面格式来判断，如果能搜索到就说明被收录，否则就没有，用你的域名替代我的http:chaooo.github.io
``` bash
    site:chaooo.github.io
```
#### 3.2 验证网站
两个搜索引擎入口：
[Google搜索引擎提交入口](https://www.google.com/webmasters/tools/home?hl=zh-CN)、[百度搜索引擎入口](http://zhanzhang.baidu.com/linksubmit/url)。
不管谷歌还是百度都要先添加域名，然后验证网站，这里统一都使用文件验证，就是下载对应的html文件，放到域名根目录下，也就收博客根目录下的`source/`下面 。
然后部署到服务器,输入地址：`http://chaooo.github.io/google4cc3eef6ff5975bf.html`和`http://chaooo.github.io/baidu_verify_wjJ25Q3cv2.html`能访问到就可以点验证按钮(按照谷歌或百度的引导步骤就好)。
注意：若出现验证失败，则是因为hexo编译文件时，会给下载的HTML文件中添加其他的内容，导致验证失败。
则需要在Github里手动修改验证HTML文件，或者不编译。
我的做法是，删除根目录`source/`下面刚拷贝的两个文件，和编译后生成的`public/`下的两个同名文件（若细心会注意到`source/`和`public/`下的两个同名文件大小不一样）。
然后重新执行：
``` bash
    hexo generate -d
```
现在重新验证就通过了。

#### 3.3 安装 RSS(订阅) 和 sitemap(网站地图) 插件
``` bash
$ npm install hexo-generator-feed --save
$ npm install hexo-generator-sitemap --save
$ npm install hexo-generator-baidu-sitemap --save
```
修改 hexo\_config.yml 站点配置，添加：
``` bash
#Extensions
Plugins:
  hexo-generator-feed
  hexo-generator-sitemap
  hexo-generator-baidu-sitemap

#Feed Atom
feed:
  type: atom
  path: atom.xml
  limit: 20

#sitemap
sitemap:
  path: sitemap.xml
baidusitemap:
  path: baidusitemap.xml
```
部署后,访问 `chaooo.github.io/sitemap.xml` 和 `chaooo.github.io/baidusitemap.xml `,就能看到有内容且第一行为:`该 XML 文件并未包含任何关联的样式信息。文档树显示如下。`,就说明成功了。
RSS 也差不多，访问 `chaooo.github.io/atom.xml` ，能看到订阅信息。
注意：把`chaooo.github.io`换成你自己的个人域名（订阅是顺带安装的，也可以后在安装）。

#### 3.4 让谷歌收录我们的博客
谷歌操作比较简单，就是向[Google站长工具](https://www.google.com/webmasters/tools/home?hl=zh-CN)提交sitemap。
登录Google账号，添加了站点验证通过后，选择站点，之后在`抓取——站点地图`中就能看到`添加/测试站点地图`,然后输入`sitemap.xml`点击提交。

#### 3.5 让百度收录我们的博客
正常情况，是要等百度爬虫来爬到你的网站，才会被收录。
但是github屏蔽了百度爬虫目前，所以我们要主动出击，我们自己把网站提交给百度。
这就要使用到[百度站长平台](http://zhanzhang.baidu.com)。
1.进入站点管理，找到`网页抓取——链接提交——详情`点进去。
一般主动提交比手动提交效果好，这里介绍主动提交的两种简单的方法

##### 3.5.1 sitemap提交
直接点击`sitemap`填写数据文件地址：`chaooo.github.io/baidusitemap.xml`,输入验证码提交。
##### 3.5.2 自动推送
自动推送很简单，就是在你代码里面嵌入自动推送JS代码，在页面被访问时，页面URL将立即被推送给百度，可将代码添加到`\themes\landscape\layout\_partial\after_footer.ejs`中的最下面就行。
代码如下：
``` javascript
<script>
(function(){
    var bp = document.createElement('script');
    var curProtocol = window.location.protocol.split(':')[0];
    if (curProtocol === 'https') {
        bp.src = 'https://zz.bdstatic.com/linksubmit/push.js';        
    }
    else {
        bp.src = 'http://push.zhanzhang.baidu.com/push.js';
    }
    var s = document.getElementsByTagName("script")[0];
    s.parentNode.insertBefore(bp, s);
})();
</script>
```
[百度链接提交主动推送后不收录的原因](http://tengj.top/2016/03/14/baidunoshouluresson/)

### 4. 开启谷歌统计(google analysis)

先到[google analysis](https://analytics.google.com/)注册服务，注册时，需要正确填写 网站的URL。注册成功后，会得到一个跟踪ID，以及一段跟踪代码。
``` javascript
// 跟踪 ID
// UA-58387143-1
<script>
  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','https://www.google-analytics.com/analytics.js','ga');
  ga('create', 'UA-58387143-1', 'auto');
  ga('send', 'pageview');
</script>
```
到`\themes\landscape\layout\_config.yml`中,将google analysis打开：
``` bash
google_analytics:
  enable: true
  id: UA-58387143-1 #刚注册获取的ID
```
如果设置不起作用，检查在`themes\pacman\layout\_partial\`下有没有google_analytics.ejs ，有的话就在`\themes\landscape\layout\_partial\head.ejs`的`</head>`之前，添加下面代码试试：
``` php
<%- partial('google_analytics') %>
```
若`themes\pacman\layout\_partial\`不存在google_analytics.ejs 文件，就手动创建：
``` javascript
<% if (theme.google_analytics){ %>
<!-- Google Analytics -->
<script type="text/javascript">
(function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
(i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
})(window,document,'script','//www.google-analytics.com/analytics.js','ga');
ga('create', '<%= theme.google_analytics %>', 'auto');
ga('send', 'pageview');
</script>
<!-- End Google Analytics -->
<% } %>
```
最笨的方法就是删除`google_analytics.ejs`和刚在`_config.yml`配置google analysis的几行代码，直接从注册来的代码拷贝到`\themes\landscape\layout\_partial\head.ejs`的`</head>`之前。

















### 5. 文章永久链接

默认文章链结是以: `http://chaooo.github.io/2016/05/24/文章标题/` 的格式，末尾没有.html结尾，有点动态页面的感觉，好像对搜索引擎不太友好，于是可以修改根目录下的 `_config.yml` 文件里:
``` bash
permalink: :year/:month/:day/:title/
```
改为：
``` bash
permalink: :year/:month/:day/:title.html
```
最后浏览器访问就是`http://chaooo.github.io/2016/05/24/文章标题.html` 的格式了。

