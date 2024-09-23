# ISIS基础概述
## ISIS的产生的背景
###### 最初是iso基于osi(open system interconnection)七层协议clns(connection less network service)服务,但是之后因为tcp/ip的壮大而扩展了,最初是为了自己的无线网络连接协议而开发clnp(connection less network protocol)

## 和ospf的一些基础区别
###### ospf是基于ip协议之上的传输层协议协议号89,***而isis是工作在数据链路层的传输协议***
![[osi和tcp的对比.png]]
在上图中以TCP/IP的协议栈来对比,可以看出

TCP中的IP的协议,对标open system interconnection
的clnp(connection less network protocol)

TCP的ip地址对标OSI(open system interconnection)
clap(connection less address protocol)

ospf 中使用 area id 和router id来表示区域
isis 中使用 NET标识符

### NSAP (network service access point)
相当于tcp协议栈的ip简单对network service access point 的描述
主要可以分为两部分IDP（Initial Domain part）初始化区域部分和DSP（Domain specific Part）设定域部分这两部分
##### IDP（initial domain part）
idp之中两部分组成AFI（authority format identity）认证格式识别和idi（initial domain identity）和出事域识别

##### DSP（domain special part）
指定域部分主要三部分组成High order DSP （分割区域），systemid（相当于ip中的前缀和掩码），sel（服务类型不用管）

## 在实际中配置的是NCE代替ip，可以近似看做nsap（sel全0）

## HCIE的域概念
在ospf中area0是骨干域，而在isis中包含level-2的域是骨干域（level 1和level 1-2）
**注意：在isis中不支持vlinkk所以isis不能够骨干区域分裂**

*当一个路由器的端口既有levle1和level2的话，那么这个路由器是level 1-2*

######  下面这部分是一些小知识点
``` huawei
关于system id,一般来说就是将ip补零凑成每段三位之后移动小数点凑成三段
关于isis的cost的值rip中的缺点是等于调数,isis中默认是10,但是可以调节,ospf是10^^8/贷款 来计算
由于isis属于数据链路层,所以可以忽略ip地址完成邻居的建立和传输
关于lsa level1可以进入level2.但是2进入不了1
```

49.0001.0000.0000.0001.00
这里面idp部分initial domain port 有afi authority format identified 这是企业固定的一般49就行了，idi （initial domain identity ）初始化身份域和Dsp（domain specific port）设定域部分中的high order dsp共同划分域的信息就是.0001，之后system id 对应ip地址就是三段的4位地址，最后.00表示服务类型默认00即可
## isis的基础
#### isis三张表
邻居表：                   disp isis pee
链路状态数据库表    disp isis lsdb 
路由表：                   disp isis route
#### 静默端口
silent-interface ：不能够建立isis的邻居关系
配置方法
端口下：isis silent（前提：isis enable）
int g0/0/0
isis enable
isis silnet

---
#### 和ospf的一些区别点
首先ospf 是传输层89号协议，isis是数据链路层不同于ospf，可以忽略ip建立邻居

相比较ospf，运行商更喜欢isis，因为更稳定平滑，并且isi支持p2p、hdlc和ether，ospf支持vlink
p2mp,nbma等网络