Ipsec实际上也是vpn技术的一种,是ipsecurity的缩写在了解ipsec之前需要先了解之前学习一下安全

## 什么是安全
### 安全一般需要具备四种元素
1.  源认证--认证
	1.  作用是用来确认是否是我们需要的合法用户
		1.  可以用来认证的元素:用户名,密码(key),指纹虹膜声音等等
		2.  认证元素越多越安全,但是安全是相对
	2.  私密性--加密
		1.  加密分为对称性加密和非对称加密
	3.  完整性--检测数据是否被修改
		1.  常用的检测完成性算法mda和sha
			1.  作为完整性检测都可以制定检测后制定长度
		2.  具有雪崩效应,一点改动可以是联动整体变化
		3.  不可逆向,无法通过倒推明文
**注意:md5已经被破解现在建议使用sha**
	4. 不可否认性
		1.  数字签名psk或者数字证书ca

___
## Ipsec ip security ip 安全
### Ipsec的具体需要的特性
1.  认证
2.  私密
3.  完整
4.  不可否认
5.  (新加的)防止重发

### ipsec的基本框架
#### ipsec运行的协议
##### AH:authentication header 认证头
这个协议封装在ip协议后面,协议号51,主要的功能是认证,防止重发,完整性检测.不具有加密功能
ICV:integrity check value 一致性检测,利用hmac地址计算hash,来比较完整性
序列号:防止重发

##### esp : encapsulation security payload 负载安全封装
和 ah一样封装在ip后面,esp协议号50,其功能在ah的基础上面,完整性,认证,防止重发的基础上额外具有私密性
和icv一样integrity check value 进行一致性检测
序列号防止重发

#### Ipsec的封装模式
ipsec的封装有两种tunnel和transfer
![[ipsec的封装.png]]结合上面的图简单阐释下两种封装方式
transfer模式的封装某头模式下不会产生隧道的ip,因此疯转加密也没有对原ip进行加密
所以tunnl更加安全所以,transfer多用于主机之间,tunnel多用于lan to lan方面


### IKE internet key exchange 互联网秘钥交换协议
一般是用于动态协商生成esp协议的各种加密参数

---
### ipesc 的基本组件
1.  对等体
2.  ipsec的隧道
3.  sa: security associate 安全联盟,用于定义ipsec的各类安全参数
	1.  sa的组成 (三元组)
		1.  spi security parameter index 安全索引参数,用于区分多ipsec情况下的秘钥
		2.  destination ip 目的ip
		3.  安全协议 : AH esp

![[Ipsec  配置.png]]
用静态配置ipsec
``` huawei
# 首先配置基本环境,ip address
之后需要保证基本的core ip通讯正常
采用ospf 宣告各自的端口
ar1的0/1和ar2的0/0和0/0和ar3的0/0注意两个网关端口不能宣告否则体现不了ipsec的功效

# 配置ar1和ar3之间的ipsec隧道 
先配置那些ip可以走隧道,这里利用acl进行筛选
acl 3000
rule 5 permit ip source 192.168.1.0 0.0.0.255
# 这里是配置ar1可以走隧道的路径
acl 3000
```