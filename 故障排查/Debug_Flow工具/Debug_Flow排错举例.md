# Debug Flow排错举例

## "Denied by forward policy check (policy 0)"根本原因

```
从Debug Flow输出发现192.168.1.10 ping 114.114.114.114 匹配了策略0（隐式丢包策略），即该数据包没有匹配的策略

2022-11-30 18:16:43 id=20085 trace_id=30 func=print_pkt_detail line=5845 msg="vd-root:0 received a packet(proto=1, 192.168.1.10:2328->114.114.114.114:2048) tun_id=0.0.0.0 from port5. type=8, code=0, id=2328, seq=1."
2022-11-30 18:16:43 id=20085 trace_id=30 func=init_ip_session_common line=6024 msg="allocate a new session-000e4d3d, tun_id=0.0.0.0"
2022-11-30 18:16:43 id=20085 trace_id=30 func=vf_ip_route_input_common line=2605 msg="find a route: flag=04000000 gw-192.168.89.254 via port1"
2022-11-30 18:16:43 id=20085 trace_id=30 func=fw_forward_handler line=719 msg="Denied by forward policy check (policy 0)"
```

## "Denied by forward policy check (policy XX)"根本原因

```
从Debug Flow输出发现192.168.1.10 ping 114.114.114.114被policy 9丢弃

2022-11-30 18:22:05 id=20085 trace_id=37 func=print_pkt_detail line=5845 msg="vd-root:0 received a packet(proto=1, 192.168.1.10:2335->114.114.114.114:2048) tun_id=0.0.0.0 from port5. type=8, code=0, id=2335, seq=1."
2022-11-30 18:22:05 id=20085 trace_id=37 func=init_ip_session_common line=6024 msg="allocate a new session-000e5327, tun_id=0.0.0.0"
2022-11-30 18:22:05 id=20085 trace_id=37 func=vf_ip_route_input_common line=2605 msg="find a route: flag=04000000 gw-192.168.89.254 via port1"
2022-11-30 18:22:05 id=20085 trace_id=37 func=fw_forward_handler line=719 msg="Denied by forward policy check (policy 9)"

policy id 为9的策略如下：
config firewall policy
    edit 9
        set srcintf "port5"
        set dstintf "port1"
        set srcaddr "all"
        set dstaddr "all"
        set schedule "always"
        set service "ALL"
        set logtraffic disable
        set action deny
    next
end
```

## "reverse path check fail, drop"根本原因

```
拓扑：PC1(192.168.5.10)-----(192.168.5.1)Router(192.168.1.2)-----(port5:192.168.1.1)FortiGate(port1:192.168.89.35)----Internet

FortiGate路由配置
config router static
    edit 1
    	set dst 0.0.0.0/0
        set gateway 192.168.89.254
        set device "port1"
    next
end

从Debug Flow输出发现192.168.5.10 ping 114.114.114.114被反向路径检查丢弃，数据包是从port5接口收到的，FortiGate会检查port5接口是否有到源地址192.168.5.10的路由，如果没有，则将丢弃该报文。

2022-11-30 19:02:32 id=20085 trace_id=38 func=print_pkt_detail line=5845 msg="vd-root:0 received a packet(proto=1, 192.168.5.10:2471->114.114.114.114:2048) tun_id=0.0.0.0 from port5. type=8, code=0, id=2471, seq=1."
2022-11-30 19:02:32 id=20085 trace_id=38 func=init_ip_session_common line=6024 msg="allocate a new session-000e7d86, tun_id=0.0.0.0"
2022-11-30 19:02:32 id=20085 trace_id=38 func=ip_route_input_slow line=2267 msg="reverse path check fail, drop"
2022-11-30 19:02:32 id=20085 trace_id=38 func=ip_session_handle_no_dst line=6110 msg="trace"

解决方法：增加到192.168.5.0/24的路由
config router static
    edit 2
        set dst 192.168.5.0/24
        set gateway 192.168.1.2
        set device "port5"
    next
end

```

## **"iprope_in_check() check failed on policy 0, drop"根本原因**

1. **访问一个防火墙上未开放的端口（包括自身管理端口/VIP端口等）**

   ```
   port5接口只允许ping
   config system interface
       edit "port5"
           set ip 192.168.1.1 255.255.255.0
           set allowaccess ping
       next
   end
   
   从Debug Flow发现192.168.1.10访问FortiGate port5接口192.168.1.1的22端口被拒绝，因为port5没有允许SSH访问。
   
   2022-11-30 19:27:23 id=20085 trace_id=52 func=print_pkt_detail line=5845 msg="vd-root:0 received a packet(proto=6, 192.168.1.10:53814->192.168.1.1:22) tun_id=0.0.0.0 from port5. flag [S], seq 2134956194, ack 0, win 29200"
   2022-11-30 19:27:23 id=20085 trace_id=52 func=init_ip_session_common line=6024 msg="allocate a new session-000eb1ae, tun_id=0.0.0.0"
   2022-11-30 19:27:23 id=20085 trace_id=52 func=vf_ip_route_input_common line=2605 msg="find a route: flag=84000000 gw-192.168.1.1 via root"
   2022-11-30 19:27:23 id=20085 trace_id=52 func=fw_local_in_handler line=500 msg="iprope_in_check() check failed on policy 0, drop"
   ```

2. **访问FGT本机流量，接口下已经开启了此服务，但是配置了管理员可信任主机，可信任主机不包括发起访问的源IP**

   ```
   port5接口允许ping，ssh
   config system interface
       edit "port5"
           set ip 192.168.1.1 255.255.255.0
           set allowaccess ping ssh
       next
   end
   config system admin
       edit "admin"
           set trusthost1 192.168.88.0 255.255.255.0
       next
   end
   
   从Debug Flow发现192.168.1.10访问FortiGate port5接口192.168.1.1的22端口被拒绝，因为FortiGate设置了信任主机，只允许192.168.88.0/24段的地址访问
   
   2022-11-30 19:35:13 id=20085 trace_id=58 func=print_pkt_detail line=5845 msg="vd-root:0 received a packet(proto=6, 192.168.1.10:53818->192.168.1.1:22) tun_id=0.0.0.0 from port5. flag [S], seq 419563927, ack 0, win 29200"
   2022-11-30 19:35:13 id=20085 trace_id=58 func=init_ip_session_common line=6024 msg="allocate a new session-000ec1a6, tun_id=0.0.0.0"
   2022-11-30 19:35:13 id=20085 trace_id=58 func=vf_ip_route_input_common line=2605 msg="find a route: flag=84000000 gw-192.168.1.1 via root"
   ```

3. **访问FGT本机流量，接口下已经开启了此服务，但是访问是从另外的一个接口发起的，而另外一个接口到本接口没有配置相应的防火墙策略**

   ```
   port5接口允许ping，ssh
   
   config system interface
       edit "port1"
           set ip 192.168.89.35 255.255.255.0
           set allowaccess ping ssh
       next
   end
   
   从Debug Flow发现192.168.1.10访问FortiGate port1接口192.168.89.32的22端口被拒绝，因为数据包是从port5接口来的，因此被拒绝
   
   2022-11-30 19:42:32 id=20085 trace_id=72 func=print_pkt_detail line=5845 msg="vd-root:0 received a packet(proto=6, 192.168.1.10:54384->192.168.89.35:22) tun_id=0.0.0.0 from port5. flag [S], seq 2800378420, ack 0, win 29200"
   2022-11-30 19:42:32 id=20085 trace_id=72 func=init_ip_session_common line=6024 msg="allocate a new session-000ed260, tun_id=0.0.0.0"
   2022-11-30 19:42:32 id=20085 trace_id=72 func=vf_ip_route_input_common line=2605 msg="find a route: flag=84000000 gw-192.168.89.35 via root"
   2022-11-30 19:42:32 id=20085 trace_id=72 func=fw_local_in_handler line=500 msg="iprope_in_check() check failed on policy 0, drop"
   
   解决方法：增加port5到port1接口的策略
   config firewall policy
       edit 0
           set srcintf "port5"
           set dstintf "port1"
           set action accept
           set srcaddr "all"
           set dstaddr "192.168.89.35/32"
           set schedule "always"
           set service "ALL"
           set logtraffic all
       next
   end
   ```

   

