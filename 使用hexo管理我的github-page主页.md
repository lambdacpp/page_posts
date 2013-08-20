title: 使用hexo管理我的Github Page主页
date: 2013-08-18 16:23:08
tags: github
---

### 终于做了这个决定

__『学而不思则罔』__

__『写作是为了更好的思考』__

常看些杂七杂八的东西，本着“通过写作来进行总结和思考是最好的学习方法”的原则，也做些笔记。既然做笔记，那就开了Github Page发布出来吧，一来避免随便记录不成文章，二来给自己一些压力不会轻易放弃。

<!--more-->

### 选来选去用了Hexo

用过的笔记工具可谓多亦。从纯文本、Emacs Wiki，云时代开始，从Dropbox、Evernote到“印象笔记”。最后明白了一个道理：要找到兼顾易用、富文本、协作、版本控制、排版控制、丰富输出的解决方案是绝不可能的，一个工具不行，一堆工具来Hack也不行。所以我得把要求降低一些：

* 使用轻量标记语言
* 便于版本控制
* 部署简单
* 不用再让注册新的互联网服务

看到这样需求其实就很简单了：使用Markdown,配合git，套用模板，部署到Github或Heroku。

1. _为什么用Markdown?_  因为它莫名其妙的就成为了最流行的轻量标记语言，大家都支持;
2. _为什么是git?_ 这个不是莫名其妙成为最流行的版本控制系统，它确实是最好的版本控制系统
3. _为什么需要hexo一类的平台?_ 当然可以手动或自制脚本来搞定剩下的事情，我也曾经乐此不疲，后来发现我不是写笔记，是在玩脚本
4. _为什么Github?_ 因为正在用

至于用Jekyll、Octopress还是hexo，完全随机选择的结果。其实装好了都非常好用。另外，[简书](http://jianshu.io/)上有个[Drafts + Scriptogr.am + Dropbox 打造移动端 Markdown 风格博客](http://jianshu.io/p/63HYZ6)很是诱人，但本着不注册新服务的原则，忍了。


### 使用步骤记录

参考：

* [hexo官网](http://zespia.tw/hexo/)
* [hexo github 地址](https://github.com/tommy351/hexo)
* [使用hexo搭建博客](http://yangjian.me/workspace/building-blog-with-hexo/)

#### 安装和基本配置

```
npm install -g hexo
hexo init blogdir
```
在`blogdir`目录下就是一个新的hexo网站的框架。需要配置的就是 `_config.yml`文件。

可以到hexo template网站找一个喜欢的模板clone到`themes`下，在`_config.yml`进行相应的配置即可。


#### 部署到github.io

在github中创建名为`username.github.io`的git库，在该库的Setting中,应用`Automatic Page Generator`。在hexo的`_config.yml`中配置部署的相关信息:

```
# Deployment
## Docs: http://zespia.tw/hexo/docs/deploy.html
deploy:
  type: github
  repository: git@github.com:username/username.github.io.git
  branch: master
```

#### 测试

新创建一个Blog，部署到github上，如果一切顺利，通过 http://username.github.io 即可访问新的Blog站点。

```
hexo new "新的Blog"
hexo deploy --generate
```

### 一些心得

1. Git只需要将`source/_post`下的文件进行版本控制即可，不需要将整个hexo项目目录纳入git库。
2. 中文文章最好使用中文作为标题
3. 部署之前使用 `hexo server --generate` 在本地测试
4. 项目下的`db.json`是hexo自动生成的，如果在测试中产生了一些不必要的标签和数据，可将其删除，运行`hexo generator`重新生成即可
5. [8.20补充] 保证`source/_post`下只有需要的.md文件，如果有编辑器自动备份的文件，hexo在生成网页时的时候会一并生成产生重复的网页


### 使用Git将md文档管理起来

前面提到只需要将`source/_post`文件管理起来就可以了。在github下再创建一个新的库，比如叫`page_posts`。在`_post`目录下如下操作 ：

```
git init
git add .
git commit -m "创建post库"
git remote add orgin git@github.com:username/page_posts.git
git push -u orgin master
```

_最后一句push 中的-u是--set-upstream的简写，目的在于指定master分支的upstream是orgin。这样定义以后，今后需要提交直接push就可以了。_


