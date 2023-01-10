# Manual（手动选路）

## 基本概念

1. 顾名思义让指定的SD-WAN流量从人工选择的固定线路出去。这时候SD-WAN规则的出口是固定的，不存在切换的问题，除非线路DOWN/SD-WAN健康检查失败/Link-Moniter等因素影响让其出口的相关路由消失，SD-WAN规则才会失效，流量才会切换到其他的线路上去，否则将强制让SD-WAN的流量出口固定到这条线路上。
2. 一般有两种常见的使用场景：将优先保障的重要业务固定指向到一条质量好的线路上去；或者反过来使用，将一些优先级不高（比如一些自动更新等占用较大带宽的应用）的业务固定指向到一条质量差的线路上去。具体的使用可按照自己的需求去定义。
3. 手工选路更像传统的策略路由，只能选择一个出接口，但相比策略路由会多跟多的匹配项，比如应用、Internet Service Database等等，更加灵活一些。
4. 总结：
   * 相当于在SD-WAN接口上配置常规的策略路由
   * 不需要调用Performance SLA（SD-WAN状态检查）
   * 源IP地址建议配置为明细，不要配置为any，避免匹配到本机发起流量
   * 目的地址可以调用 IP地址+协议+端口或Internet Service + Application Control
   * 只能选择一个出接口（单接口成员选择）

## 配置举例

### 网络拓扑

<img src="../../../../images/image-20230103191133167.png" alt="image-20230103191133167" style="zoom:50%;" />

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

3. SD-WAN规则1：Office 365业务固定走联通出口。

   <img src="../../../../images/image-20230103184849171.png" alt="image-20230103184849171" style="zoom:50%;" />

   ```
   config system sdwan
           config service
           edit 1
               set name "OFFICE_365_OUT_port2_Unicom"
               set src "LAN_192.168.1.0"
               set internet-service enable
               set internet-service-name "Microsoft-Office365"
               set priority-members 1
               set gateway enable
               set default enable
           next
       end
   end
   ```

4. SD-WAN规则2：Microsoft-Update业务固定走PPPOE出口。

   <img src="../../../../images/image-20230103185048245.png" alt="image-20230103185048245" style="zoom:50%;" />

   ```
   config system sdwan
           config service
           edit 2
               set name "Microsoft-Update-OUT-PPPoE"
               set src "LAN_192.168.1.0"
               set internet-service enable
               set internet-service-name "Microsoft-Microsoft.Update"
               set priority-members 4
           next
       end
   end
   ```

5. 剩余流量匹配隐藏规则，根据源IP进行负载。

   <img src="../../../../images/image-20230103190343640.png" alt="image-20230103190343640" style="zoom:50%;" />

6. 配置上网策略，放通SDWAN流量。

   <img src="../../../../images/image-20230103191959602.png" alt="image-20230103191959602" style="zoom:50%;" />

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
           set nat enable
       next
   end
   ```

### 结果验证

1. 查看SD-WAN成员状态。

   ```
   SDWAN # diagnose sys sdwan member
   Member(1): interface: port2, flags=0x0 , gateway: 202.100.1.192, priority: 1 1024, weight: 0
   Member(2): interface: port3, flags=0x0 , gateway: 101.100.1.192, priority: 1 1024, weight: 0
   Member(3): interface: port4, flags=0x0 , gateway: 111.100.1.192, priority: 1 1024, weight: 0
   Member(4): interface: PPPOE1_DR_PENG, flags=0x8 , gateway: 114.100.1.196, priority: 1 1024, weight: 0
   ```

2. 查看SDWAN规则状态，两条规则中的线路均为alive状态。

   ```
   SDWAN # diagnose sys sdwan service
   
   Service(1): Address Mode(IPV4) flags=0x260 use-shortcut-sla
     Gen(2), TOS(0x0/0x0), Protocol(0: 1->65535), Mode(manual)
     Members(1): 
       1: Seq_num(1 port2), alive, selected
     Internet Service(1): Microsoft-Office365(327782,0,0,0) 
     Src address(1): 
           192.168.10.0-192.168.10.255
   
   Service(2): Address Mode(IPV4) flags=0x200 use-shortcut-sla
     Gen(2), TOS(0x0/0x0), Protocol(0: 1->65535), Mode(manual)
     Members(1): 
       1: Seq_num(4 PPPOE1_DR_PENG), alive, selected
     Internet Service(1): Microsoft-Microsoft.Update(327793,0,0,0) 
     Src address(1): 
           192.168.10.0-192.168.10.255
   ```

3. 查看策略路由条目，可以看到每条策略对应的出接口SD-WAN成员编号与接口index。

   ```
   SDWAN # diagnose firewall proute list 
   list route policy info(vf=root):
   
   id=2097545217(0x7d060001) vwl_service=1(OFFICE_365_OUT_port2_Unicom) vwl_mbr_seq=1 dscp_tag=0xff 0xff flags=0x0 tos=0x00 tos_mask=0x00 protocol=0 sport=0-65535 iif=0(any) dport=1-65535 path(1) oif=4(port2) gwy=202.100.1.192 default
   source(1): 192.168.10.0-192.168.10.255 
   destination wildcard(1): 0.0.0.0/0.0.0.0 
   internet service(1): Microsoft-Office365(327782,0,0,0) 
   hit_count=11 last_used=2023-01-03 03:13:30
   
   id=2131099650(0x7f060002) vwl_service=2(Microsoft-Update-OUT-PPPoE) vwl_mbr_seq=4 dscp_tag=0xff 0xff flags=0x0 tos=0x00 tos_mask=0x00 protocol=0 sport=0-65535 iif=0(any) dport=1-65535 path(1) oif=16(PPPOE1_DR_PENG) 
   source(1): 192.168.10.0-192.168.10.255 
   destination wildcard(1): 0.0.0.0/0.0.0.0 
   internet service(1): Microsoft-Microsoft.Update(327793,0,0,0) 
   hit_count=0 last_used=2023-01-03 02:59:29
   
   SDWAN # diagnose ip address list 
   ...
   IP=202.100.1.10->202.100.1.10/255.255.255.0 index=4 devname=port2
   IP=101.100.1.10->101.100.1.10/255.255.255.0 index=5 devname=port3
   IP=111.100.1.10->111.100.1.10/255.255.255.0 index=6 devname=port4
   ...
   IP=114.100.1.204->114.100.1.196/255.255.255.255 index=16 devname=PPPOE1_DR_PENG
   ...
   ```

4. 查看SD-WAN设备路由表。

   ```
   SDWAN # get router info routing-table all
   ...
   S*      0.0.0.0/0 [1/0] via 101.100.1.192, port3, [1/0]
                     [1/0] via 111.100.1.192, port4, [1/0]
                     [1/0] via 114.100.1.196, PPPOE1_DR_PENG, [1/0]
                     [1/0] via 202.100.1.192, port2, [1/0]
   ...
   ```

5. 在客户端PC上访问Office365的Internet资源，在SD-WAN设备上查看转发流量日志，Office 365只走port2，MS.Windows.Update只走PPPOE接口出。

   <img src="../../../../images/image-20230103192433819.png" alt="image-20230103192433819" style="zoom:50%;" />

### 故障切换测试

1. 模拟port2故障（SD-WAN状态检查，虽然不需要调用，但是依旧对切换切关键作用，也包括DOWN接口/Link-Moniter等其他切换方式）。

   <img src="../../../../images/image-20230103192843687.png" alt="image-20230103192843687" style="zoom:50%;" />

   ```
   SDWAN # diagnose sys sdwan health-check 
   Health Check(114_Check): 
   Seq(1 port2): state(dead), packet-loss(100.000%) sla_map=0x0
   Seq(2 port3): state(alive), packet-loss(0.000%) latency(20.345), jitter(1.146), bandwidth-up(9999999), bandwidth-dw(9999999), bandwidth-bi(19999998) sla_map=0x1
   Seq(3 port4): state(alive), packet-loss(0.000%) latency(20.197), jitter(1.158), bandwidth-up(9999998), bandwidth-dw(9999997), bandwidth-bi(19999995) sla_map=0x1
   Seq(4 PPPOE1_DR_PENG): state(alive), packet-loss(0.000%) latency(20.336), jitter(1.103), bandwidth-up(9999999), bandwidth-dw(9999999), bandwidth-bi(19999998) sla_map=0x1
   ```

2. 查看SD-WAN成员状态，port2已经变为dead。

   ```
   SDWAN # diagnose sys sdwan member 
   Member(1): interface: port2, flags=0x0 , gateway: 202.100.1.192, priority: 1 1024, weight: 0
   Member(2): interface: port3, flags=0x0 , gateway: 101.100.1.192, priority: 1 1024, weight: 0
   Member(3): interface: port4, flags=0x0 , gateway: 111.100.1.192, priority: 1 1024, weight: 0
   Member(4): interface: PPPOE1_DR_PENG, flags=0x8 , gateway: 114.100.1.196, priority: 1 1024, weight: 0
   
   SDWAN # diagnose sys sdwan service 
   Service(1): Address Mode(IPV4) flags=0x260 use-shortcut-sla
     Gen(3), TOS(0x0/0x0), Protocol(0: 1->65535), Mode(manual)
     Service disabled caused by no outgoing path.
     Members(1): 
       1: Seq_num(1 port2), dead    //检测到port2 dead
     Internet Service(1): Microsoft-Office365(327782,0,0,0) 
     Src address(1): 
           192.168.10.0-192.168.10.255
   Service(2): Address Mode(IPV4) flags=0x200 use-shortcut-sla
     Gen(2), TOS(0x0/0x0), Protocol(0: 1->65535), Mode(manual)
     Members(1): 
       1: Seq_num(4 PPPOE1_DR_PENG), alive, selected    //PPPOE接口不受影响
     Internet Service(1): Microsoft-Microsoft.Update(327793,0,0,0) 
     Src address(1): 
           192.168.10.0-192.168.10.255
   ```

3. 查看SD-WAN设备的路由表，健康检查失败，会触发port2的SD-WAN路由失效。

   ```
   SDWAN # get router info routing-table all
   ...
   S*      0.0.0.0/0 [1/0] via 101.100.1.192, port3, [1/0]
                     [1/0] via 111.100.1.192, port4, [1/0]
                     [1/0] via 114.100.1.196, PPPOE1_DR_PENG, [1/0]
   ...
   ```

4. 查看策略路由列表。

   ```
   SDWAN # diagnose firewall proute list 
   list route policy info(vf=root):
   id=2097545217(0x7d060001) vwl_service=1(OFFICE_365_OUT_port2_Unicom) dscp_tag=0xff 0xff flags=0x8 disable tos=0x00 tos_mask=0x00 protocol=0 sport=0-65535 iif=0(any) dport=1-65535 path(0) default
   source(1): 192.168.10.0-192.168.10.255    //由于FIB路由表中的port2路由消失，相应的对应SD-WAN规则1也会失效。流量将会匹配到最后一条默认的SD-WAN负载均衡规则，走普通路由表进行转发处理
   destination wildcard(1): 0.0.0.0/0.0.0.0 
   internet service(1): Microsoft-Office365(327782,0,0,0) 
   hit_count=39 last_used=2023-01-03 03:20:36
   
   id=2131099650(0x7f060002) vwl_service=2(Microsoft-Update-OUT-PPPoE) vwl_mbr_seq=4 dscp_tag=0xff 0xff flags=0x0 tos=0x00 tos_mask=0x00 protocol=0 sport=0-65535 iif=0(any) dport=1-65535 path(1) oif=16(PPPOE1_DR_PENG) 
   source(1): 192.168.10.0-192.168.10.255    //SD-WAN规则2不受影响
   destination wildcard(1): 0.0.0.0/0.0.0.0 
   internet service(1): Microsoft-Microsoft.Update(327793,0,0,0) 
   hit_count=20 last_used=2023-01-03 03:21:06
   ```

   <img src="../../../../images/image-20230103193550255.png" alt="image-20230103193550255" style="zoom:50%;" />

5. 切换后，将不再匹配SD-WAN规则1了，而会匹配SD-WAN的默认规则，实际为查询默认路由（按照源IP负载的方式）选择出接口，此处选择的时候port3接口为出接口。

   <img src="../../../../images/image-20230103194144437.png" alt="image-20230103194144437" style="zoom:50%;" />

   
