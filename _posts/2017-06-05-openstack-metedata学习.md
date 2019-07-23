---
layout: post
title:  (3)openstack metedate学习
date:   2017-06-05 13:31:01 +0800
categories: openstack
tag: openstack
---

* content
{:toc}

### 服务介绍
    
    在云计算中，Metadata 并不是一个陌生的概念。从字面上看，Metadata 是元数据的意思。而在云计算中，Metadata 服务能够向虚机注入一些额外的信息，这样虚机在创建之后可以有一些定制化的配置。在 OpenStack 中，Metadata 服务能够向虚机提供主机名，ssh 公钥，用户传入的一些定制数据等其他信息。这些数据被分为两类：metadata和user data，metadata主要包括虚机自身的一些数据比如hostname、ssh秘钥、网络配置等，而user data主要包括一些定制的脚本、命令等。但是不管是哪一种数据，openstack向虚机提供数据的方式是一致的。

### 服务提供方式

    在 OpenStack 中，虚拟机获取 Metadata 信息的方式有两种：Config drive 和 metadata RESTful 服务。首先肯定的是提供metadata服务的组件是nova，而nova提供了两种方式，一种直接通过nova-api服务提供，另外一种是通过nova-api-metadata服务提供，现在基本就是nova-api来提供metadata服务了。
    
### metadata早期实现方法

    在早期的openstack版本中，大概F版之前没有neutron组件，网络由nova-network提供，也没有现在这么复杂的namespace，虚机的ip地址是不重复的，此时metadata的实现主要通过添加路由规则实现，这里还要提到169.254.169.254/32 这个地址，主要是早期亚马逊提出的metadata理念，openstack沿用了这个地址而已，早期实现代码如下:
    
![](http://blogdata.zhaolibin.com/FoTY0akFleSETeHcq3UPItyXijPf)
    
    明显随着openstack的发展，多租户的网络隔离网络不再那么简单单一，虚机ip地址的重叠比比皆是，此时只通过增加一条转发规则明显是不够的，因为相同ip的虚机传来请求我们无法识别，此时就衍生了neutron-metadata-agent、neutron-ns-metadata-proxy等服务，但是其核心思路还是沿用的。

### 
