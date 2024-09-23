
---
## MP-BGP

# BGP-MP ipv6 Unicast

[[Read Me#如果不额外说明的情况下一般 或者（）~~~~包含的内容一般是根据实际的地址改变的变量没有具体意义]]

主要依靠mp-reach-nlri
	 ~~nlriipv4中也存在主要包含下一条等等信息~~
```Huawei
	# 大致流程参考和ipv4基本相同
	# 先启动建立邻居
	bgp ~~as-numer~~
		bgp router-id （loopback address）
		peer ~~ip-address~~ as-number # 正常进行bgp邻居指定
	# 之前pv4可以直接配置邻居，这是因为ipv4可以默认情况下自动会在ipv4的ipv4-family里进行相关配置，这里我们配置ipv6，所以需要进入ipv6的family中配置
		ipv6-family
			peer ~~ip-address（destiny）~~ enable
			


```

### 抓包图例 
![[NLRI.png]]

# 过渡 ：transition
## 双栈 : dual-stack

	### 简单来说就是在ipv4前面加上Global Ip 200/3的前缀
	### 实现了ipv4和v6的互存保证通讯

## 隧道 : tunnel
	 # 场景运行商ipv4网络,但是客户是ipv6网络保证通讯
![[v4和v6拓扑.png]]

v4和v6互通隧道配置
![[v4和v6互通隧道配置.png]]

![[GRE封装.png]]

这样存在一个问题每个保温都要封装一个GRE头部信息造成性能损耗

### 将tunnel protocol 从GRE改成专用的ipv6-ipv4
	**注意需要从新配置source和destination**

## 动态的v6和v4通信
	之前的隧道我们手动配置的,但是如果站点stations很多的时候,隧道基本配置不过来
![[动态配置图例.png]]
**配置动态需要在v4和v6的链接端口和换回口上再次执行ipv6 address**
	注意:不同于v4在v6中地址不会覆盖
	# 注意这里配置的地址将loopback 和链接端对端对应地址融入了进去
			example:
				ar2上
					loopback 2001:172:15:2::2 64
					# 新增      2002:a01:202:12::2 64 ~  相当于2002:  10.1.2.2.12::2  )  
				ar1上
					