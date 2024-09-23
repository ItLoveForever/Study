今天练习发现的问题
# X园区
## 忘记配置core上面的核心放行vlan导致wifi可以连接但是无法获取地址
## 忘记配置无线强制认证导致ap无法上线
## 复制时候多复制一行防止stack堆叠失效

### 这里补充命令
```huawei
<huawei>reset save conf
<huawei>reset stack conf
这个命令无法修复stack 0 slot 所以需要手动把他调整成默认方
```

## 有线接入
### 配置有线认证后没有部署导致有线无法获取地址

# x排查总结
## ap上线问题检查是否配置vlan实例,是否放行vlan
**注意不要相信预配置,都需要放一下,尤其是ac上面**


___
***中秋节联系发现的问题
首先dis eth 显示链路聚合失败.disp lldp nei bri无信息,查看物理端口是否被shutdown

**之后在无线设置网络个里时候发现不能获得ip地址
首先确定vlan放行,确认无线.254可以ping通,不同确认disp IP routing vpn guest ,没有设定deault-route,确认vpn下设置vpn sim

***注意这里查看sec 下的dis secu rule all,这里出现employee可以被in_to_internet,原因是.60没有分配实力导致
## 这里是配置时候忘记一些配置细节的指令
### core上
	acl 3000
		rule permit ip source 10.1.51.0 0.0.0.255 dest 10.1.60.0 0.0.0.255
	int vlan 51		
		traffic-redirect inbound acl 3000 vpn Employee ip 10.1.200.22
	int g 0/0/3可以配置
### FW1
	公网静态路由
		ip route-static vpn employee 10.1.101.0 24 vpn Guestt
		ip route-static vpn guest 10.1.60.99 32 vpn Employee
	
#### stack 设置失败回复
reset save con
sys
	reset stack
stack slot pri 100

stack slot re 0

___


# Y 园区
## 问题点在于设备导入阶段在在修改控制vlan之前一定先确定管理vlan给core分配的ip填好并且core上线之后再去修改否则core上不了线就gg了
**除了两个export设备其他设备reset save config和reset netconf-db即可**
export复杂一些首先需要fac set from default和fac reset最后重启

# python
### 易错配置点
local-user huawei
undo local-aaa 
## 连接上之后突然中断无报错
***检查X_T1_AGG1上面的配置是否配置,大多是一开始就连接不上
***之后上次出现中间中断主要问题点是两个一个没有配置screen-length 0 temporary 
个人认为主要原因是
client.connect()的参数没有用self.来引用
faninfo.find()配置成了faninfo.open()
