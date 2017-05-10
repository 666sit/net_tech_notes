# MPLS
Multi-Protocol Label Switching，多协议标签交换。其应用最广的为MPLS VPN以及MPLS TE。
## 1. IOS平台交换机制
分为进程交换、快速交换和CEF（Cisco Express Forwarding，思科快速转发）
进程交换：路由器对每个报文进行route表和arp表的查询。缺点消耗CPU资源以及增加延迟，优点支持各种负载均衡。
快速交换：路由器会对报文进行分类，将去往相同目的地的报文分到一类。对每一类的第一个报文进行route表和arp表的查询，将结果存到cache中。后续报文会查看cache进行转发。缺点：第一个报文还是需要进程查表，无法实现基于报文的负载均衡。优点：比进程交换速度快。
CEF：开启cef之后，路由器会生成两张表被ASIC调用：邻接表和FIB（Forwarding Information Base，转发信息库）。FIB为优化后的路由表，邻接表为优化后的二层表（ARP表）。
标签是基于FIB分发的。
2.  名词解释
FEC, Forwarding Equivalent Class，转发等价类，具有相同属性的一类（报文），这种相同的属性可以是：L3 VPN地址，相同的目的地址。可以理解为一条路由条目，对应一个标签。
LDP, Label Distribution Protocol，标签分发协议，基于TCP/UDP 646端口，与TDP相比，该协议支持认证。该协议用于标签的分发。
LIB, Label Information Base，标签信息库，是一种拓扑表，存放路由条目与标签的映射关系。
LFIB, Label Forwarding Information Base，标签转发信息库，发送带标签的报文时查询入标签与出标签映射的表。
LSR, Label Switch Router，也成为P路由器。
3.  MPLS体系结构
转发标签的协议有：LDP，BGP（MPLS VPN），RSVP（MPLS TE）
 
3.1.    标签
 
Label：20 bit，取值范围是16 ~ 220-1
EXP：3 bit，实验位，作QoS
S：1 bit，栈底标签标志位
TTL：8 bit，防止出现环路
 
该报文在二层到三层报文之间。以太网的类型值为0x8847（单播）, 0x8848（组播和广播）
3.2.    LSR以及LSR Edge转发流程
LSR（P路由器），LSR Edge（PE路由器）
 
 
3.3.    分发标签的过程
1，通过路由协议建立IP路由表；
2，为路由表中的每个条目分发标签，除了BGP路由条目；
3，传递给其他LSR，标签只具有本地意义；
4，建立LIB，LFIB以及FIB供数据传递。
加入LFIB的过程：
LDP建立邻居是使用环回口建立邻居，并且该环回口需要路由可达，路由器会将自己的更新源地址以及所有物理接口的地址告诉邻居路由器，邻居路由器会将这些信息存入到邻居表中。
 
当B收到C的标签47以及E的标签26时，B会查看自己的邻居表以及FIB，确定C的标签加入到LFIB中。
MPLS分发标签会将标签分发给目的地的上游和上游的LSR通告。
3.4.    PHP
PHP（Penultimate Hop Popping，倒数第二跳弹出），关于一个目的网段的最后一跳路由器对目的网段不会发送常规标签，而会发送3号标签，意为弹出标签。倒数第二条路由器收到该标签后，并将其加入LFIB，当收到去往目的网段的数据包时，会执行弹出标签的操作，以常规报文发送给最后一跳路由器，最后一跳路由器只需查询FIB表就可以将报文进行转发。
最后一条路由器的定义：对去往目的网段接口没有启用MPLS，或者去往该接口的下一跳没有LDP邻居的路由器。
3.5.    数据转发
 
路由器A收到邻居B发送来的标签，会将其加入FIB表和LFIB表。A收到去往X网络的报文，会查看其FIB表，然后对其加入标签发送给B，B路由器查询LFIB将标签去除，并且在查询FIB表将其转发给D。
3.6.    配置
0. 启用IGP
1. 启用cef，ip cef
2. 开启mpls，接口上使用mpls ip
3. 修改router-id（可选），全局模式下mpls ldp router-id loopback 0 force
4. 修改接口MTU：mpls mtu 1504（VPN修改为1508，TE修改为1512）
5. 相关show命令：
查看邻居show mpls ldp neighbor
查看fib：show ip cef [details]
查看lib：sh mpls ldp bindings
查看lfib：sh mpls forwarding-table
6. 高级命令：
修改Label范围：mpls label range 101 150
不通告标签：no mpls ldp advertise-labels
通告标签给某些ACL
mpls ldp advertise-labels for 10 to 20
acl 10为路由条目
acl20 为通告给邻居的接口范围
4.  MPLS VPN
VRF（Virtual Routing Forwarding），一个VRF就是虚拟的路由器，可以逻辑的隔离路由。该例中，一个VRF就是一个MPLS VPN中的实例（进程），PE创建了VRF之后，要与相应的端口进行关联。VRF只是本地有意义。
RD，Route Distinguisher，区分公司的的路由条目，RD只具有本地意义。不同VRF的路由条目通过该属性区分，RD+IPv4路由为VPNv4路由，可以使用MP BGP传递这种路由。
RT, Route Target，BGP community属性中的扩展属性，PE路由器收到VPNv4路由之后，会加上相应的RT之后，通告给BGP邻居，然后进行私网路由传递。根据target的不同，传递给不同的RD。
5.  MPLS VPN配置实例
MPLS

----------------------------------------------------------
1. topo

R4(0/0)----\                                 /----(0/0)R5
           (1/0)                          (1/0)
             R2(0/0)---(0/0)R1(0/1)---(0/1)R3
           (1/1)                          (1/1)
R6(0/0)----/                                 \----(0/0)R7

----------------------------------------------------------
2. ip routing

R1:
interface Loopback0
 ip address 1.1.1.1 255.255.255.0
 no shutdown 
interface FastEthernet0/0
 ip address 12.1.1.1 255.255.255.0
 no shutdown 
interface FastEthernet0/1
 ip address 13.1.1.1 255.255.255.0
 no shutdown 
router rip
 version 2
 network 0.0.0.0
 no auto-summary

R2:
interface Loopback0
 ip address 2.2.2.2 255.255.255.0
 no shutdown 
interface FastEthernet0/0
 ip address 12.1.1.2 255.255.255.0
 no shutdown 
router rip
 version 2
 network 2.0.0.0
 network 12.0.0.0
 no auto-summary

R3:
interface Loopback0
 ip address 3.3.3.3 255.255.255.0
 no shutdown 
interface FastEthernet0/1
 ip address 13.1.1.3 255.255.255.0
 no shutdown 
router rip
 version 2
 network 3.0.0.0
 network 13.0.0.0
 no auto-summary

R4:
interface Loopback0
 ip address 4.4.4.4 255.255.255.0
 no shutdown
interface FastEthernet0/0
 ip address 24.1.1.4 255.255.255.0
 no shutdown

R5:
interface Loopback0
 ip address 5.5.5.5 255.255.255.0
 no shutdown 
interface FastEthernet0/0
 ip address 35.1.1.5 255.255.255.0
 no shutdown 

R6:
interface Loopback0
 ip address 6.6.6.6 255.255.255.0
 no shutdown 
interface FastEthernet0/0
 ip address 26.1.1.6 255.255.255.0
 no shutdown 

R7:
interface Loopback0
 ip address 7.7.7.7 255.255.255.0
 no shutdown 
interface FastEthernet0/0
 ip address 37.1.1.7 255.255.255.0
 no shutdown

----------------------------------------------------------
3. mpls

R1:
interface range FastEthernet 0/0-1
 mpls ip

R2:
interface FastEthernet 0/0
 mpls ip

R3:
interface FastEthernet 0/1
 mpls ip

sh mpls ldp nei
sh mpls ldp binding
sh mpls forwarding-table

----------------------------------------------------------
4. vrf

R2:
ip vrf r45
 rd 10:10
 route-target export 10:10
 route-target import 10:10
ip vrf r67
 rd 20:20
 route-target export 20:20
 route-target import 20:20
interface FastEthernet1/0
 ip vrf forwarding r45
 ip address 24.1.1.2 255.255.255.0
 no shutdown 
int fa1/1
 ip vrf forwarding r67
 ip add 26.1.1.2 255.255.255.0
 no shutdown 

R3:
ip vrf r45
 rd 10:10
 route-target export 10:10
 route-target import 10:10
ip vrf r67
 rd 20:20
 route-target export 20:20
 route-target import 20:20
interface FastEthernet1/0
 ip vrf forwarding r45
 ip address 35.1.1.3 255.255.255.0
 no shutdown 
interface FastEthernet1/1
 ip vrf forwarding r67
 ip address 37.1.1.3 255.255.255.0
 no shutdown 

----------------------------------------------------------
5. bgp

R2:
router bgp 1
 bgp router-id 2.2.2.2
 no bgp default ipv4-unicast
 neighbor 3.3.3.3 remote-as 1
 neighbor 3.3.3.3 update-source Loopback0
 address-family vpnv4
  neighbor 3.3.3.3 activate
  neighbor 3.3.3.3 send-community both

R3:
router bgp 1
 bgp router-id 3.3.3.3
 no bgp default ipv4-unicast
 neighbor 2.2.2.2 remote-as 1
 neighbor 2.2.2.2 update-source Loopback0
 address-family vpnv4
  neighbor 2.2.2.2 activate
  neighbor 2.2.2.2 send-community both

sh ip bgp all summary

----------------------------------------------------------
6. inner igp

R2:
router ospf 110 vrf r45
 router-id 2.2.2.2
 redistribute bgp 1 subnets
 network 24.1.1.2 0.0.0.0 area 0
router eigrp 3000
 address-family ipv4 vrf r67 autonomous-system 90
  redistribute bgp 1 metric 10000 100 255 1 1500
  network 26.1.1.2 0.0.0.0
router bgp 1
 address-family ipv4 vrf r45
  redistribute ospf 110 match internal external 1 external 2
 address-family ipv4 vrf r67
  redistribute eigrp 90 metric 10

R3:
router rip
 address-family ipv4 vrf r45
  redistribute bgp 1 metric transparent
  network 35.0.0.0
  no auto-summary
router ospf 110 vrf r67
 router-id 3.3.3.3
 redistribute bgp 1 subnets
 network 37.1.1.3 0.0.0.0 area 0
router bgp 1
 address-family ipv4 vrf r45
  redistribute rip metric 10
 address-family ipv4 vrf r67
  redistribute ospf 110 match internal external 1 external 2

R4:
router ospf 110
 router-id 4.4.4.4
 network 0.0.0.0 255.255.255.255 area 0

R5:
router rip
 version 2
 network 0.0.0.0
 no auto-summary

R6:
router eigrp 90
 network 0.0.0.0

R7:
router ospf 110
 router-id 7.7.7.7
 network 0.0.0.0 255.255.255.255 area 0






