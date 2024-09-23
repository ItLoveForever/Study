```huawei
display igmp interface 查看igmp给选举的querier和max route time等信息
dispaly igmp group 参看igmp的表项
# 交换机系统视图下
igmp snooping enable 使能交换机嗅探
display igmp-snooping 。。。。查看ipgmpsnooping的相关角色 
display pim interface 参看接口的pim信息
display rpf 地址   查看rpf的信息

######
配置rpf的静态路由
ip rpf-route-staitc source 地址 下一跳地址
muticat longest-macth
# 接口下
	igmp 
		ssm-mapping （group address）指定的源定制
			# 当收到group的address他的源就是指定的地址

转发组播流量的设备需要开启组播功能
multicast routing-enable
	接口下
		pim dm
```

## 配置RP相关指令
```huawei
pim
	static-rp 地址
dispaly pim rp-info
# 静态rp接口下不需要其pim

## 动态RP
pim 
	c-bsr 对应的接口
#并且接口同样需要开启pim
```