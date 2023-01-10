# Sniffer、Debug Flow与NP加速的矛盾

**疑问1：diagnose  sniffer抓不到任何数据包或者数据包抓不全的情况，是什么原因呢？** 

**疑问2：Debug Flow没有输出，是什么原因呢？** 

**答案：** 这是由于数据被NP加速后，数据直接由NP转发，不经过kernel，因此sniffer，debug flow将抓不到相应的数据。 

只有走kernel的数据才能被完整的抓获，通常是新建数据流才可以抓到包，比如：ICMP的第一个/第二个包、TCP三次握手和四次拆链的包、UDP的第一个/第二个包。



**原因分析**

```
拓扑：PC1(192.168.1.10)-----(port5：192.168.1.1)FGT(port1:192.168.89.35)-----Internet

PC1(192.168.1.10) ping 114.114.114.114

设置Debug Flow
diagnose debug flow filter addr 114.114.114.114
diagnose debug flow filter proto 1
diagnose debug flow show function-name enable
diagnose debug console timestamp enable
diagnose debug flow trace start 10
diagnose debug enable

第1个数据包：ping请求echo request
2022-11-30 19:54:07 id=20085 trace_id=87 func=print_pkt_detail line=5845 msg="vd-root:0 received a packet(proto=1, 192.168.1.10:2589->114.114.114.114:2048) tun_id=0.0.0.0 from port5. type=8, code=0, id=2589, seq=1."
2022-11-30 19:54:07 id=20085 trace_id=87 func=init_ip_session_common line=6024 msg="allocate a new session-000ee70c, tun_id=0.0.0.0"
2022-11-30 19:54:07 id=20085 trace_id=87 func=vf_ip_route_input_common line=2605 msg="find a route: flag=04000000 gw-192.168.89.254 via port1"
2022-11-30 19:54:07 id=20085 trace_id=87 func=get_new_addr line=1221 msg="find SNAT: IP-192.168.89.35(from IPPOOL), port-63005"
2022-11-30 19:54:07 id=20085 trace_id=87 func=fw_forward_handler line=881 msg="Allowed by Policy-9: SNAT"
2022-11-30 19:54:07 id=20085 trace_id=87 func=__ip_session_run_tuple line=3471 msg="SNAT 192.168.1.10->192.168.89.35:63005"

第2个数据包：ping响应echo reply
2022-11-30 19:54:07 id=20085 trace_id=88 func=print_pkt_detail line=5845 msg="vd-root:0 received a packet(proto=1, 114.114.114.114:63005->192.168.89.35:0) tun_id=0.0.0.0 from port1. type=0, code=0, id=63005, seq=1."
2022-11-30 19:54:07 id=20085 trace_id=88 func=resolve_ip_tuple_fast line=5931 msg="Find an existing session, id-000ee70c, reply direction"
2022-11-30 19:54:07 id=20085 trace_id=88 func=__ip_session_run_tuple line=3484 msg="DNAT 192.168.89.35:0->192.168.1.10:2589"
2022-11-30 19:54:07 id=20085 trace_id=88 func=vf_ip_route_input_common line=2605 msg="find a route: flag=00000000 gw-192.168.1.10 via port5"
2022-11-30 19:54:07 id=20085 trace_id=88 func=npu_handle_session44 line=1183 msg="Trying to offloading session from port1 to port5, skb.npu_flag=00000000 ses.state=00010204 ses.npu_state=0x04000000"
2022-11-30 19:54:07 id=20085 trace_id=88 func=fw_forward_dirty_handler line=410 msg="state=00010204, state2=00000001, npu_state=04000000"

第3个数据包：ping请求echo request，数据包发起方向推送到NP加速：npu session installation succeeded
2022-11-30 19:54:08 id=20085 trace_id=89 func=print_pkt_detail line=5845 msg="vd-root:0 received a packet(proto=1, 192.168.1.10:2589->114.114.114.114:2048) tun_id=0.0.0.0 from port5. type=8, code=0, id=2589, seq=2."
2022-11-30 19:54:08 id=20085 trace_id=89 func=resolve_ip_tuple_fast line=5931 msg="Find an existing session, id-000ee70c, original direction"
2022-11-30 19:54:08 id=20085 trace_id=89 func=npu_handle_session44 line=1183 msg="Trying to offloading session from port5 to port1, skb.npu_flag=00000400 ses.state=00010204 ses.npu_state=0x04000000"
2022-11-30 19:54:08 id=20085 trace_id=89 func=ip_session_install_npu_session line=346 msg="npu session installation succeeded"
2022-11-30 19:54:08 id=20085 trace_id=89 func=fw_forward_dirty_handler line=410 msg="state=00010204, state2=00000001, npu_state=04000400"

第4个数据包：ping响应echo reply，数据包的响应方向推送到NP加速：npu session installation succeeded
2022-11-30 19:54:08 id=20085 trace_id=89 func=__ip_session_run_tuple line=3471 msg="SNAT 192.168.1.10->192.168.89.35:63005"
2022-11-30 19:54:08 id=20085 trace_id=90 func=print_pkt_detail line=5845 msg="vd-root:0 received a packet(proto=1, 114.114.114.114:63005->192.168.89.35:0) tun_id=0.0.0.0 from port1. type=0, code=0, id=63005, seq=2."
2022-11-30 19:54:08 id=20085 trace_id=90 func=resolve_ip_tuple_fast line=5931 msg="Find an existing session, id-000ee70c, reply direction"
2022-11-30 19:54:08 id=20085 trace_id=90 func=__ip_session_run_tuple line=3484 msg="DNAT 192.168.89.35:0->192.168.1.10:2589"
2022-11-30 19:54:08 id=20085 trace_id=90 func=npu_handle_session44 line=1183 msg="Trying to offloading session from port1 to port5, skb.npu_flag=00000400 ses.state=00010204 ses.npu_state=0x04000400"
2022-11-30 19:54:08 id=20085 trace_id=90 func=ip_session_install_npu_session line=346 msg="npu session installation succeeded"
2022-11-30 19:54:08 id=20085 trace_id=90 func=fw_forward_dirty_handler line=410 msg="state=00010204, state2=00000001, npu_state=04000c00"

FGT # diagnose sniffer packet any 'host 114.114.114.114 and icmp' 4
interfaces=[any]
filters=[host 114.114.114.114 and icmp]
2.544370 port5 in 192.168.1.10 -> 114.114.114.114: icmp: echo request
2.544394 port1 out 192.168.89.35 -> 114.114.114.114: icmp: echo request

2.564639 port1 in 114.114.114.114 -> 192.168.89.35: icmp: echo reply
2.564941 port5 out 114.114.114.114 -> 192.168.1.10: icmp: echo reply

3.546345 port5 in 192.168.1.10 -> 114.114.114.114: icmp: echo request
3.546356 port1 out 192.168.89.35 -> 114.114.114.114: icmp: echo request

3.566460 port1 in 114.114.114.114 -> 192.168.89.35: icmp: echo reply
3.566467 port5 out 114.114.114.114 -> 192.168.1.10: icmp: echo reply

后续的ping报文直接通过NP芯片转发，因此sniffer和debug flow都没有输出
此时查看session list，offload=8/8表示数据包卸载到NP，即由NP芯片转发
# diagnose sys session list 

session info: proto=1 proto_state=00 duration=9 expire=51 timeout=0 flags=00000000 socktype=0 sockport=0 av_idx=0 use=3
origin-shaper=
reply-shaper=
per_ip_shaper=
class_id=0 ha_id=0 policy_dir=0 tunnel=/ vlan_cos=0/255
state=log may_dirty npu f00 
statistic(bytes/packets/allow_err): org=168/2/1 reply=168/2/1 tuples=2
tx speed(Bps/kbps): 0/0 rx speed(Bps/kbps): 0/0
orgin->sink: org pre->post, reply pre->post dev=13->9/9->13 gwy=192.168.89.254/192.168.1.10
hook=post dir=org act=snat 192.168.1.10:2589->114.114.114.114:8(192.168.89.35:63005)
hook=pre dir=reply act=dnat 114.114.114.114:63005->192.168.89.35:0(192.168.1.10:2589)
misc=0 policy_id=9 pol_uuid_idx=521 auth_info=0 chk_client_info=0 vd=0
serial=000ee70c tos=ff/ff app_list=0 app=0 url_cat=0
rpdb_link_id=00000000 ngfwid=n/a
npu_state=0x4000c00 ofld-O ofld-R
npu info: flag=0x81/0x81, offload=8/8, ips_offload=0/0, epid=148/156, ipid=156/148, vlan=0x0000/0x0000
vlifid=156/148, vtag_in=0x0000/0x0000 in_npu=1/1, out_npu=1/1, fwd_en=0/0, qid=0/3

```



**解决方法1：** 

1. 对于需要抓包的流量新建一条临时策略关闭NP加速 或者 在对应策略（找到流量对应的策略及策略ID）里临时关掉NP加速。

   ```
   FGT # config firewall policy
   FGT (policy) # edit 27                      //注意此处27是策略ID号，不是策略顺序号
   FGT (27) # set auto-asic-offload disable    //在策略中临时关闭NP加速,Sniffer完毕可再enable此命令
   FGT (27) # end
   这样数据的处理会全部走kernel处理，因此抓包才能抓完整。
   ```

2. 对于已经建立起来的会话（数据已经走NP处理了），可以先过滤出来并清除掉，然后再抓包。

   ```
   diagnose sys session filter dst 202.106.1.100
   diagnose sys session filter proto 1
   diagnose sys session list                //通过session list可以找到对应的策略ID，还可以看到NP加速的标志位 
   diagnose sys session clear               //清空该会话
   ```

3. 最后再开启抓包命令，这样就可以抓到完全的数据流了。

   ```
   diagnose sniffer packet any "host 202.106.1.100 and icmp" 4
   ```

**解决方法2**

1. 不关闭NP情况下的组合抓包脚本，但是只能抓到一次性的加速前的数据包，加速后的包将无法显示出来

   ```
   先将需要抓包的数据流对应的会话清除
   diagnose sys session filter proto 1 
   diagnose sys session filter dst 114.114.114.114 
   diagnose sys session clear
   
   再通过抓包命令抓取，可以抓取NP加速前的报文
   diagnose sniffer packet any "host 114.114.114.114 and icmp" 4 0 l
   ```

   

