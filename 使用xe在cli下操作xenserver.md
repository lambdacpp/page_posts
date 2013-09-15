title: 使用xe在CLI下操作XenServer
date: 2013-09-15 22:03:58
tags: 虚拟化 XenServer 
---

XenServer 的管理通常都是在Windows下的XenCenter中完成，如果希望直接在服务器上操作虚拟机就需要熟悉xe这个命令。xe非常强大，大部分XenServer的管理都可以使用它来完成，通过XenServer的Shell下的xe补全功能可以大大提高工作效率。


### 先了解一些名词

* **pool**	一堆安装了XenServer的物理服务器可以构成一个Pool
* **vm**	虚拟机 
* **host**	物理机 
* **network**	一个虚拟网络
* **pif**	一个物理网络接口 (通过vSwitch划分创建的VLAN接口算pif)
* **vif**	一个虚拟网络接口
* **sr**	一个物理存储资源，如一个SAN、一个NFS空间等 
* **vdi**	一个虚拟机使用的虚拟磁盘
* **vbd**	一个虚拟的块设备
* **pbd**	通过物理主机访问的一个物理块设备


### 虚拟机基本操作 

##### 列出所有的虚拟机 vm-list

> [root@xenserver-m ~]# xe vm-list 
> uuid ( RO)           : 33633a0b-5ee0-6be6-96d4-49292fa19083
>      name-label ( RW): tempserver
>     power-state ( RO): running
> 
> uuid ( RO)           : bb9486bf-14a4-2254-5c0f-f9327d5e9761
>      name-label ( RW): Mail
>     power-state ( RO): running

##### 开机 

> vm-start uuid=[uuid] 
> vm-start vm=[vmname]

##### 关机 

> vm-shutdown uuid=[uuid] 
> vm-shutdown vm=[vmname]


### SR相关操作

##### 显示一个虚拟机的vbd(virtual block device) 

> vbd-list vm-uuid=[uuid]

### 快照相关操作

##### 创建快照

> xe vm-snapshot uuid=[要创建快照的虚拟机的uuid] new-name-label=snapshotname

##### 将快照导出存储为文件

> xe template-param-set is-a-template=false ha-always-run=false uuid=[snapshot-uuid]
> xe vm-export vm=[snapshot-uuid] filename=filename.xva

##### 删除快照

> xe vm-uninstall uuid=[snapshot-uuid]

##### 恢复快照

> xe snapshot-revert uuid=[snapshot-uuid]

##### 列出所有快照

> xe snapshot-list

##### 删除对应快照的命令：

> xe snapshot-uninstall snapshot-uuid=[snapshot-uuid]


