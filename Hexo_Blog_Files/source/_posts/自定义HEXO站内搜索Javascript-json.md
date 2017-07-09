---
title: 自定义HEXO站内搜索Javascript+json
date: 2016-11-09 09:24:56
tags: [javascript,hexo]
categories: hexo博客折腾
---

### 开始之前

目前很多[Hexo](https://hexo.io/)博客都用的Swiftype和Algolia等第三方搜索服务。其实针对无数据库的情况下，Hexo本身也提供了两个插件来生成数据文件作为数据源：
    [hexo-generator-search](https://github.com/PaicHyperionDev/hexo-generator-search)生成`xml`格式的数据文件。
    [hexo-generator-json-content](https://github.com/alexbruno/hexo-generator-json-content) 生成`json`格式的数据文件。 
今天的主角是[hexo-generator-json-content](https://github.com/alexbruno/hexo-generator-json-content)，对于 Javascript语言来说还是解析 json 更方便，如果需要用 xml 做数据文件也可以使用已有的atom.xml。
### 1.安装 

``` bash 
$ npm install hexo-generator-json-content@2.2.0 --save
```
然后执行`hexo generate`时会自动生成`content.json`文件，若使用默认设置，生成的数据结构如下 
``` json
meta: {
    title: hexo.config.title,
    subtitle: hexo.config.subtitle,
    description: hexo.config.description,
    author: hexo.config.author,
    url: hexo.config.url
},
pages: [{ //-> all pages
    title: page.title,
    slug: page.slug,
    date: page.date,
    updated: page.updated,
    comments: page.comments,
    permalink: page.permalink,
    path: page.path,
    excerpt: page.excerpt, //-> only text ;)
    keywords: null //-> it needs settings
    text: page.content, //-> only text minified ;)
    raw: page.raw, //-> original MD content
    content: page.content //-> final HTML content
}],
posts: [{ //-> only published posts
    title: post.title,
    slug: post.slug,
    date: post.date,
    updated: post.updated,
    comments: post.comments,
    permalink: post.permalink,
    path: post.path,
    excerpt: post.excerpt, //-> only text ;)
    keywords: null //-> it needs settings
    text: post.content, //-> only text minified ;)
    raw: post.raw, //-> original MD content
    content: post.content, //-> final HTML content
    categories: [{
        name: category.name,
        slug: category.slug,
        permalink: category.permalink
    }],
    tags: [{
        name: tag.name,
        slug: tag.slug,
        permalink: tag.permalink
    }]
}]
```
### 2.配置 

hexo-generator-json-content默认生成的json数据内容非常全，默认配置如下：
``` yml
jsonContent:
  meta: true
  keywords: false # (english, spanish, polish, german, french, italian, dutch, russian, portuguese, swedish)
  pages:
    title: true
    slug: true
    date: true
    updated: true
    comments: true
    path: true
    link: true
    permalink: true
    excerpt: true
    keywords: true # but only if root keywords option language was set
    text: true
    raw: false
    content: false
  posts:
    title: true
    slug: true
    date: true
    updated: true
    comments: true
    path: true
    link: true
    permalink: true
    excerpt: true
    keywords: true # but only if root keywords option language was set
    text: true
    raw: false
    content: false
    categories: true
    tags: true
```
因为默认生成了很多我们不需要的数据，所以我们要对其进行配置让它只生成我们想要的内容,在`hexo/_config.yml`中加入：
``` yml
jsonContent:
  meta: false
  pages: false
  posts:
    title: true #文章标题
    date: true #发表日期
    path: true #路径
    text: true #文本字段
    raw: false
    content: false
    slug: false
    updated: false
    comments: false
    link: false
    permalink: false
    excerpt: false
    categories: false
    tags: true #标签
```
这样，就只生成每篇文章的标题，日期，路径，标签和文本字段，同时也减小了文件的大小。
例如：
``` json
{
  "title": "自定义HEXO站内搜索Javascript+json",
  "date": "2016-11-09T01:24:56.000Z",
  "path": "2016/11/09/自定义HEXO站内搜索Javascript-json.html",
  "text": "目前很多Hexo博客都用的Swiftype和Algolia等第三......#这里显示整篇文章的内容",
  "tags": [{
    "name": "javascript,hexo",
    "slug": "javascript-hexo",
    "permalink": "http://chaoo.oschina.io/tags/javascript-hexo/"
  }]
}
```
### 3.JavaScript实现代码

接下来就是用JS实现查询方法并把结果渲染到页面。
#### 3.1 xhr加载数据
``` javascript
var searchData;
function loadData(success) {
    if (!searchData) {
        var xhr = new XMLHttpRequest();
        xhr.open('GET', '/content.json', true);
        xhr.onload = function() {
            if (this.status >= 200 && this.status < 300) {
                var res = JSON.parse(this.response || this.responseText);
                searchData = res instanceof Array ? res : res.posts;
                success(searchData);
            } else {
                console.error(this.statusText);
            }
        };
        xhr.onerror = function() {
            console.error(this.statusText);
        };
        xhr.send();
    } else {
        success(searchData);
    }
}
```
#### 3.2 匹配文章内容返回结果
``` javascript
function matcher(post, regExp) {
    // 匹配优先级：title > tags > text
    return regtest(post.title, regExp) || post.tags.some(function(tag) {
        return regtest(tag.name, regExp);
    }) || regtest(post.text, regExp);
}
function regtest(raw, regExp) {
    regExp.lastIndex = 0;
    return regExp.test(raw);
}
```
#### 3.3 结果渲染到页面
``` javascript
function render(data) {
    var html = '';
    if (data.length) {
        html = data.map(function(post) {
            return tpl(searchTpl, {
                title: post.title,
                path: post.path,
                date: new Date(post.date).toLocaleDateString(),
                tags: post.tags.map(function(tag) {
                    return '<span>' + tag.name + '</span>';
                }).join('')
            });
        }).join('');
    } 
}
```
#### 3.3 查询匹配
``` javascript
function search(key) {
    // 关键字 => 正则，空格隔开的看作多个关键字
    // a b c => /a|b|c/gmi
    var regExp = new RegExp(key.replace(/[ ]/g, '|'), 'gmi');
    loadData(function(data) {
        var result = data.filter(function(post) {
            return matcher(post, regExp);
        });
        render(result);
    });
}
```