
## 1. 修改iptables规则之后，能不能立即对已经存在的数据流生效？ 

要看数据流是不是走CPU转发，走CPU转发的包可以立即应用新增/修改的iptables 规则。
如果发现修改了iptables规则之后，对当前的数据流不起作用，说明当前的数据流可能是走硬件转发。

观察 从LAN侧PC(192.168.1.3) ping WAN 侧server (61.1.1.1)

- Qualcomm板子: 所有Ping包走CPU转发，在Beacon 10上增加规则 iptables -I FORWORD -s 192.168.1.3 -p icmp -j DROP 之后， ping包立刻不通了。
- Realtek板子: 只有第一个Ping包走CPU转发，后续ping包都走硬件转发，看conntrack表的统计可以看出。 增加规则  iptables -I FORWORD -s 192.168.1.3 -p icmp -j DROP 之后 , ping包还通的。 
应该需要等硬件转发表过期之后，ping包才会不通。 

修改了iptables能不能立即生效，其实跟有没有conntrack表没有关系，主要是看报文是不是走CPU。那为什么Realtek上增加了iptables之后flush conntrack表，iptables 就会立即生效呢？ 
可能是硬件转发和conntrack之间有某种同步机制，不得而知。 

数据包走CPU还是硬件转发，每个芯片有自己的实现。

## 2. NAT表只有第一个包会走，后面的同一个数据流的包上CPU也不再走了。

iptables -t nat -nvL POSTROUTING
Chain POSTROUTING (policy ACCEPT 27 packets, 2096 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    1    60 MASQUERADE  all  --  *      eth0    192.168.18.0/24      0.0.0.0/0   
