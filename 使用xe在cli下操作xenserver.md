title: 使用xe在CLI下操作XenServer
date: 2013-09-15 22:03:58
tags: 
    - XenServer 
    - 虚拟化
---

XenServer 的管理通常都是在Windows下的XenCenter中完成，如果希望直接在服务器上操作虚拟机就需要熟悉xe这个命令。xe非常强大，大部分XenServer的管理都可以使用它来完成，通过XenServer的Shell下的xe补全功能可以大大提高工作效率。


## 先了解一些名词

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

<!--more-->



## 虚拟机基本操作 

#### 列出所有的虚拟机 vm-list
```
[root@xenserver-m ~]# xe vm-list 
uuid ( RO)           : 33633a0b-5ee0-6be6-96d4-49292fa19083
     name-label ( RW): tempserver
    power-state ( RO): running

uuid ( RO)           : bb9486bf-14a4-2254-5c0f-f9327d5e9761
     name-label ( RW): Mail
    power-state ( RO): running
```
#### 开机 
```
vm-start uuid=[uuid] 
vm-start vm=[vmname]
```
#### 关机 

```
vm-shutdown uuid=[uuid] 
vm-shutdown vm=[vmname]
```


## sr相关操作

### vdi vbd pbd sr之间的关系

vbd和pbd从主机的角度来看的都是块设备，只不过一个是物理真实的一个是虚拟的；一个从物理主机角度看一个从虚拟机角度看。sr是从XenServer管理存储的需要提出的概念，通常一个主机的物理块设备pbd对应了一个sr。vdi简单理解就是在sr上的磁盘分区，通过vbd交给虚拟机使用。

#### 实践：手动为虚拟机创建一个磁盘

* 创建一个vdi。vdi总是从属于某个sr下,命令将返回新vdi的uuid

```
xe vdi-create name-label=new-disk type=user sr-uuid=[sr的uuid] virtual-size=10GiB 
```

* 为虚拟机创建vbd。vbd总是绑定在某个vdi上，从属于某个虚拟机，device指出是虚拟机的第几个块设备，返回vbd的uuid

```
xe vbd-create vdi-uuid=[刚才创建的vdi的uuid] device=2 vm-uuid=[虚拟机的uuid]
```

* 将vbd在虚拟机中生效(plug)

```
xe vbd-plug uuid=[刚才创建的vbd的uuid]
```


#### 显示一个虚拟机的vbd(virtual block device) 

```
vbd-list vm-uuid=[uuid]
```

## 网络接口相关操作

### network pif vif 之间的关系

#### 实践：手动为虚拟机创建一个网络连接

```
[root@XenServer1 ~]# xe vif-list vm-name-label=mysql-slave53 
uuid ( RO)            : d4db8d3f-6544-047e-f96a-f33b1b14ac3e
         vm-uuid ( RO): 5d177e6e-99fb-d2e3-a326-cdf90c09c1a7
          device ( RO): 2
    network-uuid ( RO): 03d63aca-0e81-bb11-11ac-fa259f661d98

[root@XenServer1 ~]# xe vif-unplug uuid=d4db8d3f-6544-047e-f96a-f33b1b14ac3e
[root@XenServer1 ~]# xe vif-destroy uuid=d4db8d3f-6544-047e-f96a-f33b1b14ac3e
[root@XenServer1 ~]# xe vif-create vm-uuid=5d177e6e-99fb-d2e3-a326-cdf90c09c1a7 network-uuid=0e3cac31-2f2b-589d-7f62-b6ae55de2a93 mac=random device=0 
96a04e4b-fce5-7632-3ed9-54446a95c213
[root@XenServer1 ~]# xe vif-plug uuid=96a04e4b-fce5-7632-3ed9-54446a95c213
```



## 快照相关操作

#### 创建快照

```
xe vm-snapshot uuid=[要创建快照的虚拟机的uuid] new-name-label=snapshotname
```

#### 将快照导出存储为文件

```
xe template-param-set is-a-template=false ha-always-run=false uuid=[snapshot-uuid]
xe vm-export vm=[snapshot-uuid] filename=filename.xva
```

#### 删除快照

```
xe vm-uninstall uuid=[snapshot-uuid]
```

#### 恢复快照

```
xe snapshot-revert uuid=[snapshot-uuid]
```

#### 列出所有快照

```
xe snapshot-list
```

#### 删除对应快照的命令：

```
xe snapshot-uninstall snapshot-uuid=[snapshot-uuid]
```

## 使用VNC连接到虚拟机的控制台 

参考这篇[文章](http://blogs.citrix.com/2011/02/18/using-vnc-to-connect-to-a-xenserver-vms-console/)

每台虚拟机开机后都会在本地宿主机上创建一个VNC stream，可通过本地的一个VNC端口访问，所以我们需要做的事情就是找到在哪台宿主机上的哪个端口，然后通过ssh建立一个隧道就可以打开VNC stream了。

#### 确定虚拟机的宿主机和在宿主机上的编号

```
[root@XenServer1 ~]#  xe vm-list params=dom-id,resident-on name-label=mysql-slave53 
resident-on ( RO)    : 8f69621a-ef3e-41b5-a5f8-e938976f787e
         dom-id ( RO): 98
```

#### 根据宿主机的uuid获取宿主机的名称

```
[root@XenServer1 ~]# xe host-list uuid=8f69621a-ef3e-41b5-a5f8-e938976f787e
uuid ( RO)                : 8f69621a-ef3e-41b5-a5f8-e938976f787e
          name-label ( RW): XenServer1
    name-description ( RW): Default install of XenServer
```


#### 根据宿主机的uuid获取宿主机的管理IP

```
[root@XenServer1 ~]# xe pif-list management=true params=IP host-uuid=8f69621a-ef3e-41b5-a5f8-e938976f787e
IP ( RO)    : 192.168.189.10
```

#### 登录到相映的宿主机读取vnc的端口号

这个需要使用xenstore-read访问XenStore。至于XenStore可以参考[这里](http://blog.csdn.net/zhengtingt108/article/details/5409820)

```
[root@XenServer1 ~]# xenstore-read /local/domain/98/console/vnc-port
5924
```

#### VNC登录

现在我们已经清楚了所有的已知，宿主机的管理IP以及VNC端口。在客户端创建一个ssh隧道，通过vncviewer即可连接上VNC:

```
[my@client ~]$ ssh -L 5924:localhost:5924 root@192.168.189.10
[my@client ~]$ vncviewer localhost:5924
```

### 直接用脚本吧

写了[shell脚本](https://gist.github.com/lambdacpp/247f4adef98873bcc4ab)来找端口。





