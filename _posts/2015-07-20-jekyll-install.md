---
layout: post
title:  "centos7安装配置jekyll"
date:   2017-07-20 13:31:01 +0800
categories: jekyll
tag: jekyll
---

* content
{:toc}


#### 系统环境 ：
    CentOS Linux release 7.2.1511 (Core)

#### 安装ruby
    
 ```
    gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
    curl -sSL https://get.rvm.io | bash -s stable
    source /etc/profile.d/rvm.sh
    rvm list known   #查看可安装版本
    rvm install 2.4.6
    ruby -v
```

#### 安装RubyGems
```
    yum install gem
    # 添加 TUNA 源并移除默认源
    gem sources --add https://mirrors.tuna.tsinghua.edu.cn/rubygems/ --remove https://rubygems.org/
    # 列出已有源
    gem sources -l
    # 应该只有 TUNA 一个
```

#### 安装Jekyll

```
gem install jekyll bundler
gem install jekyll-feed
gem install jekyll-seo-tag
```
 
