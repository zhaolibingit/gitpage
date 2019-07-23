---
layout: post
title:  (1) openstack 创建虚机流程
date:   2017-06-01 13:31:01 +0800
categories: openstack
tag: openstack
---

* content
{:toc}   

1. Dashboard 或者 CLI 获取用户的登录信息，调用 Keystone 的 REST API 去做用户身份验证。
2. Keystone 对用户登录信息进行校验，然后产生验证token并发回。它会被用于后续 REST 调用请求。
3. Dashboard 或者 CLI 将创建虚机的 REST请求中的‘launch instance’ 或‘nova-boot’ 部分进行转换，然后调用 nova-api 的 REST 接口。
4. nova-api 接到请求，向 keystone 发送 auth-token 校验和权限认证请求。
5. Keystone 校验 token，并将 auth headers 发回，它包括了 roles 和 permissions。
6. nova-api 和 nova-database 进行交互。
7. nova-database 为新实例创建一个数据库条目。
8. nova-api 向 nova-scheduler 发送  rpc.call 请求，期望它能通过附带的 host ID 获取到数据库条目。
9. nova-scheduler 从 queue 中获取到请求。
10. nova-scheduler 和 nova-database 交互，获取集群中计算节点的信息和状态。
11. nova-scheuler 通过过滤（filtering）和称重（weighting）找到一个合适的计算节点（host）。
12. nova-scheduler 向找到的那个host上的 nova-compute 发送 rpc.cast 请求去启动虚机。
13. 目标 host 上的 nova-compute 从 queue 中获取到请求。
14. nova-compute 向 nova-condutor 发送 rpc.call 请求去获取待创建虚机的信息比如 host ID 和 flavor 等。
15. nova-conductor 从queue 中获取到请求。
16. nova-conductor 和 nova-database 交互。
17. nova-database 向 nova-conductor 返回虚机的信息。
18. nova-conductor 向 nova-compute 发送 rpc.call，附带所请求的信息。图中应该是遗漏了一个步骤，就是 nova-compute 从queue 中获取返回的数据。
19. nova-compute 调用 glance-api 的 REST API，传入 auth-token，去根据镜像 ID 获取镜像 URI，从镜像存储中下载（原文为upload）镜像。
20. glance-api 向 keystone 校验 auth-token。
21. nova-compute 获取 image 的元数据。
22. nova-compute 调用 Neutron API ，传入 
23. auth-token，去分配和配置网络，比如虚机的IP地址。
24. neutron-server 通过 keystone 校验 auth-token。
25. nova-compute 获得网络信息。
26. nova-compute 调用 Cinder API，传入 auth-token，去将 volume 挂接到实例。
27. cinder-api 通过 keystone 校验 auth-token。
28. nova-compute 获得块存储信息。
29. nova-compute 为 hypervisor driver 产生数据，并调用 Hypersior 执行请求（通过 libvirt 或者 api）。

![](http://blogdata.zhaolibin.com/Fo9TQi1qW6zhTjg1O0djCRc5kWWR)

#### 第一阶段：KeyStone验证

1. 用户使用Dashboard Horizon或者命令行CLI，通过REST API给Identity 服务Keystone发送用户凭据（credentials）并验证（authenticates）。Keystone使用用户凭据进行验证，然后返回一个auth-token。然后后续操作就可以使用这个auth-token通过REST调用请求OpenStack其他的组件。
2. Horizon或者CLI将launch instance 或者nova-boot转换形成为一个REST API的请求发送给nova-api。
3. nova-api接到这个请求后，首先向keystone发送一个请求来确认auth-token是否有效和是否有访问权限。Keystone确认auth-token后，发送一个包含角色和权限的更新后的认证头。
4. nova-api和Nova数据库交互，将用户的创建虚拟机的请求在nova 数据库里记录下来。

#### 第二阶段：Nova服务组件交互

5. nova-api以rpc.call的方式发送一个请求给nova-schedule,让nova-scheduler去选择一个计算节点来创建虚拟机。注意是通过消息队列发送给nova-scheduler。
6. nova-schedule调度服务会侦听Scheduler队列，从队列中获取数据。
7. nova-scheduler和Nova数据库交互，通过调度算法，也就是filtering 和weighing最终选择一台运行nova-compute的计算节点，然后nova-schedule将虚拟机信息使用rpc.cast的模式发送至nova-compute.计算节点队列。让nova-compute在选择好的计算节点中去创建实例。
8. Nova-Compute从队列获取请求。
9. nova-compute发送一个rpc.call请求给nova-conductor，去获取实例的信息，比如host ID和选择的Flavor（CPU、内存和磁盘）。
10. nova-conductor从队列中获取请求。
11. nova-conductor与nova的数据库进行交互。nova-conductor返回实例的信息。nova-compute从队列中获取实例的信息。

#### 第三阶段：OpenStack其它服务交互

在第二阶段nova-compute为了获取到创建实例所需要的资源，比如镜像、网络、存储。会使用在第一阶段用户验证后获取到的auth-tokon分别和镜像服务Glance的glance-api，网络服务Neutron的neutron-server已经块存储服务Cinder的cinder-api进行交互。而且每次对方收到请求后都需要到keystone上去验证auth-token是否有效。

12. nova-compute使用验证后获取的auth-token发起一个REST调用给glance-api获取镜像。然后nova-compute使用使用镜像ID。从镜像服务中得到Image URI。从（image storage）镜像存储中加载镜像。
13. glance-api去Keystone上验证auth-token是否有效。如果有效，nova-compute就可以获取镜像的元数据metadata。
14. nova-compute使用验证后获取的auth-token执行一个REST调用给neutron-server，让neutron-server给分配和配置网络，为实例分配IP地址。
15. neutron-Server去Keystone验证auth-token是否有效。如果有效，nova-compute就可以获取到网络的相关信息。
16. nova-compute使用验证后获取的auth-token执行一个REST调用给cinder-api，给实例附加卷存储，也就是云硬盘。
17. cinder-api去Keystone验证auth-token是否有效，如果有效，那么nova-compute就可以获取到块存储的相关信息。

#### 第四阶段：执行创建

在第三阶段，nova-compute已经通过Glance、Neutron和Cinder分别获取到了镜像、网络和存储相关的信息。那么在第四阶段nova-compute就开始创建虚拟机了。

18. nova-compute为hypervisor的驱动生成数据，并且通过libvirt或者其他API让hypervisor执行请求来创建虚拟机。这样虚拟机创建的交互流程基本结束，剩下的步骤就是hypervisor最终创建虚拟机的流程。

然后nova-api去轮训nova  database，查看虚拟机的状态是否变成正确创建虚拟机的状态(Active,none,sunning)，若状态正确，则虚拟机创建正常成功。

![](http://blogdata.zhaolibin.com/FpnitAuSz-XCRD8bJvB1mxr__ND-)