# PIM DM [[PIM组播分布树(multicast distribute tree)]]

### MDT Multicast Distribute tree 分类

#### 源树 : shortest path tree 最短路径树
-  最短无环的树(S G)资源消耗巨大,每个设备都有全部的*,G和S,G
#### 共享树RPT :RP tree只维护*,g资源消耗小很多,但是很容易出现次有路径甚至,有一定可能出环路,尤其的RP点

#### 书的建立组播路由协议[[PIM组播分布树(multicast distribute tree)]]
-  密集模式:pim-dm构建源树
	-  问题:资源消耗大,依靠流量触发spt(shorest path tree)建立 pull模式
-  稀疏模式: pim-sm,rpt和spt构建可以切题成spt
	- RPT:维护last-router到RP的,每个路由器维护(*,G)
	- SPT:第一跳rp到达firet-router
		- 这个前提first收到组流量想RP注册
	- SPT-switchover
		- 默认switchover-threshold 0 kps,最波条一旦收到就切换stp
			- 因此在这个基础上可以修改让低俗走组合树,高速走spt
			- 也可以走共享树
## PIM SM 建立过程 
### PIM SM的建立和转发都有RPF检测 
	在之前的PM[[PIM组播分布树(multicast distribute tree)]]的学习时候我们知道,RPF只有在收的时候(转发)会进行RPF检测,这样导致生成的树存在次优路径,自考prune报文进行修剪操作

**不同于,PIM的DM,SM在建立的时候就进行了RPF检测,并且DM建立时候以来单播,SM建立树的时候并不依赖流量**

![[PIM的SM建立过程.png]]
#### 如果探索式,PIM的SM模式同时存在两个树一个是SPT源树,一个是共享树RPT.
	这里以R3为RP分析,首先AR作为接受者段的last-router,和R3建立的是RPT,流量到达R4,进行RPF查询到达RP的peer邻居地址,之后传递*,G(因为源段没有传递过来,这里是组播分布段),再传递给R4,再次重复最后到达RP,收到之后记录到*,G;这样RPF共享树形成,在R1那面前往到RP因为采用的是SM,他没有组播分布树,所以先走的单播发送注册报文到RP,并且由此形成了SPT源表,RP将源信息想RPT部分进行扩散形成伪表,当有数据传递过来的伪表变成了真表
**可以看出在RPT形成的过程中已经进行了RPF检测,但是存在各问题那就是次优路径**类似上图,可以看出前往源走的是5-4-3-2-1,这是次优路径,实际上可以1-5这样走

#### 为此SM可以进行SPT和RPT的切换
```huawei
switchover-threshold 0 kbs
```

通过这个命令可以实现低速走RPT告诉走走SPT,在RPT的S,G是拷贝过来的所以说资源消耗要小很多

#### 靠switchover解决的不是最短无环问题,那么RP成为了瓶颈和故障点
	rp限制了转发能力和一旦孙欢RPT就失灵

### RP (rendezvous point) 汇集点

分类:
-  静态RP
		pim下
			static-rp 地址 这个地址必须可达(route information base可以查询切能ping通)
				可达的原因是因为需要做rpf检测知道想那个接口发送join,上游接口
				对端邻居信息
-  优点:简单
- 缺点:没有备份
- 启动静态RP所有设备都需要配置static-rp
- 查看方法display pim rp-info
---
-  动态RP (可以实现选举RP,冗余,负载)
	- auto -rp思科特有不讲
	-  BSR *共有重点*
		- BSR: bootstrap router 自枚举路由器
			- 角色:
				-  BSR-C : BSR candidate : BS BSR的候选者
				- BSR : BSR-C 通过发送bootstrap报文选举BSR
					- 选举:
					-  1. 优先 proority 默认 0 选大的
					-   2. 选ip大的
		- 配置方式
			- pim
				- c--bsr 如果只在一个设备上面配置的话那么他就会默认他自己是bsr
						- 如果在多个设备都配置了c-bsr会进行选举比较
					-  先比较priority的大小大的优先,优先级的相同再比较接口ip,大的优先
					-  在配置配置了多个c-bsr的情况下竞选失败的c-bsr会成为备选
				-  c-rp (RP candidatee)配置完成之后会想bsr发送单播cp-rp的advertise/announce 包含group信息,rp地址,hodle 时间150秒
				- bsr收到之后会将收到rp信息打包成rp-set发送全网,根据rp-set选择rp
			- rp的选择规则掩码大的
			-  优先级小的
			-  掩码优先级一样hash值大的
				-  默认情况下30 32-30=2 所以每四个组网段会分配一次rp的地址(负载)
			- 如果都已想选rp地址大的
#### 动态RP优点冗余备份,rp负载维护状态(bsr主备没有负载)
缺点:千幻时间长150s,消耗资源,次优路径(RP的问题)
- anycast RP (任播RP,实际上就是静态RP,园区网RP的最佳时间(intra 域内网络))
- ![[anycast 任播.png]]
###### anycast RP 原理分析
之前的我么学习了PIM的几种机制,静态rp缺点,配置量大,不灵活,rp故障点问题无法避免,无法实现备份,动态rp,通过选举c-bsr和c-rp实现了rp的备份,*但是没有实现rp的负载分担*

任播anycast本事通过静态rp实现可以说是园区网的最佳配置方式
对于同一个组播地址配置多个路由器设备,每一个都设置一个环回口10.1.3.3当做rp接口,组网中的设备静态指定rp10.1.3.3,组网设备通过ospf打通了链接,这样每一个设备都会选择自己最近的路径的rp,这样实现了rp的冗余和负载,

我们虽然貌似设置了很多rp,但是对于根据上面的原理分析,根据ip的路由确定生成spt选择的rp只有一个需要将只有这个rp才能够获得相应的sg地址,因此,需要将sg信息扩散到其他rp上面

###### rp之间配置解决办法有两种
msdp multi source detective protocol 多源发现协议(域间的技术但是域内也可以用)
	peer rp first router地址 (想msdp的peer发送souce rp消息)
之后还需要防止rp之间互相传递sg信息造成资源浪费
(发送SA,souce,active信息)

pim 
	anycast-rp(只接受从first souce发送过来的注册包,并且想peer,转发,peer收到后接受,但不转发)

## 域间组播

![[域间组播.png]]

域间组播第一个问题?不同区域有着自己的源信息如果和他区域间联通的时候使能pim sm会导致bsr和rp重新选举打破域间的规划
	解决办法:通过在边界的接口上配置bsr-boundary,使其对bsr消息不收不发

###### 但是新问题产生?as之间如何发现组播源
这个就简单了!不过要跳出牛角尖,每个区域都有自己的rp,那么类似之前的anycast任播,互相建立任播邻居,spt选出接受注册包的rp后有他发送souce active到qita
rp实现rp的冗余,负载设置.

###### 新问题rp协议自带的次优路径问题
*可以依靠和之前解决rp次优问题一样自动调用了RTF来解决依赖单播,如果出现单播路径和和组播路径选取不同的情况可以手动的静态指定组播路径或者不知mbpp(本质也是手动干预)*


## IPV6的组播
### 可以说和ipv4进本一模一样

接受者段:ipv4使用的internet broadcast group protocol;ipv6使用MLD(multicast listener discovery 组播广播侦听)

不同点:*ibgp默认v2,v1缺少离组管理,v3针对special source multicast ,mld只有v1,v2其中v2是ssm(special source )**

剩下配置基本和ipv4雷同

**configuration example**

配置： 
AR5： Interface g1/0/0 
	Mld enable 组播网络分布树段：组播网络路由器 组播网络分布树 分区 
组播网络分布树 组播网络路由协议 Ipv6 pim dm Ipv6 pim dm配置： 
Multicast ipv6 routing-enable-------使能ipv6组播功能 
Interface g0/0/1 Pim ipv6 dm 
	Ipv6 pim sm Ipv6 pim 
	Sm配置： Multicast ipv6 routing-enable-------使能ipv6组播功能 
	
Interface g0/0/1 Pim ipv6 Sm Pim ipv6 bsr-boundary # Pim-ipv6 c-bsr + ipv6地址，建议loopback接口 c-rp + ipv6地址，建立loopback接口