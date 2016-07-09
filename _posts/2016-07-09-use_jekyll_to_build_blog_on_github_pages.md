---
layout: post
title: 使用Jekyll在github上搭建技术博客
---

Jekyll是一种静态网页生成工具，虽然有很多其他优秀的同类工具，Jekyll最大优点是Github原生支持。你不用在本地编译，Github自动会帮你编译Jekyll的资源文件生成静态网页。

## 选择一个好看的主题
Jekyll的主题好的真的不多，而且加上Jekyll3.x比较大的版本更新导致很多2.x的优秀框架都不兼容了，所以选择很窄，还需要自己试。最好的办法是看到github上好的Jekyll项目直接Clone一份。
我选择了hyde主题，就遇到没有升级到3.x的问题。需要自己改一下`_config.yml`。

	Dependencies
	highlighter:      rouge
	
	Permalinks
	permalink:        pretty
	
	Setup
	title:            Hyde
	tagline:          'A Jekyll theme'
	description:      'A brazen two-column <a href="http://jekyllrb.com" target="_blank">Jekyll</a> theme that pairs a prominent sidebar with uncomplicated content. Made by <a href="https://twitter.com/mdo" target="_blank">@mdo</a>.'
	url:              http://hyde.getpoole.com
	baseurl:          /
	
	author:
	  name:           'Mark Otto'
	  url:            https://twitter.com/mdo
	
	paginate:         5
	
	Custom vars
	version:          2.1.0
	
	github:
	  repo:           https://github.com/poole/hyde
	
	Gems
	gems: [jekyll-paginate]

## 部署到github
直接用git工具push到gh-pages分支，你就能访问了。但是直接部署后你会发现css文件都404了。

### 解决静态文件404无法找到的问题
假如github pages的地址是 xxx.github.io/yyy，那么css的地址生成逻辑是：

	<link rel="stylesheet" href="{{ site.baseurl }}public/css/poole.css">

这里baseurl是`xxx.github.io`，所以就会访问 `xxx.github.io/public/css/poole.css`。一个临时的解决方法是在config里加上baseurl的选项
	baseurl: /blog

但这里又带来一个问题。github pages支持绑定到自己的域名，如果你绑定到了`blog.example.com` 的域名后，你是希望baser恢复到/而不是/blog下的。github上有个issue讨论这个问题：[链接](https://github.com/jekyll/jekyll/issues/332)，但最后也没有什么结果。我们暂时不绑定自定义的域名，所以就先在config中把baseurl改成/blog （github项目名是blog）

另外，post的url地址也需要修改，以前是

	<a href="{{ post.url }}">

修改为

	<a href="{{site.baseurl}}{{ post.url }}">

### 　github pages 延时更新的问题
**注意：部署到更新有一些延时的**，感觉是修改静态文件延时最大，更新post的话马上就能生效

### 修改Jekyll模板
默认的hyde模板会显示博客的所有内容，而我希望能在首页只显示概要内容。所以，我们需要对模板进行改造。

修改index.html的内容，使用Jekyll自带的 excerpt功能不是很理想，所以还是采用比较土的办法：

{% raw %}

	{{ post.content }} 
	修改为
	{{ post.content | truncatewords:50 | strip_html }}

{% endraw %}

## 开始写一篇吧
可以先在编辑器里写好后，把markdown直接复制到github网页中新建的md文件里。
在`_posts_`的文件格式参照其他文件就可以了。有几个注意的地方：

1. 文件名的格式是固定的，用来指明创建时间
2. 如果用ulysses工具生成markdown默认不会在`##`前空行，这可能导致二级标题或者代码段不能被识别出来。Jekyll中需要在`##`标记前空行。现在暂时只能在ulysses里写作时注意空行。
3. 现在ulysses本地图片还不能自动上传。

## 未来的工作
写一个mac上的app，支持ulysses一键发布到github pages。需要的主要功能：

- 支持markdown兼容（比如上面说到的空行问题）
- 支持文件自动上传

可能会更方便的功能：

- 主题选择和编辑
