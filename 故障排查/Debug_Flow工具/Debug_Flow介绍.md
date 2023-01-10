# Debug Flow介绍

Debug Flow  通常用于定位调试穿过或访问FortiGate数据流的处理过程，对于定位问题有很大的帮助。

## 命令解析

```
Debug Flow命令解析：
diagnose debug flow filter addr x.x.x.x                //过滤某个IP
diagnose debug flow show function-name enable          //显示功能模块名称
diagnose debug console timestamp enable                //显示时间戳
diagnose debug flow trace start 10                    //开启debug flow trace并显示10条debug信息
diagnose debug enable                                  //开启debug命令

diagnose debug flow trace stop                         //关闭debug flow trace
diagnose debug flow filter clear                       //清除过滤条件
diagnose debug disable                                 //关闭debug命令
diagnose debug reset                                   //重置所有的debug命令
```

## 过滤条件

```
# diagnose debug  flow filter            //敲 ? 查看Debug Flow支持的过滤条件，如IP，端口，协议等
clear      Clear filter.
vd         Index of virtual domain.
vd-name    Name of virtual domain.
proto      Protocol number.
addr       IP address.
saddr      Source IP address.
daddr      Destination IP address.
port       port
sport      Source port.
dport      Destination port.
negate     Inverse filter.


# diagnose debug  flow filter           //直接敲回车查看当前的过滤条件
        vf: any
        proto: any
        Host addr: any
        Host saddr: any
        Host daddr: any
        port: any
        sport: any
        dport: any

# diagnose debug  flow filter clear     //清除filter过滤条件
```

## Debug Flow流程分析

1. 拓扑和策略配置如下。

   ```
   PC1(192.168.1.10)-----(port5：192.168.1.1)FGT(port1:192.168.89.35)-----Internet
   
   config firewall policy
       edit 9
           set srcintf "port5"
           set dstintf "port1"
           set action accept
           set srcaddr "all"
           set dstaddr "all"
           set schedule "always"
           set service "ALL"
           set nat enable
       next
   end
   ```

2. 设置Debug Flow，并使用PC1 ping 114.114.114.114。

   ```
   diagnose debug flow filter addr 114.114.114.114
   diagnose debug flow filter proto 1
   diagnose debug flow show function-name enable
   diagnose debug console timestamp enable
   diagnose debug flow trace start 10
   diagnose debug enable
   ```

3. Debug Flow输出分析。

   - 第1个数据包：ping的请求报文echo reqeust：

     ```
     2022-11-30 17:22:54 id=20085 trace_id=26 func=print_pkt_detail line=5845 msg="vd-root:0 received a packet(proto=1, 192.168.1.10:2280->114.114.114.114:2048) tun_id=0.0.0.0 from port5. type=8, code=0, id=2280, seq=1."
     //received a packet表示FortiGate从port5收到一个192.168.1.10到114.114.114.114的echo request报文
     
     2022-11-30 17:22:54 id=20085 trace_id=26 func=init_ip_session_common line=6024 msg="allocate a new session-000e1480, tun_id=0.0.0.0"
     //创建会话，会话id是000e1480
     
     2022-11-30 17:22:54 id=20085 trace_id=26 func=vf_ip_route_input_common line=2605 msg="find a route: flag=04000000 gw-192.168.89.254 via port1"
     //查找路由，该报文需要从port1转发出去
     
     2022-11-30 17:22:54 id=20085 trace_id=26 func=get_new_addr line=1221 msg="find SNAT: IP-192.168.89.35(from IPPOOL), port-62696"
     //需要执行源NAT
     
     2022-11-30 17:22:54 id=20085 trace_id=26 func=fw_forward_handler line=881 msg="Allowed by Policy-9: SNAT"
     //报文匹配id是9的策略
     
     2022-11-30 17:22:54 id=20085 trace_id=26 func=__ip_session_run_tuple line=3471 msg="SNAT 192.168.1.10->192.168.89.35:62696"
     //将数据包的源地址192.168.1.10转换为192.168.89.35并发出
     ```
   
   - 第2个数据包：ping的响应报文 echo reply：
   
     ```
     2022-11-30 17:22:54 id=20085 trace_id=27 func=print_pkt_detail line=5845 msg="vd-root:0 received a packet(proto=1, 114.114.114.114:62696->192.168.89.35:0) tun_id=0.0.0.0 from port1. type=0, code=0, id=62696, seq=1."
     //received a packet表示FortiGate从port1收到一个114.114.114.114到的192.168.89.35 echo reply报文
     
     2022-11-30 17:22:54 id=20085 trace_id=27 func=resolve_ip_tuple_fast line=5931 msg="Find an existing session, id-000e1480, reply direction"
     //匹配已创建的会话id-000e1480
     
     2022-11-30 17:22:54 id=20085 trace_id=27 func=__ip_session_run_tuple line=3484 msg="DNAT 192.168.89.35:0->192.168.1.10:2280"
     //反向报文需要进行目的NAT转换，将192.168.89.35转换为PC1的真实IP 192.168.1.10
     
     2022-11-30 17:22:54 id=20085 trace_id=27 func=vf_ip_route_input_common line=2605 msg="find a route: flag=00000000 gw-192.168.1.10 via port5"
     //查找路由，该报文需要从port5转发出去
     
     2022-11-30 17:22:54 id=20085 trace_id=27 func=npu_handle_session44 line=1183 msg="Trying to offloading session from port1 to port5, skb.npu_flag=00000000 ses.state=00010200 ses.npu_state=0x04000000"
     2022-11-30 17:22:54 id=20085 trace_id=27 func=fw_forward_dirty_handler line=410 msg="state=00010200, state2=00000000, npu_state=04000000"
     ```
   
   - 第3个数据包：ping的请求报文 echo request直接匹配会话转发：
   
     ```
     2022-11-30 17:22:55 id=20085 trace_id=28 func=print_pkt_detail line=5845 msg="vd-root:0 received a packet(proto=1, 192.168.1.10:2280->114.114.114.114:2048) tun_id=0.0.0.0 from port5. type=8, code=0, id=2280, seq=2."
     2022-11-30 17:22:55 id=20085 trace_id=28 func=resolve_ip_tuple_fast line=5931 msg="Find an existing session, id-000e1480, original direction"
     2022-11-30 17:22:55 id=20085 trace_id=28 func=npu_handle_session44 line=1183 msg="Trying to offloading session from port5 to port1, skb.npu_flag=00000400 ses.state=00010200 ses.npu_state=0x04000000"
     2022-11-30 17:22:55 id=20085 trace_id=28 func=ip_session_install_npu_session line=346 msg="npu session installation succeeded"
     2022-11-30 17:22:55 id=20085 trace_id=28 func=fw_forward_dirty_handler line=410 msg="state=00010200, state2=00000000, npu_state=04000400"
     2022-11-30 17:22:55 id=20085 trace_id=28 func=__ip_session_run_tuple line=3471 msg="SNAT 192.168.1.10->192.168.89.35:62696"
     ```
   
   - 第4个数据包：ping的影响报文echo reply直接匹配会话转发：
   
     ```
     2022-11-30 17:22:55 id=20085 trace_id=29 func=print_pkt_detail line=5845 msg="vd-root:0 received a packet(proto=1, 114.114.114.114:62696->192.168.89.35:0) tun_id=0.0.0.0 from port1. type=0, code=0, id=62696, seq=2."
     2022-11-30 17:22:55 id=20085 trace_id=29 func=resolve_ip_tuple_fast line=5931 msg="Find an existing session, id-000e1480, reply direction"
     2022-11-30 17:22:55 id=20085 trace_id=29 func=__ip_session_run_tuple line=3484 msg="DNAT 192.168.89.35:0->192.168.1.10:2280"
     2022-11-30 17:22:55 id=20085 trace_id=29 func=npu_handle_session44 line=1183 msg="Trying to offloading session from port1 to port5, skb.npu_flag=00000400 ses.state=00010200 ses.npu_state=0x04000400"
     2022-11-30 17:22:55 id=20085 trace_id=29 func=ip_session_install_npu_session line=346 msg="npu session installation succeeded"
     2022-11-30 17:22:55 id=20085 trace_id=29 func=fw_forward_dirty_handler line=410 msg="state=00010200, state2=00000000, npu_state=04000c00"
     ```
   
   
