---
title: github博客迁移
date: 2016-08-17 11:14:25
tags: [github,hexo]
categories: hexo博客折腾
---
由于github博客在国内访问非常慢而且经常不稳定，百度爬虫也无法抓取github博客内容，所以决定将博客迁移至码云。
### 1.迁移代码
把代码从[github](https://github.com/)迁移至[码云（oschina）](http://git.oschina.net/)。
首先，你要先在码云注册一个账号，和GitHub操作基本一样，这里不再赘述。
码云的Pages服务基本和GitHub的一样，不过码云的Pages服务更灵活一些。
<!-- more -->
在GitHub上，可以新建一个以`用户名`命名的仓库，将网站代码放在master分支下，即可自动部署到：`http://用户名.github.io/`，若其他命名的仓库则新建一个`gh-pages`的分支，网站代码放在`gh-pages`下，，即可自动部署到：`http://用户名.github.io/仓库名/`。
对于码云，基本和GitHub一样，不过还需要手动开启Pages服务，而且其他仓库虽然默认在`osc-pages`下，但可自定到自己喜欢的分支上。
代码迁移步骤如下：
##### 1.1 新建码云项目
以我自己的博客为例，项目地址：[https://github.com/chaooo/chaooo.github.io.git](https://github.com/chaooo/chaooo.github.io.git)。
它在Github上的Pages地址是：[http://chaooo.github.io](http://chaooo.github.io)
如果想把它转移到码云Pages，只需要登录你的码云账户，点击右上角的`+`号，选择新建项目:
![博客迁移至码云1](http://obzf7z93c.bkt.clouddn.com/blog/oschina.jpg)
##### 1.2 开启pages服务
然后点击创建，项目会在后台自动导入，导入成功后，点击菜单栏的`Pages`,码云默认的Pages服务分支是osc-pages，但是你也已选择自己静态页面所在的分支，这里我的博客项目的静态页面分支是master，选择master并点击启动服务。
![博客迁移至码云2](http://obzf7z93c.bkt.clouddn.com/blog/oschina1.jpg)
至此，博客已经部署成功，访问提供的地址：[http://chaoo.oschina.io](http://chaoo.oschina.io)即可查看到我的博客。
![博客迁移至码云3](http://obzf7z93c.bkt.clouddn.com/blog/oschina2.jpg)

### 2.修改hexo配置
打开博客根目录的_config.yml文件，找到：
``` bash
# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://chaooo.github.io

# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repository: https://github.com/chaooo/chaooo.github.io.git
  branch: master
```
修改为(根据自己的仓库地址修改)：
``` bash
# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
#url: http://chaooo.github.io
url: http://chaoo.oschina.io

# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
#- type: git
#  repository: https://github.com/chaooo/chaooo.github.io.git
#  branch: master
- type: git
  repository: https://git.oschina.net/chaoo/chaoo.git
  branch: master
```
然后执行下面命令，以重新生成`baidusitemap.xml`。
``` bash
hexo generate -d
```
#### 3.验证网站
百度搜索引擎入口：
[百度搜索引擎入口](http://zhanzhang.baidu.com/linksubmit/url)。
先添加域名，然后验证网站，这里统一都使用文件验证，就是下载对应的html文件，放到域名根目录下，也就收博客根目录下的`public/`下面 。
然后部署到服务器,输入地址：`http://chaoo.oschina.io/baidu_verify_wjJ25Q3cv2.html`能访问到就可以点验证按钮(按照百度的引导步骤就好)。

### 4.sitemap提交
直接点击`sitemap`填写数据文件地址：`http://chaoo.oschina.io/baidusitemap.xml`,输入验证码提交。
##### 自动推送
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

这样就可以等待百度收录了。

![博客迁移至码云4](http://obzf7z93c.bkt.clouddn.com/blog/oschina3.jpg)