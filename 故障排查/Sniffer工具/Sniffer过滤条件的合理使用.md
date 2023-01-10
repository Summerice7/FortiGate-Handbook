# Sniffer过滤条件的合理使用

## Sniffer过滤条件使用总结

**Sniffer 应该抓取一条流的五元组不变的那个参数，且过滤的条件越精确抓取的报文越准确！**

在没有NAT场景下，源目的IP及目的端口都不会变化，则过滤条件可以设置为"host 源ip and host 目的ip and 目的端口"；

在源NAT场景下，源IP会做NAT转换，因此目的IP和目的端口是不会变化的，则过滤条件可以设置为"host 目的ip and 目的端口"；

在目的NAT场景下，目的IP会做NAT转换，因此源IP和目的端口是不会变化的，则过滤条件可以设置为"host 源ip and 目的端口"；

在目的端口NAT场景下，目的IP和端口都会做NAT转换，因此源IP是不会变化，则过滤条件可以设置为"host 源ip"；

在源NAT和目的NAT都存在的场景下，源IP和目的IP都会发生变化，则只能通过or分别抓取NAT前后的报文，则过滤条件可以设置为"(源NAT前的源ip and 目的NAT前的目的ip and 目的NAT前目的端口） or (源NAT后的源IP and 目的NAT后的目的ip and 目的NAT后的目的端口)"

## Sniffer过滤条件使用举例

ping是没有端口的，直接使用icmp协议过滤即可。

1. **普通三层转发，没有NAT场景**

   **普通三层转发，没有NAT的情况下，抓 ''源ip and 目的ip and icmp协议'' 可以抓到完整的ping数据流**

   拓扑：PC1(172.16.1.100)-----(port9：172.16.1.1)FGT(port10:202.106.1.1)-----PC2(202.106.1.100)

   ```
   策略配置
   config firewall policy
       edit 1
           set srcintf "port9"
           set dstintf "port10"
           set action accept
           set srcaddr "all"
           set dstaddr "all"
           set schedule "always"
           set service "ALL"
       next
   end
   
   FGT # diagnose sniffer packet  any "host 172.16.1.100 and host 202.106.1.100 and icmp" 4  // 抓源IP
   interfaces=[any]
   filters=[host 172.16.1.100 and icmp]
   3.583522 port9 in 172.16.1.100 -> 202.106.1.100: icmp: echo request
   3.583541 port10 out 172.16.1.100 -> 202.106.1.100: icmp: echo request
   3.583808 port10 in 202.106.1.100 -> 172.16.1.100: icmp: echo reply
   3.583818 port9 out 202.106.1.100 -> 172.16.1.100: icmp: echo reply
   ```

2. **普通三层转发，有源NAT场景** 

   **普通三层转发，有源NAT的情况下，抓 ''目的ip and icmp协议'' 可以抓到完整的ping数据流**

   拓扑：PC1(172.16.1.100)-----(port9：172.16.1.1)FGT(port10:202.106.1.1)-----PC2(202.106.1.100)

   ```
   策略配置
   config firewall policy
       edit 1
           set srcintf "port9"
           set dstintf "port10"
           set action accept
           set srcaddr "all"
           set dstaddr "all"
           set schedule "always"
           set service "ALL"
           set nat enable
       next
   end
   
   FGT # diagnose sniffer packet  any “host 172.16.1.100 and icmp” 4  // 抓源IP and icmp协议 抓不全 
   interfaces=[any]
   filters=[host 172.16.1.100 and icmp]
   2.821403 port9 in 172.16.1.100 -> 202.106.1.100: icmp: echo request
   2.821707 port9 out 202.106.1.100 -> 172.16.1.100: icmp: echo reply      // 只能在port9看到数据包IN/OUT，port10的看不到
   
   FGT # diagnose sniffer packet  any "host 202.106.1.100 and icmp" 4  // 需要抓目的IP and icmp协议
   interfaces=[any]filters=[host 202.106.1.100 and icmp]
   4.624647 port9 in 172.16.1.100 -> 202.106.1.100: icmp: echo request
   4.625032 port10 out 202.106.1.1 -> 202.106.1.100: icmp: echo request
   4.625235 port10 in 202.106.1.100 -> 202.106.1.1: icmp: echo reply
   4.625243 port9 out 202.106.1.100 -> 172.16.1.100: icmp: echo reply
   ```

3. **普通三层转发，有目的NAT（一对一NAT）场景**

   **普通三层转发，有目的NAT的情况下，抓 ''源ip and icmp协议'' 可以抓到完整的ping数据流**

   拓扑：PC1(172.16.1.100)-----(port9：172.16.1.1)FGT(port10:202.106.1.1)-----PC2(202.106.1.100)
   
   ```
   策略配置： 
   config firewall vip
       edit "VIP1"
           set extip 202.106.1.10
           set mappedip "172.16.1.100"
           set extintf "any"
       next
   end
   
   config firewall policy
       edit 1
           set srcintf "port10"
           set dstintf "port9"
           set action accept
           set srcaddr "all"
           set dstaddr "VIP1"
           set schedule "always"
           set service "ALL"
       next
   end
   
   访问VIP（DNAT），流量从外网侧发起 
   FGT # diagnose sniffer packet any “host 202.106.1.10” 4  // 抓目的IP and icmp协议，抓不全
   interfaces=[any]
   filters=[host 202.106.1.10]
   3.809005 port10 in 202.106.1.100 -> 202.106.1.10: icmp: echo request
   3.809294 port10 out 202.106.1.10 -> 202.106.1.100: icmp: echo reply    // 只能在port10看到数据包IN/OUT，port9的看不到
   
   FGT # diagnose sniffer packet any “host 202.106.1.100” 4  // 需要抓源IP and icmp协议
   interfaces=[any]
   filters=[host 202.106.1.100]
   3.523666 port10 in 202.106.1.100 -> 202.106.1.10: icmp: echo request
   3.523687 port9 out 202.106.1.100 -> 172.16.1.100: icmp: echo request
   3.523937 port9 in 172.16.1.100 -> 202.106.1.100: icmp: echo reply
   3.523943 port10 out 202.106.1.10 -> 202.106.1.100: icmp: echo reply
   ```
   
4. **普通三层转发，有双向NAT(源NAT+目的NAT）场景**

   **普通三层转发，有双NAT的情况下，抓 ''(源NAT之前的源ip and 目的NAT之前的目的IP) and (源NAT之后的源ip and 目的NAT之后的目的IP)  and icmp协议'' 可以抓到完整的ping数据流**

   拓扑：PC1(172.16.1.100)-----(port9：172.16.1.1)FGT(port10:202.106.1.1)-----PC2(202.106.1.100)
   
   ```
   策略配置：
   config firewall vip
       edit "VIP1"
           set extip 202.106.1.10
           set mappedip "172.16.1.100"
           set extintf "any"
       next
   end
   
   config firewall policy
       edit 1
           set srcintf "port10"
           set dstintf "port9"
           set action accept
           set srcaddr "all"
           set dstaddr "VIP1"
           set schedule "always"
           set service "ALL"
           set nat enable
       next
   end
   
   访问VIP，流量从外网侧发起 ，同时在内网接口侧开启了SNAT
   
   FGT # diagnose sniffer packet any "host 202.106.1.100 and icmp" 4 （抓源IP 不全）
   interfaces=[any]
   filters=[host 202.106.1.100 and icmp]
   0.416187 port10 in 202.106.1.100 -> 202.106.1.10: icmp: echo request
   0.416576 port10 out 202.106.1.10 -> 202.106.1.100: icmp: echo reply 
   
   FGT # diagnose sniffer packet any "host 172.16.1.100 and icmp" 4  （抓目的IP 不全）
   interfaces=[any]
   filters=[host 172.16.1.100 and icmp]
   1.417838 port9 out 172.16.1.1 -> 172.16.1.100: icmp: echo request
   1.418326 port9 in 172.16.1.100 -> 172.16.1.1: icmp: echo reply  
   
   FGT # diagnose sniffer packet any "host 202.106.1.100 or host 172.16.1.1 and icmp" 4 （or组合变化前源目的ip和变化后源目的IP可抓全）
   interfaces=[any]
   filters=[host 202.106.1.100 or host 172.16.1.1 or icmp]
   0.419407 port10 in 202.106.1.100 -> 202.106.1.10: icmp: echo request
   0.419436 port9 out 172.16.1.1 -> 172.16.1.100: icmp: echo request
   0.419781 port9 in 172.16.1.100 -> 172.16.1.1: icmp: echo reply      
   0.419794 port10 out 202.106.1.10 -> 202.106.1.100: icmp: echo reply
   ```
   