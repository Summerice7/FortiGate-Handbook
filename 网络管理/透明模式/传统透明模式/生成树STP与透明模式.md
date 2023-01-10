# 生成树(STP)与透明模式

## 组网需求

SW1和SW2之间运行了STP协议，防火墙以透明模式运行在两台SW之间，SW和SW之间原本存在环路，通过STP进行隔离阻断了环路接口，在加入透明模式的FGT之后，需要继续确保STP工作正常。

## 网络拓扑

<img src="../../../images/image-20221114170712038.png" alt="image-20221114170712038" style="zoom:50%;" />

## 配置要点

- SW1/SW2交换机的基础配置
- 将防火墙配置为透明模式并开启网管

- 默认情况下，观察STP报文的情况

- 配置l2forward解决STP流量转发问题

## 配置步骤与结果验证

1. SW1/SW2交换机的基础配置。

   **SW1：**

   ```
   hostname SW1
   spanning-tree vlan 1 priority 32768
   ```

   **SW2：**

   ```
   hostname SW2
   !
   spanning-tree vlan 1 priority 4096
   ```

2. 查看没有加入透明模式的FortiGate之前STP状态，STP选举和接口状态正常。

   ```
   SW1#show spanning-tree 
   
   VLAN0001
     Spanning tree enabled protocol rstp
     Root ID    Priority    4097
                Address     5000.0008.0000
                Cost        4
                Port        1 (GigabitEthernet0/0)
                Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
     Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
                Address     5000.0007.0000
                Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
                Aging Time  300 sec
   Interface           Role Sts Cost      Prio.Nbr Type
   ------------------- ---- --- --------- -------- --------------------------------
   Gi0/0               Root FWD 4         128.1    Shr
   Gi0/3               Altn BLK 4         128.4    Shr     //G0/3被阻断，STP工作正常
   
   SW2#show spanning-tree 
   VLAN0001
     Spanning tree enabled protocol rstp
     Root ID    Priority    4097
                Address     5000.0008.0000
                This bridge is the root
                Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
   
     Bridge ID  Priority    4097   (priority 4096 sys-id-ext 1)
                Address     5000.0008.0000
                Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
                Aging Time  300 sec
   
   Interface           Role Sts Cost      Prio.Nbr Type
   ------------------- ---- --- --------- -------- --------------------------------
   Gi0/0               Desg FWD 4         128.1    Shr
   Gi0/3               Desg FWD 4         128.4    Shr
   ```

3. 将防火墙配置为透明模式并开启网管并加入到SW1和SW2之间，进入设备命令行（CLI）中进行配置，将模式修改为"透明模式"同时为设备配置好管理地址和网关。

   ```
   FortiGate-VM64-KVM # config system global
   FortiGate-VM64-KVM (global) # set hostname FortiGate_Transparent
   FortiGate_Transparent (global) # set timezone 55
   FortiGate_Transparent (global) # set language simch
   FortiGate-VM64-KVM (global) # end
   FortiGate_Transparent #
   FortiGate_Transparent # config system settings
   
   FortiGate_Transparent (settings) # set opmode transparent    //修改FGT的运行模式为透明模式，默认为NAT路由模式。注意切换透明模式防火墙需要防火墙没有相关接口、策略、路由等配置。
   FortiGate_Transparent (settings) # set manageip 192.168.1.100 255.255.255.0    //配置可以管理防火墙的本地IP和网关，以便HTTP/SSH管理防火墙及防火墙的服务更新。
   FortiGate_Transparent (settings) # set gateway 192.168.1.99
   FortiGate_Transparent (settings) # end
   Changing to TP mode
   
   MGMT1或MGMT2口默认有管理权限，以要通过port1（LAN）接口管理设备为例，开启port1（LAN）管理FGT的命令如下：
   FortiGate_Transparent # config system interface
   FortiGate_Transparent (interface) # edit port1
   FortiGate_Transparent (port1) # set allowaccess https http ping ssh  // 允许网管协议从Port1接口通过https/http/SSH/Ping访问透明模式的FortiGate
   FortiGate_Transparent (port1) # end
   ```

4. 加入FGT在没有任何配置的情况下，观察STP的运行情况，一旦FGT加入到SW1和SW2之间，很明显网络中出现了环路，设备操作很卡顿，SW1和SW2基本上不再响应请求，三台设备均出现了异常情况（默认情况下透明模式的FGT只转发ARP报文，因此只需要一个ARP报文就会产生广播风暴，引起环路导致设备、业务中断）。

   ```
   FortiGate_Transparent # diagnose sniffer packet any "" 4
   interfaces=[any]
   filters=[none]
   0.814720 port2 in arp who-has 192.168.1.99 tell 192.168.1.100
   0.814733 port1 out arp who-has 192.168.1.99 tell 192.168.1.100
   0.817141 port1 in arp who-has 192.168.1.99 tell 192.168.1.100
   0.817148 port2 out arp who-has 192.168.1.99 tell 192.168.1.100
   0.818761 port1 in arp who-has 192.168.1.99 tell 192.168.1.100
   0.818771 port2 out arp who-has 192.168.1.99 tell 192.168.1.100
   0.819709 port2 in arp who-has 192.168.1.99 tell 192.168.1.100
   0.819720 port1 out arp who-has 192.168.1.99 tell 192.168.1.100
   0.821533 port2 in arp who-has 192.168.1.99 tell 192.168.1.100
   0.821544 port1 out arp who-has 192.168.1.99 tell 192.168.1.100
   0.825484 port1 in arp who-has 192.168.1.99 tell 192.168.1.100
   0.825492 port2 out arp who-has 192.168.1.99 tell 192.168.1.100
   0.827856 port1 in arp who-has 192.168.1.99 tell 192.168.1.100
   0.827866 port2 out arp who-has 192.168.1.99 tell 192.168.1.100
   0.828752 port1 in arp who-has 192.168.1.99 tell 192.168.1.100
   0.828762 port2 out arp who-has 192.168.1.99 tell 192.168.1.100
   0.829235 port2 in arp who-has 192.168.1.99 tell 192.168.1.100
   0.829244 port1 out arp who-has 192.168.1.99 tell 192.168.1.100
   0.831930 port2 in arp who-has 192.168.1.99 tell 192.168.1.100
   0.831947 port1 out arp who-has 192.168.1.99 tell 192.168.1.100
   0.834842 port2 in arp who-has 192.168.1.99 tell 192.168.1.100
   0.834853 port1 out arp who-has 192.168.1.99 tell 192.168.1.100
   0.836686 port2 in arp who-has 192.168.1.99 tell 192.168.1.100
   0.836694 port1 out arp who-has 192.168.1.99 tell 192.168.1.100
   0.838965 port1 in arp who-has 192.168.1.99 tell 192.168.1.100
   0.838972 port2 out arp who-has 192.168.1.99 tell 192.168.1.100
   0.841965 port1 in arp who-has 192.168.1.99 tell 192.168.1.100
   0.841974 port2 out arp who-has 192.168.1.99 tell 192.168.1.100
   0.843858 port2 in arp who-has 192.168.1.99 tell 192.168.1.100
   0.843866 port1 out arp who-has 192.168.1.99 tell 192.168.1.100
   ```

5. 查看SW1/SW2的 STP状态均异常，所有接口都处于转发状态。

   ```
   SW1#show spanning-tree
   VLAN0001
     Spanning tree enabled protocol rstp
     Root ID    Priority    4097
                Address     5000.0008.0000
                Cost        4
                Port        4 (GigabitEthernet0/3)
                Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
     Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
                Address     5000.0007.0000
                Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
                Aging Time  300 sec
   Interface           Role Sts Cost      Prio.Nbr Type
   ------------------- ---- --- --------- -------- --------------------------------
   Gi0/0               Desg FWD 4         128.1    Shr
   Gi0/3               Root FWD 4         128.4    Shr     //SW1的两个接口都处于转发状态
   
   SW2#show spanning-tree
   VLAN0001
     Spanning tree enabled protocol rstp
     Root ID    Priority    4097
                Address     5000.0008.0000
                This bridge is the root
                Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
     Bridge ID  Priority    4097   (priority 4096 sys-id-ext 1)
                Address     5000.0008.0000
                Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
                Aging Time  300 sec
   Interface           Role Sts Cost      Prio.Nbr Type
   ------------------- ---- --- --------- -------- --------------------------------
   Gi0/0               Desg FWD 4         128.1    Shr
   Gi0/3               Desg FWD 4         128.4    Shr  //SW2的两个接口都处于转发状态
   ```

6. 在FGT上抓包可以看到STP的BPDU报文没有被转发，而是被默认丢弃，这是引起环路的根本原因。STP BPDU数据默认不会被透明模式的FGT所转发，因此STP计算不正确，会计算不出网络中的环路，从而引起广播风暴。

   ```
   FortiGate_Transparent # diagnose sniffer packet any "none" 4
   interfaces=[any]
   filters=[none]
   1.234590 port2 in stp 802.1w, rapid stp, flags [learn, forward], bridge-id 1001.50:00:00:08:00:00.8001
   2.182694 port1 in stp 802.1w, rapid stp, flags [learn, forward], bridge-id 8001.50:00:00:07:00:00.8001
   3.247650 port2 in stp 802.1w, rapid stp, flags [learn, forward], bridge-id 1001.50:00:00:08:00:00.8001
   4.193908 port1 in stp 802.1w, rapid stp, flags [learn, forward], bridge-id 8001.50:00:00:07:00:00.8001
   5.260154 port2 in stp 802.1w, rapid stp, flags [learn, forward], bridge-id 1001.50:00:00:08:00:00.8001
   6.204003 port1 in stp 802.1w, rapid stp, flags [learn, forward], bridge-id 8001.50:00:00:07:00:00.8001
   7.278226 port2 in stp 802.1w, rapid stp, flags [learn, forward], bridge-id 1001.50:00:00:08:00:00.8001
   8.210732 port1 in stp 802.1w, rapid stp, flags [learn, forward], bridge-id 8001.50:00:00:07:00:00.8001
   9.294582 port2 in stp 802.1w, rapid stp, flags [learn, forward], bridge-id 1001.50:00:00:08:00:00.8001
   10.223021 port1 in stp 802.1w, rapid stp, flags [learn, forward], bridge-id 8001.50:00:00:07:00:00.8001
   11.302829 port2 in stp 802.1w, rapid stp, flags [learn, forward], bridge-id 1001.50:00:00:08:00:00.8001
   12.232585 port1 in stp 802.1w, rapid stp, flags [learn, forward], bridge-id 8001.50:00:00:07:00:00.8001
   13.310023 port2 in stp 802.1w, rapid stp, flags [learn, forward], bridge-id 1001.50:00:00:08:00:00.8001
   14.242616 port1 in stp 802.1w, rapid stp, flags [learn, forward], bridge-id 8001.50:00:00:07:00:00.8001
   ```

7. 那么要如何解决此情况呢？在FGT的二层接口下配置set stpforward enable即可！

   ```
   FortiGate_Transparent # config system interface
   FortiGate_Transparent (interface) # edit port1
   FortiGate_Transparent (port1) # set stpforward enable
   
   FortiGate_Transparent (port1) # next
   FortiGate_Transparent (interface) # edit port2
   FortiGate_Transparent (port2) # set stpforward enable
   FortiGate_Transparent (port2) # end
   ```

8. 再次抓包查看，此时FGT可以转发STP BPDU报文了。

   ```
   FortiGate_Transparent # diagnose sniffer packet any "stp" 4
   interfaces=[any]
   filters=[stp]
   2.070782 port2 in stp 802.1w, rapid stp, flags [learn, forward], bridge-id 1001.50:00:00:08:00:00.8001
   2.070797 port1 out stp 802.1w, rapid stp, flags [learn, forward], bridge-id 1001.50:00:00:08:00:00.8001
   4.085338 port2 in stp 802.1w, rapid stp, flags [learn, forward], bridge-id 1001.50:00:00:08:00:00.8001
   4.085355 port1 out stp 802.1w, rapid stp, flags [learn, forward], bridge-id 1001.50:00:00:08:00:00.8001
   6.097429 port2 in stp 802.1w, rapid stp, flags [learn, forward], bridge-id 1001.50:00:00:08:00:00.8001
   6.097444 port1 out stp 802.1w, rapid stp, flags [learn, forward], bridge-id 1001.50:00:00:08:00:00.8001
   8.107806 port2 in stp 802.1w, rapid stp, flags [learn, forward], bridge-id 1001.50:00:00:08:00:00.8001
   8.107822 port1 out stp 802.1w, rapid stp, flags [learn, forward], bridge-id 1001.50:00:00:08:00:00.8001
   ```

9. 此时再次查看STP的状态。

   ```
   SW1#show spanning-tree
   VLAN0001
     Spanning tree enabled protocol rstp
     Root ID    Priority    4097
                Address     5000.0008.0000
                Cost        4
                Port        1 (GigabitEthernet0/0)
                Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
     Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
                Address     5000.0007.0000
                Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
                Aging Time  300 sec
   Interface           Role Sts Cost      Prio.Nbr Type
   ------------------- ---- --- --------- -------- --------------------------------
   Gi0/0               Root FWD 4         128.1    Shr
   Gi0/3               Altn BLK 4         128.4    Shr    //G0/3 被阻断，STP工作正常
   
   SW2#show spanning-tree
   VLAN0001
     Spanning tree enabled protocol rstp
     Root ID    Priority    4097
                Address     5000.0008.0000
                This bridge is the root
                Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
     Bridge ID  Priority    4097   (priority 4096 sys-id-ext 1)
                Address     5000.0008.0000
                Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
                Aging Time  300 sec
   Interface           Role Sts Cost      Prio.Nbr Type
   ------------------- ---- --- --------- -------- --------------------------------
   Gi0/0               Desg FWD 4         128.1    Shr
   Gi0/3               Desg FWD 4         128.4    Shr
   ```

> 要特别注意此情况！避免透明模式的FGT一上线引起STP计算异常，引起广播风暴，造成重大业务故障。
