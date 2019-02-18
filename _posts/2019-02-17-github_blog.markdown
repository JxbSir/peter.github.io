---
layout: post
title: 在github搭建技术博客
date: 2019-02-17 17:32:24.000000000 +09:00
---

### 准备
- **域名** 需要一个可访问的域名，例如 `jxb.name`，任何域名都可以，没有的话购买一个也很便宜
- **github** 想必，做技术的一般都有

### 这里使用Jekyll系统 + vno主题
> [Jekyll 中文介绍](https://jekyllcn.com/)
> [喵神的vno主题](https://github.com/onevcat/vno-jekyll)

> ##### 注意：
> 安装环境时关于homebrew的安装、镜像的修改，不是本次的重点，这边不再多过介绍

- **下载主题** 将喵神的vno主题下载到本地文件夹`myblog`
- **安装Jekyll** 
	- Jekyll
		- `sudo gem install jekyll`
	- bundle
		- `sudo gem install bundle`
	- 安装时出现以下错误，请在命令后面加`-n /usr/local/bin`
	```
	ERROR:  While executing gem ... (Gem::FilePermissionError)
	You don't have write permissions for the /usr/bin directory.
	```
- **本地启动Jekyll**
	- 在终端进入到myblog下，执行`bundle exec jekyll serve`，效果如下
	
```shell
Peter@Peter-MacBook:~/Private/blog/peter.github.io$ bundle exec jekyll serve
Configuration file: /Users/Peter/Private/blog/peter.github.io/_config.yml
            Source: /Users/Peter/Private/blog/peter.github.io
       Destination: /Users/Peter/Private/blog/peter.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
                    done in 0.502 seconds.
 Auto-regeneration: enabled for '/Users/Peter/Private/blog/peter.github.io'
    Server address: http://127.0.0.1:4000
  Server running... press ctrl-c to stop.
```
- **访问**
	- 在上一步可以看到，`server address`，在浏览器中访问该地址即可访问
- **修改配置** 在`_config.yml`文件里修改即可

> ok，到这里在本地的blog系统已经搭建完毕，那么接下来就是如何上传到`github`上面去

### Github Pages
- **CNAME** 首先在myblog下面创建`cname`文件，里面只要填写你想访问这个博客的域名，不需要加上协议头，比如我的就是：`www.jxb.name`
- **Repository** 在`github`上创建一个仓库，然后去setting里面，可以找到`Github pages`，在这边设置好`Source` = 分支，`custom domain`= `cname`中的域名
- **Push** 然后将`myblog`中的内存上传到这个仓库对象的`Source`分支
- **域名解析** 最后一步，只要去域名服务商那边添加`cname`的解析到`用户名.github.io`，具体设置`cname`这边也不多介绍
- **访问** 然后就可以直接访问了 [`blog`](http://www.jxb.name)

### 写博客
- **写文档** 此类博客系统都是没有后台系统的，只要你编写好`markdown`格式的文件（`markdown`后缀），保存到`myblog`下面的`_posts`下面，然后`push`到`github`即可，里面已经有2个默认的模板了，系统会自动发现并显示，很简单哦~~~
- **设置layout** 需要在文档的加入下面的格式，`title`与`date`根据自己修改哦
``` markdown
---
layout: post
title: Hello World - Vno
date: 2016-02-16 15:32:24.000000000 +09:00
---
``` 
- **文件格式** 必须已日期开头+名称的格式，如：`2019-02-17-blog.markdown`，否则无法自动识别
