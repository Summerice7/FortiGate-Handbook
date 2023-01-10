# SNMP的设置

## 需求

配置FortiGate的SNMP。

## 组网拓扑

<img src=".\..\..\..\images\image-20220905153745080.png" alt="image-20220905153745080" style="zoom:50%;" />

## 操作步骤

1. 接口上开启SNMP的允许访问。

   <img src=".\..\..\..\images\image-20220905161457124.png" alt="image-20220905161457124" style="zoom:50%;" />

2. 开启SNMP的总开关。

   <img src=".\..\..\..\images\image-20220905162012335.png" alt="image-20220905162012335" style="zoom:50%;" />

3. 配置SNMP v2的属性，比如配置SNMPv2的SNMP属性为Fortinet123#，建议不要使用通用的public，容易被探测和攻击。需要添加SNMP客户端的主机IP（192.168.77.77）的SNMP权限，把主机的IP地址（192.168.77.77）添加到SNMPv2的配置里面。

   <img src=".\..\..\..\images\image-20220905162159483.png" alt="image-20220905162159483" style="zoom:50%;" />

4. 配置SNMPv3，配置SNMPv3的用户名为fortinet，认证算法选择SHA1，密码为forti3389，加密算法选择为AES，密码为forti5566。需要添加SNMP客户端的主机IP（192.168.77.77）的SNMP权限，把主机的IP地址（192.168.77.77）添加到SNMPv2的配置里面。

   <img src=".\..\..\..\images\image-20220905162752462.png" alt="image-20220905162752462" style="zoom:50%;" />

5. SNMP命令行配置汇总。

   ```
   config system interface
       edit "port1"
           set allowaccess ping https ssh snmp http probe-response
       next
   end
   
   config system snmp sysinfo
       set status enable
       set description "Fortinet_Beijing_LAB"
       set contact-info "bbai@fortinet.com"
       set location "China_Beijing_Lab"
   end
   
   config system snmp community
       edit 1
           set name "Fortinet123#"
           config hosts
               edit 1
                   set ip 192.168.77.77 255.255.255.255
               next
           end
       next
   end
   
   config system snmp user
       edit "fortinet"
           set notify-hosts 192.168.77.77
           set security-level auth-priv
           set auth-pwd ENC MTAwNG2U70/eePNhi1N3/u4q6FSRuv2ebcPGgqV+yxRuFtKxFilE6SmYZfpMvYOQBU4InxdTXnlIeYdMguMT8x7Hsqz/Q+3G2DWXlmtJohv0RukHWQK4nkY/aYKCnujZkKGGyPxDKysAj4LDkR1CazeJkMKtVgGyoPF2WEjbPEt6PijsUZ67cDptDDzqnABzQFUemw==
           set priv-pwd ENC MTAwNFIePySDQu7hqz+SFic3AUQ+1G4HX5er9qOeMxFTkZ8I7DnW+rw3XYNwgSfQtHuWEwTZGmztt6kUYUaT+oxLSdLG/RqU/3wbbE1/m4MH64bSYZgd+c+Pks8S44UPmGBdQeewwpwQi7xHmL/y9Gyv+t7Wb6ge0WS3dQFe971Vp7n4evJgU51EK90Cnt7TjNX0lQ==
       next
   end
   ```

> **注意：有一个特别的场景需要特别注意，那就是HA独立管理口的场景，需要一些特殊的配置才可以搞定。后边的章节[SNMP有HA独立管理的设置](./SNMP有HA独立管理的设置.md)会描述这种情况。其他情况下都是以上类似的配置即可开启FortiGate的SNMP。**

## 结果验证

1. Net-SNMP SNMP V2结果验证。

   ```bash
   [Thu Aug 06 11:46:54 root@centos7~#snmpwalk -v2c -c Fortinet123# 192.168.77.1 .1.3.6.1.4.1.12356.101.4.1.1.0
   SNMPv2-SMI::enterprises.12356.101.4.1.1.0 = STRING: "v7.0.6,build0366,220606 (GA.F)"
   [Thu Aug 06 11:46:55 root@centos7~#snmpwalk -v2c -c Fortinet123# 192.168.77.1 1.3.6.1.2.1.1.3.0
   DISMAN-EVENT-MIB::sysUpTimeInstance = Timeticks: (301635896) 34 days, 21:52:38.96
   [Thu Aug 06 11:46:58 root@centos7~#snmpwalk -v2c -c Fortinet123# 192.168.77.1 .1.3.6.1.4.1.12356.100.1.1.1.0
   SNMPv2-SMI::enterprises.12356.100.1.1.1.0 = STRING: "FGVM08TM22000173"
   [Thu Aug 06 11:47:46 root@centos7~#snmpwalk -v2c -c Fortinet123# 192.168.77.1 1.3.6.1.2.1.31.1.1.1.1.1
   IF-MIB::ifName.1 = STRING: port1
   [Thu Aug 06 11:47:47 root@centos7~#snmpwalk -v2c -c Fortinet123# 192.168.77.1 1.3.6.1.2.1.31.1.1.1.1.2
   IF-MIB::ifName.2 = STRING: port2
   [Thu Aug 06 11:47:51 root@centos7~#snmpwalk -v2c -c Fortinet123# 192.168.77.1 1.3.6.1.4.1.12356.101.4.1.8.0
   SNMPv2-SMI::enterprises.12356.101.4.1.8.0 = Gauge32: 77
   ```

2. Net-SNMP SNMPv3结果验证。

   ```bash
   [Thu Aug 06 12:01:01 root@centos7~#net-snmp-config --create-snmpv3-user -ro -a SHA -A forti3389 -x AES -X forti5566 fortinet 
   adding the following line to /var/lib/net-snmp/snmpd.conf:
      createUser fortinet SHA "forti3389" AES forti5566
   adding the following line to /etc/snmp/snmpd.conf:
      rouser fortinet
   [Thu Aug 06 12:01:10 root@centos7~#chkconfig snmpd on
   Note: Forwarding request to 'systemctl enable snmpd.service'.
   Created symlink from /etc/systemd/system/multi-user.target.wants/snmpd.service to /usr/lib/systemd/system/snmpd.service.
   [Thu Aug 06 12:01:30 root@centos7~#service snmpd start
   Starting snmpd (via systemctl):                            [  OK  ]
   [Thu Aug 06 12:06:38 root@centos7~#snmpwalk -v3 -u fortinet -l authPriv -a SHA -A forti3389 -x AES -X forti5566 192.168.77.1
   SNMPv2-MIB::sysDescr.0 = STRING: Fortinet_Beijing_LAB
   SNMPv2-MIB::sysObjectID.0 = OID: SNMPv2-SMI::enterprises.12356.101.1.60
   DISMAN-EVENT-MIB::sysUpTimeInstance = Timeticks: (301754102) 34 days, 22:12:21.02
   SNMPv2-MIB::sysContact.0 = STRING: bbai@fortinet.com
   SNMPv2-MIB::sysName.0 = STRING: FortiGate
   SNMPv2-MIB::sysLocation.0 = STRING: China_Beijing_Lab
   SNMPv2-MIB::sysServices.0 = INTEGER: 78
   SNMPv2-MIB::sysORLastChange.0 = Timeticks: (0) 0:00:00.00
   SNMPv2-MIB::sysORIndex.1 = INTEGER: 1
   SNMPv2-MIB::sysORID.1 = OID: SNMPv2-SMI::zeroDotZero.0
   SNMPv2-MIB::sysORDescr.1 = STRING: 
   SNMPv2-MIB::sysORUpTime.1 = Timeticks: (0) 0:00:00.00
   IF-MIB::ifNumber.0 = INTEGER: 19
   IF-MIB::ifIndex.1 = INTEGER: 1
   IF-MIB::ifIndex.2 = INTEGER: 2
   [Thu Aug 06 12:10:23 root@centos7~#snmpwalk -v3 -u fortinet -l authPriv -a SHA -A forti3389 -x AES -X forti5566 192.168.77.1 .1.3.6.1.4.1.12356.100.1.1.1.0
   SNMPv2-SMI::enterprises.12356.100.1.1.1.0 = STRING: "FGVM08TM22000173"
   [Thu Aug 06 12:10:45 root@centos7~#
   [Thu Aug 06 12:10:46 root@centos7~#snmpwalk -v3 -u fortinet -l authPriv -a SHA -A forti3389 -x AES -X forti5566 192.168.77.1 1.3.6.1.2.1.1.3.0
   DISMAN-EVENT-MIB::sysUpTimeInstance = Timeticks: (301780209) 34 days, 22:16:42.09
   [Thu Aug 06 12:11:00 root@centos7~#
   [Thu Aug 06 12:11:12 root@centos7~#snmpwalk -v3 -u fortinet -l authPriv -a SHA -A forti3389 -x AES -X forti5566 192.168.77.1 .1.3.6.1.4.1.12356.101.4.1.1.0
   SNMPv2-SMI::enterprises.12356.101.4.1.1.0 = STRING: "v7.0.6,build0366,220606 (GA.F)"
   ```

3. iReasoning MIB Browser SNMPv2验证。

   <img src=".\..\..\..\images\image-20220905171716616.png" alt="image-20220905171716616" style="zoom:50%;" />

4. MIB Browser SNMP V3验证。

   <img src=".\..\..\..\images\image-20220906153349790.png" alt="image-20220906153349790" style="zoom:50%;" />

5. iReasoning MIB Browser SNMP Trap验证，SNMPv2和SNMPv3均可以正常接收Trap消息。

   <img src=".\..\..\..\images\image-20220906153808416.png" alt="image-20220906153808416" style="zoom:50%;" />

   <img src=".\..\..\..\images\image-20220906162131795.png" alt="image-20220906162131795" style="zoom:50%;" />

   <img src=".\..\..\..\images\image-20220906162839844.png" alt="image-20220906162839844" style="zoom:50%;" />

## SNMP排错

1. 抓包，确认SNMP UDP 161、162的通信是正常的，正常SNMP是有去有回的。

   ```
   # diagnose sniffer packet any "port 161 and host 192.168.77.1" 4
   ```

2. 通过第一步抓包看到SNMP流量有去有回了，但是SNMP还是失败，则开启SNMP进程的debug，查看具体原因。

   ```
   # diagnose debug application  snmpd -1
   # diagnose debug enable
   ```

   比如，最常见的错误“community”不匹配、管理员配置了可信任主机，但是没有包括SNMP的IP地址等等...

   ```
   HUB2-ShangHai # snmpd: updating cache: idx_cache
   snmpd: <msg> 47 bytes 192.168.90.254:44539 -> 200.1.1.1/200.1.1.1:161 (itf 3.3)
   snmpd: checking if community "Fortinet123#2" is valid
   snmpd: updating cache: vdom_idx_map_cache
   snmpd: updating vdom idx mapping
   snmpd: Creating vdom_idx_cache for root
   snmpd: Vdom created kernel-index=0, snmp-index=1, name=root
   snmpd: checking against community "Fortinet123#"
   snmpd: vdom name mismatch
   snmpd: checking against community "FortiManager"
   snmpd: name mismatch.
   snmpd: failed to match community "Fortinet123#2"
   snmpd: </msg> 0
   ```
