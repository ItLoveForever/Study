## 横向虚拟化技术
堆叠
m-lag
有个技术已经被淘汰现在使用NVoc

### NVO3 network virtualization over layer 3
扩展性极其高
network virtuallization 网络虚拟化
虚拟化交换机---- BD bridge domain (桥接域)
虚拟化路由器 --- vrf (vpn-instance)
虚拟化防火墙---- FW 通过防火墙虚拟
虚拟化均衡设备 --- LB通过服务器虚拟
针对某个业务进行虚拟化

network virtualization overlayer 3 在原有的网络基础上叠加虚拟化各种设备

#### 叠加网络
顶层:overlayernv
底层underlay,三层ip网络

在底层underlay基础上在overlayer上进行虚拟扩张
**所以底层网络要求很高,高可用,并发,吞吐,扩展**

___
#### 底层网络underlayer

##### 一些术语
tor top of rack 用户使用的交换机设备机架中最上面的设备
mor middle or rack 顾名思义中间的设备
leaf 用于接入网络层的核心,数据中心交换机
fabric 织物 三层和leaf链接的路线叫做织物
fabric可以不断扩展用于保证织物的负载高可用
用来构建支付fabric的路由器设备叫做spine 脊
同样spine上层可以接入一个leaf这个leaf是service leaf
service 可以惊醒虚拟FW,LB等等
也可以接入leaf border来介入外网
**这种架构叫做clos胖树**
fabric织物中间full mesh 全互联
![[胖树.png]]