## IGMP
	Internet group multicast protocol

组播组协议,应用于接受者段
配置命令
```huawei
#赋能组播路由功能
multicast routing enable
#接口下使能igmp,注意总是是接受者断但也有可能是组播网络分布段的成员需要配置PIM()
接口下
	igmp enable(注意如果先使能igmp那么无法配置pim)


```