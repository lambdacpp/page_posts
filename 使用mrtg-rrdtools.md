title: 使用MRTG-RRDTools
date: 2014-01-26 10:50:00
tags: 
    - 网络管理 
    - Linux Tips
---

__东西好，坑太多，记之!__

很早很早以前用MRTG，后来赶时髦改成Cacti。我们这个场景监控2台设备，Cacti一点不好。这几天网络大升级，顺便将监控改回去。

<!--more-->


### 打开交换机SNMP

####  创建一个SNMP使用的ACL

```
[Quidway-7703] acl number 2050
[Quidway-7703-acl-baisc-2050] rule 1 permit source 192.168.1.100 0
[Quidway-7703-acl-baisc-2050] rule 2 permit source 192.168.100.3 0
[Quidway-7703-acl-basic-2050] rule 10 deny source any
[Quidway-7703-acl-baisc-2050] quit
```

####   打开交换机的SNMP

```
[Quidway-7703]snmp-agent community read public acl 2050
```

#### 客户端测试

```
#snmpwalk -v 2c -c public 192.168.190.1
```
 
### mrtg 

#### 安装
```
aptitude install mrtg mrtgutils mrtg-rrd
```

#### 生成mrtg.cfg文件

mrtg.cfg通过命令cfgmaker生成。cfgmaker的使用可以参考[这个文档](http://people.binf.ku.dk/~hanne/technotes/mrtg/)，非常好！

```
#cfgmaker \
    --global 'HtmlDir: /var/www/mrtg/html' \
    --global 'Imagedir: /var/www/mrtg/img' \
    --global 'Logdir: /var/www/mrtg/log' \
    --global 'IconDir: /mrtg' \
    --global 'LogFormat: rrdtool' \
    --global 'Options[_]: growright,bits' \
    --ifdesc=descr   \
    --snmp-options=:::::2    \
    --output /etc/mrtg.190.1.cfg \
     public@192.168.190.1 
```

说明一下：

1. 其中public是设备的community名
2. 参数`--global 'LogFormat: rrdtool'`是告诉mrtg使用rrdtool格式来记录日志
3. `--snmp-options=:::::2` 配置snmp的参数，__这个配置的第一个坑__。前面配置设备的snmp服务的时候，使用`snmpwalk`的时候，我们都采用了`2c`这个版本号。但如果你在cfgmaker的snmp_option中使用了`2c`那运行mrtg的时候将会出现类似如下的错误：
```
Target[172.16.0.68_1][_IN_] ' $target->[0]{$mode} c' (warn): (Missing operator before c?)
```
__mrtg只认数字的版本号，1可以，2可以，3可以，2c不可以__。

#### 美化 mrtg.cfg 文件

`mrtg.cfg`也就是个模板文件，所以你希望加什么，怎么汉化直接编辑即可。mrtg通过snmp探测到的端口都会出现在cfg文件中，但如果端口没有Up或者被手动Down掉的，cfgmaker会将其注释掉，并在注释中添加：

```
### * it is operationally DOWN
```
__这里是第二个坑__，后面会用 `indexmaker` 生成展示页面，如果端口有这样的注释，`indexmaker`是不会将端口加到页面中的，哪怕端口的相关配置没有注释。

### 定时采集数据

`/etc/cron.d/mrtg`中已经配置每5分钟采集一次数据。如果没有问题，可以在`/var/www/mrtg/log`下面看见rrd格式的日志文件，但是，`/var/www/mrtg/img/`下面什么也没有。MRTG不会去生成图像，那是RRDTools的事情。 

### 使用RRDTool

`RRDTool`是MRTG的升级版，其实RRDTools只是一个将日志数据转换为图像的工具。使用MRTG-RRDTools与MRTG的不同地方表现在：
1. MRTG一般采集了数据就将图像生成放在那儿，而MRTG-RRDTools只是将数据采集回来放在RRDTools格式的文件中。
2. MRTG展示图像是通过Web服务器的静态页面方式，而MRTG-RRDTools需要通过CGI程序在用户需要的时候生成图像。

所以，在MRTG的[官方网站](http://oss.oetiker.ch/mrtg/doc/mrtg-rrd.en.html)上提供了3种不同的CGI实现。我们使用第三种 [mrtg-rrd](http://www.fi.muni.cz/~kas/mrtg-rrd/)。使用非常简单，只要保证安装了`mrtg-rrd`，就可以运行了，关键是通过`indexmaker`生成正确的链接，另外应该注意配置`/etc/mrtg-rrd.conf`，这个文件中列出了使用`mrtg-rrd`展示的mrtg监控的配置文件。举个例子，如果有两台设备，mrtg监控的配置分别是`/etc/mrtg.cfg`和`/etc/mrtg2.cfg`，那就应该将2个文件都配置到`/etc/mrtg-rrd.conf`中。



#### indexmaker 的坑真多

使用方法：
```
# indexmaker --rrdviewer=/cgi-bin/mrtg-rrd.cgi --output /var/www/mrtg/index.html /etc/mrtg.cfg

```
但有问题，刚才提到的那个[参考](http://people.binf.ku.dk/~hanne/technotes/mrtg/)上提到了要按[这个操作](http://people.binf.ku.dk/~hanne/technotes/mrtg/indexmaker-hack.txt)hack一下这个perl程序。Hack的内容就是要修改一下模板。修改后 indexmaker就可以工作了，生成了index文件，通过Apache打开，那些图像就开始实时的生成了。

#### 下面是Hacker的内容

```
# Approx line 471 in /usr/bin/indexmaker

                    if ( !($picfirstloop^$$opt{picfirst}) ) {
                        # figure show name for rrd viewer
                            if ($$cfg{logformat} eq 'rrdtool') {
               my $sep = $$opt{rrdviewer} =~ /\?/ ? '&amp;' : '?';
               $index .= "<A HREF=\"$$opt{rrdviewer}/$item.html\">".
               "<IMG BORDER=$$opt{imgborder} ALT=\"$item Traffic Graph\" ".
               "SRC=\"$$opt{rrdviewer}/$item-day.png\"></A>"
# <orig>
#               $index .= "<A
#               HREF=\"$$opt{rrdviewer}".$sep."log=$item&amp;cfg=$$cfgfile{$item}\">".
#                                         "<IMG BORDER=$$opt{imgborder}
#                                         ALT=\"$item Traffic Graph\" ".
# "SRC=\"$$opt{rrdviewer}".$sep."log=$item&amp;cfg=$$cfgfile{$item}&amp;png=$$opt{show}.".
#                                         "s&amp;small=1\"></A>"
# </orig>
                            } else {

```

但还有问题：

- perl的SNMP包有不规范的地方，运行时会出一堆Warning。

其实也不影响使用。如果有洁癖，可以按[这个提示](http://mark.orbum.net/2013/06/07/fix-for-mrtg-generating-snmp_session-error-in-debian-wheezy-and-possibly-ubuntu/)，修改一下Perl的包。具体方法就是将149行和609行的：
```
import Socket6;
```
改为：
```
Socket6->import(qw(inet_pton getaddrinfo));
```

- 生成的网页`charset=iso88591-15`，不支持中文，需要手动修改index.html。

至于中文嘛，已经Hack indexmaker了，那继续就是了。后面是Patch文件。包括修改了题目、修改了缺省的cgi方式（运行的时候可以不要rrdviewer参数了）加入了一些css。

```
91c91
< 	       title => 'MRTG Index Page',
---
> 	       title => '我的监控网页',
97c97
< 	       rrdviewer => '/cgi-bin/14all.cgi',
---
> 	       rrdviewer => '/cgi-bin/mrtg-rrd.cgi',
100,102c100,102
< 	       boldon => '<B>',
< 	       boldoff => '</B>',
< 	       div => 'DIV',
---
> 	       boldon => '<b>',
> 	       boldoff => '</b>',
> 	       div => 'div',
107,108c107,108
<     $opt{headon} = "<H$opt{headlevel}>";
<     $opt{headoff} = "</H$opt{headlevel}>";
---
>     $opt{headon} = "<h$opt{headlevel}>";
>     $opt{headoff} = "</h$opt{headlevel}>";
313,315c313,318
< <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
< <HTML>
< <HEAD>
---
> <!DOCTYPE html>
> <html lang="zh-CN" >
> <head>
>     <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
> 
> 
317,325d319
<     <!-- Command line is easier to read using "View Page Properties" of your browser -->
<     <!-- But not all browsers show that information. :-(                             -->
<     <meta http-equiv="content-type" content="text/html; charset=iso-8859-15" >
<     <META NAME="Command-Line" CONTENT="$metaCmdLine" >
<     <META HTTP-EQUIV="Refresh" CONTENT="$refresh" >
<     <META HTTP-EQUIV="Cache-Control" content="no-cache" >
<     <META HTTP-EQUIV="Pragma" CONTENT="no-cache" >
<     <META HTTP-EQUIV="Expires" CONTENT="$expiration" >
<     <LINK HREF="${gifPath}favicon.ico" rel="shortcut icon" >
329a324
> 
331,332c326,348
< /* commandline was: $argz */
< /* sorry, no style, just abusing this to place the commandline and pass validation */
---
> body {
>     background-color: #EFEFEF;
>     color: #000000;
>     font-family: sans-serif;
>     font-size: 12px;
>   }
>   
>   h1 {
>     background-color: #CCCCCC;
>     font-family: sans-serif;
>     font-size: 18px;
>     font-weight: bold;
>   }
>   
>   h2 {
>     font-family: sans-serif;
>     font-size: 16px;
>     font-weight: bold;
>   }
>   
>   a {
>     text-decoration: none;
>   }
333a350,351
> 
> 
481,484c499,506
<                $index .= "<A HREF=\"$$opt{rrdviewer}".$sep."log=$item&amp;cfg=$$cfgfile{$item}\">".
< 		                          "<IMG BORDER=$$opt{imgborder} ALT=\"$item Traffic Graph\" ".
<  "SRC=\"$$opt{rrdviewer}".$sep."log=$item&amp;cfg=$$cfgfile{$item}&amp;png=$$opt{show}.".
< 					  "s&amp;small=1\"></A>"
---
>                $index .= "<A HREF=\"$$opt{rrdviewer}/$item.html\">".
>                "<IMG BORDER=$$opt{imgborder} ALT=\"$item Traffic Graph\" ".
>                "SRC=\"$$opt{rrdviewer}/$item-day.png\"></A>"
> 
>  #               $index .= "<A HREF=\"$$opt{rrdviewer}".$sep."log=$item&amp;cfg=$$cfgfile{$item}\">".
>  #    	                          "<IMG BORDER=$$opt{imgborder} ALT=\"$item Traffic Graph\" ".
>  # "SRC=\"$$opt{rrdviewer}".$sep."log=$item&amp;cfg=$$cfgfile{$item}&amp;png=$$opt{show}.".
>  #    				  "s&amp;small=1\"></A>"
```

