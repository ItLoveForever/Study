用此图举例
![[mpsl.png]]
###### 解析
之前在AR2属于PE(provider edger),3(管线内部的复杂链路),4(provider edger)属于运营商的管线,r2,r3,r4底层通过ospf打通,ar1,和ar5都属于同一个客户,因此产生了需求,ar1和ar5通讯,这时候就需要借助BGP,因为我只想AR1访问AR2但是不能够访问我具体内部网络

AR2和AR4建立ibgp邻居,这样2和4就有了纳入bgp管理中的所有的路由
	同样的mpls vpn也是这样的纳入
