# MPLS

## mpls vpn

### vpn简介

#### virtual personal network分类

##### 安装层次来分(osi分类)
###### 二层 virtual personal network
传统二层virtual personal network (基本上不用了)
FR(帧中继/淘汰)  ATM(淘汰)

multi protocol Label switch 二层vpn
vpws ----virtual private wan service   虚拟私有广域服务 链接方式p2p 将mpls的domian 看成多条线缆
vpls ---- virtual private lan sevice      share 共享网络  将mpls domain 大型交换机

###### 三层virtual personal network

###### GRE general route encapsulation 通用路由封装

封装的格式 source ip  | gre | 隧道ip
优点:
可以运行路由协议,站点互通
假设运行ospf 的情况,ospf的hellow包可以进行类似下面格式的封装
Ospf hello | 接口ip 224.0.0.5 | GRE | 隧道sip，隧道dip
缺点:
安全性差

###### ip ssec
具体在后面讲解
优点:安全可靠,可以进行验证
缺点:不能运行路由协议

###### GRE OVER IP SEC
结合上面两种开发的的模式
##### 按照客户和运行商的关系分类
###### overlay vpn:覆盖性vpn
多用于中小型企业,soho,对于网络的要求不高
**将运行商看做一条链路,想GRE和IPEC都是overlap类型的vpn**
优点:运营商不参与客户路由成本低
缺点:因为运营商不参与,所以在运营商走的可能是次优路径

###### peer to peer vpn 点到点的vpn
用于大型网络,对于网络质量有要求
典型例子:BGP

术语： 
	P：provider，运营商核心设备 
	C：customer，客户设备，归客户管理 
	PE：provider edge：运营商边界设备，对接CE 独享
	PE：一个PE连接一个CE 优点：稳定可靠 缺点：成本高 
	共享PE：一个PE服务多个CE 优点：成本低 缺点：一旦CE太多，导致资源消耗过高
	 CE：customer edge：客户边界设备，属于运营商 传统三层vpn，通过bgp来实现，路由控制和过滤，策略很复杂

###### overlay vpn 和 peer to peer vpn的优缺点
overlay vpn由于将运营商看做链路没运营商不参与,所欲会存在次优路径问题
peer to peer 由于运营商参与用户的路由,所以两个要分离
peer to peer 中不同的客户路由需要分离开不能冲突
peer to peer 中独立pe provide 和共享pe 互斥
peer to peer 中的ce client edger 需要进行复杂的bgp配置

###### [[MPLS VPN]]

