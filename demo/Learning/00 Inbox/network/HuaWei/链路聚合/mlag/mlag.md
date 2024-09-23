## 什么是m-lag
m-lag（muntichassi link aggregation group）跨框的链路聚合之前的[[LAG]]可以知道传统的lag只能实现一个设备的聚合，之前学习的堆叠，smart-link也可以实现，但是由于基于rstp，mstp并不能真正的负载分分担

并且传统的堆叠，是多虚拟成一个，所以管理平面只有一个
![[Pasted image 20240721190005.png]]

## m-lag的必要几个要素

### DFS-GROUP
DFS-GROUP（dynamic fabric servie group）动态交换服务器组
通过dynamic fabric servi group 部署对mlag进行配对，实现多台设备间链路聚合

**dfs-group的组id只能是1，m-lag是0~2048，主设备之间的m-lag要保证一直**
m-lag之间的同步需要以来dynamic fabric servie group协议来进行同步 

#### DFS的贮备设备的选举
先比较dfs group的优先级来比较，之后比较mac都是越小越优先，***实际上他会谁先开机谁会被选中*

### peer link
##### peer link 链路
peer link是配置mlag必须存在的一条二层链路，此链路必须采用了链路聚合
##### peer link 接口
peer link 两端的直连接口，放行所有vlan ，peer-link的id只能够是1

### 双主检测链路（心跳链路，keepalive链路）

是一条三层链路，用于mlag设备之间进行DAD（dynamic active detect）检测设备是否正常





