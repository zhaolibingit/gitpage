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
    
#### 安装配置
    gpg2 --keyserver hkp://pool.sks-keyservers.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
    curl -sSL https://get.rvm.io | bash -s stable
    /usr/local/rvm/bin/rvm list known   # 查看有哪些可安装版本
    /usr/local/rvm/bin/rvm install ruby-2.4.6
    rvm use ruby-2.4.6
    gem install jekyll
 
