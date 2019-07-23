---
layout: post
title:  (2)openstack heat学习
date:   2017-06-02 13:31:01 +0800
categories: openstack
tag: openstack
---

* content
{:toc}

### Heat 介绍
    基于预先定义的模板，Heat通过自身的orchestration Engine来实现复杂应用的创建启动。Heat原生的模板格式目前还在不停地演进中，但是对CloudFormation的格式具有良好的支持。存在的CloudFormation的模板可以在OpenStack平台通过heat来启动。

#### Heat Client
    Heat client是Heat project 提供的CLI工具，类似于其他项目的client。对于heat tools的使用，可以通过安装后查看，或者通过此链接来查看。

#### heat-api
    Heat-api 类似于nova-api，提供了原生的restful API对外使用。用户对API的调用，由heat-api处理之后，最终通过RPC传递给Heat-engine来进一步处理。

#### heat-api-cfn
    heat-api-cfn组件则提供了Amazon style 的查询 API，因此可以完全兼容于Amazon的CloudFormation，对于API的请求，同heat-api类似，处理之后，通过RPC传递给heat-engine进一步处理。

#### heat-engine
    heat-engine是heat中的核心模块，主要的逻辑业务处理模块。此模块最终完成应用系统的创建和部署。

#### heat-cfntools
    这个工具是一个单独的工具，代码没在heat project里面，可以单独下载。这个工具主要用来完成虚拟机实例内部的操作配置任务。在创建虚拟机竟像时，需要在镜像中安装heat-cfntools工具。
    
    
#### 模板

    Heat templates支持多种模板内容展示格式，包括：HOT Syntax， YAML Syntax，JSON Syntax；每种格式表达的内容都一样，只是在表现形式上存在差别。关于模板的配置选项具体可以参考此链接。

    同时，Heat templates还支持多种资源格式，包括OpenStack自身的资源格式，Amazon的资源格式和RackSpace的资源格式。目前，Heat templates所支持的内容还在进一步演变中，会越来越完善。

    现在先参考官方贴出的一个简单例子，该模板中包含一个虚拟机实例，链接在此。

   - 虚拟机将使用的key-pair
   - 虚拟机的flavor配置，heat的模板可以定义选项的默认值，还可以配置选项的值的选择范围。
   - 虚拟机镜像的ID
   - 虚拟机实例内部数据库的配置密码
   - 虚拟机实例内部数据库的端口
   - 指定创建虚拟机实例过程需要获取的输出，如实例的ip地址

示例：
```json
parameters:
    KeyName:
        type: string
        description: Name of an existing key pair to use for the instance
    InstanceType:
        type: string
        description: Instance type for the instance to be created
        default: m1.small
        constraints:
            - allowed_values: [m1.tiny, m1.small, m1.large]
            description: Value must be one of 'm1.tiny', 'm1.small' or 'm1.large'
    ImageId:
        type: string
        description: ID of the image to use for the instance]
```
模板文件大致分为以下几个部分：
  - AWSTemplateFormatVersion: 版本信息
  - Description: 描述信息
  - Parameters: 一些自定义的变量，可以提供默认值，可选的值等等
  - Mappings: 一些映射，例如”AWSInstanceTypeArch:” {“m1.tiny”:{“Arch”:”32”}, “m1.small” : {“Arch”:”64”}}，这就是一个字典
  - Resources: 描述了一些资源，例如Instance，Network等等，资源有Type，Properties等等信息
  - Outputs: 返回值

        模板文件的这些部分中最为关键的是Resources段，Resource具有很多的attribute，例如type，properties等等，还有一些可选的attribute，例如DependsOn, DeletionPolicy, Metadata等等。除了属性之外，属性中还会用到一些函数，例如：Fn:Base64, Fn:FindInMap, Fn:GetAttr等等。通过resource，我们可以描述应用系统包含的虚拟机，虚拟机的属性，开机初始化信息，虚拟机软件栈的配置以及应用系统的网络等等信息。
例如，从heat-templates中摘取的一个resources例子，描述的LoadBalancer，有类型，属性等，用到了Fn::GetAZs函数。

```json
"LoadBalancer" : {
      "Type" : "AWS::ElasticLoadBalancing::LoadBalancer",
      "Properties" : {
        "AvailabilityZones" : { "Fn::GetAZs" : "" },
        "Instances" : [{"Ref": "WikiServerOne"}],
        "Listeners" : [ {
          "LoadBalancerPort" : "80",
          "InstancePort" : "80",
          "Protocol" : "HTTP"
        }]
      }
    }
```
对于Instance类型的resource，常会有下面这些可选的attribute：

 - Metadata: 作为nova创建虚拟机时指定的meta选项， 主要完成虚拟机启动时的基本配置，例如安装软件包，启动服务等等。
 - Properties中的UserData: 一般是脚本文件内容，根据环境来配置虚拟机的软件栈。

        上面例举的attribute与虚拟机软件栈的配置，服务的安装启动密切相关，通过在虚拟机内部安装cloudinit和heat-cfntools这两个工具，结合nova-metadata完成应用的自动化部署的大量工作。



#### heat和其它模块的关系

    Heat 是一个基于模板来编排复合云应用的服务。 它目前支持亚马逊的 CloudFormation 模板格式，也支持 Heat 自有的 Hot 模板格式。模板的使用简化了复杂基础设施，服务和应用的定义和部署。模板支持丰富的资源类型，不仅覆盖了常用的基础架构，包括计算、网络、存储、镜像，还覆盖了像 Ceilometer 的警报、Sahara 的集群、Trove 的实例等高级资源。
    
[](http://blogdata.zhaolibin.com/FmtRy7cMZXtDjIxy2z40PCe9Ye6j)

#### Heat 架构
    
    用户在 Horizon 中或者命令行中提交包含模板和参数输入的请求，Horizon 或者命令行工具会把请求转化为 REST 格式的 API 调用，然后调用 Heat-api 或者是 Heat-api-cfn。Heat-api 和 Heat-api-cfn 会验证模板的正确性，然后通过 AMQP 异步传递给 Heat Engine 来处理请求。
![](http://blogdata.zhaolibin.com/FgSaFiK2jihArnYTxLMV7IF-iY81)
    
    当 Heat Engine 拿到请求后，会把请求解析为各种类型的资源，每种资源都对应 OpenStack 其它的服务客户端，然后通过发送 REST 的请求给其它服务。通过如此的解析和协作，最终完成请求的处理。
    
    Heat Engine 在这里的作用分为三层： 第一层处理 Heat 层面的请求，就是根据模板和输入参数来创建 Stack，这里的 Stack 是由各种资源组合而成。 第二层解析 Stack 里各种资源的依赖关系，Stack 和嵌套 Stack 的关系。第三层就是根据解析出来的关系，依次调用各种服务客户段来创建各种资源。

#### Heat Engine 结构

![](http://blogdata.zhaolibin.com/FmW8E6t-0DQTAFKcCgc8EpTinSVy)


#### 编排

    编排，顾名思义，就是按照一定的目的依次排列。在 IT 的世界里头，一个完整的编排一般包括设置服务器上机器、安装 CPU、内存、硬盘、通电、插入网络接口、安装操作系统、配置操作系统、安装中间件、配置中间件、安装应用程序、配置应用发布程序。对于复杂的需要部署在多台服务器上的应用，需要重复这个过程，而且需要协调各个应用模块的配置，比如配置前面的应用服务器连上后面的数据库服务器。下图显示了一个典型应用需要编排的项目。
![](http://blogdata.zhaolibin.com/Fqr_0npCtJnQ1PzwvWGYGWB5pYry)
    
    在云计算的世界里，机器这层就变为了虚拟的 VM 或者是容器。管理 VM 所需要的各个资源要素和操作系统本身就成了 IaaS 这层编排的重点。操作系统本身安装完后的配置也是 IaaS 编排所覆盖的范围。除此之外，提供能够接入 PaaS 和 SaaS 编排的框架也是 IaaS 编排的范围。
    
#### heat编排
    OpenStack 从最开始就提供了命令行和 Horizon 来供用户管理资源。然而靠敲一行行的命令和在浏览器中的点击相当费时费力。即使把命令行保存为脚本，在输入输出，相互依赖之间要写额外的脚本来进行维护，而且不易于扩展。用户或者直接通过 REST API 编写程序，这里会引入额外的复杂性，同样不易于维护和扩展。 这都不利于用户使用 Openstack 来进行大批量的管理，更不利于使用 OpenStack 来编排各种资源以支撑 IT 应用。
    Heat 在这种情况下应运而生。Heat 采用了业界流行使用的模板方式来设计或者定义编排。用户只需要打开文本编辑器，编写一段基于 Key-Value 的模板，就能够方便地得到想要的编排。为了方便用户的使用，Heat 提供了大量的模板例子，大多数时候用户只需要选择想要的编排，通过拷贝-粘贴的方式来完成模板的编写。
    Heat 从四个方面来支持编排。首先是 OpenStack 自己提供的基础架构资源，包括计算，网络和存储等资源。通过编排这些资源，用户就可以得到最基本的 VM。值得提及的是，在编排 VM 的过程中，用户可以提供一些简单的脚本，以便对 VM 做一些简单的配置。然后用户可以通过 Heat 提供的 Software Configuration 和 Software Deployment 等对 VM 进行复杂的配置，比如安装软件、配置软件。接着如果用户有一些高级的功能需求，比如需要一组能够根据负荷自动伸缩的 VM 组，或者需要一组负载均衡的 VM，Heat 提供了 AutoScaling 和 Load Balance 等进行支持。如果要用户自己单独编程来完成这些功能，所花费的时间和编写的代码都是不菲的。现在通过 Heat，只需要一段长度的 Template，就可以实现这些复杂的应用。Heat 对诸如 AutoScaling 和 Load Blance 等复杂应用的支持已经非常成熟，有各种各样的模板供参考。最后如果用户的应用足够复杂，或者说用户的应用已经有了一些基于流行配置管理工具的部署，比如说已经基于 Chef 有了 Cookbook，那么可以通过集成 Chef 来复用这些 Cookbook，这样就能够节省大量的开发时间或者是迁移时间。本文稍后会分别对这四个方面做一些介绍。
![](http://blogdata.zhaolibin.com/FhzJ37Gm1ftbQ_BArX_gSnLBFFX9)

#### Heat 模板
    Heat 目前支持两种格式的模板，一种是基于 JSON 格式的 CFN 模板；另外一种是基于 YAML 格式的 HOT 模板。CFN 模板主要是为了保持对 AWS 的兼容性。HOT 模板是 Heat 自有的，资源类型更加丰富，更能体现出 Heat 特点的模板。
    一个典型的 HOT 模板由下列元素构成：
    模板版本：必填字段，指定所对应的模板版本，Heat 会根据版本进行检验。
        参数列表：选填，指输入参数列表。
        资源列表：必填，指生成的 Stack 所包含的各种资源。可以定义资源间的依赖关系，比如说生成 Port，然后再用 port 来生成 VM。
        输出列表：选填，指生成的 Stack 暴露出来的信息，可以用来给用户使用，也可以用来作为输入提供给其它的 Stack。
     对于 CFN 模板和 HOT 模板的不同，包括所支持的资源类型的不同，不在本文的讨论范围内。本文会主要用 HOT 模板。HOT 模板的全称是 Heat Orchestration Template，是 Heat 发展的重心。
     

    
https://www.ibm.com/developerworks/cn/cloud/library/1511_zoupx_openstackheat/index.html
 
  
  




