![[mpsl.png]]
```huawei
# ar2,3,4配置ibgp的基本ospf部分省略
开启mpls

ar2
	system
	sysname ar2
	mpls lsr-id loopback (这里一般都使用环回口而且必须保持链接可通,因为mpls)
```