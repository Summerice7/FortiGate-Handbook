# 基于轮询的Maximize Bandwidth (SLA)

## 概念介绍

### SLA

Service-Level Agreement，服务品质协议，是服务提供者和客户之间的一个协议，用来保证可计量的网络性能达到所定义的服务品质。SD-WAN的选路基于这个SLA品质标准来判断，SD-WAN规则保障让流量一直走符合SLA品质的链路出去。从而达到业务/客户的SLA品质要求。

### SLA-Target

1. SD-WAN可定义SLA-Targets设置最低保障的延迟、抖动和丢包率，一旦超过SLA-Targets提供的保障最低值，则立即切换另外一条线路，以确保持续提供SLA-Targets品质级别的服务。

2. SLA Target 有三种类型的判断阈值：

   - Latency (ms)

   - Jitter (ms)

   - Packet Loss (%)

   <img src="../../../../../images/image-20230104113020731.png" alt="image-20230104113020731" style="zoom:50%;" />

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
2. 如果有多个接口满足SLA-Targets，这些接口将按照配置的负载方式进行负载均衡来转发SD-WAN的流量，以便达到带宽最大利用率的效果。
3. 负载方法支持round-robin（默认）、source-ip-based、source-dest-ip-based、inbandwidth、outbandwidth、bibandwidth。
4. Maximize Bandwidth (SLA) 完全基于SLA-Targets进行工作，因此首先需要在SD-WAN状态检查里面配置具体的SLA-Targets标准，然后再SD-WAN规则中选择相应的SLA-Targets，只有符合选择的SLA-Targets标准的出口，才会被SD-WAN规则所计算并用于出口数据的转发，符合SLA-Targets的接口都会被用于数据的转发，多个接口按照配置的负载方式进行负载均衡处理。

## 配置举例

### 网络拓扑

<img src="../../../../../images/image-20230104154112158.png" alt="image-20230104154112158" style="zoom:50%;" />

### 配置步骤

1. SD-WAN接口成员定义。

   <img src="../../../../../images/image-20230103184552085.png" alt="image-20230103184552085" style="zoom:50%;" />

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

   <img src="../../../../../images/image-20230103184652006.png" alt="image-20230103184652006" style="zoom:50%;" />

   ```
   config router static
       edit 1
           set distance 1
           set sdwan-zone "virtual-wan-link"
       next
   end
   ```

3. 配置健康检查，SLA目标监控阿里云的延迟及丢包率。

   <img src="../../../../../images/image-20230104141022511.png" alt="image-20230104141022511" style="zoom:50%;" />

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

   <img src="../../../../../images/image-20230104141249855.png" alt="image-20230104141249855" style="zoom:50%;" />

4. 配置SD-WAN规则，目标为阿里云相关Internet服务，引用上步配置的SLA Target，默认的负载方式为轮询，负载方式只能在CLI下修改。

   <img src="../../../../../images/image-20230104154559405.png" alt="image-20230104154559405" style="zoom:50%;" />

   ```
   config system sdwan
   config service
       edit 1
           set name "To_Aliyun"
           set mode load-balance
           set hash-mode round-robin    //默认负载方式为轮询，即基于会话的负载
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

   <img src="../../../../../images/image-20230104141849533.png" alt="image-20230104141849533" style="zoom:50%;" />

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
   Seq(1 port2): state(alive), packet-loss(0.000%) latency(29.467), jitter(1.361), bandwidth-up(9999998), bandwidth-dw(9999998), bandwidth-bi(19999996) sla_map=0x1
   Seq(2 port3): state(alive), packet-loss(0.000%) latency(29.235), jitter(1.343), bandwidth-up(9999997), bandwidth-dw(9999995), bandwidth-bi(19999992) sla_map=0x1
   Seq(3 port4): state(alive), packet-loss(0.000%) latency(29.169), jitter(1.304), bandwidth-up(9999998), bandwidth-dw(9999998), bandwidth-bi(19999996) sla_map=0x1
   ```

2. 查看SD-WAN规则状态。

   ```
   SDWAN # diagnose sys sdwan service
   
   Service(1): Address Mode(IPV4) flags=0x200 use-shortcut-sla
     Gen(1), TOS(0x0/0x0), Protocol(0: 1->65535), Mode(load-balance  hash-mode=round-robin)
     Members(3): 
       1: Seq_num(1 port2), alive, sla(0x1), gid(2), num of pass(1), selected
       2: Seq_num(2 port3), alive, sla(0x1), gid(2), num of pass(1), selected
       3: Seq_num(3 port4), alive, sla(0x1), gid(2), num of pass(1), selected
     Internet Service(6): Alibaba-Alibaba.Cloud(6881402,0,0,0) Alibaba-DNS(6881283,0,0,0) Alibaba-ICMP(6881282,0,0,0) Alibaba-NTP(6881288,0,0,0) Alibaba-SSH(6881286,0,0,0) Alibaba-Web(6881281,0,0,0) 
     Src address(1): 
           192.168.10.0-192.168.10.255
   ```

3. 查看策略路由中SD-WAN规则的列表，符合SLA目标的接口就会用于基于会话的负载均衡。

   ```
   SDWAN # diagnose firewall proute list
   list route policy info(vf=root):
   
   id=2133458945(0x7f2a0001) vwl_service=1(To_Aliyun) vwl_mbr_seq=1 2 3 dscp_tag=0xff 0xff flags=0x10 load-balance hash-mode=round-robin  tos=0x00 tos_mask=0x00 protocol=0 sport=0-65535 iif=0(any) dport=1-65535 path(3) oif=4(port2) num_pass=1 oif=5(port3) num_pass=1 oif=6(port4) num_pass=1 
   source(1): 192.168.10.0-192.168.10.255 
   destination wildcard(1): 0.0.0.0/0.0.0.0 
   internet service(6): Alibaba-Alibaba.Cloud(6881402,0,0,0) Alibaba-DNS(6881283,0,0,0) Alibaba-ICMP(6881282,0,0,0) Alibaba-NTP(6881288,0,0,0) Alibaba-SSH(6881286,0,0,0) Alibaba-Web(6881281,0,0,0) 
   hit_count=25 last_used=2023-01-03 22:49:28
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

5. 查看SD-WAN设备的流量日志，可以看到客户端访问阿里云相关的业务时，会根据轮询算法负载到三条线路上。

   <img src="../../../../../images/image-20230104160244313.png" alt="image-20230104160244313" style="zoom:50%;" />

### 线路切换测试

1. 让port2的延迟超过SLA-Target。

   ```
   SDWAN # diagnose sys sdwan health-check
   Health Check(Aliyun): 
   Seq(1 port2): state(alive), packet-loss(0.000%) latency(187.104), jitter(0.777), bandwidth-up(9999999), bandwidth-dw(9999999), bandwidth-bi(19999998) sla_map=0x0    //超过SLA的目标值
   Seq(2 port3): state(alive), packet-loss(0.000%) latency(28.857), jitter(0.705), bandwidth-up(9999999), bandwidth-dw(9999999), bandwidth-bi(19999998) sla_map=0x1
   Seq(3 port4): state(alive), packet-loss(0.000%) latency(28.824), jitter(0.750), bandwidth-up(9999999), bandwidth-dw(9999999), bandwidth-bi(19999998) sla_map=0x1
   ```

   <img src="../../../../../images/image-20230104161134990.png" alt="image-20230104161134990" style="zoom:50%;" />

2. 查看SD-WAN规则状态，port2不符合SLA目标，从接口列表中剔除，SLA置位为0x0。

   ```
   SDWAN # diagnose sys sdwan service
   
   Service(1): Address Mode(IPV4) flags=0x200 use-shortcut-sla
     Gen(1), TOS(0x0/0x0), Protocol(0: 1->65535), Mode(load-balance  hash-mode=round-robin)
     Members(3): 
       1: Seq_num(2 port3), alive, sla(0x1), gid(2), num of pass(1), selected
       2: Seq_num(3 port4), alive, sla(0x1), gid(2), num of pass(1), selected
       3: Seq_num(1 port2), alive, sla(0x0), gid(2), num of pass(0), selected  //不符合SLA目标，从接口列表中剔除，SLA置位为0x0
     Internet Service(6): Alibaba-Alibaba.Cloud(6881402,0,0,0) Alibaba-DNS(6881283,0,0,0) Alibaba-ICMP(6881282,0,0,0) Alibaba-NTP(6881288,0,0,0) Alibaba-SSH(6881286,0,0,0) Alibaba-Web(6881281,0,0,0) 
     Src address(1): 
           192.168.10.0-192.168.10.255
   ```

3. 查看策略路由列表，port2的num_pass为0，不用于数据转发。

   ```
   SDWAN # diagnose firewall proute list
   list route policy info(vf=root):
   
   id=2133458945(0x7f2a0001) vwl_service=1(To_Aliyun) vwl_mbr_seq=2 3 1 dscp_tag=0xff 0xff flags=0x10 load-balance hash-mode=round-robin  tos=0x00 tos_mask=0x00 protocol=0 sport=0-65535 iif=0(any) dport=1-65535 path(3) oif=5(port3) num_pass=1 oif=6(port4) num_pass=1 oif=4(port2) num_pass=0    //port2不用于数据转发
   source(1): 192.168.10.0-192.168.10.255 
   destination wildcard(1): 0.0.0.0/0.0.0.0 
   internet service(6): Alibaba-Alibaba.Cloud(6881402,0,0,0) Alibaba-DNS(6881283,0,0,0) Alibaba-ICMP(6881282,0,0,0) Alibaba-NTP(6881288,0,0,0) Alibaba-SSH(6881286,0,0,0) Alibaba-Web(6881281,0,0,0) 
   hit_count=86 last_used=2023-01-04 00:02:56
   ```

4. 查看SD-WAN设备的流量日志，客户端访问阿里云的流量被负载均衡到port3和port4上，不会负载到port2上。

   <img src="../../../../../images/image-20230104161826674.png" alt="image-20230104161826674" style="zoom:50%;" />

5. 如果port2、port3、port4 三者都无法满足SLA目标了，那么SD-WAN规则如何选择出接口呢？将三条链路的延迟均调整为无法满足SLA Target。

   ```
   SDWAN # diagnose sys sdwan health-check
   Health Check(Aliyun): 
   Seq(1 port2): state(alive), packet-loss(0.000%) latency(182.008), jitter(2.069), bandwidth-up(9999999), bandwidth-dw(9999999), bandwidth-bi(19999998) sla_map=0x0
   Seq(2 port3): state(alive), packet-loss(0.000%) latency(139.951), jitter(2.078), bandwidth-up(9999999), bandwidth-dw(9999999), bandwidth-bi(19999998) sla_map=0x0
   Seq(3 port4): state(alive), packet-loss(0.000%) latency(168.968), jitter(2.055), bandwidth-up(9999999), bandwidth-dw(9999999), bandwidth-bi(19999998) sla_map=0x0
   ```

   <img src="../../../../../images/image-20230104150725105.png" alt="image-20230104150725105" style="zoom:50%;" />

6. 答案是：如果三者都无法满足SLA 目标值了，那么还是将会以port2、port3、port4进行基于会话的负载均衡转发。

7. 查看SD-WAN规则状态，三条链路均被选中。

   ```
   SDWAN # diagnose sys sdwan service
   
   Service(1): Address Mode(IPV4) flags=0x200 use-shortcut-sla
     Gen(4), TOS(0x0/0x0), Protocol(0: 1->65535), Mode(load-balance  hash-mode=round-robin)
     Members(3): 
       1: Seq_num(1 port2), dead, sla(0x0), gid(1), num of pass(0), selected
       2: Seq_num(2 port3), alive, sla(0x0), gid(2), num of pass(0), selected
       3: Seq_num(3 port4), alive, sla(0x0), gid(2), num of pass(0), selected
     Internet Service(6): Alibaba-Alibaba.Cloud(6881402,0,0,0) Alibaba-DNS(6881283,0,0,0) Alibaba-ICMP(6881282,0,0,0) Alibaba-NTP(6881288,0,0,0) Alibaba-SSH(6881286,0,0,0) Alibaba-Web(6881281,0,0,0) 
     Src address(1): 
           192.168.10.0-192.168.10.255
   ```

8. 查看策略路由列表。

   ```
   SDWAN # diagnose firewall proute list
   list route policy info(vf=root):
   
   id=2133458945(0x7f2a0001) vwl_service=1(To_Aliyun) vwl_mbr_seq=1 2 3 dscp_tag=0xff 0xff flags=0x10 load-balance hash-mode=round-robin  tos=0x00 tos_mask=0x00 protocol=0 sport=0-65535 iif=0(any) dport=1-65535 path(3) oif=4(port2) num_pass=0 oif=5(port3) num_pass=0 oif=6(port4) num_pass=0 
   source(1): 192.168.10.0-192.168.10.255 
   destination wildcard(1): 0.0.0.0/0.0.0.0 
   internet service(6): Alibaba-Alibaba.Cloud(6881402,0,0,0) Alibaba-DNS(6881283,0,0,0) Alibaba-ICMP(6881282,0,0,0) Alibaba-NTP(6881288,0,0,0) Alibaba-SSH(6881286,0,0,0) Alibaba-Web(6881281,0,0,0) 
   hit_count=25 last_used=2023-01-03 22:49:28
   ```

9. 查看SD-WAN设备的流量日志，依旧会在三条链路进行负载均衡。

   <img src="../../../../../images/image-20230104162933497.png" alt="image-20230104162933497" style="zoom:50%;" />
