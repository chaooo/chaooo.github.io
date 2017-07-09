---
title: BlueLake博客主题的详细配置
date: 2016-12-29 11:25:33
tags: [hexo,BlueLake]
categories: hexo博客折腾
---
### 开始之前

[BlueLake主题](https://github.com/chaooo/hexo-theme-BlueLake)写了有一段时间了，经常会有朋友发消息给我问一些配置的问题，这篇博文主要也是为了解决这些问题。主题以简洁轻量自居(实则简陋)，去掉了Jquery和Fancybox,用原生JS实现站内搜索功能和回到顶部效果。这个主题只是一个小小的雏形，期待您来帮助它成长。

在阅读本文之前，假定您已经成功安装了[Hexo](https://hexo.io/zh-cn/)，并使用 Hexo 提供的命令创建了一个静态博客。Hexo是一个快速、简洁且高效的博客框架。Hexo基于Node.js ，使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

> 需要特别注意的是Hexo有两个`_config.yml`配置文件，一份位于站点根目录下，主要包含 Hexo 站点本身的配置，下文中会称为**`根_config.yml`**；另一份位于主题目录下（themes/主题名/_config.yml），这份配置由主题作者提供，主要用于配置主题相关的选项,下文中会称为**`主题_config.yml`**。

### 1. 安装

您可以直接到[BlueLake发布页](https://github.com/chaooo/hexo-theme-BlueLake)下载，然后解压拷贝到`themes`目录下，修改配置即可。
不过我还是推荐使用`GIT`来checkout代码，之后也可以通过`git pull`来快速更新。

#### 1.1 安装主题

在根目录下打开终端窗口：
``` bash git bash
$ git clone https://github.com/chaooo/hexo-theme-BlueLake.git themes/BlueLake
```

#### 1.2 安装主题渲染器

BlueLake是基于`jade`和`stylus`写的，所以需要安装`hexo-renderer-jade`和`hexo-renderer-stylus`来渲染。
``` bash git bash
$ npm install hexo-renderer-jade@0.3.0 --save
$ npm install hexo-renderer-stylus --save
```

#### 1.3 启用主题

打开`根_config.yml`配置文件，找到theme字段，将其值改为`BlueLake`(先确认主题文件夹名称是否为BlueLake)。
``` yml 根_config.yml https://hexo.io/zh-cn/docs/configuration.html _config.yml
theme: BlueLake
```

#### 1.4 验证

首先启动 Hexo 本地站点，并开启调试模式：
``` bash git bash
$ hexo s --debug
```
在服务启动的过程，注意观察命令行输出是否有任何异常信息，如果你碰到问题，这些信息将帮助他人更好的定位错误。 当命令行输出中提示出：`INFO  Hexo is running at http://0.0.0.0:4000/. Press Ctrl+C to stop.`
此时即可使用浏览器访问 `http://localhost:4000`，检查站点是否正确运行。

#### 1.5 更新主题

今后若主题添加了新功能正是您需要的，您可以直接`git pull`来更新主题。
``` bash git bash
cd themes/BlueLake
git pull
```

### 2. 配置

#### 2.1 配置网站头部显示文字

打开`根_config.yml`，找到：
``` yml 根_config.yml https://hexo.io/zh-cn/docs/configuration.html _config.yml
title: 
subtitle: 
description: 
author: 
```
`title`和`subtitle`分别是网站主标题和副标题，会显示在网站头部；`description`在网站界面不会显示，内容会加入网站源码的`meta`标签中，主要用于SEO；`author`就填写网站所有者的名字，会在网站底部的`Copyright`处有所显示。

#### 2.2 设置语言

该主题目前有七种语言：简体中文（zh-CN），繁体中文（zh-TW），英语（en），法语（fr-FR），德语（de-DE），韩语 （ko）,西班牙语（es-ES）；例如选用简体中文，在`根_config.yml`配置如下：
``` yml 根_config.yml https://hexo.io/zh-cn/docs/configuration.html _config.yml
language: zh-CN
```

#### 2.3 设置菜单

打开`主题_config.yml`，找到：
``` yml 主题_config.yml https://github.com/chaooo/hexo-theme-BlueLake/blob/master/_config.yml themes/BlueLake/_config.yml
menu:
  - page: home
    directory: .
    icon: fa-home
  - page: archive
    directory: archives/
    icon: fa-archive
  # - page: about
  #   directory: about/
  #   icon: fa-user
  - page: rss
    directory: atom.xml
    icon: fa-rss
```
主题默认是展示四个菜单，即`主页home`，`归档archive`，`关于about`，`订阅RSS`；about需要手动添加，RSS需要安装插件，若您并不需要，可以直接注释掉。
每个页面底部的`footer`中`联系博主`的三个图标分别是`邮箱`，`微博主页链接地址`，`GitHUb个人页链接地址`，直接使用`主题_config.yml`中`about页面`的配置，若不需要about页面，只需要如下配置就好：
``` yml 主题_config.yml https://github.com/chaooo/hexo-theme-BlueLake/blob/master/_config.yml themes/BlueLake/_config.yml
# About page 
about:
  email: ## 个人邮箱 
  weibo_url: ## 微博主页链接地址
  github_url: ## github主页链接地址
```

##### 2.3.1 添加about页

此主题默认page页面是关于我页面的布局，在根目录下打开命令行窗口，生成一个关于我页面：
``` bash git bash
$ hexo new page 'about'
```
打开`主题_config.yml`，补全关于我页面的详细信息：
``` yml 主题_config.yml https://github.com/chaooo/hexo-theme-BlueLake/blob/master/_config.yml themes/BlueLake/_config.yml
# About page 
about:
  photo_url: ## 头像的链接地址
  email: ## 个人邮箱 
  weibo_url: ## 微博主页链接地址
  weibo_name: ## 微博用户名 
  github_url: ## github主页链接地址
  github_name: ## github用户名
```
当然您也可以自定义重新布局about页面，只需要修改`layout/page.jade`模板就好。

##### 2.3.2 安装 RSS(订阅) 和 sitemap(网站地图) 插件

在根目录下打开命令行窗口：
``` bash git bash
$ npm install hexo-generator-feed --save
$ npm install hexo-generator-sitemap --save
$ npm install hexo-generator-baidu-sitemap --save
```
添加`主题_config.yml`配置：
``` yml 主题_config.yml https://github.com/chaooo/hexo-theme-BlueLake/blob/master/_config.yml themes/BlueLake/_config.yml
Plugins:
  hexo-generator-feed
  hexo-generator-sitemap
  hexo-generator-baidu-sitemap

feed:
  type: atom
  path: atom.xml
  limit: 20

sitemap:
  path: sitemap.xml
baidusitemap:
  path: baidusitemap.xml
```

#### 2.4 添加本地搜索
默认本地搜索是用原生JS写的，但还需要HEXO插件创建的JSON数据文件配合。安装插件[hexo-generator-json-content](https://github.com/alexbruno/hexo-generator-json-content)来创建JSON数据文件：
``` bash git bash
$ npm install hexo-generator-json-content@2.2.0 --save
```
然后在`根_config.yml`添加配置：
``` yml 根_config.yml https://hexo.io/zh-cn/docs/configuration.html _config.yml
jsonContent:
  meta: false
  pages: false
  posts:
    title: true
    date: true
    path: true
    text: true
    raw: false
    content: false
    slug: false
    updated: false
    comments: false
    link: false
    permalink: false
    excerpt: false
    categories: false
    tags: true
```
最后在`主题_config.yml`添加配置：
``` yml 主题_config.yml https://github.com/chaooo/hexo-theme-BlueLake/blob/master/_config.yml themes/BlueLake/_config.yml
local_search: true
```

#### 2.5 修改站点图标

站点图标存放在主题的`Source`目录下，已经默认为您准备了两张图片。您也可以自己设计站点LOGO。
您需要准备一张ico格式并命名为** favicon.ico **，请将其放入hexo目录的`source`文件夹，建议大小：32px X 32px。
您需要为苹果设备添加网站徽标，请命名为** apple-touch-icon.png **的图像放入hexo目录的“source”文件夹中，建议大小为：114px X 114px。
(有很多网站都可以在线生成ico格式的图片。)

#### 2.6 添加站点关键字

请在hexo目录的`根_config.yml`中添加keywords字段，如：
``` yml 根_config.yml https://hexo.io/zh-cn/docs/configuration.html _config.yml
# Site
title: Hexo
subtitle: 副标题
description: 网站简要描述,如：Charles·Zheng's blog.
keywords: 网站关键字, key, key1, key2, key3
author: Charles
language: zh-CN
```

#### 2.7 其他配置
`主题_config.yml`的其他配置
1. `show_category_count`——是否显示分类下的文章数。
2. `widgets_on_small_screens`——是否在小屏显示侧边栏，若`true`,则侧边栏挂件将显示在底部。
``` yml 主题_config.yml https://github.com/chaooo/hexo-theme-BlueLake/blob/master/_config.yml themes/BlueLake/_config.yml 
show_category_count: true 
widgets_on_small_screens: true 
```

### 3.集成第三方服务

#### 3.1 添加评论

目前主题集成六种第三方评论，分别是[多说评论](http://duoshuo.com)、[Disqus评论](https://disqus.com)、[来必力评论](https://livere.com)、[友言评论](http://www.uyan.cc/)、[网易云跟帖评论](https://gentie.163.com/info.html)、[畅言评论](http://changyan.kuaizhan.com)，多说马上就要停止服务了，友言好像也没怎么维护,目前我已把自己的博客评论从多说转移到畅言了，在国内目前`网易云跟帖`和`畅言`还不错。
1. 注册并获得代码。
  - 若使用[多说评论](http://duoshuo.com)，注册多说后获得short_name。
  - 若使用[Disqus评论](https://disqus.com)，注册Disqus后获得short_name。
  - 若使用[来必力评论](https://livere.com)，注册来必力,获得data-uid。
  - 若使用[友言评论](http://www.uyan.cc/)，注册友言,获得uid。
  - 若使用[网易云跟帖评论](https://gentie.163.com/info.html)，注册网易云跟帖,获得productKey。
  - 若使用[畅言评论](http://changyan.kuaizhan.com)，注册畅言，获得appid，appkey。
2. 配置`主题_config.yml`：
``` yml 主题_config.yml https://github.com/chaooo/hexo-theme-BlueLake/blob/master/_config.yml themes/BlueLake/_config.yml
#Cmments
comment:
  duoshuo: ## duoshuo_shortname
  disqus: ## disqus_shortname
  livere: ## 来必力(data-uid)
  uyan: ## 友言(uid)
  cloudTie: ## 网易云跟帖(productKey)
  changyan: ## 畅言需在下方配置两个参数，此处不填。
    appid: ## 畅言(appid)
    appkey: ##畅言(appkey)
```

#### 3.2 百度统计

1. 登录[百度统计](http://tongji.baidu.com/)，定位到站点的代码获取页面。
2. 复制`//hm.baidu.com/hm.js?`后面那串统计脚本id(假设为：8006843039519956000)
3. 配置`主题_config.yml`:
``` yml 主题_config.yml https://github.com/chaooo/hexo-theme-BlueLake/blob/master/_config.yml themes/BlueLake/_config.yml 
baidu_analytics: 8006843039519956000
```
> 注意： `baidu_analytics`不是你的百度`id`或者百度统计`id`
如若使用谷歌统计，配置方法与百度统计类似。

#### 3.3 卜算子阅读次数统计

``` yml 主题_config.yml https://github.com/chaooo/hexo-theme-BlueLake/blob/master/_config.yml themes/BlueLake/_config.yml
busuanzi: true
```
若设置为`true`将计算文章的阅读量(Hits)，并显示在文章标题下的`小手图标`旁。

#### 3.4 微博秀

微博秀挂件的代码放在`layout/_widget/weibo.jade`下，需要您去[微博开放平台](http://open.weibo.com/)获取您自己的微博秀代码来替换。
1. 登录[微博开放平台](http://open.weibo.com/)，选择微博秀。
2. 为了与主题风格统一，作如下配置
  - 基础设置：高`400px`；勾选宽度自适应；颜色选择`白色`；
  - 样式设置：主字色`#333`；链接色`#40759b`；鼠标悬停色`#f7f8f8`；
  - 模块设置：去掉`标题`、`边框`、`粉丝`的勾选框，只留`微博`。
3. 复制代码里`src=""`里引号包裹的内容，替换到`layout/_widget/weibo.jade`
{% codeblock weibo.jade lang:stylus mark:1,7-8,10 https://github.com/chaooo/hexo-theme-BlueLake/blob/master/layout/_widget/weibo.jade layout/_widget/weibo.jade %}
.widget
  .widget-title
    i(class='fa fa-weibo')= ' ' + __('新浪微博')
  iframe(width="100%",height="400",class="share_self",frameborder="0",scrolling="no",src="http://widget.weibo.com/weiboshow/index.php?language=&width=0&height=400&fansRow=2&ptype=1&speed=0&skin=5&isTitle=0&noborder=0&isWeibo=1&isFans=0&uid=1700139362&verifier=85be6061&colors=d6f3f7,ffffff,333,40759b,f7f8f8&dpc=1")
{% endcodeblock %}
这只是为了和主题的风格统一，当然您也可以自由随意发挥。
> 注意：最主要是是要把`src`里`uid=`和`verifier=`后面的字段替换为您自己代码里的就好。