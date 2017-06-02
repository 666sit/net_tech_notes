## 5. MPLS TE
### 5.1 TE
* TE（Traffic Engineering，流量工程）由于IGP选择的均为代价最小、距离最近的路由，所以导致链路利用率极不均衡的问题。TE对现有网络流量合理的规划和引导，实现资源的优化配置和提升网络性能。
### 5.2 IP TE
其使用广泛，但是非常粗糙，主要方法：
1. 利用IGP协议，改变metric或者cost值，过滤路由，或者LSA的方法
2. 利用BGP丰富的路由策略。
* 其优点为简单，缺点为相互影响严重。
### 5.3 MPLS TE概述
* 主要实现方式有：RSVP（Resource Reservation Protocol，资源预留协议）TE，CR LDP（Constraint-based Routing Label Distribution Protocol，基于路由受限标签分发协议）TE。这里只讨论RSVP TE。
* 必要条件
1. 支持P2P的LSP流量tunnel，tunnel中的LSP是固定的，故，报文进入tunnel之后，只能从tunnel另一端出来。
2. LSP tunnel的建立支持自动建立和手动建立；
3. 根据不同的优先级进行隧道抢占；
4. 支持预建立备份路径的功能；
5. 支持隧道随着网络环境的变化而重优化；
5. 支持LSP优先级隧道。
* 四大基本组件：信息发布、路径计算、信令、报文转发组件
* 扩展组件：FRR（Fast reRoute）、隧道的备份、宽带自动调整、路径的重优化

![mpls.te.implement.framework](https://github.com/Minions1128/net_tech_notes/blob/master/img/mpls.te.implementation.frameworks.jpg "mpls.te.implement.framework")

### 5.4 


检查mpls te环境ok
show mpls traffic-eng topology brief
还可以查看标签




信息发布：
