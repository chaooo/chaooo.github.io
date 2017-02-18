---
title: Hexo博客优化——主题、分享、评论、微博秀
date: 2016-05-23 14:30:13
tags: hexo
categories: hexo博客折腾
---

继上一篇[Hexo3.2+GitHub搭建静态博客](Hexo3-2-github搭建静态博客.html)。

### 1.主题
Hexo提供了很多主题，具体可参见[Themes|Hexo](https://hexo.io/themes/)，这里我选择默认主题[landscape](https://github.com/hexojs/hexo-theme-landscape)(此主题默认已安装)。
<!-- more --> 
#### 1.1安装主题
将Git Shell切换到根目录，执行下列命令，将主题下载到themes/spfk目录下：
``` bash
$ git clone https://github.com/hexojs/hexo-theme-landscape.git themes/landscape
```
获取landscape主题的最新版本：
``` bash
$ cd themes/landscape
$ git pull
```
修改在根目录下_config.yml 配置：
``` bash
theme: landscape
```

### 2.修改添加分享链接
#### 2.1原生分享的修改
在`themes\landscape\source\js\script.js`中，57行 `<div class="article-share-links">`下面的四个链接就是 Facebook 等社交网站的分享链接。将其替换或添加如下代码，即可实现分享到国内社交网站：
``` javascript
'<a href="http://service.weibo.com/share/share.php?&title=好东西就要一起分享&language=zh_cn&url=' + encodedUrl + '" class="article-share-sina" target="_blank" title="微博"></a>',
'<a href="http://share.renren.com/share/buttonshare.do?link=' + encodedUrl + '" class="article-share-renren" target="_blank" title="人人"></a>',
'<a href="http://sns.qzone.qq.com/cgi-bin/qzshare/cgi_qzshare_onekey?url=' + encodedUrl + '" class="article-share-qq" target="_blank" title="QQ空间"></a>',
'<a href="http://qr.liantu.com/api.php?text=' + encodedUrl + '" class="article-share-wechat" target="_blank" title="微信"></a>',
```
同时，还需要替换图标。本主题使用 Font Awesome 来显示图标，但内置的 Font Awesome 版本较旧，无法显示 QQ、微信等图标，所以，需要下载最新版 [Font Awesome](http://fontawesome.io/)，替换掉 `themes\landscape\source\css\fonts`中相关文件，并在`themes\landscape\source\css\_variables.styl `中27行的 `font-icon-version` 修改为最新的 Font Awesome 版本号。

然后，在 `themes\landscape\source\css\_partial\article.styl` 中，找到四段以 `.article-share-*** `开头的代码（273行起），添加如下内容：
``` stylus
.article-share-sina
  @extend $article-share-link
  &:before
    content: "\f18a"
  &:hover
    background: color-sina
    text-shadow: 0 1px darken(color-sina, 20%)

.article-share-qq
  @extend $article-share-link
  &:before
    content: "\f1d6"
  &:hover
    background: color-qq
    text-shadow: 0 1px darken(color-qq, 20%)

.article-share-renren
  @extend $article-share-link
  &:before
    content: "\f18b"
  &:hover
    background: color-renren
    text-shadow: 0 1px darken(color-renren, 20%)

.article-share-wechat
  @extend $article-share-link
  &:before
    content: "\f1d7"
  &:hover
    background: color-wechat
    text-shadow: 0 1px darken(color-wechat, 20%)
```

最后，找到 `themes\landscape\source\css\_variables.styl` 中 Colors 部分（16行），最后四行分别为社交网站图标的背景色，可根据这些网站的主题色修改。
``` stylus
color-sina = #ff8140
color-qq = #ffcc33
color-renren = #227dc5
color-wechat = #44b549
```

#### 2.2加入百度分享
首先在_config.yml中增加bdshare_shortname: 你站点的short_name，这里的short_name也就是你的二级域名。
``` bash
bdshare_shortname: http://chaooo.github.io/
```
在百度分享获取代码后，代码可分为两部分。
在`themes\landscape\layout\_partial\article.ejs`中第26行插入第一段代码并添加判断条件，若当前页为文章展开页则显示百度分享框，若是缩略则采用原生分享链接，避免百度分享框获取的 URL 错误：
``` html
<% if ((page.layout == 'post'|| page.layout == 'page')){ %>
<div class="bdsharebuttonbox"><span style="float:left;line-height:16px;height:16px;margin: 6px 6px 6px 0;">分享到：</span><a title="分享到新浪微博" href="#" class="bds_tsina" data-cmd="tsina"></a><a title="分享到QQ空间" href="#" class="bds_qzone" data-cmd="qzone"></a><a title="分享到微信" href="#" class="bds_weixin" data-cmd="weixin"></a><a title="分享到人人网" href="#" class="bds_renren" data-cmd="renren"></a><a title="分享到Facebook" href="#" class="bds_fbook" data-cmd="fbook"></a><a title="分享到一键分享" href="#" class="bds_mshare" data-cmd="mshare"></a><a href="#" class="bds_more" data-cmd="more"></a></div>
<% } else { %>
<a data-url="<%- post.permalink %>" data-id="<%= post._id %>" class="article-share-link"><%= __('share') %></a>
<% } %>
<!-- Baidu Share Start -->
<script>window._bd_share_config={"common":{"bdSnsKey":{},"bdText":"好东西就要一起分享~","bdMini":"2","bdMiniList":["mshare","qzone","tsina","weixin","sqq","douban","tqq","renren","kaixin001","tqf","linkedin","ty","fbook","twi","copy","print"],"bdPic":"","bdStyle":"1","bdSize":"16"},"share":{},"image":{"viewList":["mshare","weixin","qzone","tsina"],"viewText":"分享到：","viewSize":"16"},"selectShare":{"bdContainerClass":null,"bdSelectMiniList":["mshare","weixin","qzone","tsina"]}};with(document)0[(getElementsByTagName('head')[0]||body).appendChild(createElement('script')).src='http://bdimg.share.baidu.com/static/api/js/share.js?v=89860593.js?cdnversion='+~(-new Date()/36e5)];</script>
<!-- Baidu Share End -->
```

### 3.加入多说评论
首先在_config.yml中增加duoshuo_shortname: 你站点的short_name，这里的short_name也就是你的二级域名。
``` bash
duoshuo_shortname: http://chaooo.github.io/
```
如果使用的是默认的landscape主题只需要修改`themes\landscape\layout\_partial\article.ejs`中的disqus评论：
``` html
<% if (!index && post.comments && config.disqus_shortname){ %>
 <section id="comments">
   <div id="disqus_thread">
     <noscript>Please enable JavaScript to view the <a href="//disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
   </div>
 </section>
 <% } %>
```
改为多说评论：
``` html
<% if (!index && post.comments && config.duoshuo_shortname){ %>
<section id="comments">
<!-- 多说评论框 start -->
  <div class="ds-thread" data-thread-key="<%= post.path %>" data-title="<%= post.title %>" data-url="<%= post.url %>"></div>
<!-- 多说评论框 end -->
<!-- 多说公共JS代码 start (一个网页只需插入一次) -->
<script type="text/javascript">
var duoshuoQuery = {short_name:"chaooo"};
  (function() {
    var ds = document.createElement('script');
    ds.type = 'text/javascript';ds.async = true;
    ds.src = (document.location.protocol == 'https:' ? 'https:' : 'http:') + '//static.duoshuo.com/embed.js';
    ds.charset = 'UTF-8';
    (document.getElementsByTagName('head')[0] 
     || document.getElementsByTagName('body')[0]).appendChild(ds);
  })();
  </script>
<!-- 多说公共JS代码 end -->
</section>
<% } %>
```
如果是其他主题，也只需要修改主题\layout\_partial\comment.ejs
注意`多说的Thread Key一定不要改变，Thread Key相当于是识别码；如果改变了的话，评论清空。`

### 4. 侧栏微博秀

登录[新浪微博开放平台](http://app.weibo.com/tool/weiboshow)来获取微博秀的代码，将其样式调整与主题一致，关掉各种边框和标题栏。然后新建themes/landscape/layout/_widget/weibo.ejs这个文件，将刚刚获取到的代码添加到这个文件中。最后编辑themes/landscape/_config.yml，在widgets:标签后面的适当位置添加- weibo。这样微博秀应该就可以显示在你的博客上了。
``` html
<div class="widget-wrap">
    <h3 class="widget-title">微博</h3>
    <div class="widget" style="padding: 0">
        <iframe width="100%" height="400" class="share_self"  frameborder="0" scrolling="no" src="http://widget.weibo.com/weiboshow/index.php?language=&width=0&height=400&fansRow=2&ptype=1&speed=0&skin=2&isTitle=0&noborder=0&isWeibo=1&isFans=0&uid=1700139362&verifier=85be6061&colors=d6f3f7,dddddd,555555,837f86,cccccc&dpc=1"></iframe>
    </div>
</div>
```
其中，`<iframe...></iframe>`为获取微博秀的代码。