由于堆叠的故障域大,所以目前堆叠只在园区网使用多并且多接入层强耦合

数据中心的接入层:M_LAG是一种二层技术.同样横向虚拟化,只能两台两个框

### LAG和M-LAG的对比
如果两台交换机使用链路链接如果运行stp防环会导致链路利用率低
所以可以通过LAG(Link )来防环冗余
![[Mlag.png]]
**但是存在问题如果sw1坏了就彻底坏了**
那么配置sw冗余
但是
LAG是同一个设备的多条链路聚合,跨设备链路聚合怎么处理?运行m-LAG(多框的链路聚合组multichassis link aggregate group)
**注意mlag是链路级别,mlag是两套的转空管系统,对于下游设备感知是一个,数据同步二层的peer-link链接创建关系,dfs-group 虚拟除一个sw**
![[Mlag的实例图.png]]
链路级别:有两个控制系统,但在链路层面认为是一个,单因两个控制系统会有环路问题,因此mlag有自己的防环机制

## 术语
LAG link aggregate group 链路端口组
M-LAG multichassic link aggregate group 跨框链路聚合组
DFS-group
	定义 动态交换服务组(dynamic fabriv service group) 两台设备接入,一主一备(不同堆叠的主备,后面详细讲解)
		我们会跑一个协议构成框架dfs-group 用来交互配置

	DFS-group id :默认值是1 一般也配置一因为只有一个组
	在DFS中会通过peer-link 交互信令,选举主设备master和被设备DFS的standy
	DFS的主备选举的是DFS交互的master,而外面的控制和数据仍然是双活
	DFS的主备平时美作用.只有当peerlink故障分裂为了防止分裂才会主动挡掉之前的备业务接口
	由于DFS- group的主备并不影响业务,所以可以抢占保持回复后主转发
	主备的交互依靠peer-link 对等体链路.必须是LAG(并且一般建议配置eth-trunk 0) 二层链路默认放行所有vlan

### peer link作用
1.  信令交互,同步表现(arp,mac)
2.  转发当上行down,跨框转发

### 分裂检测DAD(dual-active detect)双主检测链路也加叫做HB(heart beast)心跳,必须用三层链路进行配置

	三层链路可以是mgmt管理口
	物理接口
	loopback 
	建议管理huozheloopback
#### 因为peerlink很重要所以一般peer链路一般都绑定多条链路

### 如果peerlink和DAD都死了证明设备死了

### 一些配置的说明
DSF-group 1 这个是堆叠设备的管理组
DSF-group m-lag xx  将不同设备的lag绑定区分运行mlag的成员接口6

### M-LAG的成员接口
运行mlag的接口会选举出主接口和备接口,并且不抢占
对于端口来讲:谁先up谁就是master

*mlag针对单播流量没有任何影响双活*
**针对三层组播流量,基数目的组ip走主,偶数目的组ip走备**

### mlag的选录根据hash的值计算随机数走哪条
mlag配置举例
### malg的配置需要配置两边并且相同 

---
### mlag的转发
#### 单播转发
单归接入:可能会出现跨设备转发,主要出现在peerlink路径上
双归接入:本地优先尽量少走跨框,所以建议双归接入

---
## BUM(brooadcast unkonw unicast--未知单播的广播)广播的转发

![[mlag数据转发参考图.png]]
当收到bum之后A泛洪所有
水平分割从一个口接受的不会转发回去,b收到x有可能收到x,y,a的产生了复制包并且有可能产生回灌()图示不可能因为水平分割但如果是duolag口的情况就会出现
并且目前x会受到a和b转发的数据

**为了防止x收到a和b的bum所以在mlag中从peerlink收到的数据不会转发bum这叫做mlag的单向防环,对单播无影响他的实现是通过内部的acl规则实现**
单归来不生效

## 不同于lag需要重启恢复设置,mlag240秒同步配置恢复转发,非成员端口会立即进行转发,
peerlink故障malg上面除了
逻辑端口,网管口和peerlink堆叠口之外会直接errordown停止数据转发

---
### vlan转发:vlanif
我们的peerlink默认所有vlan可以通过当vlan100互相访问的时候,从peerlink通过,vlanid在mlag的tag里面封装

### 在带级别wangluo可以通过配置相同mac和通过设置优先级保证root bridge
stp bridge-address 
stp root primary
注意stp会阻塞端口为了保证peerlink正常,所以在mlag下
stp disable
stp或者rstp下
stp v-stp enable