# IPSec业务不通问题

## 网络拓扑

PC1-----------(port5:192.168.0.1/24)FGT-BJ(port2:100.1.1.2)-------------Internet-------------(port2:200.1.1.2)FGT-SH(port5:192.168.1.1/24)-----------PC2

## 隧道UP了，但是业务不通，如何排查

1. 抓包查看数据是否通过IPSEC接口转发

   ```
   # diagnose sniffer packet any icmp 4
   interfaces=[any]
   filters=[icmp]
   2.615008 port5 in 192.168.0.10 -> 192.168.1.10: icmp: echo request
   2.615030 VPN-to-SH out 192.168.0.10 -> 192.168.1.10: icmp: echo request     #VPN-to-SH是IPSEC接口
   2.615891 VPN-to-SH in 192.168.1.10 -> 192.168.0.10: icmp: echo reply
   2.616067 port5 out 192.168.1.10 -> 192.168.0.10: icmp: echo reply
   ```

2. 通过Debug Flow查看匹配的策略和路由是否正确

   ```
   #diagnose debug flow filter addr 192.168.1.10
   #diagnose debug flow show function-name enable
   #diagnose debug console timestamp enable
   #diagnose debug flow trace start 5
   #diagnose debug enable
   
   从PC1 192.168.0.10 ping PC2 192.168.1.10 第1个数据包 echo request
   2022-12-30 17:50:12 id=20085 trace_id=1 func=print_pkt_detail line=5845 msg="vd-root:0 received a packet(proto=1, 192.168.0.10:23728->192.168.1.10:2048) tun_id=0.0.0.0 from port5. type=8, code=0, id=23728, seq=1."
   2022-12-30 17:50:12 id=20085 trace_id=1 func=init_ip_session_common line=6024 msg="allocate a new session-00003fab, tun_id=0.0.0.0"
   2022-12-30 17:50:12 id=20085 trace_id=1 func=vf_ip_route_input_common line=2605 msg="find a route: flag=04000000 gw-200.1.1.2 via VPN-to-SH"
   2022-12-30 17:50:12 id=20085 trace_id=1 func=fw_forward_handler line=881 msg="Allowed by Policy-4:"
   2022-12-30 17:50:12 id=20085 trace_id=1 func=ipsecdev_hard_start_xmit line=669 msg="enter IPSec interface VPN-to-SH, tun_id=0.0.0.0"
   2022-12-30 17:50:12 id=20085 trace_id=1 func=_do_ipsecdev_hard_start_xmit line=229 msg="output to IPSec tunnel VPN-to-SH"
   2022-12-30 17:50:12 id=20085 trace_id=1 func=esp_output4 line=844 msg="IPsec encrypt/auth"
   2022-12-30 17:50:12 id=20085 trace_id=1 func=ipsec_output_finish line=544 msg="send to 100.1.1.1 via intf-port2"
   
   PC2到PC1的响应包 echo reply
   2022-12-30 17:50:12 id=20085 trace_id=2 func=print_pkt_detail line=5845 msg="vd-root:0 received a packet(proto=1, 192.168.1.10:23728->192.168.0.10:0) tun_id=200.1.1.2 from VPN-to-SH. type=0, code=0, id=23728, seq=1."
   2022-12-30 17:50:12 id=20085 trace_id=2 func=resolve_ip_tuple_fast line=5931 msg="Find an existing session, id-00003fab, reply direction"
   2022-12-30 17:50:12 id=20085 trace_id=2 func=vf_ip_route_input_common line=2605 msg="find a route: flag=00000000 gw-192.168.0.10 via port5"
   2022-12-30 17:50:12 id=20085 trace_id=2 func=npu_handle_session44 line=1183 msg="Trying to offloading session from VPN-to-SH to port5, skb.npu_flag=00000000 ses.state=00010200 ses.npu_state=0x05040000"
   2022-12-30 17:50:12 id=20085 trace_id=2 func=fw_forward_dirty_handler line=410 msg="state=00010200, state2=00000000, npu_state=05040000"
   ```


