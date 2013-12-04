title: 创建自己的Debian Vagrant Box
date: 2013-12-03 09:44:48
tags: 虚拟化
---

___『懒惰是程序员的美德』--Larry Wall ___


云计算管理离不开自动化部署，于是Puppet火了。Chef、Saltstack、Ansible跟着也火，做好配置脚本，真正的一键部署。但攻城师们懒惰已经无法救药了，他们连自动化部署本身也不想重复了，于是Docker、Vagrant横空出世。自动化部署变成一种云服务的时候，那些七七八八的部署过程放到了一种叫Box的东西里面了，随需随取，丰简由人。

最近想用Vagrant做测试，但访问Vagrant太慢，而且各路Boxes中的Box不能真正满足自己需要，所以决定自己做一个放服务器上。基本参考[这篇](https://mikegriffin.ie/blog/20130418-creating-a-debian-wheezy-base-box-for-vagrant/)

<!--more-->

## 1.安装VirtualBox和Vagrant

官网下载VirtualBox deb包安装，需要注意的是配置`/etc/init.d/vboxdrv setup` Debian需要安装`dkms`和Linux的源码。不过只需要apt `dkms` 就行了， 相关的东西会自动安装
```
[root@desktop ~]# aptitude install dkms
下列“新”软件包将被安装。         
  dkms fakeroot{a} linux-headers-3.11-2-686-pae{a} 
  linux-headers-3.11-2-common{a} linux-headers-686-pae{a} 
  linux-kbuild-3.11{a} 
```

## 2.使用Debian安装盘创建一个虚拟机

1. 创建虚拟机取名wheezy64
2. 关闭虚拟机的Audio和USB
3. 将netinst ISO 镜像挂载到虚拟机
4. 运行虚拟机，开始安装
5. 机器名 vagrant-debian-wheezy 
6. root口令(vagrant)
7. 创建用户vagrant，口令vagrant

## 3.登录虚拟机进行配置

以root用户登录进行如下操作:

### 安装一下需要的软件

```
aptitude install python python-apt python-pycurl tmux
aptitude install build-essential module-assistant
m-a prepare
aptitude install openssh-server zerofree sudo
mkdir /home/vagrant/.ssh
cd /home/vagrant/.ssh
wget https://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub
mv vagrant.pub authorized_keys
chmod 700 /home/vagrant/.ssh
chown -R vagrant:vagrant /home/vagrant/.ssh
```

### 安装VirtualBox guest additions

1. VirtualBox Devices菜单中选择`Install Guest Additions`
2. 在虚拟机中mount光驱
3. `sh /media/cdrom/VBoxLinuxAdditions.run`

### 配置vagrant用户

```
groupadd admin
usermod -G admin vagrant
```
添加如下内容到`/etc/sudoers`

```
vagrant    ALL = NOPASSWD: ALL
```

### 清除不需要的文件
```
aptitude clean
rm -rf /usr/share/doc
rm -rf /usr/src/vboxguest*
rm -rf /usr/src/virtualbox-ose-guest*
find /var/cache -type f -exec rm -rf {} \;
rm -rf /usr/share/locale/{af,am,ar,as,ast,az,bal,be,bg,bn,bn_IN,br,bs,byn,ca,cr,cs,csb,cy,da,de,de_AT,dz,el,en_AU,en_CA,eo,es,et,et_EE,eu,fa,fi,fo,fr,fur,ga,gez,gl,gu,haw,he,hi,hr,hu,hy,id,is,it,ja,ka,kk,km,kn,ko,kok,ku,ky,lg,lt,lv,mg,mi,mk,ml,mn,mr,ms,mt,nb,ne,nl,nn,no,nso,oc,or,pa,pl,ps,pt,pt_BR,qu,ro,ru,rw,si,sk,sl,so,sq,sr,sr*latin,sv,sw,ta,te,th,ti,tig,tk,tl,tr,tt,ur,urd,ve,vi,wa,wal,wo,xh,zh,zh_HK,zh_CN,zh_TW,zu}
```  
### 收缩box的尺寸

```
init 1

mount -o remount,ro /dev/sda1
zerofree /dev/sda1
```

## 4.创建box

```
$vagrant package --base wheezy64
[wheezy64] Clearing any previously set forwarded ports...
[wheezy64] Creating temporary directory for export...
[wheezy64] Exporting VM...
[wheezy64] Compressing package to: /Users/hehe/package.box
```  

## 5. 使用新创建的box
```
vagrant box add MyDebianBox package.box
```






