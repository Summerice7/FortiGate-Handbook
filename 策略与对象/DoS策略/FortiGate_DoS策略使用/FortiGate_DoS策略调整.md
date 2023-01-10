# FortiGate DoS策略调整

1. 应该如何调整DoS策略以防误报或阻止正常流量？DoS策略不依赖于会话状态，所以FortiGate的出站和入站流量都会被处理。例如如下流量抓包：

   ```
   # di sniff pack Vlan_11 'port 80 and tcp[tcpflags] == tcp-syn' 1 25
   interfaces=[Vlan_11]
   filters=[port 80 and tcp[tcpflags] == tcp-syn]
   0.876904 10.95.13.204.24088 -> 10.95.136.204.80: syn 3585436935
   0.894848 10.95.4.223.7273 -> 10.95.128.117.80: syn 2436279189
   0.947586 10.95.10.90.5871 -> 10.95.132.100.80: syn 1124757321
   0.972220 10.95.5.29.26781 -> 10.95.128.217.80: syn 2500978264
   ```

2. 基于这一点，你需要使用如下推荐的配置。

## 从Internet保护内网服务器

1. 确保仅将内网服务器映射的IP（比如VIP）配置为DoS策略的目的地址。

2. 在被保护的服务器上开启需要的服务。在DoS策略的服务中仅配置服务器对外开启的服务。

3. DoS策略中仅配置与服务器开启的服务有关的异常。例如服务器只开启了SMTP服务，那么在配置DoS策略时，只配置tcp_syn_flood、tcp_src_session或ip_src_session异常。

4. 将这些异常配置到一个DoS策略中，根据实际使用情况，阈值设置为正常的值。例如配置tcp_src_session异常，对于单个源客户端来说，最大的并发会话数为100，则使用如下配置（对于每一个异常，都有一个“正常”的阈值，需要根据实际情况进行配置）：

   ```
   config firewall DoS-policy
       edit 1
           set interface "wan1"
           set srcaddr "all"
           set dstaddr "200.201.202.1"
           set service "SMTP"
               config anomaly
                   edit "tcp_src_session"
                       set status enable
                       set action block
                       set quarantine attacker
                       set quarantine-expiry 10
                       set quarantine-log enable
                       set threshold 100
                   next
               end
       next
   end
   ```

5. 如果使用tcp_dst_session或ip_dst_session，则DoS策略将限制服务器处理的并发会话数，不建议这么使用。

## 从内网保护FortiGate

1. 将DoS策略的源配置为本地网络的网段（不要使用any）。
2. 将DoS策略的目的配置为any（攻击流量会通过默认路由到达防火墙），服务配置为ALL。
3. 开启这些异常：tcp_syn_flood、tcp_dst_session、ip_dst_session、tcp_port_scan，并根据平时使用情况配置正常的阈值。
4. 需要注意的是，在UDP相关异常和tcp_src异常的情况下，某些流量可能是由torrent（P2P）或Skype软件导致的。
