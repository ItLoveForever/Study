# 组播
	## ospf的组播地址224.5/.6
## 讲组播之前先了解下ipv4的传播方式

### 单播 :unicast
	# 实际上是点对点,效率最高
		sip ABC DIP ABC

#### 对方收到之后会检查目的ip,是就接受,不是转发出去
### 广播: broadcast 点到所有 
	# sip ABC DIP 广播地址 (全网广播255.255.255,子网广播1.255.255.255 ~主类的广播地址~)

#### 目的是广播无脑接受广播

### 多播 : multicast 
	# 点到多点 p2mp
	SIP ABC DIP D(D类地址)
	**实际上组播的范围变成全部就是广播,可以说是特殊的广播**
		- 也因此在ipv6中广播被取消
#### D类地址代表的是一个组,既不是广播也不是一个人

多播慢慢和组播画等号,每个D类地址代表之歌组,未知接受者

**不同于broadcast 和 unicast 他们的接受者是已知的,二multi是未知的他的接受者是动态变化的**

---

## 广播和组播的对比

![[单播组播对比topo.png]]


### 如上图所示存在一些问题
	1. 性能问题
		* 上图在单播情况下每个设备都需要建立连接
	2. BW
		*  每个设备都占用了一定的贷款,导致贷款不够
	3. 同时性
		* 设备转发存在先后顺序,无法同时发送接受

### 对比组播
还是对比上图,将三台全部加入一个组,假设地址224.1.1.1
	這樣f发送的实收只发送一份数据,复制转发
	![[组播对比举例.png]]
		上图黄色箭头代表复制
	这样就同时解决了unicast的缺点

#### 综上所属在p2mp下的组播的效率最高

### 组播的缺点

#### 因为组播基于udp,所以继承udp的缺点
1. 不可靠
2. 无序
3. 没有拥塞避免
4. 复制包
  *  复制包的本质原因是冗余造成,当在冗余链路上,冗余的链路会产生复制,之前的缺点不可靠,无序,拥塞避免都可以通过组播上层英语解决,唯独复制包要依靠***组播路由协议解决***重要	

## 组播

### 组播地址
* 组播地址是一个D类的组播地址224.0.0.0 - 239.255.255.255
	* 这里写一个acl来匹配所有的组播地址
	```huawei
	acl 2000
		rule permit source 224.0.0.0  15.255.255.255
	```

- #### 组播分类:
	- ASM : 任意源组(无论源地址是什么,只要组地址相同接受)
		- *any source multicast*
	- SSM:特定源组(校验组播地址的同时,还会校验源)
		- special source multicast
- **因此组播网段分成了两端**
- ##### 地址分类
	-  1. 保留组播地址
		- 224.0.0.1 所有主机地址
		- 224.0.0.2 所有路由器地址
		- 224.0.0.5 所有ospf路由器地址
		- 224.0.0.6 所有ospf DR/BDR地址
		- 224.0.0.9 所有rip的
		- 224.0.0.13 PIM
		- 224.0.0.8 VRRPV2
	-  2. ASM  任意源组播 
		- 224.0.1.0 ~231.255.255.255
		- 233.0.1.0 ~238.255.255.255
	-  3. SSM 特定源组播地址
		- 232.0.0.0~232.255.255.255
	-  4. 管理组播地址
		- 239.0.0.0 ~239.255.255
- ##### 组播的体系框架
	- ###### 域内组播(同一个as域内)
	 -  1. 组成组播的五种设备
		- 1. source : 源设备
		-  2. first-hop : 第一调路由器  *也叫做根*
		-  3 . multicast router : 组播网络路由器 
		-  4. last-hop : 最末跳路由器 *也叫做叶子*
		-  5. receiver : 接受者  
	 -  2.  按照功能划分可以分成三个段 ^6afe66
		-  1. 源段 : source和first-hop组成,只是不断产生组播流量,没有其他设置
		-  2. 接受者段: last-hop 和received段路由器组成
			- 职责 : 判断是否有人加组离组
			- 原则:
				- IGMP internet group multicast protocol 互联网组协议
					- 存在v1,v2,和v3,只有v3对应special source multicast
						- **默认情况下启动的是v2**
				- 原理: 给最末跳路由器生成igmp group表象.针对某一个组是否租在last router
		-  3. 组播网络分布书段[[PIM组播分布树(multicast distribute tree)]] ^PIM
			- 由multicast router 组成
			- 职责:形成从源到接受者最短的无环路路径,将源流量发送给接受者段
				- 最短无环路径生成协议
			-  PIM
				- protocol independent multicast 协议无关组播
				- 原理就是在原有的单播路由协议基础上开发组播路由协议
					- mospf :ospfv lsa type6
					- DVMPR : 距离矢量路由协议
					- **很早的协议了,适用性不高了解**
				- 开发独立的路由器协议,一次产生了pim protocol independent multicast
				- PIM模式
					- 密集模式:DM: dense-mode push (退)方式构建 源树
					- 稀疏模式:sparse-mode  pull(拉)方式来构建 组合树：
						- 源到RP：源树
						- ro到接受者：共享树
				- pim构建的树也分为集中
					- 源树:spt,最短路径树.接受者和源最短无环,适用于小型网络
						- 缺点:小号资源
						- 每个设备都要形成(S,G/source group),设备越多形成的SG越多，
					- 共享树
						- RPT，RP树，适用于大型网络
						- RP：汇集点
							- 源到RP：叫作源树 （S，G）
						- 接受者到RP ：共享树，（*，G）any source 到 group
							- 共享树相较而言减少了Group的数量只需要
	 -  3. 接受者段 IGMP internet group multi protocol ^IGMP
		 -  图例
		 - ![[IGGMP图例.png]]
		如图例所示:作为接受者我需要知道我所链接的设备有没有加组
		- 报文和工作机制
			- ***简单总结下***
				- *这里当las-router发送general query报文的时候发送的组地址内所有的last service都会收到，并且都会发送report，**但是在inernet group multi protocol 中的表项知识知道有人就可以了**不需要其他收到后会发送report想组地址中的所有设备，其他设备接受到report，如果他还没发送report就会抑制其report报文的发送 ，但是同时发怎么办？在query中有个mrt（max-reonse-time）最大响应时间，随机生成设备的发送时间不超过10s ^yizhi
					- 问题：单行网络没啥用，因为最大10s随机一定会有重复的
			- ####### V1 ^v1
				- 在v1中有一个general query 通用子查询,用于查询是否有人加组
				- 封装的sip是自己,dip是所有主句组包地址224.0.0.1 mac为全0
				- 这个组内的设备接受到之后会回复一个report 报告,目的地址是当前网段的组播地址
				- 形成了IGMP的 group表象
					| G组播地址 | last-report 最新报告的设备地址| 端口  |
				- 缺陷:如果三层网络中二层接了交换机,会导致翻红,组播网段所有人都收到
				-  report :报告分文两种运行模式
				- 周期性:收到general query之后周期性回复report
					- last-router周期性发送general query (时间间隔60s)收到之后,回复report
				- 触发性:
					- 当设备加入组后立即发送report报文
			- 注意:last report字段永远会被最新的设备字段所顶替掉
			- 离开组：离开时不做任何动作，这样也导致在老化期间三分钟内会一直占用
			-  问题 ：
			- 复制包问题 ^packet-duplicate-answer
				- 假设是个设备都是224.1.1.1这个组，，那么有多少个设备就会复制多少个包
				- 解决方案
					- report 抑制
						- [[Multicast 组播#^yizhi | 抑制原理参照简单总结]]
							- 根据上面的总结原理还是存在问题假设一起发送report 怎么办？
								- 引入 max-reponse-time 10
								- 缺点：大型网络还是重复，增加时间优惠响应变慢
			-  路由器查询的复制 ^route-duplicate-anser
				- 一般情况下路由器都会冗余，这就造成了在查询的时候冗余路径也会发送general query报文，导致双倍的general query和report
					-  解决方案： 
						- 选举查询者：querier，internet group multi potcol没有选举机制，通过借用PIM protocol independent multicast（[[Multicast 组播#^PIM|multi router段源树共享树选择最短路径的协议]]）的DR来充当查询者，只有查询者可以发送general query
		- V2 解决V1的缺点
			- 基本机制：[[Multicast 组播#^v1|v1基本机制]]
			- 问题
				- [[Multicast 组播#^packet-duplicate-answer|包复制问题]]
					- v2修改max-route-time 的粒度细化，可以10.1，10.2这样
					- 针对大型网络
						- internet group multiple protocol proxy 代理功能；example，十个report一个代理变成一个发送出去
				- [[Multicast 组播#^route-duplicate-anser|路由复制问题]]
					- v2存在querier的查询机制，通过ip地址来选（PIM选的是地址**大的**）**小的**---目的是区分增加可靠性
			 -  ==**离组：==
				-  v2中离组需要发送离组报文
					- **注意：飞last-report离组没有任何问题，如果是last-report离组会先确认组内还是否还有其他的接受者，发送特定组查询**
						- 特定组查询（最末跳路由器last-router）
							- igmp的查询地址是特定组地址
							- mrt（1秒之内发送两个查询），如果有会回复，将响应者作为新的last-report
		- V3
			-  根据下面的二层组播问题，v2的解决机制可以看出对于源source没有任何管理，而v3针对ssm - *special souce multi* 建立的针对源的校验机制
			- **在v3中leave被report管理，因为在v3的report功能很多进行了覆盖**
			- 特定组查询
			- 特定源组查询
			-  兼容性
				- 驾车接受这是v2，因为v2的report不包含leave信息，但是v3的report需要，这时候需要配置ssm的配额制
					-  配额距离
						- ssm-mapping （group address）指定的源定制
							-  当收到group的address他的源就是指定的地址\
			-  v3的几种模式
				- record字段下
					- 分为三类
					- 用于对general query报文响应
						- mode is_include 默认接受source中的地址，不疲惫拒绝
						- mode is_exclude 默认波不接受
					- 但include和exclude互相切换的时候
						- change to inclde mode
						- change to exclude mode
					- 当指定的源改变的是源地址时候
						- allow new souce
						- block_old_sources
			
	-  ~~二层主播问题（~~了解即可笔试可能出现~~）~~
		-  1. 组播ip映射组播man
		- 组播mac地址01005e
		- 组播ip为2的28次方，mac的oui是2的24，4个连续oui才能和oui对应
		- 实际上用01005e +ip后25位形成组播mac
			- 问题：多个ip映射一个mac；（但是可以忽略不计，后台会区分开）
			- 二层组播会泛洪没必要的流量
				- 解决方案：
					- igmp snooping：igmp的嗅探
						- 使能之后交换机能够识别igmp的报文，收到general query的接口称为router port，report的接口称之为port-info---交换机只会想port-info发送对应的报文，避免广播的泛洪
						- 对于leave他不做任何拦截直接传递，但是也会将leave对应的组播成员删除
					- igm snooping proxy 
						-  交换机中继,当组内设备很多的时候,设备接受到的report请求过多可以让交换机中继有他整理同意发送过来
						-  ==注意模拟器bug会导致看不出来离组效果==
- ### *模拟器此时有问题会导致看不见抑制能力*




	

### rp的具体讲解会在后面详细讲解

---
**上面我们将的组播都是跨网段的组播,跨越路由器,我们之前ospf的组播组播网络没有跨越网段**
