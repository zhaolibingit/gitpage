---
layout: post
title:  openstack 中快照与备份
date:   2018-04-22 13:31:01 +0800
categories: openstack
tag: openstack
---

* content
{:toc}




## openstack 中快照与备份

### 快照作用

	快照保存的是瞬时数据，依赖于源数据，主要用于做重要升级之前先做个snapshot，以备出现差错回滚。备份主要是为了容灾，不依赖于源数据，且一般备份数据在不同数据中心。
	备份作为一种常见的容灾机制，是系统设计中不可或缺的一环

### nova 相关的快照/备份机制

#### nova snapshot
	严格来说，该快照并不是真正意义的快照，更像是创建一个镜像，所以命令为image-create.它并没有使用virt提供的virsh snapshot-create来创建snapshot，仅仅将虚拟机的系统磁盘文件完整的复制一份，然后把复制的文件上传至 glance 中作为镜像，之后二者毫无关联，类似于全量快照。
- Libvirt snapshot: 采用 virDomainSnapshotCreateXML() (CLI为virsh snapshot-create) 创建快照，新建的快照与虚拟机有关联：若为内置快照，新建的快照和虚拟机磁盘文件存在同一个 qcow2 文件中；若为外置快照，快照为一个新建的 qcow2 文件，原虚拟机的磁盘文件将变为 read-only 模板，快照文件记录与模板的差异数据。它包含快照链信息，也保留了 RAM 信息，可回滚至快照点。
- Nova snapshot: nova 并未采用 virDomainSnapshotCreateXML() 实现快照，它仅仅将虚拟机磁盘文件完整拷贝一份并上传至 glance 中。这种快照不包含快照链和 RAM，只保留了磁盘文件信息，无法回滚至快照点。

因 nova 的 snapshot 只是对虚拟机磁盘文件的完整拷贝，所以和真正意义的 snapshot 相比，存在以下缺点：
- 没有快照链信息，不支持虚拟机回滚至某个快照点
- 只对系统盘进行快照，不支持内存快照
- 不支持同时对虚拟机和磁盘做快照
- 需要用户进行一致性操作
- 不支持含元数据导出，不支持含元数据导入
- 只支持虚拟机全量数据快照（与快照的实现方式有关，因为是通过image进行保存的），过程较长（需要先通过存储快照，然后抽取并上传至glance)，快照以Image方式保存，而非以cinder卷方式保存，无法充分利用存储本身能力加快快照的创建和使用

####Cold Snapshot & Live Snapshot
受磁盘文件格式以及 libvirt、qemu 版本的影响，nova 的 snapshot 分两种，cold snapshot 和 live snapshot，本文用命令行表示对应的步骤，：
- Cold snapshot: 创建 snapshot 时，需先关闭虚拟机，快照完成后再启动虚拟机
- Live snapshot: 创建 snapshot 时，无需关闭虚拟机

   nova 中的快照支持冷快照和live snapshot,两者的命令都是 nova image-create。当满足QEMU 1.3+ and libvirt 1.0+版本要求时会使用live snapshot,否则使用冷快照。


##### Cold Snapshot 流程:
- 1.卸载 pci 设备，保持虚拟机状态并关机。
```shell
    Python:
    virt_dom.managedSave(0)
    Cli:
    $ virsh managedsave vm
```
- 2.复制虚拟机的磁盘文件, 然后把复制的文件上传至 glance 中。
```shell
$ qemu-img convert -f qcow2 vm.qcow2 -O qcow2 vm_snapshot.qcow2
$ glance image-update vm_snapshot --file vm_snapshot.qcow2
```
- 3.启动虚拟机，挂载 pci 设备
```shell
$ virsh start vm
```
##### Live Snapshot 流程
- 1.暂停所有的 block job:

```shell
Python
virt_dom.blockJobAbort(disk_path, 0)

Cli
$ virsh blockjob vm vda --abort
```

- 2.新建一个带有 backing file 的 vm_delta.qcow2 文件，大小和虚拟机磁盘大小一致：

```shell
$ qemu-img create -f qcow2 vm_delta.qcow2 --backing_file=base_image --size=root_disk_size
```
- 3.Undefine 虚拟机，以便保持数据一致性，期间虚拟机依旧运行：

```shell
Python
domain.undefine()

Cli
$ virsh undefine vm
```
- 4.把虚拟机磁盘文件复制到新建的 vm_delta.qcow2 文件中：

```shell
Python
domain.blockRebase()

Cli
$ virsh blockcopy --domain vm vda vm_delta.qcow2 --wait --verbose
```
- 5.Define 虚拟机

```shell

Python
_conn.defineXML(xml)

Cli
$ virsh define vm
```
- 6.因 vm_delta.qcow2 只是一个增量文件，所以需把它和 backing file 文件合并成一个 snapshot.qcow2 文件，然后把这个完整的 snapshot.qcow2 文件上传至 glance 中:

```shell
$ qemu-img convert -f qcow2 -O raw vm_delta.qcow2 snapshot.qcow2
$ glance image-update vm_snapshot --file vm_snapshot.qcow2
```

#### 基于 Ceph RBD 的 Snapshot

上述 snapshot 的流程都是基于本地存储，基于 Ceph RBD 的 Snapshot 相对复杂些，下面介绍基于 Ceph RBD 的虚拟机 snapshot 流程，由于 live snapshot 和 cold snapshot 的区别和基于本地存储的区别不大，所以本文着重关注 RBD 层面的流程。

如果是使用ceph作为统一存储(即nova,glance,cinder都将ceph rbd作为默认存储)，那么snapshot可以利用ceph的cow等特性实现更快速的快照，L版本已实现，流程大概如下：
![](http://blogdata.zhaolibin.com/Frludp8KqZoyDqJjnKKceMT6a0D-)
这里需要注意一点，在ceph中rbd clone出来的卷会和被clone卷有一个父子依赖关系，如果你想去除这个依赖关系，可以在图中标注的flatten处做flatten操作，但是flatten的过程是非常慢的，如果你的root disk size比较大，那就有的等了，看自己的需求来取舍吧。
**这样做的坏处是会产生新建镜像对原虚拟机的依赖性，需要定期flatten。**
如果我们做了flatten，那么在原始虚机或者glance快照删除的时候，一般不会有什问题，因为flatten后，他们不存在任何依赖关系。如果没做flatten，那么删除的时候需要考虑，rbd snapshot的protection机制，这个机制会锁定相关的disk，导致不能删除，只能解除依赖去除保护后，才能安全删除，流程看下图。
![](http://blogdata.zhaolibin.com/Fpxl_84V18YlYay0A_mRK5tp76AL)
当然逻辑上肯定还有疏漏的地方，很可能会出现rbd中遗留了大量后缀为 _to_be_deleted_by_glance 的卷没能被删除，此时可以考虑起个定时任务去删除它
- 1.创建快照并设置快照属性为 protected：

```shell
$ rbd snap create nova-pool/vm@nova_vm_snap
$ rbd snap protect nova-pool/vm@nova_vm_snap
```
- 2.Clone 该快照至 glance pool 中：

```shell
$ rbd clone nova-pool/vm@nova_vm_snao glance-pool/glance_vm_snap_clone
```

- 3.Flatten the clone in glance pool：

```shell
$ rbd flatten glance-pool/glance_vm_snap_clone
```
- 4.对 glance pool 的 glance_vm_snap_clone 做快照，并设置其属性为 protected：

```shell
$ rbd snap create glance-pool/glance_vm_snap_clone@vm_snapshot
$ rbd snap protect glance-pool/glance_vm_snap_clone@vm_snapshot
```
- 5.删除 nova pool 的快照：

```shell
$ rbd snap unprotect nova-pool/vm@nova_vm_snap
$ rbd snap rm nova-pool/vm@nova_vm_snap
```

![](http://blogdata.zhaolibin.com/Fm2lfhyClRSN0DcpLNroGpaJMZZv)

##### RBD是什么？
- RBD就是Ceph里的块设备，一个4T的块设备的功能和一个4T的SATA类似，挂载的RBD就可以当磁盘用。
- resizable:这个块可大可小。
- data striped:这个块在ceph里面是被切割成若干小块来保存，不然1PB的块怎么存的下。
- thin-provisioned:精简置备，我认为的很容易被人误解的RBD的一个属性，1TB的集群是能创建无数1PB的块的。说白了就是，块的大小和在ceph中实际占用的大小是没有关系的，甚至，刚创建出来的块是不占空间的，今后用多大空间，才会在ceph中占用多大空间。打个比方就是，你有一个32G的U盘，存了一个2G的电影，那么RBD大小就类似于32G，而2G就相当于在ceph中占用的空间。

