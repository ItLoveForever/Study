1.  两台level 1的邻居area id 不统一（level 2 可以跨区域传输）
2.   宣告的地址是否冲突错误
3.  isis邻居的认证参数不一致
4.  isis的mtu值不一致
5.  isis 邻居接口的网络类型不一致
6.  接口设置成了isis silent
2 way 和 3way、max area 等都需要一致
*注意：isis之中cost存在窄带和宽带默认窄带如果两端的cost类型不一致可以建立邻居，但是数据无法传输*

**现实情况下，一般一个isis进程设置3个区域地址**