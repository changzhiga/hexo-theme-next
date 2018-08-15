---
title: Hexo 博客搭建
date: 2018-08-14 17:09:42
tags: hexo
categories: Tech
description: 我的 Hexo 博客搭建笔记
---

# GitHub Pages

## 创建 GitHub 仓库

新建名为`[[username]].github.io`的仓库，`https://[[username]].github.io`为博客的访问地址。

## 配置 Git

参考《Git 及 GitHub 配置指南》

------

# Hexo

## 安装 Hexo 框架

```shell
cd ~/Dropbox/Projects;
mkdir hexo
```

```shell
npm install -g hexo
```

## 操作命令

### 初始化

```shell
cd ~/Dropbox/Projects/hexo;
hexo init
```

### 生成静态文件

```shell
hexo g
```
执行以上命令之后，Hexo会在 public 文件夹生成相关 html 文件，这些文件将提交到 GitHub 仓库。

### 本地预览

```shell
hexo s --debug
```
打开浏览器访问 http://localhost:4000 即可看到内容，Ctrl+C 停止本地预览。

### 新建文章

```shell
hexo new "postName"
```
可以在`~/hexo/scaffolds/`中自定义模板，模板将声明文章的标题、时间、标签等信息：
```
---
title: {{ title }}
date: {{ date }}
tags:
categories:
description:
---
```
以 blog.md 为例，以后新建文章用：
```shell
hexo new blog "postName" #缩写：hnb
```

### 其他命令

```shell
hexo generate #生成静态页面至public目录
hexo server #开启预览访问端口
hexo deploy #部署到GitHub
hexo help  #查看帮助
hexo version  #查看Hexo的版本
```

```shell
hexo s -g #生成并本地预览
hexo d -g #生成并上传到 GitHub
```

## 主题设定

主题设定可参考 [NexT 官方使用指南](http://theme-next.iissnan.com/)。

### 安装 NexT

```shell
cd ~/Dropbox/Projects/hexo;
git clone https://github.com/iissnan/hexo-theme-next themes/next
```

### 启用并验证主题

编辑**站点配置文件**`_config.yml`，将`theme`字段更改为`next`；执行`hexo s --debug`在本地验证主题。

## 主题优化

### 主页文章边框添加阴影

编辑`themes/next/source/css/_custom`下的`custom.styl`：

```css
// 主页文章添加阴影效果
.post {
   margin-top: 0px;
   margin-bottom: 60px;
   padding: 25px;
   -webkit-box-shadow: 0 0 5px rgba(202, 203, 203, .5);
   -moz-box-shadow: 0 0 5px rgba(202, 203, 204, .5);
}
```

### 自定义代码块样式

编辑`themes/next/source/css/_custom`下的`custom.styl`：

```css
// code styles
code {
    color: #ff7600;
    background: #fbf7f8;
    margin: 2px;
}
// code highlight styles
.highlight, pre {
    margin: 5px 0;
    padding: 5px;
    border-radius: 3px;
}
.highlight, code, pre {
    border: 1px solid #d6d6d6;
}
```

### 代码块复制按钮

* 下载 [clipboard.min.js](https://raw.githubusercontent.com/zenorocha/clipboard.js/master/dist/clipboard.min.js)，将文件保存在`./themes/next/source/js/src`

* 在`./themes/next/source/js/src`目录下新建`clipboard-use.js`：
        
```js
/*页面载入完成后，创建复制按钮*/
!function (e, t, a) { 
/* code */
var initCopyCode = function(){
    var copyHtml = '';
    copyHtml += '<button class="btn-copy" data-clipboard-snippet="">';
    copyHtml += '  <i class="fa fa-globe"></i><span>copy</span>';
    copyHtml += '</button>';
    $(".highlight .code pre").before(copyHtml);
    new ClipboardJS('.btn-copy', {
        target: function(trigger) {
            return trigger.nextElementSibling;
        }
    });
}
initCopyCode();
}(window, document);
```
* 在`./themes/next/source/css/_custom/custom.styl`样式文件中添加如下代码：
        
```css
//代码块复制按钮
.highlight{
    //方便copy代码按钮（btn-copy）的定位
    position: relative;
}
.btn-copy {
    display: inline-block;
    cursor: pointer;
    background-color: #eee;
    background-image: linear-gradient(#fcfcfc,#eee);
    border: 1px solid #d5d5d5;
    border-radius: 3px;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
    -webkit-appearance: none;
    font-size: 13px;
    font-weight: 700;
    line-height: 20px;
    color: #333;
    -webkit-transition: opacity .3s ease-in-out;
    -o-transition: opacity .3s ease-in-out;
    transition: opacity .3s ease-in-out;
    padding: 2px 6px;
    position: absolute;
    right: 5px;
    top: 5px;
    opacity: 0;
}
.btn-copy span {
    margin-left: 5px;
}
.highlight:hover .btn-copy{
    opacity: 1;
}
```
* 在`./themes/next/layout/_layout.swig`的末尾添加引用：

```html
<!-- 代码块复制功能 -->
<script type="text/javascript" src="/js/src/clipboard.min.js"></script>  
<script type="text/javascript" src="/js/src/clipboard-use.js"></script>
```

### 字数和阅读时间统计

* 在 Hexo 站点下，安装`hexo-wordcount`插件：
        
```shell
npm i --save hexo-wordcount
```
* 编辑`主题配置文件`的`post_wordcount`字段：

```
# Post wordcount display settings
# Dependencies: https://github.com/willin/hexo-wordcount
post_wordcount:
    item_text: true
    wordcount: true
    min2read: true
    totalcount: false
    separated_meta: true
```

### 修改文章内文本连接样式为蓝色

打开`themes/next/source/css/_custom`下的`custom.styl`，添加代码：

```css
// 文章内链接文本样式
.post-body p a{
  color: #0593d3;
  border-bottom: none;
  border-bottom: 1px solid #0593d3;
  &:hover {
    color: #fc6423;
    border-bottom: none;
    border-bottom: 1px solid #fc6423;
  }
}
```

### 博文置顶

打开 Hexo 站点下的`node_modules/hexo-generator-index/lib/generator.js`，将代码全部替换为：

```js
'use strict';
var pagination = require('hexo-pagination');
module.exports = function(locals){
  var config = this.config;
  var posts = locals.posts;
    posts.data = posts.data.sort(function(a, b) {
        if(a.top && b.top) { // 两篇文章top都有定义
            if(a.top == b.top) return b.date - a.date; // 若top值一样则按照文章日期降序排
            else return b.top - a.top; // 否则按照top值降序排
        }
        else if(a.top && !b.top) { // 以下是只有一篇文章top有定义，那么将有top的排在前面（这里用异或操作居然不行233）
            return -1;
        }
        else if(!a.top && b.top) {
            return 1;
        }
        else return b.date - a.date; // 都没定义按照文章日期降序排
    });
  var paginationDir = config.pagination_dir || 'page';
  return pagination('', posts, {
    perPage: config.index_generator.per_page,
    layout: ['index', 'archive'],
    format: paginationDir + '/%d/',
    data: {
      __index: true
    }
  });
};
```
在文章的模板中添加`top`字段并设置数值，数值越大文章越靠前：

```
---
layout: layout
title: 标签1
date: 2017-08-18 15:41:18
tags: 标签1
top: 100
---
```

### 静态资源压缩

* 安装`gulp`：

```shell
npm install gulp -g
```
* 安装`gulp`插件：

```shell
npm install gulp-minify-css --save;
npm install gulp-htmlmin --save;
npm install gulp-htmlclean --save;
npm install gulp-uglify --save
```
* 在 Hexo 站点下添加`gulpfile.js`文件：

```js
var gulp = require('gulp');
var minifycss = require('gulp-minify-css');
var htmlmin = require('gulp-htmlmin');
var htmlclean = require('gulp-htmlclean');
var uglify = require('gulp-uglify');

// 压缩 public 目录 css
gulp.task('minify-css', function() {
  return gulp.src('./public/**/*.css')
    .pipe(minifycss())
    .pipe(gulp.dest('./public'));
});

// 压缩 public 目录 html
gulp.task('minify-html', function() {
  return gulp.src('./public/**/*.html')
    .pipe(htmlclean())
    .pipe(htmlmin({
      removeComments: true,//清除 HTML 注释
      collapseWhitespace: true,//压缩 HTML
      collapseBooleanAttributes: true,//省略布尔属性的值 <input checked="true"/> ==> <input /
      removeEmptyAttributes: true,//删除所有空格作属性值 <input id="" /> ==> <input />
      removeScriptTypeAttributes: true,//删除 <script> 的 type="text/javascript"
      removeStyleLinkTypeAttributes: true,//删除 <style> 和 <link> 的 type="text/css"
      minifyJS: true,//压缩页面 JS
      minifyCSS: true,//压缩页面 CSS
      minifyURLs: true,
    }))
    .pipe(gulp.dest('./public'))
});

// 压缩js文件
gulp.task('minify-js', function() {
    return gulp.src(['./public/**/.js','!./public/js/**/*min.js'])
        .pipe(uglify())
        .pipe(gulp.dest('./public'));
});

// 执行 gulp 命令时执行的任务
gulp.task('default', [
  'minify-html','minify-css','minify-js'
]);
```
* 在`package.json`添加自动部署命令：

```json
"scripts": {
    "build": "hexo clean && hexo g && gulp && hexo d",
    "test": "hexo clean && hexo g && gulp && hexo s"
},
```
* 运行`npm run build`即自动删除老文件，生成新文件，压缩html、css然后发布到github或其他静态服务器资源；运行`npm run test`即自动删除老文件，生成新文件，压缩html、css然后发布到本地服务器做预览。

### 开启版权声明

编辑`主题配置文件`的`post_copyright`字段：

```
# Declare license on posts
post_copyright:
  enable: true
  license: CC BY-NC-SA 3.0
  license_url: https://creativecommons.org/licenses/by-nc-sa/3.0/
```

### 修改文章底部带#号的标签

打开`themes/next/layout/_macro`下的`post.swig`文件,搜索`rel="tag">#`,将`#`换成`<i class="fa fa-tag"></i>`：

```
<div class="post-tags">
    {% for tag in post.tags %}
       <a href="{{ url_for(tag.path) }}" rel="tag"><i class="fa fa-tag"></i> {{ tag.name }}</a>
    {% endfor %}
</div>
```
执行`hexo clean`，重新部署测试`hexo s --debug`。

### 添加顶部加载条

打开`主题配置文件`，搜索关键字`pace`，设置为`true`，可以更换加载样式：

```
# Progress bar in the top during page loading.
pace: true
# Themes list:
#pace-theme-big-counter
#pace-theme-bounce
#pace-theme-barber-shop
#pace-theme-center-atom
#pace-theme-center-circle
#pace-theme-center-radar
#pace-theme-center-simple
#pace-theme-corner-indicator
#pace-theme-fill-left
#pace-theme-flash
#pace-theme-loading-bar
#pace-theme-mac-osx
#pace-theme-minimal
# For example
# pace_theme: pace-theme-center-simple
pace_theme: pace-theme-minimal #替换更换样式
```

### 添加本地搜索功能

* 在 Hexo 站点下安装插件：

```shell
npm install hexo-generator-searchdb --save
```
* 在`站点配置文件`添加配置：

```yml
search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```
将`主题配置文件`的`local_search`设置为`true`：

```
# Local search
# Dependencies: https://github.com/flashlab/hexo-generator-search
local_search:
  enable: true
  # if auto, trigger search by changing input
  # if manual, trigger search by pressing enter key or search button
  trigger: auto
  # show top n results per article, show all results by setting to -1
  top_n_per_article: 1
```