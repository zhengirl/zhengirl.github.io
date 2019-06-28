---
title: Hexo之NexT博客美化
date: 2019-01-20 15:29:46
categories: Hexo
tags: [hexo,next,美化设置]
---
## Hexo之NexT博客美化

>**写在前面**：默认的hexo界面看起来还是太简陋了，可以给Hexo换一个主题，这里推荐NexT，这是一个比较成熟的主题，使用的人也是最多的，优化，配置扩展都集成了，使用起来比较简单。然后再对功能界面做一些扩展，博客重质量，界面做的干净、清爽就行。本文详细介绍了博客美化的步骤。
<!--more-->
### 一、安装NexT主题

在命令行输入

```bash
git clone https://github.com/theme-next/hexo-theme-next themes/next
```

下载主题。打开根目录下的`_config.yml`为博客的站点配置文件，主题配置文件在`./themes/_config.yml`。本文的整个配置基本是在修改这两个配置文件，所以你需要区分清楚。在站点配置文件`./config.yml`查找`theme`并修改：

```yaml
## Themes: https://hexo.io/themes/
theme: next 
```

这样就启用了主题next，可以输入`hexo s`查看效果。注意，有时候通过`hexo s`预览时，你会发现自己所做的修改并没有生效，这时不要着急，命令行输入`hexo clean`清理下`database`文件夹和`public`文件夹即可。

### 二、博客设置

需要先对博客基本信息做一些设置，**注意，设置时冒号后面都要有一个空格，这是yml语法格式**。否则会报错或修改不生效。

#### 2.1、设置语言

在站点配置文件`./_config.yml`中，将language设置成所需要的语言。例如简体中文，配置如下：

```yaml
language: zh-CN
```

#### 2.2、基本信息配置

在站点配置文件`./_config.yml`的开头，填上自己博客的相应信息：

```yaml
title: #标题
subtitle: #子主页标题 
description: #描述
keywords: #关键字
author: #作者zhen
```

#### 2.3、设置主题的Scheme

Next自带了几种外观，在主题配置文件`./themes/next/_config.yml`里找到`schemes`，可以自行选择布局，根据个人喜好，把前面的注释符#去掉即可：

```yaml
# Schemes
# scheme: Muse
scheme: Mist
#scheme: Pisces
#scheme: Gemini
```

#### 2.4、菜单栏设置

在网站首页有归档等菜单，在主题配置文件`./themes/next/_config.yml`里找到`menu`，把需要的菜单取消注释。另外也可以自己添加菜单栏，`||`后面是`font awesome`图标栏，如下：

```yaml
menu:
  home: / || home
  archives: /archives/ || archive
  categories: /categories/ || th
  tags: /tags/ || tags
  about: /about/ || user 
  #schedule: /schedule/ || calendar
  #sitemap: /sitemap.xml || sitemap
  #commonweal: /404/ || heartbeat
```

#### 2.5、创建页面

设置完菜单但是没有相关页面的话点击进去就会显示错误。在命令行输入

```bash
hexo new page tags
hexo new page categories
hexo new page about
```

然后在`./source/_posts`文件夹下面会生成对应的文件夹，打开将页面的type设置为相应的内容。例：

```markdown
---
title: 这里是所有分类的汇总
categories: 分类名 
type: "categories"
date: 2019-01-17 15:29:31
---
```

#### 2.6、文章显示设置

默认首页的文章会显示全文，在发表文章的内容中加上`<!--more-->`

这样首页中文章会显示到你插入这句话的前面，点击阅读全文才会显示整篇文章。

#### 2.7、使用RSS

在命令行中输入:

```bash
npm install --save hexo-generator-feed
```

安装插件，然后在主题配置文件`./themes/next/_config.yml`中找到`rss`并修改：

```yaml
rss: /atom.xml
```

#### 2.8、设置博客favicon图标

在`./themes/next/source/images`目录下放置图标，和默认的图标类似，然后在主题配置文件`./themes/next/_config.yml`找到`favicon`并修改：

```yaml
favicon:
  small: /images/favicon-16x16-next.ico
  medium: /images/favicon-32x32-next.ico
  apple_touch_icon: /images/apple-touch-icon-next.png
  safari_pinned_tab: /images/logo.svg
  #android_manifest: /images/manifest.json
  #ms_browserconfig: /images/browserconfig.xml
```

#### 2.9、侧边栏社交链接

在主题配置文件`./themes/next/_config.yml`找到`social`把需要的内容取消注释，填好你的链接就可以。`||`后面的是图标名称，和菜单一样，也是使用的Font Awesome`图标。

```yaml
social:
  GitHub: https://github.com/zhengirl || github
  #E-Mail: mailto:yourname@gmail.com || envelope
  #Weibo: https://weibo.com/yourname || weibo
  #Google: https://plus.google.com/yourname || google
  #Twitter: https://twitter.com/yourname || twitter
  #FB Page: https://www.facebook.com/yourname || facebook
  #VK Group: https://vk.com/yourname || vk
  #StackOverflow: https://stackoverflow.com/yourname || stack-overflow
  #YouTube: https://youtube.com/yourname || youtube
  #Instagram: https://instagram.com/yourname || instagram
  #Skype: skype:yourname?call|chat || skype
```

#### 2.10、设置背景动画

在主题配置文件中，找到canvas_nest，改为`true`：

```yaml
canvas_nest:
  enable: true
  onmobile: true # display on mobile or not
  color: '0,0,255' # RGB values, use ',' to separate
  opacity: 0.5 # the opacity of line: 0~1
  zIndex: -1 # z-index property of the background
  count: 99 # the number of lines
```

#### 2.11、修改文章底部的#号标签

打开`./themes/next/layout/_macro/post.swig`文件中，搜索`rel="tag">#`,将`#`替换为`Font Awesome`图标：

```JavaScript
rel="tag"><i class="fa fa-tag"></i>
```

#### 2.12、搜索服务

在命令行输入：

```bash
npm install hexo-generator-searchdb --save
```

安装`hexo-generator-searchdb`插件，然后在站点配置文件`./_config.yml`添加以下代码：

```yaml
# search
search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```

然后在主题配置文件`./themes/next/_config.yml`中找到`local_search`改为`true`即可：

```yaml
local_search:
  enable: true
```

#### 2.13、代码高亮

在站点配置文件`./_config.yml`内找到`highlight`，并设置如下：

```yaml
highlight:
  enable: true
  line_number: true
  #代码自动高亮
  auto_detect: true
  tab_replace:
```

然后在主题配置文件`./themes/next/_config.yml`中找到`highlight_theme`，设置成你喜欢的代码高亮主题：

```yaml
# Code Highlight theme
# Available values: normal | night | night eighties | night blue | night bright
# https://github.com/chriskempson/tomorrow-theme
highlight_theme: night
```

#### 2.14、头像圆形和旋转

将头像显示成圆形，鼠标放上去有旋转效果，在`.\themes\next\source\css\_common\components\sidebar\sidebar-author.styl`文件将里面的内容替换为：

```css
.site-author-image {
  margin: 0 auto;
  padding: $site-author-image-padding;
  max-width: $site-author-image-width;
  height: $site-author-image-height;
  border: $site-author-image-border-width solid $site-author-image-border-color;

  border-radius: 50%;
  -webkit-border-radius: 50%;
  -moz-border-radius: 50%;
  transition: 1.4s all;
}

.site-author-image:hover {
    background-color: #e6be93;
    -webkit-transform: rotate(360deg);
    -moz-transform: rotate(360deg);
    -ms-transform: rotate(360deg);
    -transform: rotate(360deg);
}

.site-author-name {
  margin: $site-author-name-margin;
  text-align: $site-author-name-align;
  color: $site-author-name-color;
  font-weight: $site-author-name-weight;
}

.site-description {
  margin-top: $site-description-margin-top;
  text-align: $site-description-align;
  font-size: $site-description-font-size;
  color: $site-description-color;
}

```

#### 2.15、添加文章字数和阅读时长统计功能

首先需要在命令行输入：

```bash
npm install hexo-symbols-count-time --save
```

安装统计插件，然后在站点配置文件`./_config.yml`末尾添加如下使能统计功能的代码：

```yaml
# reading time
symbols_count_time:
  symbols: true
  time: true
  total_symbols: true
  total_time: true
```

#### 2.16、保留文章本身的编号

我们在写博客的时候，会自己给文章编号，但next主题默认的也有编号，这样多个编号就比较奇怪，所以把默认的编号取消，在主题配置文件查找`toc`，修改如下：

```yaml
toc:
  enable: true
  # Automatically add list number to toc.
  number: false
```

#### 2.17、自定义博客

Next中留出给使用者自我设计的空间，在`/themes/next/source/css/_custom/cutom.styl`文件中可以自行添加一些小样式让博客有所不同：

```yaml
//文章阴影与边缘强化
.post {
    margin-top: 0px;
    margin-bottom: 0px;
    border-radius: 16px;
    padding: 25px;
    -webkit-box-shadow: 0 0 5px rgba(0, 0, 0, 0.70);
    -moz-box-shadow: 0 0 5px rgba(0, 0, 0, 0.70);
}
```

### 三、在文章中显示图片

首先在站点配置文件中将`post_asset_folder`后面修改为true，在建立一篇新的博客时，Hexo会自动建立一个与文章同名的文件夹，这样一来，就可以把图片存储在这个文件夹中方便调用。

其次本人习惯于在`typora`中将markdown文件编辑好之后直接复制到hexo中，所以需要对typora的设置做一些更改，打开偏好设置，选择将图片复制到指定文件夹中，这样在typora中也有指定的与文章同名的文件夹，所以将文章和文件夹都复制到Hexo中即可。

![1547956225987](美化/1547956225987.png)

在命令行输入

```bash
npm install https://github.com/CodeFalling/hexo-asset-image --save
```

安装插件，等待一段时间。输入`hexo s`在本地预览网站图片就可以显示啦！

### 四、致谢

在搭建博客的过程中，遇到了一些问题，参考了很多大佬的解决方案，感谢他们的分享。本文只是将自己搭建博客的过程复述了一遍，希望对后来者有所帮助。

推荐在配置的时看官方文档：http://theme-next.iissnan.com/third-party-services.html