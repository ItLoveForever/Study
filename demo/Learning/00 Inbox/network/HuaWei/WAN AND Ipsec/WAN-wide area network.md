 广域骨干网:一般运营商构建
 core igp:isis
 bgp:ipv4,evpn/vpnv4v6
 mpls:sr/srv6

## 广域网怎么互联?
wan的链路技术
在物理层:通过ethernet
wan的互联
使用vpn:
	三层vpn
	GRE general route encapsulation 通过GRE封装
	interface 开隧道
	ip address 配置ip
	tunnel protocol 配置隧洞协议
	source 
	destinationg 配置源目地址
但是上面配置的是点对点的,为此可以使用
MGRE:dsvpn
优点:支持路由协议,支持广泛,配置检点
缺点:不安全
### mgre工作原理
让设备知道multi general route encapsulation 这个过程叫做注册register
用Ip电话来举例子
ip电话不通于普通电话,链接网线通过callm-manager链接
首先spoke 会向hub发送register,hub中记录并生成nhpr(next-hop)表项记录了隧道peer地址和nbma的地址信息双发在hub进行交互最后双方都会学习到对方的信息

# 注意网络类型造成的影响
ospf如果网络类型是p2p的话那么他只能建立一个ospf邻居第二个邻居无法建立,所以需要修改网络类型
### Ipsec 安全但是不支持路由
#### 保证安全需要哪些要素
1.  源认证
	1.  就是我们的认证用来确定用户是否是我们的合法用户通过认证元素来认证
		1. 认证元素: 卡(用户名) 密码(key) 指纹 虹膜 声音
		2.  认真元素越多越安全
2.  私密性
	1.  加密(gre genenral route encapsulation 最大的问题没有加密,明文传输导致容易被窃取)
		1.  加密学原理: 同一个秘钥加密同意个秘钥解密
			1.  ;优点: 效率高 密文小
			2.  缺点:秘钥的分发阐述不安全,储存不方便
		2.  非对称加密 
				1.  不同的密码来加解密
					1.  产生一对秘钥 公钥public 私钥private
						1.  原理只要一个坏了都坏了,并且一个加密一个解密(public 或者private都可以进行加解密)
					他的工作的机制.假设我给了一个人的文件并且给他公钥,那么他就可以用我发给他的public进行加密,那么只有private的持有者可以进行解密
					**优点:秘钥的管理分发储存方便,缺点:加解密效率低下,密文较大**
***总结:基于对称和非对称的特性相互作为优缺点所以我们一般结合使用***
3.  结合用法
	1.  首先发送用对称加密方式将文件加密,这样产生了对称秘钥,之后再用public进行非对称加密进行重复加密
4.  完整性
	1.  有可能存在一种请款那就是我不需要破解你的文件我只要对你的秘钥或者什么文件进行篡改,让你有问题
	2.  md5 和 sha 完整性检测(目前MD5已经被破解了)
		1. MD5 定长度 128/256/512
		2.  不可逆没办法通过hash反推出明文
		3.  雪崩效应,明文一丁点的改动密文就会雪崩一般进行篡改
		*md5或者sha都是封装在密文包中,惊醒加密*
**为此可以使用md5或者sha记性完整性检测**

对方收到文件之后先不会进行密文打开而是先进行md5或者sha的校验(根据md5或者sha的值进行hash计算根据hash的值进行比较)
5.  不可否认性
	1.  问了确认这个文件到底谁发送
	2.  数字签名和数字证书
		1.  实际上就是通过private进行加密,用公钥进行解密
		2.  但是上面的方法,也可以被抵赖,就是对方说你的public文件就是伪造,因此产生了数字证书的概念
			1.  通欧数字证书签发机构(CA进行认证)
				1.  发送一个证书 里面包含了公钥,个人信息等
				2.  这是后再抵赖那么CA就来证明

**没有绝对的安全都是相对**

### IPsec
Ipdec:Ipsec security IP不安全,ipv4后来发现ip很重要才进行ipsecurity的认证

##### IP security 的基本要求
1.  认证
2.  私密
3.  一致
4.  不可否认性
5.  防止重发:序列号

##### ip security 的框架
安全协议:
	支持的协议: authentication header 认证头部
	 封装在 ip后面 协议号51 作用主要是防止重发,认证和完整性
		. IVC integrity check value ,一致性检测,根据max 计算has值比较完整性
		序列好防止重发
	ESP encapsulation security playload 
		同样封装ip后面协议号50,功能认证,完整性,防止重发,**多了私密性(可以记性加密)**
		同样依靠ivc(integrity check value)和序列号
封装模式
	传输模式
	隧道模式
组合模式
传输模式下的和ip的组合模式
图例![[传输模式.png]]
这种模式没有进行新怎所以简单效率高使用主机和主机
![[隧道模式.png]]
隧道模式进行了新格式使用性更高一般用于网管更安全LAN to LAN 默认情况下是隧道模式的封装
IKE(internet key exchange)互联网秘钥交换
**ike主要保障双发能够生成相同的秘钥保障认证**
IKEv1(老模式v2新出的还没使用)
-  ikev1分文两个阶段
	-  第一阶段
		-   主模式(main mode ) ,这个模式通过六个包进行交互,一般这种模式用于网管之间
		-  野蛮魔兽(aggressive mode) 3 个包多用于主机之间
	-  第二阶段
		-  快速模式

#### ip security 的组件
1.  peer 对等体
2.  ip security 隧道:数据对通过隧道的数据进行加解密
3.  SA: security association 安全联盟 ip security 隧道安全参数的约定
	1.  security 的组成
		1.  SPi security parameter index 安全参数索引
			1.  确定对面使用哪个spi进行嫁接密,类似rt促进在出和入
			2.  目的ip 
			3.  安全协议 AH(authentication header),ESH encapsulation security protocol

配置示范
```huawei
ar1
inter g 0.0.1
ip ass
1.1.12.1 24
ospf 
network

ar2
同上配置
## 确认关于网通常
多种方法这里先通过静态方法保障底层通常
int g 0/0/0/
pc上面设置网关

ar1 
ip 0.0.0.0 目的ip
注意先不要配置ar2因为需要再r2上面进行ipsecpeizhi
配置感兴趣流信息
ar1上面配置acl过滤1网段
ar3 alc 3000 192.168.3.0 dest 192.168.
# 配置相关的ipsec
默认情况下会有默认的ipsec 都可以修改
进入ip sec下面
transform 修改传输类型
修改认证方式
esp encapsulation
修改加密模式
esp encrption algori

配置ipsec policy
结合之前设置的acl修改sa spi inboun esp 属性
```
lan to lan 远程接入
