#### srv6的优点
1.  简洁,不走标签,全部是ipv6的报文转发,不在是2,5层直接进就是ip转发payload减小,方便之后对接ipv6的拓扑和应用
2.  可编程针对目的的ipv6头部进行ipv6改造:
![[对比mplsvpn的实例.png]
	1. (针对ipv6目的地址的部分字段赋予不同的行为)
		和mpls vpn 对比
			公网标签:用于送达pe
			私网标签 : 用于到达pe的行为.查询vpn-instance使用
	 2.  srv6中公网标签不存在,在srv6中locator2001:4::/96代表了pe,相当于代替了公网标签,私网标签在srv6中opcode::41 end-dt4 vpninstance A
		 1.  公网标签举例说明ipv6 2001:4(这里叫做 locaor)::41(这里叫做function)  (后面还有argment,但是目前用不到)
			 1.  目前知道的::41表示end-dt4 vpn-instance
			 2.  ::1 end(node)
			 3.  ::45 end-x 接口(adje id)
			 4. ** ::x这不重要,重要的是后面的end,后面跟前面的数字进行绑定 **
srv6 BE只有目的地需要配置srv6 locator ,其他的路由器只需要配置ipv6路由既可以
缺点:不能控制路径

srv6 te: 真正的遵从我们源路由(source routing)操作,集中式操作(所有的都需要配置locator)
 带来的问题:变大的这部分source routing封装在哪里
	 回忆ipv6基础
	ipv6 header
		basic header :基本头部:next-header 43(指向扩展包头)
		srH(sr的封装信息,之间存在sl:3(深度3)),传递到下一个的时候去掉一个sl list ,sl变成2
		![[srv6的鱼形图.png]]