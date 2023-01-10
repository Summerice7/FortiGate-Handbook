# Lowest Cost (SLA)

## 概念介绍

### SLA

Service-Level Agreement，服务品质协议，是服务提供者和客户之间的一个协议，用来保证可计量的网络性能达到所定义的服务品质。SD-WAN的选路基于这个SLA品质标准来判断，SD-WAN规则保障让流量一直走符合SLA品质的链路出去。从而达到业务/客户的SLA品质要求。

### SLA-Target

1. SD-WAN可定义SLA-Targets设置最低保障的延迟、抖动和丢包率，一旦超过SLA-Targets提供的保障最低值，则立即切换另外一条线路，以确保持续提供SLA-Targets品质级别的服务。

2. SLA Target 有三种类型的判断阈值：

   - Latency (ms)

   - Jitter (ms)

   - Packet Loss (%)

   <img src="../../../../images/image-20230104113020731.png" alt="image-20230104113020731" style="zoom:50%;" />

   ```
   config system sdwan
   	config health-check
       edit "114_Check"
           set server "114.114.114.114"
           set members 0
           config sla
               edit 1
                   set latency-threshold 200
                   set packetloss-threshold 2
               next
               edit 2
                   set latency-threshold 250
                   set jitter-threshold 10
                   set packetloss-threshold 5
               next
               edit 3
                   set latency-threshold 300
                   set jitter-threshold 15
                   set packetloss-threshold 8
               next
           end
       next
   end
   ```

3. **Lowest Cost (SLA) 和Maximize Bandwidth (SLA)都需要调用SLA Target，基于SLA Target进行相关的判断和选路。**

## 选路原则

1. 只有满足SLA-Targets的出接口才有机会被选中，如果低于SLA-Targets的接口将会被移除选中列表中。
2. 如果有多个接口满足SLA-Targets，那么就选择SD-WAN规则配置接口顺序靠前的接口用于转发SD-WAN流量。
3. 同时只能一个出接口被优选，用于SD-WAN流量的转发。
4. Lowest Cost（SLA）完全基于SLA-Targets进行工作，因此首先需要在SD-WAN状态检查里面配置具体的SLA-Targets标准，然后在SD-WAN规则中选择相应的SLA-Targets，只有符合选择的SLA-Targets标准的出口，才会被SD-WAN规则所计算并用于出口数据的转发，将选择符合SLA-Targets且接口顺序靠前的出口用于数据转发，同时只有一个接口用于数据的转发。

## 配置举例

### 网络拓扑

<img src="../../../../images/image-20230104135906852.png" alt="image-20230104135906852" style="zoom:50%;" />

### 配置步骤

1. SD-WAN接口成员定义。

   <img src="../../../../images/image-20230103184552085.png" alt="image-20230103184552085" style="zoom:50%;" />

   ```
   config system sdwan
       set status enable
       config zone
           edit "virtual-wan-link"
           next
       end
       config members
           edit 1
               set interface "port2"
               set gateway 202.100.1.192
           next
           edit 2
               set interface "port3"
               set gateway 101.100.1.192
           next
           edit 3
               set interface "port4"
               set gateway 111.100.1.192
           next
           edit 4
               set interface "PPPOE1_DR_PENG"
           next
       end
    end
   ```

2. 配置SD-WAN关联的默认路由。

   <img src="../../../../images/image-20230103184652006.png" alt="image-20230103184652006" style="zoom:50%;" />

   ```
   config router static
       edit 1
           set distance 1
           set sdwan-zone "virtual-wan-link"
       next
   end
   ```

3. 配置健康检查，SLA目标监控阿里云的延迟及丢包率。

   <img src="../../../../images/image-20230104141022511.png" alt="image-20230104141022511" style="zoom:50%;" />

   ```
   config system sdwan
   config health-check
       edit "Aliyun"
           set server "cn.aliyun.com"
           set members 1 2 3
           config sla
               edit 1
                   set link-cost-factor latency packet-loss
                   set latency-threshold 120
                   set packetloss-threshold 2
               next
           end
       next
   end
   ```

   <img src="../../../../images/image-20230104141249855.png" alt="image-20230104141249855" style="zoom:50%;" />

4. 配置SD-WAN规则，目标为阿里云相关Internet服务，引用上步配置的SLA Target。

   <img src="../../../../images/image-20230104141621090.png" alt="image-20230104141621090" style="zoom:50%;" />

   ```
   config system sdwan
   config service
       edit 1
           set name "To_Aliyun"
           set mode sla
           set src "LAN_192.168.10.0"
           set internet-service enable
           set internet-service-name "Alibaba-Alibaba.Cloud" "Alibaba-DNS" "Alibaba-ICMP" "Alibaba-NTP" "Alibaba-SSH" "Alibaba-Web"
           config sla
               edit "Aliyun"
                   set id 1
               next
           end
           set priority-members 1 2 3
       next
   end
   ```

5. 配置安全策略允许SD-WAN流量。

   <img src="../../../../images/image-20230104141849533.png" alt="image-20230104141849533" style="zoom:50%;" />

   ```
   config firewall policy
       edit 1
           set name "To_Internet"
           set srcintf "port8"
           set dstintf "virtual-wan-link"
           set action accept
           set srcaddr "LAN_192.168.10.0"
           set dstaddr "all"
           set schedule "always"
           set service "ALL"
           set utm-status enable
           set ssl-ssh-profile "certificate-inspection"
           set webfilter-profile "default"
           set application-list "default"
           set logtraffic all
           set nat enable
       next
   end
   ```

### 结果验证

1. 查看健康检查状态，三条链路均为alive状态。

   ```
   SDWAN # diagnose sys sdwan health-check 
   Health Check(Aliyun): 
   Seq(1 port2): state(alive), packet-loss(0.000%) latency(19.893), jitter(0.531), bandwidth-up(9999999), bandwidth-dw(9999999), bandwidth-bi(19999998) sla_map=0x1
   Seq(2 port3): state(alive), packet-loss(0.000%) latency(19.704), jitter(0.408), bandwidth-up(9999999), bandwidth-dw(9999999), bandwidth-bi(19999998) sla_map=0x1
   Seq(3 port4): state(alive), packet-loss(0.000%) latency(19.719), jitter(0.556), bandwidth-up(9999999), bandwidth-dw(9999999), bandwidth-bi(19999998) sla_map=0x1
   ```

2. 查看SD-WAN规则状态，出接口顺序为port2，port3，port4。

   ```
   SDWAN # diagnose sys sdwan service 
   
   Service(1): Address Mode(IPV4) flags=0x200 use-shortcut-sla
     Gen(11), TOS(0x0/0x0), Protocol(0: 1->65535), Mode(sla), sla-compare-order
     Members(3): 
       1: Seq_num(1 port2), alive, sla(0x0), gid(0), cfg_order(0), cost(0), selected
       2: Seq_num(2 port3), alive, sla(0x0), gid(0), cfg_order(1), cost(0), selected
       3: Seq_num(3 port4), alive, sla(0x0), gid(0), cfg_order(2), cost(0), selected
     Internet Service(6): Alibaba-Alibaba.Cloud(6881402,0,0,0) Alibaba-DNS(6881283,0,0,0) Alibaba-ICMP(6881282,0,0,0) Alibaba-NTP(6881288,0,0,0) Alibaba-SSH(6881286,0,0,0) Alibaba-Web(6881281,0,0,0) 
     Src address(1): 
           192.168.10.0-192.168.10.255
   ```

3. 查看策略路由中SD-WAN规则的列表，出接口顺序为port2，port3，port4。

   ```
   SDWAN # diagnose firewall proute list 
   list route policy info(vf=root):
   
   id=2132213761(0x7f170001) vwl_service=1(To_Aliyun) vwl_mbr_seq=1 2 3 dscp_tag=0xff 0xff flags=0x0 tos=0x00 tos_mask=0x00 protocol=0 sport=0-65535 iif=0(any) dport=1-65535 path(3) oif=4(port2) oif=5(port3) oif=6(port4) 
   source(1): 192.168.10.0-192.168.10.255 
   destination wildcard(1): 0.0.0.0/0.0.0.0 
   internet service(6): Alibaba-Alibaba.Cloud(6881402,0,0,0) Alibaba-DNS(6881283,0,0,0) Alibaba-ICMP(6881282,0,0,0) Alibaba-NTP(6881288,0,0,0) Alibaba-SSH(6881286,0,0,0) Alibaba-Web(6881281,0,0,0) 
   hit_count=0 last_used=2023-01-03 22:16:17
   ```

4. 查看路由表。

   ```
   SDWAN # get router info routing-table all
   ...
   
   S*      0.0.0.0/0 [1/0] via 101.100.1.192, port3, [1/0]
                     [1/0] via 111.100.1.192, port4, [1/0]
                     [1/0] via 114.100.1.196, PPPOE1_DR_PENG, [1/0]
                     [1/0] via 202.100.1.192, port2, [1/0]
   ...
   ```

### 线路切换测试

1. 让port2的延迟超过SLA-Target。

   ```
   SDWAN # diagnose sys sdwan health-check
   Health Check(Aliyun): 
   Seq(1 port2): state(alive), packet-loss(1.000%) latency(187.502), jitter(8.489), bandwidth-up(9999999), bandwidth-dw(9999999), bandwidth-bi(19999998) sla_map=0x0
   Seq(2 port3): state(alive), packet-loss(1.000%) latency(48.102), jitter(9.499), bandwidth-up(9999999), bandwidth-dw(9999999), bandwidth-bi(19999998) sla_map=0x1
   Seq(3 port4): state(alive), packet-loss(1.000%) latency(48.508), jitter(8.618), bandwidth-up(9999999), bandwidth-dw(9999999), bandwidth-bi(19999998) sla_map=0x1
   ```

   <img src="../../../../images/image-20230104144127886.png" alt="image-20230104144127886" style="zoom:50%;" />

2. 查看SD-WAN策略状态，由于port2的延迟不满足SLA Target，出接口顺序变为port3，port4，port2。

   ```
   SDWAN # diagnose sys sdwan service
   
   Service(1): Address Mode(IPV4) flags=0x200 use-shortcut-sla
     Gen(6), TOS(0x0/0x0), Protocol(0: 1->65535), Mode(sla), sla-compare-order
     Members(3): 
       1: Seq_num(2 port3), alive, sla(0x1), gid(0), cfg_order(1), cost(0), selected    //port3将会优先选择wan2进行转发
       2: Seq_num(3 port4), alive, sla(0x1), gid(0), cfg_order(2), cost(0), selected
       3: Seq_num(1 port2), alive, sla(0x0), gid(0), cfg_order(0), cost(0), selected    //port2不满足的SLA，顺序置于最后面了
     Internet Service(6): Alibaba-Alibaba.Cloud(6881402,0,0,0) Alibaba-DNS(6881283,0,0,0) Alibaba-ICMP(6881282,0,0,0) Alibaba-NTP(6881288,0,0,0) Alibaba-SSH(6881286,0,0,0) Alibaba-Web(6881281,0,0,0) 
     Src address(1): 
           192.168.10.0-192.168.10.255
   ```

3. 查看策略路由列表中的SD-WAN规则，由于port2的延迟不满足SLA Target，出接口顺序变为port3，port4，port2。

   ```
   SDWAN # diagnose firewall proute list 
   list route policy info(vf=root):
   
   id=2132344833(0x7f190001) vwl_service=1(To_Aliyun) vwl_mbr_seq=2 3 1 dscp_tag=0xff 0xff flags=0x0 tos=0x00 tos_mask=0x00 protocol=0 sport=0-65535 iif=0(any) dport=1-65535 path(3) oif=5(port3) oif=6(port4) oif=4(port2)
   source(1): 192.168.10.0-192.168.10.255 
   destination wildcard(1): 0.0.0.0/0.0.0.0 
   internet service(6): Alibaba-Alibaba.Cloud(6881402,0,0,0) Alibaba-DNS(6881283,0,0,0) Alibaba-ICMP(6881282,0,0,0) Alibaba-NTP(6881288,0,0,0) Alibaba-SSH(6881286,0,0,0) Alibaba-Web(6881281,0,0,0) 
   hit_count=0 last_used=2023-01-03 22:16:17
   ```

4. 使用内网PC客户端访问阿里云相关资源，查看流量日志，可以看到流量切换到了port3。

   <img src="../../../../images/image-20230104144722040.png" alt="image-20230104144722040" style="zoom:50%;" />

#### 接口成员Cost值对选路的影响

1. SD-WAN接口成员的接口Cost值将影响Lowest Cost (SLA)的接口选择顺序，如果SD-WAN成员接口均达到了SLA Target，那么Cost值越小越优先，这将打破配置顺序而进行用户自定义的SD-WAN接口优先级。

2. 比如自定义：port2 Cost 222，port3 Cost 111，port4 Cost 333。此时Lowest Cost (SLA)将会优选Cost较小的WAN2，而忽略配置接口的顺序，Cost值优先进行比较。

   <img src="../../../../images/image-20230104145319205.png" alt="image-20230104145319205" style="zoom:50%;" />

3. 查看健康检查状态，三个接口成员均达到了SLA Target。

   ```
   SDWAN # diagnose sys sdwan health-check
   Health Check(Aliyun): 
   Seq(1 port2): state(alive), packet-loss(0.000%) latency(20.301), jitter(0.578), bandwidth-up(9999999), bandwidth-dw(9999999), bandwidth-bi(19999998) sla_map=0x1
   Seq(2 port3): state(alive), packet-loss(0.000%) latency(20.225), jitter(0.610), bandwidth-up(9999999), bandwidth-dw(9999999), bandwidth-bi(19999998) sla_map=0x1
   Seq(3 port4): state(alive), packet-loss(0.000%) latency(20.143), jitter(0.571), bandwidth-up(9999999), bandwidth-dw(9999999), bandwidth-bi(19999998) sla_map=0x1
   ```

4. 查看SD-WAN策略状态，Cost小的port3优先。

   ```
   SDWAN # diagnose sys sdwan service
   
   Service(1): Address Mode(IPV4) flags=0x200 use-shortcut-sla
     Gen(1), TOS(0x0/0x0), Protocol(0: 1->65535), Mode(sla), sla-compare-order
     Members(3): 
       1: Seq_num(2 port3), alive, sla(0x1), gid(0), cfg_order(0), cost(111), selected    //Cost小的port3优先
       2: Seq_num(1 port2), alive, sla(0x1), gid(0), cfg_order(2), cost(222), selected
       3: Seq_num(3 port4), alive, sla(0x1), gid(0), cfg_order(1), cost(333), selected
     Internet Service(6): Alibaba-Alibaba.Cloud(6881402,0,0,0) Alibaba-DNS(6881283,0,0,0) Alibaba-ICMP(6881282,0,0,0) Alibaba-NTP(6881288,0,0,0) Alibaba-SSH(6881286,0,0,0) Alibaba-Web(6881281,0,0,0) 
     Src address(1): 
           192.168.10.0-192.168.10.255
   ```

5. 查看策略路由列表中的SD-WAN规则，Cost小的port3优先。

   ```
   SDWAN # diagnose firewall proute list
   list route policy info(vf=root):
   
   id=2132869121(0x7f210001) vwl_service=1(To_Aliyun) vwl_mbr_seq=2 1 3 dscp_tag=0xff 0xff flags=0x0 tos=0x00 tos_mask=0x00 protocol=0 sport=0-65535 iif=0(any) dport=1-65535 path(3) oif=5(port3) oif=4(port2) oif=6(port4)    //Cost小的port3优先
   source(1): 192.168.10.0-192.168.10.255 
   destination wildcard(1): 0.0.0.0/0.0.0.0 
   internet service(6): Alibaba-Alibaba.Cloud(6881402,0,0,0) Alibaba-DNS(6881283,0,0,0) Alibaba-ICMP(6881282,0,0,0) Alibaba-NTP(6881288,0,0,0) Alibaba-SSH(6881286,0,0,0) Alibaba-Web(6881281,0,0,0) 
   hit_count=25 last_used=2023-01-03 22:49:28
   ```

#### 接口成员顺序对选路的影响

1. 将SD-WAN成员接口的cost全部改回0，如果port2、port3、port4 三者都无法满足SLA目标了，那么SD-WAN规则如何选择出接口呢？

2. 答案是：如果三者都无法满足SLA 目标值了，那么将会按照SD-WAN规则配置的接口顺序选择出接口，也就是以port2、port3、port4这样的顺序选择，port2将会优先转发数据。此时顺序优先。

3. 将port2、port3、port4的延迟调整超过SLA Target。

   ```
   SDWAN # diagnose sys sdwan health-check
   Health Check(Aliyun): 
   Seq(1 port2): state(alive), packet-loss(0.000%) latency(156.008), jitter(2.069), bandwidth-up(9999999), bandwidth-dw(9999999), bandwidth-bi(19999998) sla_map=0x0
   Seq(2 port3): state(alive), packet-loss(0.000%) latency(186.951), jitter(2.078), bandwidth-up(9999999), bandwidth-dw(9999999), bandwidth-bi(19999998) sla_map=0x0
   Seq(3 port4): state(alive), packet-loss(0.000%) latency(176.968), jitter(2.055), bandwidth-up(9999999), bandwidth-dw(9999999), bandwidth-bi(19999998) sla_map=0x0
   ```

   <img src="../../../../images/image-20230104150725105.png" alt="image-20230104150725105" style="zoom:50%;" />

4. 查看SD-WAN规则状态，全部不符合SLA Target的时候，按照配置的接口顺序进行选择出接口。

   ```
   SDWAN # diagnose sys sdwan service
   
   Service(1): Address Mode(IPV4) flags=0x200 use-shortcut-sla
     Gen(1), TOS(0x0/0x0), Protocol(0: 1->65535), Mode(sla), sla-compare-order
     Members(3): 
       1: Seq_num(1 port2), alive, sla(0x0), gid(0), cfg_order(0), cost(0), selected    //全部不符合的时候，按照配置的接口顺序进行选择出接口
       2: Seq_num(2 port3), alive, sla(0x0), gid(0), cfg_order(1), cost(0), selected
       3: Seq_num(3 port4), alive, sla(0x0), gid(0), cfg_order(2), cost(0), selected
     Internet Service(6): Alibaba-Alibaba.Cloud(6881402,0,0,0) Alibaba-DNS(6881283,0,0,0) Alibaba-ICMP(6881282,0,0,0) Alibaba-NTP(6881288,0,0,0) Alibaba-SSH(6881286,0,0,0) Alibaba-Web(6881281,0,0,0) 
     Src address(1): 
           192.168.10.0-192.168.10.255
   ```

5. 查看策略路由列表，全部不符合SLA Target的时候，按照配置的接口顺序进行选择出接口。

   ```
   SDWAN # diagnose firewall proute list
   list route policy info(vf=root):
   
   id=2133131265(0x7f250001) vwl_service=1(To_Aliyun) vwl_mbr_seq=1 2 3 dscp_tag=0xff 0xff flags=0x0 tos=0x00 tos_mask=0x00 protocol=0 sport=0-65535 iif=0(any) dport=1-65535 path(3) oif=4(port2) oif=5(port3) oif=6(port4)    //全部不符合的时候，按照配置的接口顺序进行选择出接口
   source(1): 192.168.10.0-192.168.10.255 
   destination wildcard(1): 0.0.0.0/0.0.0.0 
   internet service(6): Alibaba-Alibaba.Cloud(6881402,0,0,0) Alibaba-DNS(6881283,0,0,0) Alibaba-ICMP(6881282,0,0,0) Alibaba-NTP(6881288,0,0,0) Alibaba-SSH(6881286,0,0,0) Alibaba-Web(6881281,0,0,0) 
   hit_count=25 last_used=2023-01-03 22:49:28
   ```

#### sla-compare-method  number对选路的影响

1. 上述情况的一个补充命令：如果全部不符合则相对于没有SLA Target了，这显然不够合理，还有优化的空间，这时候就引出了另外一个参数：set sla-compare-method  number。

2. SD-WAN规则可以调用多个SLA Target，然后优化计算的方法。

   <img src="../../../../images/image-20230104152257559.png" alt="image-20230104152257559" style="zoom:50%;" />

   ```
   config system sdwan
   config service
       edit 1
   ...
           config sla
               edit "Aliyun"
                   set id 1
               next
               edit "114_Check"
                   set id 1
               next
               edit "Default_DNS"
                   set id 1
               next
               edit "Default_Office_365"
                   set id 1
               next
           end
   ...
           set sla-compare-method order    //默认值，如果按此设置，多个SLA目标之间的逻辑关系是and，只要有其中一个SLA目标不符合，则将该接口剔除SD-WAN规则的选择，如果SLA全部失效，则按照配置的接口顺序进行选择出接口
   ...
       next
   end
   ```

3. 默认情况下，CLI下SLA的设置为set sla-compare-method order，如果按此设置，多个SLA目标之间的逻辑关系是and，只要有其中一个SLA目标不符合，则将该接口剔除SD-WAN规则的选择，如果SLA全部失效，则按照配置的接口顺序进行选择出接口。

   ```
   SDWAN (1) # set sla-compare-method 
   order     Compare SLA value based on the order of health-check.
   number    Compare SLA value based on the number of satisfied health-check.  Limits health-checks to only configured member interfaces.
   ```

4.  set sla-compare-method  numbe，此参数是SLA目标全部失效的一个补充，如果全部失效，则选择失败数较小的接口进行SD-WAN流量的转发，比如port2失败2个，port3失败3个，port4失败1个，则会选择port4进行数据的转发。

5. 我们配置多个服务器的SLA Target，在SD-WAN规则里面也调用多个SLA Target，然后虽然全部都失败了，但是可以选择符合SLA Target条件更多的接口作为SD-WAN规则的出接口，在全部失败的矮子中选择一个最优的出接口。当然我们需要设置不同的目的IP进行健康检查，同时配置不同的SLA条目，在SD-WAN规则调用的时候也只能调用不同服务器的SLA目标，才可以进行这样的进一步比较。
