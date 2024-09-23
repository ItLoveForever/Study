## 组播转发和单播转发最大的区别

- 单薄转发会先根据destine ip 查询route inform base 找到目的网段出接口
-  看一看出单薄考虑的是包往哪里走
-  ![[组播单薄对比图.png]]
	- 对此图进行一下分析组播传递的方式
		- 首先末端设备加入的224.1.1.1的组播地址
			- last _ route和末端设备依靠internet-group-multi-protocol 构建souce和出方向等地址表项
			- multi-router根据PIM（Protocol-Independ-MultiCast)构建组播路由列表OIL（out interface list）
			- 数据包根据PIM的出接口发送所有绑定组播地址的设备最终到达接受者段，根据自身IGMP[[Multicast 组播#^IGMP|IGMP详解]]的v2/v3的绑定信息在进行范围传递
		-  问题：产生了复制包和次优路径，可能产生环路
			-  解决方法：
				- 产生原因是因为组地址是未知的，这样的形成表项是不靠谱的所以会有复制，转发等等问题。但是我们的source地址是已知的，下游路径的上游接口是已知的，这**其实就是将单播的unicast反过来，即组播去源的出接口来确定源来组播的如接口这就是RPF（route ）** 
#### 综上所述，组播的RPF基于unicast单播，通过组播地址到单播地址的出接口确定自己的如接口，==所以我们的组播路由有两部分组成，一个是上游接口表一个是下游接口列表，根据PIM（protocol independent multiple）构建==

### RPF详解
#### RPF 依赖单播构建，根据类型分为

#### 配置方法
##### PM （紧类型）
- 工作模式
	- 配置 multicast routing-enable 开启组播功能称为组播路由器
		- 接口下 pim dm
###### pim的报文接口
-  pim的hello报文：
	-  sip接口ip，dip（下游组播地址根据pim形成）
		-  pim option
			- hold time 105 超过这个时间认为你不存在
			-  DR priority 1 ## sm中才有用## MA网络中选举DR使用
			- LAN prune dealy = propgate delate + override = 3
			- state -fresh cap ：60 s 状态刷新时间
	![[dm树构建过程.png]]
###### 1. flood 这里分析一下组播完整的构建机制
-  在接受者段，根据IGMP建立出来出那些末端设备加入了组播构建表项
-  在源段，source设备根据S，G表项发送到multi router，multi router根据RPF检测source能否接受符合接受
-  因为dm无脑进行转发，问题：看下面的设备，没有加入组播也想下面的设备转发给了末端，但是末端的igmp根据general query和report设备表项未加入组播地址不接受（这是flood状态）这里是机制
	- prunne （持续210，接受后重新进行flood一次操作）这是个专有报文
	-  进行修剪掉igmp中不存在的就收者（102秒的修剪状态此期间不转发），如果一个没有就回馈个p消息
-  pm模式所有的接口都是forwarding状态
###### 2. prune 报文
* 根据之前的flood机制问题在于1.1.1.1，他没有加入组播组，但是r2仍然在会已知flood过去，为了解决这个问题通过hellow包中的 prune来解决
	* 当某个last-router设备的末端都没有加入group的情况下会发送prune报文，上游设备收到之后会进行prune修剪操作在一个prune周期102s期间不会再向其发送信息
*  *问题* ：
	*  如果类似上图 2.2.2.2推出了设备怎么办 ？
		*  如果他发送prune报文会导致r3被无关波及，因为prune的报文发送是组播的地址所以r2也会收到，r2收到之后因为自己还存在组，他就会发送lan prune delay信息，R2收到信息之后，就会在默认的时间3s之内不会修剪，同时之前已经推出的设备还是会一直发送prune信息，
		*  问题：会性能浪费并且每个路由表都要维护大量的（* g父项） 和具体的（s，g）子项
###### 3. state refresh 需要手动开启
-  前面可以知道1.1.1.1一直没有加入group，但是r1还是一直会转发，浪费资源，为此可以手动开启 state refresh 这样first回想last route 发送state refresh，这样在组播分布段prune的时间，让R5的校友端口永远不扩散

---
#####  上面的pim的基本机制，下面的两种报文是为了完善 pm 的机制
###### 4. graf ，嫁接
- 在不加入的情况下确实没问题，如果加入了group地址之后怎么办
- 还以上图来举例子，如果r5检测到下游端口接收到了report报文表示在接受者段有设备加入组，这时候r5回向上游接口发送graft报文，要求引入组流量
- 组播分布段的R1收到graft的报文后回复graft-ack的确认，这时候接口状态从prune转换到forwarding

###### 3.  state-fresh
**RPF如果是负载均衡模式会随便选择一个**

###### 4. Assert断言 彻底解决报复之问题
![[Assert断言.png]]
-  对于包复制我们知道对于从上游链路传递过来的我们可以通过RPF 确定那些包接受,那些包不接受但是想上图一样回传过来的没办法避免,对此启用了assert机制
	- 当类似这种下游链路回传的情况会发送assert,这时候会比较asset中的prefer/cost 如果都相同会比较ip,大的胜利
	- pim 的peer 存在两个一个是rpf 的一个是prime 通过assert断言选举出来的,
		- 产生的原因对于rpf来讲如果都接入并且负载分担的或他随机选了一个假设是上面,但是由于assert的断言导致上面精选输掉了,那么这样流量就会走下面

###### RPF选择机制
1. 根据路由最长匹配
2. 路由相同根据传输协议的优先级选择小的
3. 优先级相同选择cost小的
4. 都相同选择IP地址大的

###### 当单播和组播路径不一样的情况
![[单播组播路径不一致情况.png]]

如上图,冗余链路上访被选中单播路径,但是没开启pim,下方开启的pim,这时候因为下方没有单播路径导致RPF失败
-  解决方案配置静态的RFP路由
- ip rpf-route-static  1.1.6.6.0(source address) 吓一跳地址1.1.134.4 
-  display multi rpf 地址
因为组播的默认优先级1小于其他优先级所以有限

AS间路由过多的情况下怎么调整单播和组播路径不一致问题
	一般配置mbgp
默认情况下 mbgp的优先级255大于单播bgp的255
	mstatic>mbgp>urib选择的先后顺序
	也可以开启最长匹配,这样就会有限最长匹配选择路径
	multicast longest-match

---
# 总结
	首先BM生成的是源树shortest path tree 属于flood类型,在进行flood的流量下发时候没有进行任何的检测会先形成共享树,之后进入prune阶段,对对不在组内的接受者段的进行修剪,在防止复制包方面通过RPF控制上游,用assert避免回传,prune阶段对于离开的通过lan-prune 3s中不断否定修剪搞定,对于一致不连接的开启stat-refresh,刷新prune的200s计时器,对于新加入的通过graft嫁接报文回复graft ack确认保持加入,在这个过程中每个设备都存在完整的S,G信息,消耗很大;