---
layout: post
title: Windows下安装jekyll
cover: /public/images/cover/chrome-extensions.jpg
tags: ['jekyll']
---

[Jekyll是什么?](https://jekyllrb.com/){:target="_blank"}



## >事先准备

### 1.Ruby

* Windows下用RailsInstaller 来安装Ruby环境, 下载地址为：[点击进入下载页面](http://railsinstaller.org/en) 
* 如果你不能翻墙的话, 可以在我的百度云地址下载 : [RailsInstaller 3.4.0](https://pan.baidu.com/s/1c6knxF_2EPxgR2j0aKwCDw)
* 安装完成执行命令 `ruby -v` 查看结果.

安装成功示例:

```ruby
$ ruby -v
ruby 2.3.3p222 (2016-11-21 revision 56859) [i386-mingw32]
```

### 2.RubyGems

用RailsInstaller 附带Gems.

用Gem安装Jekyll:

```ruby
gem install jekyll
```

### 3.Bundle

Jekyll官方推荐用bundle启动jekyll

```ruby
gem install bundle
```

至此, 环境已经准备完毕.



## 用Jekyll创建博客站点

随便进入一个盘符

```
d:
```

创建博客目录

```
jekyll new blog
```

磁盘下多出一个叫`blog`的目录.

```
cd blog
# 进入blog目录, 然后执行启动命令
bundle exec jekyll s
```

默认启动在本地4000端口, 打开浏览器访问即可.