title: Vagrant 使用技巧
date: 2013-12-11 10:11:20
tags:
    - 虚拟化
    - Vagrant
---

__上帝借『墙』一双慧眼吧，Vagrant的文档真的做得很好__

这里记录这两天使用Vagrant的一些经验。Vagrant的命令非常简单，不赘述。虚拟机重要的配置都在文件`Vagrantfile`中。

__首先要注意__，`Vagrantfile`开头就有指出API的版本：
```
VAGRANTFILE_API_VERSION = "2"
```

<!--more-->

所以我们应该查阅Version2[文档](http://docs.vagrantup.com/v2/)，网络上搜出来好多都是`V1`的文档，不要搞错了。


### 网络配置

以下是Vagrantfile的配置例子，其实已经非常清楚了
```
  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network :forwarded_port, guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network :private_network, ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network :public_network
```  

* __端口转发__ `config.vm.network :forwarded_port, guest: 80, host: 8080`，配置后访问本地的8080就相当于访问虚拟机的80。
* __直接配置私有地址__ `config.vm.network :private_network, ip: "192.168.33.10"`，配置后`$ssh 192.168.33.10`访问虚拟机。
* __桥模式__ 使用外部DHCP，获取外部的网络的IP地址`config.vm.network :public_network`，如果有多个网络接口，就需要在后面指定清楚使用那个接口出去，多写一个参数`:public_network,:bridged=>'eth0'`。不写清楚启动的时候vagrant就会问”你使用那个做桥啊？1还是2“。_使用桥模式那虚拟机获取的ip是多少呢？_，使用`vagrant ssh`可以ssh到虚拟机，然后就`ifconfig`自己看了哦。

### 共享目录

```
  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  config.vm.synced_folder "../data", "/vagrant_data"
```

可以搞一个目录来和虚拟机同步文件。使用 `config.vm.synced_folder`,第一个参数是环境的目录，第二个参数是在虚拟机中mount的所在位置。 

### 快照

Virtualbox支持Snapshot，而原生的Vagrant没有快照方面的功能。好在Vagrant支持plugin，所以有人就为Vagrant开发了[snapshot插件](https://github.com/dergachev/vagrant-vbox-snapshot)。





