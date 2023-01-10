# sniffer工具介绍

FortiGate具备完善的sniffer功能，可以基于指定接口或所有接口附带过滤条件进行抓包。

1. **Sniffer命令如下**：

   ```
   diagnose sniffer packet <interface> <filter> <verbose> <count> <tsformat> <frame size>
   
   <interface>    可以指定一个具体的interface名字，比如"port1"或所有接口"any"
   
   <filter>       非常强大的过滤功能，可以过滤出你想要过滤的流，过滤条件需要引号括起来'filter'或者"filter",类似tcpdump语法
   '[[src|dst] host {<host1_ipv4>}] 
   [and|or] [[src|dst] host <host2_ipv4>}] 
   [and|or] [[arp|ip|gre|esp|udp|tcp] port <port1_int>] 
   [and|or] [[arp|ip|gre|esp|udp|tcp] port <port2_int>]'
   如过滤单个IP(该ip可以是源，也可以是目的)：'host 1.1.1.1'
   如过滤源ip和目的： host 1.1.1.1 and host 2.2.2.2
   如过滤目的ip和端口：host 2.2.2.2 and tcp port 80
   如过滤源ip和协议： host 1.1.1.1 and icmp
   
   <verbose>      不同级别的不同内容展示，按需定义sniffer级别
   1: print header of packets (只输出报文头部内容)
   2: print header and data from ip of packets (输出从ip头部开始，包含数据内容) 
   3: print header and data from ethernet of packets  (输出从ethernet头部开始，包含数据内容) 
   4: print header of packets with interface name (只输出报文头部和进出接口，使用最多） 
   5: print header and data from ip of packets with interface name (输出从ip头部开始，包含数据内容和进出接口) 
   6: print header and data from ethernet of packets (if available) with intf name （从ethernet头部开始，包含数据内容和进出接口，使用最多） 
   
   <count>        抓包个数，到达指定个数后停止sniffer，如100表示抓100个包
   0：不限制抓包的数量，默认
   
   <tsformat>     时间戳格式
   a: UTC绝对时间，yyyy-mm-dd hh:mm:ss.ms
   l:绝对本地时间，yyyy-mm-dd hh:mm:ss.ms
   otherwise:相对于嗅探的开始，ss.ms，默认
   
   <frame size> 设置截断前帧的大小，默认是接口MTU
   ```

2. **Sniffer命令示例**

   ```
   diagnose sniffer packet port1 "host 172.16.1.100" 1                       //抓port1上172.16.1.100的包 
   diagnose sniffer packet any "host 172.16.1.100 and icmp" 4                //抓172.16.1.100的ICMP包 
   diagnose sniffer packet any "host 172.16.1.100 and tcp" 4                 //抓172.16.1.100的TCP包 
   diagnose sniffer packet any "host 172.16.1.100 and udp" 4                 //抓172.16.1.100的UDP包   
   diagnose sniffer packet any "host 172.16.1.100 and tcp port 443" 4        //抓172.16.1.100且https的包 
   diagnose sniffer packet any "host 172.16.1.100 and !tcp port 22" 4        //抓172.16.1.100且非ssh的包 
   diagnose sniffer packet any "host 172.16.1.100 and port 10000" 4          //抓172.16.1.100且TCP/UDP-Port=10000的包 
   diagnose sniffer packet any "host 114.114.119.119 and icmp" 4 0 a         // 携带0表示不限制抓包个数，携带a表示携带抓包的时间戳，a 表示不携带是时区的绝对时间
   diagnose sniffer packet any "host 114.114.119.119 and icmp" 4 0 l         // 携带0表示不限制抓包个数，携带l表示携带抓包的时间戳，l 表示携带是时区的绝对时间，通常我们使用这个参数，和设备保持一致
   diagnose sniffer packet any 'vlan 10 and host 10.1.1.1 and port 80' 4                       //带vlan tag的包
   diagnose sniffer packet any "host 172.16.1.100 or host 172.16.1.101 and port 1701" 4        //and和or组合 
   diagnose sniffer packet internal "tcp[13]&2!=0 and port 23" 4                               //SYN置位包 
   diagnose sniffer packet internal "tcp[13]&4!=0" 4                                           //RST置位 
   diagnose sniffer packet internal "net 4.2.2.0/24" 4                                         //抓某一网段的包 
   diagnose sniffer packet internal "(ether[6:4]=0x00090f89) and (ether[10:2]=0x10ea)" 4       //根据MAC源地址抓包 
   diagnose sniffer packet any "ip[2:2] > 1500" 4                                              //观察大于1500的帧
   ```

3. **抓包分析**

   ```
   ping的请求报文是echo request，响应报文是echo reply
   # diagnose sniffer packet any 'host 192.168.2.10 and icmp' 4
   interfaces=[any]
   filters=[host 192.168.2.10 and icmp]
   in表示FortiGate收到一个报文，out表示FortiGate发出一个报文
   
   7.074724 port5 in 192.168.1.10 -> 192.168.2.10: icmp: echo request
   FortiGate从port5收到 192.168.1.10发送给192.168.2.10 ping的请求报文echo request
   
   7.074748 port6 out 192.168.1.10 -> 192.168.2.10: icmp: echo request
   FortiGate将 192.168.1.10发送给192.168.2.10 ping的请求报文echo request从port6转发出去了
   
   7.075033 port6 in 192.168.2.10 -> 192.168.1.10: icmp: echo reply
   FortiGate从port6收到 192.168.2.10发送给192.168.1.10 ping的响应报文echo reply
   
   7.075042 port5 out 192.168.2.10 -> 192.168.1.10: icmp: echo reply
   FortiGate将 192.168.2.10发送给192.168.1.10 ping的响应报文echo reply从port5转发出去了
   
   从上面的抓包来看，FortiGate成功转发了192.168.1.10和192.168.2.10之间ping的请求和响应报文
   
   TCP报文交互之前都需要完成3次握手
   # diagnose sniffer packet any 'host 192.168.2.10 and port 22' 4
   interfaces=[any]
   filters=[host 192.168.2.10 and port 22]
   in表示FortiGate收到一个报文，out表示FortiGate发出一个报文
   
   12.009383 port5 in 192.168.1.10.46028 -> 192.168.2.10.22: syn 486804292 
   FortiGate从port5收到 192.168.1.10发送给192.168.2.10 的syn报文
   
   12.009752 port6 out 192.168.1.10.46028 -> 192.168.2.10.22: syn 486804292
   FortiGate将 192.168.1.10发送给192.168.2.10 的syn报文从port6转发出去了
   
   12.009957 port6 in 192.168.2.10.22 -> 192.168.1.10.46028: syn 1279904517 ack 486804293 
   FortiGate 从port6收到 192.168.2.10发送给192.168.1.10 的 syn ack报文
   
   12.010120 port5 out 192.168.2.10.22 -> 192.168.1.10.46028: syn 1279904517 ack 486804293 
   FortiGate将 192.168.2.10发送给192.168.1.10 的syn ack报文从port5转发出去了
   
   12.010233 port5 in 192.168.1.10.46028 -> 192.168.2.10.22: ack 1279904518
   FortiGate从port5收到 192.168.1.10发送给192.168.2.10 的ack报文
   
   12.010245 port6 out 192.168.1.10.46028 -> 192.168.2.10.22: ack 1279904518 
   FortiGate将 192.168.1.10发送给192.168.2.10 的ack报文从port6转发出去了
   
   从上面的抓包来看，FortiGate成功转发了192.168.1.10和192.168.2.10之间tcp三次握手的报文
   ```

   

