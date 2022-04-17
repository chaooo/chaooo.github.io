---
title: BlueLake博客主题的详细配置
date: 2016-12-29 11:25:33
tags: [hexo, BlueLake]
categories: Hexo博客
top: 9
---
### 2021年2月3日更新：

[BlueLake主题](https://github.com/chaooo/hexo-theme-BlueLake)主题写了有些年头了，随着Hexo升级到5.X，BlueLake很多配置已经过时，在使用中难免出现问题。

所以，我利用工作之余抽了些时间进行了大改版，主题模板引擎由Jade(Pug)换成了EJS，以landscape为原型进行二次开发；保留BlueLake主题老版本的其他功能，如原生JS实现站内搜索功能，本地分享等；新添加了一些新的功能，如打赏模块，集成新的三方评论等等。

说了这么多不如你亲自体验来的直接，BlueLake主题地址：https://github.com/chaooo/hexo-theme-BlueLake。
<!-- more -->
在阅读本文之前，假定您已经成功安装了[Hexo](https://hexo.io/zh-cn/)，并使用 Hexo 提供的命令创建了一个静态博客。Hexo是一个快速、简洁且高效的博客框架。Hexo基于Node.js ，使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

![](BlueLake.jpg)

> 需要特别注意的是Hexo有两个`_config.yml`配置文件，一份位于站点根目录下，主要包含 Hexo 站点本身的配置，下文中会称为**`根_config.yml`**；另一份位于主题目录下（themes/主题名/_config.yml），这份配置由主题作者提供，主要用于配置主题相关的选项,下文中会称为**`主题_config.yml`**。

### 1. 安装

您可以直接到[BlueLake发布页](https://github.com/chaooo/hexo-theme-BlueLake)下载，然后解压拷贝到`themes`目录下，修改配置即可。
不过我还是推荐使用`GIT`来checkout代码，之后也可以通过`git pull`来快速更新。

#### 1.1 安装主题

在根目录下打开终端窗口：
``` bash git bash
git clone https://github.com/chaooo/hexo-theme-BlueLake.git themes/bluelake
```

#### 1.2 安装主题插件

在根目录下打开终端窗口：
``` bash git bash
npm install hexo-renderer-jade --save
npm install hexo-renderer-stylus --save
npm install hexo-generator-feed --save
npm install hexo-generator-sitemap --save
```

#### 1.3 启用主题

打开`根_config.yml`配置文件，找到theme字段，将其值改为`bluelake`(先确认主题文件夹名称是否为bluelake)。
``` yml 根_config.yml https://hexo.io/zh-cn/docs/configuration.html _config.yml
theme: bluelake
```

#### 1.4 验证

首先启动 Hexo 本地站点，并开启调试模式：
``` bash git bash
hexo s --debug
```
在服务启动的过程，注意观察命令行输出是否有任何异常信息，如果你碰到问题，这些信息将帮助他人更好的定位错误。 当命令行输出中提示出：`INFO  Hexo is running at http://0.0.0.0:4000/. Press Ctrl+C to stop.`
此时即可使用浏览器访问 `http://localhost:4000`，检查站点是否正确运行。

#### 1.5 更新主题

今后若主题添加了新功能正是您需要的，您可以直接`git pull`来更新主题。
``` bash git bash
cd themes/bluelake
git pull
```

### 2. 配置

#### 2.1 配置网站头部显示文字

打开`根_config.yml`，找到：
``` yml 根_config.yml https://hexo.io/zh-cn/docs/configuration.html _config.yml
title:     # 标题，如：秋过冬漫长
subtitle:  # 副标题，如：没有比脚更长的路，走过去，前面是个天！
description: # 网站关键字，如：key, key1, key2, key3
keywords:
author:
```
- `title`和`subtitle`分别是网站主标题和副标题，会显示在网站头部；
- `description`在网站界面不会显示，内容会加入网站源码的`meta`标签中，主要用于SEO；
- `keywords`在网站界面不会显示，内容会加入网站源码的`meta`标签中，主要用于SEO；
- `author`就填写网站所有者的名字，会在网站底部的`Copyright`处有所显示。

#### 2.2 设置语言

该主题目前有七种语言：简体中文（zh-CN），繁体中文（zh-TW），英语（en），法语（fr-FR），德语（de-DE），韩语 （ko）,西班牙语（es-ES）；例如选用简体中文，在`根_config.yml`配置如下：
``` yml 根_config.yml https://hexo.io/zh-cn/docs/configuration.html _config.yml
language: zh-CN
```

#### 2.3 设置菜单

打开`主题_config.yml`，找到：
``` yml 主题_config.yml https://github.com/chaooo/hexo-theme-BlueLake/blob/master/_config.yml themes/bluelake/_config.yml
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

#### 2.4 添加about页

此主题默认page页面是关于我页面的布局，在根目录下打开命令行窗口，生成一个关于我页面：
``` bash git bash
hexo new page 'about'
```
打开`主题_config.yml`，补全about页面的详细信息：
``` yml 主题_config.yml https://github.com/chaooo/hexo-theme-BlueLake/blob/master/_config.yml themes/bluelake/_config.yml
about:
  photo_url: # 头像的链接地址 e.g. http://cdn.itdn.top/blog/themeauthor.jpg
  items:
  - label: email # 个人邮箱
    url: ## Your email with mailto: e.g.  mailto:zhenggchaoo@gmail.com
    title: ## Your email e.g.  zhenggchaoo@gmail.com
  - label: github # github
    url: ## Your github'url e.g.  https://github.com/chaooo
    title: ## Your github'name e.g.  chaooo
  - label: weibo # 微博
    url: ## Your weibo's url e.g.  http://weibo.com/zhengchaooo
    title: ## Your weibo's name e.g.  秋过冬漫长
```
当然您也可以自定义重新布局about页面，只需要修改`layout/page.jade`模板就好。

#### 2.5 添加本地搜索
默认本地搜索是用原生JS写的，但还需要HEXO插件创建的JSON数据文件配合。安装插件[hexo-generator-json-content](https://github.com/alexbruno/hexo-generator-json-content)来创建JSON数据文件：
``` bash git bash
npm install hexo-generator-json-content@2.2.0 --save
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
``` yml 主题_config.yml https://github.com/chaooo/hexo-theme-BlueLake/blob/master/_config.yml themes/bluelake/_config.yml
local_search: true
```

#### 2.6 首页添加文章置顶
在根目录下打开命令行窗口安装：
``` bash git bash
npm uninstall hexo-generator-index --save
npm install hexo-generator-index-pin-top --save
```
然后在需要置顶的文章的Front-matter中加上top: true即可。
``` markdown
---
title: BlueLake博客主题的详细配置
tags: [hexo,BlueLake]
categories: Hexo博客
top: true
---
```


### 3.集成第三方服务
#### 3.1 添加评论
目前主题集成多种第三方评论，比如基于Github Issue的[GITALK](https://gitalk.github.io/)），[Valine](https://valine.js.org/)评论，[畅言评论](http://changyan.kuaizhan.com)，[Disqus评论](https://disqus.com)、[来必力评论](https://livere.com)、[友言评论](http://www.uyan.cc/)、[网易云跟帖评论](https://gentie.163.com/info.html)等。

 配置`主题_config.yml`：
``` yml 主题_config.yml https://github.com/chaooo/hexo-theme-BlueLake/blob/master/_config.yml themes/bluelake/_config.yml
# Gitalk comment
gitalk:
  enable: false
  owner: ## Your GitHub ID, e.g. username
  repo: ## The repository to store your comments, make sure you're the repo's owner, e.g. gitalk.github.io
  client_id: ## GitHub client ID, e.g. 75752dafe7907a897619
  client_secret: ## GitHub client secret, e.g. ec2fb9054972c891289640354993b662f4cccc50
  admin: ## Github repo owner and collaborators, only these guys can initialize github issues.
  language: 'zh-CN' ## Language
  pagerDirection: last # Comment sorting direction, available values are last and first.

# Valine comment. https://valine.js.org
valine:
  enable: false # if you want use valine,please set this value is true
  appId: # leancloud application app id
  appKey: # leancloud application app key
  notify: false # valine mail notify (true/false) https://github.com/xCss/Valine/wiki
  verify: false # valine verify code (true/false)
  pageSize: 10 # comment list page size
  avatar: mm # gravatar style https://valine.js.org/#/avatar
  lang: zh-cn # i18n: zh-cn/en
  placeholder: Just go go # valine comment input placeholder(like: Please leave your footprints )
  guest_info: nick,mail,link #valine comment header info

# Changyan comment. 畅言
changyan:
  enable: false
  appid: ## changyan appid
  appkey: ## changyan appkey

# Other comments
comment:
  disqus: ## disqus_shortname
  livere: ## 来必力(data-uid)
  uyan: ## 友言(uid)
  cloudTie: ## 网易云跟帖(productKey)
```

#### 3.2 站点统计

``` yml 主题_config.yml https://github.com/chaooo/hexo-theme-BlueLake/blob/master/_config.yml themes/bluelake/_config.yml 
busuanzi: true  # 卜算子阅读次数统计
baidu_analytics: # 百度统计
google_analytics: # 谷歌统计
gauges_analytics: # Gauges
```

##### 3.2.1 卜算子阅读次数统计

若busuanzi设置为true将计算文章的阅读量，及网站的访问量与访客数，并显示在文章标题下和网站底部。

##### 3.2.2 百度统计
登录百度统计，定位到站点的代码获取页面。复制`//hm.baidu.com/hm.js?`后面那串统计脚本id(假设为：`8006843039519956000`)

``` yml 主题_config.yml https://github.com/chaooo/hexo-theme-BlueLake/blob/master/_config.yml themes/bluelake/_config.yml 
baidu_analytics: 8006843039519956000
```

> 注意： baidu_analytics不是你的百度id或者百度统计id，如若使用其他统计，配置方法与百度统计类似。
