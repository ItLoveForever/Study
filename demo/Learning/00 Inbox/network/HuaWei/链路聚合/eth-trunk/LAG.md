# LAG （Linke Aggregation ）链路聚合

## 在华为路由器之中就是eth-trunk

link aggregation的主要作用就是将多条物理链路虚拟聚合成一条

#### eth-trunk的作用
增加带宽
冗余
负载均衡
高可用性
### 上面的三点基本上是所有聚合堆叠类型共有的特点

### Eth-trunk一般和mstp和vrrp（virtual reduncy routing protocol）一起实现负载均衡
#### 注意华为设备最多绑定8根，cissco吹的厉害实际测试也是8根

#### 最好速率双工一致

## link aggregation 配置
### 手工模式
```huawei
# 配置之前需要保证绑定默认配置所以最好的配置之前将端口设置清空
clear configuration g x/x/x 
# 创建eth-trunk端口
interface eth-trunk 1 #(这里的接口号默认0-63,但是设备不同也会有些许不同)
mode manual load-balance  #默认情况下,eth-trunk就是手工
# 创建eth-trunk之后绑定纳入的物理接口
trunk g 0/0/1
trunk g 0/0/2
# 进入物理端口将接口纳入eth-trunk
interface g 0/0/1
eth 1
interface g 0/0/2
eth 2
#检查
dis eth
```

#### 手工模式的缺点
假设绑定了三条端口,但是只聚合两根,1根冷备份,手工模式实现不了
## LACP (link aggregation control protocol)
#### 从在交换机角色master和slave
master:主交换机
salve:从交换机

##### lacp会根据lacp systemid选举主从交换机小的,如果一样根据lacp port-id选举小的
lacp system-id 默认的priority 32768+mac地址
lacp port-id 默认priority port-id

###### eth-trunk的负载分担通过源目ip,源目mac就计算hash,hash相同走统一路线
逐包负载分担——以报文为单位分别从不同的成员链路上发送（可能会产生报文乱序的现象）

逐流负载分担——不同的流在不同的成员链路上发送

如何判断是同一条流：根据不同的负载分担类型有不同的判断流的方式

负载分担类型
![[Pasted image 20240721091615.png]]

例：当负载分担类型为Des-ip时，表示目的IP相同的报文为同一流

### 在默认情况下使用的链路好是active状态,而eth-trunk默认是不抢占的,当电路故障冷备inactive的链路,转换成active,故障链路变成inactive
#### 但是如果冷备实际上链路带宽低需要回复之后切换回去就需要启用抢占

```huawei
lacp preempt enable
```
##### 因此产生的有可能网络震荡引起eth-trunk动荡
```huawei
lacp preempt delay 10
```

```huawei
# 注意如果配置了clear configuration interface g x 的端口包括eth注意之后去除shutdown

mode lacp
interface eth1
trunkport g x/x/x
lacp max act 设置多少是热备
lacp max ba  设置出现几条链路故障报错
```