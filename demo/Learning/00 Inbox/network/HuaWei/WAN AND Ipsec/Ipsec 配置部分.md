配置ip security的时候的几个基本组成部分
1.  peer: 对等体
	1. 在后面使用ike(interfnet key exchange | 互联网秘钥交换技术)协议动态配置ipsec的时候需要配置,目前在静态配置的时候暂未发现需要配置
2.  Ipsec 隧道 
	1.  GRE(general route encapsulation)和ipsec都是一种隧道技术,所以需要配置隧道
3.  SA : security association 安全联盟 
	1.  用来定义保护ipsec安全隧道的安全参数的约定
	2.  SA的组成(三元组)
		1.  SPI(security parameter index)安全索引参数,主要是用于区分设备上面多个隧道的情况
		2.  目的ip
		3.  ipsec 用什么安全洗衣
			1.  EA 
			2.  ESH
---
### 静态的ipsec
```Huawei

```