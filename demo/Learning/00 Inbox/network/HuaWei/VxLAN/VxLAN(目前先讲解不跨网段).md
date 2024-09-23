## VN的简单描述
vn
 virtual network 虚拟网路的二层和三层的网络转发功能
 二层:BD(bridge domain)
 三层 : vrf (vpn -instance )
 这里由于跳过之前的两天内容稍微有点断层大致理解
 
 ### 目前讲解的都是用户侧

### vlan和BD之间的关系(BD的意义)
#### 什么是vlan
vlan (virtual lan)
vlan 虚拟区域网,通过vlan将广播域划分,一个vlan相当于一个虚拟设备
#### vlan的不足
因为一个vlan是全局的,并且如果想实现vlan通讯两端设备的vlan 必须相同,并且vlan的数量有限,只有4096个,对于运营商来说根本不够用

传统的二层还存在一个问题,也就是如果多个vlan的情况下可能出现环路,运行stp会产生不能负载的问题.

**为了解决这个问题出现了vxlan**
#### VxLAN

vxlan 通过vap(virtual access point)将vlan和bd(bridge domain)进行绑定

对比传统的vlan,VxLAN的数量扩展到2^32,大约百万个;而且相比vlan更加灵活,将vlan本地化实现vlan的复用

为了解决传统vlan的无法负载和环路的问题将原本二层overlay上升到lager 3网络层,通过隧道实现CE之间的互通,结局传统vlan的一些缺点.**隧道叫做NVE(NV EDGE)**
-  可以将NVE看做两边overlay的物理接口,NVE隧道用来通信的地址叫做VTEP

##### VNI
virtual network identity 是一个全局的设置,他就相当于vlan可以配置2^32的范围,通过vni(virual network identity)将通过vap(virtual access point)进行和bd绑定的vlan和nve隧道绑定
-  前面通过通过vap将bd和vlan进行绑定,之后通过vni(virtual network identity)

##### BD(Bridge Edge)和VAP(virtual access point)用户侧 NVE和VTEP属于网络侧

![[vxlan 基本原理图例.png]]

###### 配置简单流程
需要再数据中心交换机CE重配置
```huawei
bridge-domain 这里随便配置大约百万级别
vxlan vni           (virtual network identity)配置vxlan所用的区别前缀
# 配置虚拟专用端口
# ce交换机设置
进入二层虚拟接口
interface ge x/x/x.xxx mode l2
encapsulation (这里配置对于vlan的处理结果)
如果配置dot 那么就是dot的规则模式去除并打上
如果配置default 不会操作,但是如果是default 那么主端口需要是trunk或者access
并且一旦配置default那么就再也配置不了其他的子接口了
untag和default相同
将此子接口和bd(bridge domain绑定)
bridge-domain
# 配置nte也就是vxlan的隧道
interface nve(network virtual edge 网络虚拟端口)
source 配置虚拟隧道的源端口
将vni和隧道邻居绑定
vni xxx peer 目的nve

```

#### PS vxlan封装在udp当中

