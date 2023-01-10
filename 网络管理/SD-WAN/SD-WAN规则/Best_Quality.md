# Best Quality（最佳质量）

## 基本概念

1. SD-WAN规则选路原则是：选择具有“最佳质量”的链路。

2. 最佳质量的标准有：

   <img src="../../../../images/image-20230103195026967.png" alt="image-20230103195026967" style="zoom:50%;" />

   - 延迟
     - 一个数据包从一个节点到另一个节点所需的时间
     - 更少的延迟=更好的吞吐量
     - 延迟大引起的问题：访问速度慢，连接失败
     - 推荐应用：建议用于需要最佳响应时间的应用程序。例如：视频/VoIP
   - 抖动
     - 在网络中传输数据包，相邻的数据包被转发之间的时间延迟差异（毫秒级别）。这是设备在按序转发数据包，包和包之间的一个短暂的中断时间。抖动通常是由IP网络中的拥塞引起的
     - 抖动大引起的问题：实时应用程序延迟
     - 推荐应用：建议用于需要实时转发传递数据包的应用程序。例如：VoIP
   - 丢包
     - 发生在当通过网络传输的一个或多个数据包无法到达其目标地时候
     - 丢包而引起的问题：信息超时、加载时间慢、加载中断、连接关闭和信息丢失
     - 推荐应用：Oracle DB和SSH这样的CS应用程序
   - 下游带宽
     - 内部用户通过网络传输下载数据
     - 带宽不足而引起的问题：传输速度慢
     - 推荐应用：需要网络资源下载数据的应用程序。例如：文件服务器、云存储（Dropbox、OneDrive、百度网盘）
   - 上游带宽
     - 内部用户通过网络传输上传数据
     - 带宽不足而引起的问题：传输速度慢，无法完成上传
     - 推荐应用：需要网络资源上传数据的应用程序。例如：备份系统
   - 总体带宽
     - 内部用户通过网络传输上传下载数据的总和
   - 自定义配置文件
     - 基于延迟、抖动、丢包、带宽权重的自定义  {计算公式：【Link Quality Index = (packet-loss-weight * packet loss) + (latency-weight * latency) + (jitter-weight * jitter) + (bandwidth-weight / bandwidth)】}
     - 带宽不足而引起的问题：传输速度慢
     - 推荐应用：需要网络资源上传和下载数据的应用程序。示例：文件服务器、云存储（Dropbox、OneDrive）

3. 最佳质量顾名思义就是在SD-WAN规则里配置的多个出接口里面：选择最好的！

   <img src="../../../../images/image-20230103195239652.png" alt="image-20230103195239652" style="zoom:50%;" />

## 配置举例

### 网络拓扑

<img src="../../../../images/image-20230103195444267.png" alt="image-20230103195444267" style="zoom:50%;" />

### 配置步骤

------

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

3. 配置性能SLA，使用HTTP方法监控www.skype.com网站。

   <img src="../../../../images/image-20230104093714077.png" alt="image-20230104093714077" style="zoom:50%;" />

   ```
   config system sdwan
       config health-check
           edit "SKYPE"
               set server "www.skype.com"
               set protocol http
               set members 1 2 3
           next
       end
   end
   ```

4. 配置SD-WAN规则，目标地址为Skype相关应用和Internet服务，流出接口策略选择最佳质量，SLA衡量标准选择上步创建的性能SLA，质量标准选择延迟。

   <img src="../../../../images/image-20230103201522438.png" alt="image-20230103201522438" style="zoom:50%;" />

   <img src="../../../../images/image-20230103201546217.png" alt="image-20230103201546217" style="zoom:50%;" />

   ```
   config system sdwan
       config service
           edit 1
               set name "SKYPE-OUT-Best-Quality-Latency"
               set mode priority
               set link-cost-factor latency    //标准为延迟
               set link-cost-threshold 10    //误差范围10%，避免频繁切换 （默认值）
               set src "LAN_192.168.10.0"
               set internet-service enable
               set internet-service-name "Microsoft-Skype_Teams"
               set internet-service-app-ctrl 10 28554 43540 28587 29350 28597
               set health-check "SKYPE"
               set priority-members 1 2 3
           next
       end
   end
   ```

5. 配置SD-WAN的上网策略。

   <img src="../../../../images/image-20230103200843875.png" alt="image-20230103200843875" style="zoom:50%;" />

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

1. 查看SD-WAN成员状态，以及健康检查监控状态，可以看到port2、port3、port4的监控延迟分别为160ms左右、260ms左右、360ms左右。

   ```
   SDWAN # diagnose sys sdwan member 
   Member(1): interface: port2, flags=0x0 , gateway: 202.100.1.192, priority: 1 1024, weight: 0
   Member(2): interface: port3, flags=0x0 , gateway: 101.100.1.192, priority: 1 1024, weight: 0
   Member(3): interface: port4, flags=0x0 , gateway: 111.100.1.192, priority: 1 1024, weight: 0
   Member(4): interface: PPPOE1_DR_PENG, flags=0x8 , gateway: 114.100.1.196, priority: 1 1024, weight: 0
   
   SDWAN # diagnose sys sdwan service 
   
   Service(1): Address Mode(IPV4) flags=0x200 use-shortcut-sla
     Gen(1), TOS(0x0/0x0), Protocol(0: 1->65535), Mode(priority), link-cost-factor(latency), link-cost-threshold(10), heath-check(SKYPE)
     Members(3): 
       1: Seq_num(1 port2), alive, latency: 171.847, selected
       2: Seq_num(2 port3), alive, latency: 267.559, selected
       3: Seq_num(3 port4), alive, latency: 366.488, selected
     Internet Service(7): Microsoft-Skype_Teams(327781,0,0,0) Skype(4294838051,0,0,0 10) Microsoft.Lync(4294837471,0,0,0 28554) Skype.Portals(4294838961,0,0,0 43540) Lync_Audio(4294837364,0,0,0 28587) Lync_Apps.Sharing(4294837363,0,0,0 29350) Lync_Video(4294837366,0,0,0 28597) 
     Src address(1): 
           192.168.10.0-192.168.10.255
           
   SDWAN # diagnose sys sdwan health-check 
   Health Check(SKYPE): 
   Seq(1 port2): state(alive), packet-loss(15.000%) latency(178.808), jitter(53.983), bandwidth-up(9999995), bandwidth-dw(9999954), bandwidth-bi(19999949) sla_map=0x0
   Seq(2 port3): state(alive), packet-loss(14.000%) latency(261.427), jitter(62.005), bandwidth-up(9999995), bandwidth-dw(9999952), bandwidth-bi(19999947) sla_map=0x0
   Seq(3 port4): state(alive), packet-loss(9.000%) latency(367.605), jitter(58.608), bandwidth-up(9999995), bandwidth-dw(9999951), bandwidth-bi(19999946) sla_map=0x0
   ```

   <img src="../../../../images/image-20230104094636447.png" alt="image-20230104094636447" style="zoom:50%;" />

2. 查看策略路由列表，将会按照延迟的顺序port2、port3、port4这样排序，优先使用延迟小的出口port2。

   ```
   SDWAN # diagnose firewall proute list 
   list route policy info(vf=root):
   
   id=2131492865(0x7f0c0001) vwl_service=1(SKYPE-OUT-Best-Quality-Latency) vwl_mbr_seq=1 2 3 dscp_tag=0xff 0xff flags=0x0 tos=0x00 tos_mask=0x00 protocol=0 sport=0-65535 iif=0(any) dport=1-65535 path(3) oif=4(port2) oif=5(port3) oif=6(port4) 
   source(1): 192.168.10.0-192.168.10.255 
   destination wildcard(1): 0.0.0.0/0.0.0.0 
   internet service(7): Microsoft-Skype_Teams(327781,0,0,0) Skype(4294838051,0,0,0, 10) Microsoft.Lync(4294837471,0,0,0, 28554) Skype.Portals(4294838961,0,0,0, 43540) Lync_Audio(4294837364,0,0,0, 28587) Lync_Apps.Sharing(4294837363,0,0,0, 29350) Lync_Video(4294837366,0,0,0, 28597) 
   hit_count=13 last_used=2023-01-03 17:40:51
   ```

3. 查看SD-WAN设备路由表。

   ```
   SDWAN # get router info routing-table all
   ...
   S*      0.0.0.0/0 [1/0] via 101.100.1.192, port3, [1/0]
                     [1/0] via 111.100.1.192, port4, [1/0]
                     [1/0] via 114.100.1.196, PPPOE1_DR_PENG, [1/0]
                     [1/0] via 202.100.1.192, port2, [1/0]
   ...
   ```

4. 使用客户端PC访问Skype相关资源，在SD-WAN设备上查看流量日志，可以看到相关流量被分配到port2，符合上述SD-WAN检测结果。

   <img src="../../../../images/image-20230104100653643.png" alt="image-20230104100653643" style="zoom:50%;" />

### 故障切换测试

1. 调整port2和port4的延迟，模拟延迟变化，将port2的延迟模拟为360ms，port4的延迟模拟为160ms。

   <img src="../../../../images/image-20230104101226488.png" alt="image-20230104101226488" style="zoom:50%;" />

2. 切换延迟后，查看SLA的检查状态，port2和port4的延迟进行了对调。

   ```
   Seq(1 port2): state(alive), packet-loss(0.000%) latency(367.850), jitter(36.092), bandwidth-up(9999995), bandwidth-dw(9999953), bandwidth-bi(19999948) sla_map=0x0
   Seq(2 port3): state(alive), packet-loss(0.000%) latency(270.216), jitter(37.558), bandwidth-up(9999995), bandwidth-dw(9999953), bandwidth-bi(19999948) sla_map=0x0
   SDWAN # diagnose sys sdwan health-check
   Health Check(SKYPE): 
   Seq(3 port4): state(alive), packet-loss(0.000%) latency(169.290), jitter(28.239), bandwidth-up(9999995), bandwidth-dw(9999952), bandwidth-bi(19999947) sla_map=0x0
   ```

   <img src="../../../../images/image-20230104101725750.png" alt="image-20230104101725750" style="zoom:50%;" />

3. 查看SD-WAN规则出接口的变化，出接口顺序也相应的对调，变为port4，port3，port2。

   ```
   SDWAN # diagnose sys sdwan service 
   
   Service(1): Address Mode(IPV4) flags=0x200 use-shortcut-sla
     Gen(25), TOS(0x0/0x0), Protocol(0: 1->65535), Mode(priority), link-cost-factor(latency), link-cost-threshold(10), heath-check(SKYPE)
     Members(3): 
       1: Seq_num(3 port4), alive, latency: 168.681, selected    //port4，优先从port13转发
       2: Seq_num(2 port3), alive, latency: 269.862, selected    //port3
       3: Seq_num(1 port2), alive, latency: 367.583, selected    //port2
     Internet Service(7): Microsoft-Skype_Teams(327781,0,0,0) Skype(4294838051,0,0,0 10) Microsoft.Lync(4294837471,0,0,0 28554) Skype.Portals(4294838961,0,0,0 43540) Lync_Audio(4294837364,0,0,0 28587) Lync_Apps.Sharing(4294837363,0,0,0 29350) Lync_Video(4294837366,0,0,0 28597) 
     Src address(1): 
           192.168.10.0-192.168.10.255
   ```

4. 查看策略路由中的SD-WAN规则列表，接口顺序已经变化为port4、port3、port2了，依据延迟的大小排序，优先选择延迟小的port4出口。

   ```
   SDWAN # diagnose firewall proute list
   list route policy info(vf=root):
   
   id=2131492865(0x7f0c0001) vwl_service=1(SKYPE-OUT-Best-Quality-Latency) vwl_mbr_seq=3 2 1 dscp_tag=0xff 0xff flags=0x0 tos=0x00 tos_mask=0x00 protocol=0 sport=0-65535 iif=0(any) dport=1-65535 path(3) oif=6(port4) oif=5(port3) oif=4(port2) 
   source(1): 192.168.10.0-192.168.10.255 
   destination wildcard(1): 0.0.0.0/0.0.0.0 
   internet service(7): Microsoft-Skype_Teams(327781,0,0,0) Skype(4294838051,0,0,0, 10) Microsoft.Lync(4294837471,0,0,0, 28554) Skype.Portals(4294838961,0,0,0, 43540) Lync_Audio(4294837364,0,0,0, 28587) Lync_Apps.Sharing(4294837363,0,0,0, 29350) Lync_Video(4294837366,0,0,0, 28597) 
   hit_count=19 last_used=2023-01-03 18:05:39
   ```

5. 查看客户端访问SKYPE相关资源的流量日志，可以看到流量被分配到了port4。

   <img src="../../../../images/image-20230104102931780.png" alt="image-20230104102931780" style="zoom:50%;" />

### 注意事项

1. 由于我的SD-WAN规则里面选择了应用程序Skype，这样基于应用的SD-WAN规则（策略路由）要生效的话需要满足一些条件：

   <img src="../../../../images/image-20230104103226962.png" alt="image-20230104103226962" style="zoom:50%;" />

   - 第一步：应用程序是需要先通过内网用户通过使用SKPYE，然后FGT进行应用识别的，因此策略里面一定要开启APP Control才可以（需要识别加密流量需要开启SSL深度检测）。

     <img src="../../../../images/image-20230104103330776.png" alt="image-20230104103330776" style="zoom:50%;" />

   - 第二步：识别之后将会学习到相应的应用程序（SKYPE）的IP地址，将会形成一个动态的应用程序（SKYPE）IP地址数据库，最终才会将这个学习到的IP数据库加入到SD-WAN规则（策略路由）里面，因此这需要有一个学习的过程，SD-WAN中的应用程序的调用不会立即生效，因此在没有成功学习之前，SKPYE的流量会走到其他线路上去，FGT会去学习这些识别出来的应用程序的IP地址，这并非问题，而是SD-WAN调用应用的工作逻辑就是这样，学习完毕之后，新发起的流量才会匹配到SD-WAN规则（策略路由），我们可以通过命令行查看到学习的应用程序IP数据库信息。

     ```
     SDWAN # diagnose sys sdwan internet-service-app-ctrl-list
     
     Skype(10 4294838051): 52.113.194.133 6 443 Tue Jan  3 18:05:37 2023
     Skype.Portals(43540 4294838961): 13.107.42.16 6 443 Tue Jan  3 15:40:51 2023
     ```

   - 如果需要学习将到的Skype数据库立马完全生效，旧的会话是会保持原有出口的，想要这些旧的会话立即生效需要清除旧的会话和路由缓存信息，而新建的会话则不需要此操作，具体命令行如下。

     ```
     SD-WAN # diagnose sys session filter src 192.168.10.100
     SD-WAN # diagnose sys session clear    //清除测试机器的会话，让其重新匹配新的SD-WAN规则（有skype的IP数据库更新）
     SD-WAN # diagnose ip rtcache flush    //清除路由缓存
     ```

2. SD-WAN规则中参数：link-cost-threshold的作用。

   ```
   SDWAN #  config system sdwan
   SDWAN (sdwan) # config service 
   SDWAN (service) # show full-configuration 
   config service
       edit 1
           set name "SKYPE-OUT-Best-Quality-Latency"
   ...
           set link-cost-threshold 10
   ...
       next
   end
   ```

   - 根据上述的实验测试可以看出SD-WAN规则的工作逻辑：

     - 切换前延迟：
       - port2联通延迟160ms    //选择延迟小的
       - port3电信延迟260ms
       - port4移动延迟360ms
       - 优先选择port2
     - 切换后的延迟：
       - port2联通延迟360ms
       - port3电信延迟260ms
       - port4移动延迟160ms    //选择延迟小的
       - 优先选择port4

   - 我们再来做一个测试，让延迟变成下面这个样子：

     - port2联通延迟166ms
     - port3电信延迟156ms     //设置port3的延迟比port2低，但是低的不多
     - Port4移动延迟266ms

   - 此时如何优先出接口呢？我们来看具体情况。

     ```
     SDWAN # diagnose sys sdwan health-check
     Health Check(SKYPE): 
     Seq(1 port2): state(alive), packet-loss(0.000%) latency(166.441), jitter(42.834), bandwidth-up(9999995), bandwidth-dw(9999953), bandwidth-bi(19999948) sla_map=0x0
     Seq(2 port3): state(alive), packet-loss(0.000%) latency(156.610), jitter(29.137), bandwidth-up(9999995), bandwidth-dw(9999952), bandwidth-bi(19999947) sla_map=0x0
     Seq(3 port4): state(alive), packet-loss(0.000%) latency(268.445), jitter(21.758), bandwidth-up(9999995), bandwidth-dw(9999953), bandwidth-bi(19999948) sla_map=0x0
     ```

     <img src="../../../../images/image-20230104105456971.png" alt="image-20230104105456971" style="zoom:50%;" />

   - 此时看SD-WAN规则的选择，此时port2会被优选，port3延迟虽然比port2低，但是没有被优选，这是为什么呢？

     ```
     SDWAN # diagnose sys sdwan service
     
     Service(1): Address Mode(IPV4) flags=0x200 use-shortcut-sla
       Gen(1), TOS(0x0/0x0), Protocol(0: 1->65535), Mode(priority), link-cost-factor(latency), link-cost-threshold(10), heath-check(SKYPE)
       Members(3): 
         1: Seq_num(1 port2), alive, latency: 166.993, selected    //此时port2会被优选
         2: Seq_num(2 port3), alive, latency: 156.219, selected    //port3延迟虽然比port2低，但是没有被优选，这是为什么呢？
         3: Seq_num(3 port4), alive, latency: 268.677, selected
       Internet Service(7): Microsoft-Skype_Teams(327781,0,0,0) Skype(4294838051,0,0,0 10) Microsoft.Lync(4294837471,0,0,0 28554) Skype.Portals(4294838961,0,0,0 43540) Lync_Audio(4294837364,0,0,0 28587) Lync_Apps.Sharing(4294837363,0,0,0 29350) Lync_Video(4294837366,0,0,0 28597) 
       Src address(1): 
             192.168.10.0-192.168.10.255
     ```

   - 查看策略路由列表，策略路由的接口顺序port2、port3、port4。

     ```
     SDWAN # diagnose firewall proute list 
     list route policy info(vf=root):
     
     id=2131623937(0x7f0e0001) vwl_service=1(SKYPE-OUT-Best-Quality-Latency) vwl_mbr_seq=1 2 3 dscp_tag=0xff 0xff flags=0x0 tos=0x00 tos_mask=0x00 protocol=0 sport=0-65535 iif=0(any) dport=1-65535 path(3) oif=4(port2) oif=5(port3) oif=6(port4)    //策略路由的接口顺序port2、port3、port4
     source(1): 192.168.10.0-192.168.10.255 
     destination wildcard(1): 0.0.0.0/0.0.0.0 
     internet service(7): Microsoft-Skype_Teams(327781,0,0,0) Skype(4294838051,0,0,0, 10) Microsoft.Lync(4294837471,0,0,0, 28554) Skype.Portals(4294838961,0,0,0, 43540) Lync_Audio(4294837364,0,0,0, 28587) Lync_Apps.Sharing(4294837363,0,0,0, 29350) Lync_Video(4294837366,0,0,0, 28597) 
     hit_count=24 last_used=2023-01-03 18:34:50
     ```

   - 此时SD-WAN规则选择了port2作为出接口，port3延迟虽然比port2低，但是没有被优选，这是为什么呢？

   - 答案正是因为set link-cost-threshold 10此参数在起作用：当两个SD-WAN接口成员链路之间的SLA测量值存在显著差异时，结果是可以预期的，越小则越优先被选择。然而，当标准的SLA测量值足够接近时，SD-WAN会优先考虑配置中的靠前位置位置的出口链路。比如port2/port3/port4的延迟差不多的时候，会优先选择port2，这种逻辑是为了避免当SLA相对接近时链接之间的频繁切换而设计的。它由一个名为“set link-cost-threshold”的设置进行控制，该设置默认为10%，如果差距在10%以内的值，会优先选择SD-WAN规则中配置靠前的接口进行数据转发。

     - port2联通延迟166ms，10%的最低值：166 /（1+0.1）=  150.9，150.9 ~ 166 之间的值对于port2来说都是会被忽略的，即便延迟比port2要低，但是没有超过10%，因此就忽略了比较计算值，依旧保持port2出口的选择。
     - port3电信延迟156ms，延迟比port2虽然要低，但是没有超过< 150.9 ~166 （port2百分之10的忽略范围），因此port3不会被优选。
     - port4移动延迟266ms，超过了150.9 ~166的范围，肯定不会被优选。

   - 此时将port3的延迟设置为149ms，查看SD-WAN策略中的顺序，发现port3的延迟149.5 < 150.9马上就会被优选。

     ```
     Service(1): Address Mode(IPV4) flags=0x200 use-shortcut-sla
       Gen(1), TOS(0x0/0x0), Protocol(0: 1->65535), Mode(priority), link-cost-factor(latency), link-cost-threshold(10), heath-check(SKYPE)
       Members(3): 
         1: Seq_num(2 port3), alive, latency: 149.569, selected
         2: Seq_num(1 port2), alive, latency: 164.794, selected
         3: Seq_num(3 port4), alive, latency: 259.659, selected
       Internet Service(7): Microsoft-Skype_Teams(327781,0,0,0) Skype(4294838051,0,0,0 10) Microsoft.Lync(4294837471,0,0,0 28554) Skype.Portals(4294838961,0,0,0 43540) Lync_Audio(4294837364,0,0,0 28587) Lync_Apps.Sharing(4294837363,0,0,0 29350) Lync_Video(4294837366,0,0,0 28597) 
       Src address(1): 
             192.168.10.0-192.168.10.255
     ```

   - 查看策略路由列表，出接口顺序变为port3，port2，port4。

     ```
     SDWAN # diagnose firewall proute list
     list route policy info(vf=root):
     
     id=2131623937(0x7f0e0001) vwl_service=1(SKYPE-OUT-Best-Quality-Latency) vwl_mbr_seq=2 1 3 dscp_tag=0xff 0xff flags=0x0 tos=0x00 tos_mask=0x00 protocol=0 sport=0-65535 iif=0(any) dport=1-65535 path(3) oif=5(port3) oif=4(port2) oif=6(port4) 
     source(1): 192.168.10.0-192.168.10.255 
     destination wildcard(1): 0.0.0.0/0.0.0.0 
     internet service(7): Microsoft-Skype_Teams(327781,0,0,0) Skype(4294838051,0,0,0, 10) Microsoft.Lync(4294837471,0,0,0, 28554) Skype.Portals(4294838961,0,0,0, 43540) Lync_Audio(4294837364,0,0,0, 28587) Lync_Apps.Sharing(4294837363,0,0,0, 29350) Lync_Video(4294837366,0,0,0, 28597) 
     hit_count=24 last_used=2023-01-03 18:34:50
     ```

   - 总结来看：Best Quality（最佳质量）在SD-WAN规则中会选择最好质量的线路（延迟、丢包、抖动）最低值的出接口优先用于SD-WAN流量的转发，为了避免频繁的切换，当所有的出接口质量都差不多的时候（差距在10%以内），则会选择SD-WAN规则配置接口顺序靠前的接口用于SD-WAN流量的转发。