# 计算机网络实验复习

![image-20241111155752695](C:\Users\lzfshy\AppData\Roaming\Typora\typora-user-images\image-20241111155752695.png)

## 注意事项

1. tracert看不见

   > ip ttl-expires enable
   > ip unreachables enable  

   可用 tracert –d 10.1.3.10，避免DNS配置的干扰。

2.undo vlan 1

这些出厂设置，会造成地址冲突、自动参加ospf的router id 选举，造成router id冲突等等。目前已发现其有可能会对**OSPF、BGP、组播、IPv6等实验造成不良影响，且故障隐蔽不易排查。**

建议养成习惯，实验开始时，undo这些出厂设置。

例如：

1）线下实验环境的路由器会自带interface vlan 1（ip address 192.168.0.1）

​    解决办法：[h3c]undo  interface vlan 1

2）线上平台部分路由器的Ethernet0/0接口会自带ip address 192.168.1.1

​    解决办法：[h3c-Ethernet0/0]undo ip address 

3）部分交换机的出厂设置创建了三层接口interface vlan-interface 1，并配置了dhcp自动获取地址。

​    解决办法：[h3c]undo  interface vlan 1

3. 查看端口状态及简写

   > dis inter brief

4. 关掉XGE 1/0/25 to XGE1/0/28



## 常用快捷方式

（1）<Ctrl+G>对应命令display current-configuration（显示当前配置）

（2）<Ctrl+L>对应命令display ip routing-table（显示IPv4 路由表信息）

（3）<Ctrl+O>对应命令undo debugging all（关闭设备支持的所有功能项的调试开关）

（4）<Ctrl+Z> 对应命令return（退回到用户视图）

## 常用配置命令

### 1.  重启

> <h3c> reset sa
> <h3c> y
> <h3c> reboot
> <h3c> n
> <h3c> y

### 2. dispay

可在任意视图下使用。

> dis cur 
> dis version
> dis inter e1/0/1
>
> <!--查看交换机的MAC地址表-->
>
> dis mac-addr 

### 3. 路由器基本配置

1. **基本配置**

   > <h3c>sys
   > [h3c]dis version

2. **接口配置**

   > sys
   > inter G0/0
   >
   > <!--设置网络协议地址 -->
   >
   > ip addr 192.168.0.1 24      <!--掩码24位 -->
   >
   > <!--取消网络协议地址 -->
   > undo ip addr 192.168.0.1 24      <!--掩码24位 -->

### 4. 交换机基本配置

1. **进入以太网端口**

   > sys
   > inter G1/0/1

2. **打开关闭以太网端口**

   > shutdown
   > undo shutdown

3. **设置以太网端口类型**

   端口视图下，默认access。

   > port link-type access/hybrid/trunk
   > undo port link-type

### 5. win配置电脑的ip地址

>netsh interface ip set address name="本地连接" static 192.168.2.10 255.255.255.0 192.168.2.1 0 

## 集线器、交换机、路由器

1. 集线器(Hub): 工作在物理层，主要功能是对端口接收到的电信号进行复制、整形、放大，同时有多个端口传输数据会发生冲突
2. 交换机(Switch): 工作在数据链路层，每个冲突域只有一个端口，交换机基于MAC地址表决定数据的转发端口，而不是向所有端口转发。
3. 路由器(Router): 工作在网络层，根据数据包的IP地址选择发送路径，转发数据包到相应网络，一般工作在广域网，具有两个以上的端口，分别连接不同的网络。路由器隔离广播域。

## VLAN的配置与分析

VLAN技术是一种专门为隔离二层广播报文设计的虚拟局域网技术。

路由器隔离广播域，是因为路由器的数据转发都在IP层进行，所以对于二层本地广播来说，它是无法通过路由器的。

VLAN技术中，规定凡是具有VLAN功能的交换机在转发数据报文时，都需要确认该报文属于某一个VLAN，并且该报文只能被转发到属于同一VLAN的端口或主机。每一VLAN代表一个广播域，不同的VLAN用户属于不同的广播域，他不能接受来自不同VLAN用户的广播报文。

###  VLAN的帧格式

IEEE802.1Q协议规定VLAN技术，在原有标准以太网帧格式中添加一个特殊标志域tag，用于标识数据帧所属的VLANID。

### VLAN数据帧的传输、VLAN端口的分类P50

### VLAN的配置

![image-20241111110444782](C:\Users\lzfshy\AppData\Roaming\Typora\typora-user-images\image-20241111110444782.png)

1. **创建/删除VLAN**

   > vlan 2
   > undo vlan 2

2. **向当前VLAN添加/删除端口**

   > vlan 2
   > port G1/0/2 G1/0/1
   > port G1/0/3 to G1/0/20

3. **指定端口类型**

   > inter G1/0/1
   > port link-type trunk
   > port link-type hybrid
   > undo port link-type

4. **指定/删除端口的默认vlanID**

   > inter G1/0/1
   > port trunk pvid vlan 2
   > undo port trunk pvid 2

5. **指定/删除Trunk端口可以通过的VLAN数据帧**

   > inter G1/0/1
   > port trunk permit vlan 2 to 3
   > port trunk permit vlan 3
   > port trunk permit vlan all

6. **将Hybrid端口加入到指定的已经存在的VLAN，并标记为tagged或untagged**

   > inter G1/0/1
   > port hybrid vlan 20 30 untagged

### exp1 网络实验入门P28

<!--wrong-->

**netsh interface set ip address name="本地连接" static 192.168.2.10 255.255.255.0 192.168.2.1 0**

**netsh interface set ip address name="本地连接" static 192.168.2.11 255.255.255.0 192.168.2.1 0**

**netsh interface set ip address name="本地连接" static 192.168.3.10 255.255.255.0 192.168.3.1 0**

**netsh interface set ip address name="本地连接" static 192.168.3.11 255.255.255.0 192.168.3.1 0**

true : netsh interface ip set address 

**R1**
sys
sysname R1
inter G0/0 
ip addr 192.168.2.1 24
inter G0/1
ip addr 192.168.3.1 24

### exp2 数据链路层

IEEE802.3标准

1. 交换机的MAC地址表：交换机的转发基于MAC地址表，MAC地址表是交换机在收到帧时通过学习帧中的源地址和来源帧端口生成。

2. 交换机的端口聚合

3. 实验配置：

   > netsh interface ip set address name="本地连接" static 192.168.2.10 255.255.255.0 192.168.2.1 0
   > netsh interface ip set address name="本地连接" static 192.168.2.11 255.255.255.0 192.168.2.1 0
   > netsh interface ip set address name="本地连接" static 192.168.2.12 255.255.255.0 192.168.2.1 0
   > netsh interface ip set address name="本地连接" static 192.168.2.12 255.255.255.0 192.168.2.1 0

   **S1**
   sys
   sysname S1
   vlan 2
   port G1/0/1 to G1/0/5
   vlan 3
   port G1/0/20 to G1/0/24

4. 实验配置2

   netsh interface ip set address name="本地连接" static 192.168.2.10 255.255.255.0
   **S1**
   vlan 2
   port G1/0/1 to G1/0/5
   vlan 3
   port G1/0/20 to G1/0/24
   inter G1/0/13
   port link-type trunk

   **permit vlan 2 3**

   port trunk permit vlan 2 3

   undo port link-type
   port link-type hybrid
   **port hybrid pvid vlan 1**
   **port hybrid vlan 2 tagged**
   **port hybrid vlan 3 untagged**
   **port hybrid vlan 3 tagged**
   or
   port hybrid pvid vlan 3

   **S2**
   vlan 2
   port G1/0/1 to G1/0/5
   vlan 3
   port G1/0/20 to G1/0/24
   inter G1/0/13
   port link-type trunk
   port trunk permit vlan 2 3

### 广域网数据链路层协议

#### 1.PPP

![image-20241111115751029](C:\Users\lzfshy\AppData\Roaming\Typora\typora-user-images\image-20241111115751029.png)

广域网中广泛使用的数据链路层协议

两种身份认证：PAP,CHAP

#### 2.实验步骤

##### PPP

R1-S1串接

**R1**
inter S1/0
ip addr 192.0.0.1 24

link-protocol ppp

shutdown
undo shutdown

ctrl+z
<R1>debugging ppp all
terminal debugging

**R2**
inter S1/0
ip addr 192.0.0.2 24

inter S2/0
shutdown
inter S1/0
shutdown
undo shutdown

##### **PAP**

**[R1]**
local-user RTB class network
service-type ppp
password simple aaa
inter S1/0
ppp authentication pap

**[R2]**

inter S1/0
ppp pap local-user RTB password simple aaa

配置PPP后重新启动接口生效

通过display interface serial命令，查看接口Serial2/1/0的信息，发现接口的物理层和链路层的状态都是up状态，并且PPP的LCP和IPCP都是opened状态，说明链路的PPP协商已经成功

inter S1/0
shutdown
undo shutdown

<R1>debugging ppp pap all
terminal debugging
[R1]inter S1/0
shut down
undo shutdown

##### CHAP

**[R1]**

local-user RTB class network
service-type ppp
password simple aaa
inter S0/0
ppp authentication-mode chap
ppp chap user RTA

**[R2]**

local-user RTA class network
service-type ppp
password simple aaa
inter S0/0
ppp chap user RTB

配置完后需要在接口上shutdown和undo shutdown

inter S0/0
shutdown
undo shutdown

<R1>

debugging ppp chap all
terminal debugging
[R1]
inter S0/0
shutdown
undo shutdown

### VLAN间通信P.84

**netsh interface ip set address name="本地连接" 192.168.2.10 255.255.255.0 192.168.2.1 0**

netsh interface ip set address name="本地连接" static 192.168.2.10 255.255.255.0 192.168.2.1 0

netsh interface ip set address name="本地连接" static 192.168.3.10 255.255.255.0 192.168.3.1 0

**[S1]**

undo inter vlan 1
vlan 2
port G1/0/1 to G1/0/4
vlan 3 
port G1/0/20 to G1/0/24
inter G1/0/13
port link-type trunk
port trunk permit vlan 2 3
inter vlan 2
ip addr 192.168.2.1 24
inter vlan 3
ip addr 192.168.3.1 24

**[S2]**

undo inter vlan 1
vlan 2
port G1/0/1 to G1/0/4
vlan 3 
port G1/0/20 to G1/0/24
inter G1/0/13
port link-type trunk
port trunk permit vlan 2 3

清空交换机的MAC地址表和arp缓存

[S1]undo mac-addr
<s1>reset arp all

PC:

arp -d



#####  **交换机MAC地址表说明**

以VLAN Trunk实验的交换机S1为例，显示其MAC地址表，通常里面有3部分MAC地址：

（1） PC的MAC地址，在PC上用ipconfig/all查看

（2）对端交换机S2的MAC地址，在S2上用display interface命令查看

（3）连线组网设备的MAC地址，我们没有权限查看，大家知道就好。

## OSPF

重启

reset ospf process

**查看邻居**

<!-- 详细verbose-->

dis ospf peer verbose

<!--简短-->

dis ospf peer

查看路由

dis ospf rou

**\# 查看Switch D的ABR/ASBR信息。**

<SwitchD> display ospf abr-asbr

### exp 1 P180 图7-2 

**[R1]**

undo inter vlan 1
inter G0/0
undo ip addr
ip addr 168.1.1.1 24
inter Loopback1
ip addr 1.1.1.1 32

<!--wrong-->

**ospf**
**router id 1.1.1.1**

**[R1]**
router id 1.1.1.1
ospf
area 0
network 1.1.1.0 0.0.0.255
network 168.1.1.0 0.0.0.255

**[R2]**

undo inter vlan 1
inter G0/0
undo ip addr
ip addr 168.1.1.2 24

router id 2.2.2.2
ospf
area 0
network 2.2.2.0 0.0.255
network 168.1.1.0 0.0.0.255

**[R1]**

undo router id 
router id 3.3.3.3
ospf
area 0
network 1.1.1.0 0.0.0.255
network 168.1.1.0 0.0.0.255

更改router id后需要重启ospf协议

dis ospf 

**<R1>**
debugging ospf event
terminal debugging

### exp 2 P.196

**[S1]**
undo inter vlan 1
vlan 1
port G1/0/1 to G1/0/20
inter vlan 1 
ip addr 168.1.1.3 24
inter Loopback 1
ip addr 5.5.5.5 32

router id 5.5.5.5
ospf
area 0
network 168.1.1.0 0.0.0.255
network 5.5.5.0 0.0.0.255

**[S2]**
undo inter vlan 1
vlan 1
port G1/0/1
inter vlan 1
ip addr 168.1.1.4 24
inter Loopback 1
ip addr 6.6.6.6 32

router id 6.6.6.6
ospf
area 0
network 168.1.1.0 0.0.0.255
network 6.6.6.0 0.0.0.255

**[R1]**
undo inter vlan 1
inter G0/0
undo ip addr
inter G0/0
ip addr 168.1.1.1 24
inter Loopback 1
ip addr 1.1.1.1 32

router id 1.1.1.1
ospf
area 0
network 168.1.1.0 0.0.255
network 1.1.1.0 0.0.0.255

**[R2]**
undo inter vlan 1
inter vlan 1
undo ip addr
inter G0/0
ip addr 168.1.1.2 24
inter Loopback 1
ip addr 2.2.2.2 32

router id 2.2.2.2
ospf
area 0
network 168.1.1.0 0.0.0.255
network 2.2.2.0 0.0.0.255

**显示lsdb**

display ospf lsdb brief
display ospf lsdb network/nssa/router/summary

reset ospf process

**<R1>**
debugging ospf event
terminal debugging



### exp 3 P215

**[S1]**
undo inter vlan 1
vlan 2
port G1/0/1
inter vlan 2
ip addr 192.168.1.1 24
inter Loopback 1
ip addr 4.4.4.4 32

**[S2]**
undo inter vlan 1
vlan 5
port G1/0/1
inter vlan 5
ip addr 20.1.1.2 24
inter loop 1
ip addr 6.6.6.6 32

router id 3.3.3.3
ospf
area 1
network 6.6.6.0 0.0.0.255
network 20.1.1.0 0.0.0.255

**[R2]**
undo inter vlan 1
inter G0/0
undo ip addr
inter G0/0
ip addr 20.1.1.1 24
inter S1/0
ip addr 10.1.1.2 24

**shutdown**
**undo shutdown**

router id 2.2.2.2
ospf
area 1
network 20.1.1.0 0.0.0.255
area 0
network 10.1.1.0 0.0.0.255

**[R1]**

undo inter vlan 1
inter G0/0
ip addr 192.168.1.2 24
inter S1/0
ip addr 10.1.1.1 24

<!--y??-->

**shutdown**
**undo shutdown**
inter loop 1
ip addr 5.5.5.5 32

router id 1.1.1.1
area 0
network 5.5.5.0 0.0.0.255
network 10.1.1.0 0.0.0.255

ping通

[R1]
ip route-static 4.4.4.0 24 192.168.1.1
ospf
import route-static

[S1]ip rou 0.0.0.0 0.0.0.0 192.168.1.1

dis ospf lsdb asbr
dis ospf lsdb ase



### exp 4 图7-22  P215

**ospf cost** 
**[S1]**
inter vlan 3
ospf cost 100
inter vlan 2
ospf cost 200

**[R1]**
inter G0/0
ospf cost 100
inter S1/0
ospf cost 500

SPF计算
查看router lsdb
dis ospf lsdb router

同一个OSPF区域内每台路由器上LSDB中的LSA信息一致

根据显示画图P220？？

SPF的计算

### 设计实验P227

#### 设计exp 1 图7-30 P 227

PCA
netsh interface ip set address name="本地连接" static 192.168.5.2 255.255.255.0 192.168.5.1 0

[S1]
undo inter vlan 1
vlan 1
port G1/0/1 to G1/0/5
vlan 2
port G1/0/20 to G1/0/24
inter vlan 1
ip addr 192.168.3.2 24
inter vlan 2
ip addr 192.168.5.1 24

router id 1.1.1.1
ospf
area 1
network 192.168.3.0 0.0.0.255

pc1一开始就能Ping通所有的设备只是ping不通PC2以及192.168.6.1？

方案一
S1
import direct
S2
import direct

方案二
**R1**
ip route-static 192.168.5.0 24 192.168.3.2 
ospf
import route-static

**R2**
ip route-static 192.168.6.0 24 192.168.4.2
ospf
import route-static

方案三纯静态
**S1**
ip route-static 192.168.6.0 24 192.168.3.1

**S2**
ip route-static 192.168.5.0 24 192.168.4.1

**R1**
ip route-static 192.168.5.0 24 192.168.3.2

**R2**
ip route-static 192.168.6.0 24 192.168.4.2

#### 设计exp2 图7-31 P 228

**[S1]**

router id 1.1.1.1
ospf
area 0

<!--注入的是网段.0即可不需要精确到网段号码-->

network 192.168.3.0 0.0.0.255
network 192.168.4.0 0.0.0.255
import direct

inter vlan 1
ospf cost 100
inter vlan 3
ospf cost 200

**[S2]**

ip route-static 192.168.5.0 24 202.112.1.1
ip route-static 192.168.6.0 24 202.112.2.1

**[R1]**

ip route-static 211.100.2.0 24 202.112.1.2
import route-static

router id 2.2.2.2
ospf
area 0 
network 192.168.0.0 0.0.0.255
network 192.168.3.0 0.0.0.255

inter G0/1
ospf cost100
inter S0/0
ospf cost 200

**[R2]**

ip route-static 211.100.2.0 24 202.112.2.2
import route-static

router id 3.3.3.3
ospf
area 0
network 192.168.0.0 0.0.0.255
network 192.168.4.0 0.0.0.255

inter G0/1
ospf cost 200
inter S0/0
ospf cost 200

## BGP P 230

### exp 1 BGP的i基本分析 图8-5 P236

**[R1]**
undo inter vlan 1
inter G0/0
undo ip addr
ip addr 1.1.1.1 16
inter loop 1
ip addr 5.5.5.5 32

bgp 100
peer 1.1.1.2 as-number 300

<!--ipv4单播地址簇-->

address-family ipv4 unicast 

peer 1.1.1.2 enable

network 5.5.5.5 32 

**[S1]**

vlan 2
port G1/0/1
vlan 3
port G1/0/2
inter vlan 2
ip addr 2.1.1.2 16
inter vlan 3
ip addr 3.1.1.1 16

bgp 300
peer 1.1.1.1 as-number 100
peer 3.1.1.2 as-number 300

address-family ipv4 unicast

peer 1.1.1.1 enable
peer 3.1.1.2 enable

<!--wrong-->

**peer 3.1.1.2 next-local-hop**

peer 3.1.1.2 next-hop-local

**[S2]**

vlan 3
port G1/0/2
vlan 2 
port G1/0/1
inter vlan 3
ip addr 3.3.1.1 16
inter vlan 2
ip addr 2.2.1.1 16

bgp 300
peer 3.1.1.1 as-number 300
peer 2.1.1.2 as-number 200

address-family ipv4 unicast

peer 3.1.1.1 enable
peer 2.1.1.2  enable

peer 3.1.1.1 next-hop-local

**[R2]**
undo inter vlan 1
inter G0/0
undo ip addr
ip addr 2.1.1.2 16
inter loop 1
ip addr 4.4.4.4 32

bgp 200
peer 2.1.1.1 as-number 300

address-family ipv4 unicast

peer 2.1.1.1 enable

network 4.4.4.4 32

<R1>

debug bgp event
terminal debugging
reset bgp all

dis bgp peer

### exp 2 BGP的路由聚合 P240

**[R1]**

inter loop 2
ip addr 192.168.0.1 24
inter loop 3
ip addr 192.168.1.1 24
bgp 100
network 192.168.0.1 0.0.0.255
network 192.168.1.1 0.0.0.255

bgp 100
address-family ipv4 unicast
aggregate 192.168.0.0 255.255.240.0

undo aggregate 192.168.0.0 255.255.240.0 

bgp 100
address-family ipv4 unicast
aggregate 192.168.0.0 255.255.240.0 detail-suppressed

### exp 3 BGP的基本路由属性分析 P241

**[S1]**

bgp 300

import-route direct

**[S2]**

bgp 300
network 6.6.6.6 0.255.255.255

import-route direct

**[R2]**

dis bgp rou

### exp 4 BGP的路由策略 P246

reset BGP
reset bgp all

<!--基于acl-->

**[S2]**

acl number 2001
rule 0 deny source 5.0.0.0 0.255.255.255
rule 1 permit source 0.0.0.0 255.255.255.255
quit

bgp 300
peer 2.1.1.2 filter-policy 2001 export

<!--基于AS-Path-->

**[S1]**

<!--设置拒绝来自AS200的路由-->

ip as-path 1 deny \b200$

<!--设置允许本AS的路由-->

ip as-path 1 permit ^$

bgp 300
address-family ipv4 unicast
peer 1.1.1.1 as-path-acl export

<!--基于Route Policy-->

**[S1]**

acl basic 2001
rule 0 deny source 6.0.0.0 0.255.255.255.255
rule 1 permit source any
quit

route-policy deny6 permit node 10  //deny6是路由策略的名字，数字10是节点序

if match ip address acl 2001
apply cost 888
quit

bgp 300
address-family ipv4 unicast
peer 1.1.1.1 route-policy  deny6 export



### BGP设计实验 

#### exp 2 路由策略实验 图8-13 P254

**[S1]**

ip as-path 1 deny \b300$
ip as-path 1 permit ^$
ip as-path 1 permit \b400$



#### exp 3 Local-preference和Med属性实验 图8-14 P255

Local-preference仅仅在IBGP对等体之间交换，不通告给其他AS。判断流量离开AS时的最佳路由

Med判断流量进入AS时的最佳路由，当一个BGP路由器通过不同的EBGP对等体得到目的地址相同但下一跳不同的多条路由时，在其他条件相同的情况下，将优先选择MED属性值较小者。

local-preference 默认100 越大越好
med 默认 0 越小越好

**[R2]**
bgp 100
address-family ipv4 unicast
default  local-preference 150 

**[R1]**

bgp 100
address-family ipv4 unicast
default med 10
