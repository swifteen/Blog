+++
title = "搭建HUGO博客"
description = ""
tags = [
    "HUGO",
    "blog",
]
date = "2022-04-17"
categories = [
    "工具",
]
menu = "main"

weight= 11

+++

# 搭建HUGO博客

## 安装hugo

进入[release页面下载](https://github.com/gohugoio/hugo/releases)，选择下载[hugo_extended_0.97.0_Linux-64bit.deb](https://github.com/gohugoio/hugo/releases/download/v0.97.0/hugo_extended_0.97.0_Linux-64bit.deb)带extended后缀的安装包

```shell
sudo dpkg -i hugo_extended_0.97.0_Linux-64bit.deb
```

## 创建hugo工程

```shell
mkdir ~/Public/Book
cd ~/Public/Book
hugo new site ./
```

## 下载主题

```shell
git init
git submodule add https://github.com/alex-shpak/hugo-book themes/hugo-book
cp -R themes/hugo-book/exampleSite/content .
```

## 运行Web服务

```shell
ubuntu:~/Public/Book$ hugo server --minify --theme hugo-book --bind="0.0.0.0" -p 8888
Start building sites … 
hugo v0.97.0-c07f3626e7c8160943591f4d209977efa02c3dca+extended linux/amd64 BuildDate=2022-04-14T08:45:07Z VendorInfo=gohugoio
WARN 2022/04/16 01:47:16 Expand shortcode is deprecated. Use 'details' instead.
WARN 2022/04/16 01:47:16 Page '/layout/variables' not found in 'posts/goisforlovers.md'
WARN 2022/04/16 01:47:16 Page '/layout/functions' not found in 'posts/goisforlovers.md'
WARN 2022/04/16 01:47:16 Page '/content/front-matter' not found in 'posts/goisforlovers.md'
WARN 2022/04/16 01:47:16 Page '/overview/configuration/' not found in 'posts/migrate-from-jekyll.md'
WARN 2022/04/16 01:47:16 Page '/layout/templates/' not found in 'posts/migrate-from-jekyll.md'
WARN 2022/04/16 01:47:16 Page '/doc/shortcodes/' not found in 'posts/migrate-from-jekyll.md'

                   | EN | RU | ZH  
-------------------+----+----+-----
  Pages            | 57 |  7 |  7  
  Paginator pages  |  0 |  0 |  0  
  Non-page files   |  0 |  0 |  0  
  Static files     | 78 | 78 | 78  
  Processed images |  0 |  0 |  0  
  Aliases          | 12 |  2 |  2  
  Sitemaps         |  2 |  1 |  1  
  Cleaned          |  0 |  0 |  0  

Built in 92 ms
Watching for changes in ~/Public/Book/{archetypes,content,data,layouts,static,themes}
Watching for config changes in ~/Public/Book/config.toml, ~/Public/Book/config/_default
Environment: "development"
Serving pages from memory
Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
Web Server is available at http://localhost:8888/ (bind address 0.0.0.0)
Press Ctrl+C to stop
```

# Hugo主题宽屏设置

修改文件themes/hugo-book/assets/_defaults.scss

```scss
$container-max-width: 80rem !default;
```

修改为

```scss
$container-max-width: 200rem !default;
```

