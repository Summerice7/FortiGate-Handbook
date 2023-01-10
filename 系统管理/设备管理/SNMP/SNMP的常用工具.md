# SNMP的常用工具

## Net-SNMP

网址链接：http://www.net-snmp.org/

Download：http://www.net-snmp.org/download.html

1. CentOS 7 安装Net-SNMP和SNMP V2：

   ```
   [Wed Aug 05 10:07:33 root@centos7~#yum install -y net-snmp*
   Loaded plugins: fastestmirror, langpacks
   Determining fastest mirrors
    * base: mirror2.totbb.net
    * extras: mirror.hostlink.com.hk
    * updates: mirror01.idc.hinet.net
   base                                                                                                             | 3.6 kB  00:00:00     
   extras                                                                                                           | 2.9 kB  00:00:00     
   updates                                                                                                          | 2.9 kB  00:00:00     
   (1/2): extras/7/x86_64/primary_db                                                                                | 205 kB  00:00:00     
   (2/2): updates/7/x86_64/primary_db                                                                               |3.7 MB   00:00:11     
   ......
   
   Installed:
     net-snmp.x86_64 1:5.7.2-48.el7_8.1           net-snmp-agent-libs.x86_64 1:5.7.2-48.el7_8.1  net-snmp-devel.x86_64 1:5.7.2-48.el7_8.1  
     net-snmp-gui.x86_64 1:5.7.2-48.el7_8.1       net-snmp-perl.x86_64 1:5.7.2-48.el7_8.1        net-snmp-python.x86_64 1:5.7.2-48.el7_8.1 
     net-snmp-sysvinit.x86_64 1:5.7.2-48.el7_8.1  net-snmp-utils.x86_64 1:5.7.2-48.el7_8.1      
   
   Dependency Installed:
     elfutils-devel.x86_64 0:0.176-4.el7                          elfutils-libelf-devel.x86_64 0:0.176-4.el7                              
     libdb-devel.x86_64 0:5.3.21-25.el7                           lm_sensors-devel.x86_64 0:3.4.0-8.20160601gitf9185e5.el7                
     perl-ExtUtils-Install.noarch 0:1.58-295.el7                  perl-ExtUtils-MakeMaker.noarch 0:6.68-3.el7                             
     perl-ExtUtils-Manifest.noarch 0:1.61-244.el7                 perl-ExtUtils-ParseXS.noarch 1:3.18-3.el7                               
     perl-Tk.x86_64 0:804.030-6.el7                               perl-devel.x86_64 4:5.16.3-295.el7                                      
     popt-devel.x86_64 0:1.13-16.el7                              rpm-devel.x86_64 0:4.11.3-43.el7                                        
     systemtap-sdt-devel.x86_64 0:4.0-11.el7                      tcp_wrappers-devel.x86_64 0:7.6-77.el7                                  
   
   Updated:
     net-snmp-libs.x86_64 1:5.7.2-48.el7_8.1                                                                                               
   
   Dependency Updated:
     elfutils.x86_64 0:0.176-4.el7            elfutils-libelf.x86_64 0:0.176-4.el7            elfutils-libs.x86_64 0:0.176-4.el7           
   
   Complete!
   [Thu Aug 06 09:56:37 root@centos7~#
   [Thu Aug 06 10:09:13 root@centos7~# snmpd -v
   
   NET-SNMP version:  5.7.2
   Web:               http://www.net-snmp.org/
   Email:             net-snmp-coders@lists.sourceforge.net
   
   [Thu Aug 06 10:09:18 root@centos7~#
   [Thu Aug 06 10:09:46 root@centos7~#systemctl start snmpd.service
   [Thu Aug 06 10:09:49 root@centos7~#
   [Thu Aug 06 10:10:17 root@centos7~#snmpwalk -v 1 -c public 127.0.0.1 .1.3.6.1.2.1.1.1.0
   SNMPv2-MIB::sysDescr.0 = STRING: Linux centos7.lab.fortinet.com 3.10.0-1062.el7.x86_64 #1 SMP Wed Aug 7 18:08:02 UTC 2019 x86_64
   [Thu Aug 06 10:10:19 root@centos7~#
   [Thu Aug 06 10:24:55 root@centos7~#snmpwalk -v2c -c Fortinet123# 192.168.90.3 .1.3.6.1.2.1.1.5.0
   SNMPv2-MIB::sysName.0 = STRING: HUB2-ShangHai
   [Thu Aug 06 10:24:59 root@centos7~#snmpwalk -v2c -c Fortinet123# 192.168.90.3 .1.3.6.1.4.1.12356.100.1.1.1.0
   SNMPv2-SMI::enterprises.12356.100.1.1.1.0 = STRING: "FGVM04TM20003482"
   [Thu Aug 06 10:25:01 root@centos7~#snmpwalk -v2c -c Fortinet123# 192.168.90.3 .1.3.6.1.4.1.12356.101.4.1.1.0
   SNMPv2-SMI::enterprises.12356.101.4.1.1.0 = STRING: "v6.4.1,build1637,200604 (GA)"
   [Thu Aug 06 10:25:21 root@centos7~#snmpwalk -v2c -c Fortinet123# 192.168.90.3 1.3.6.1.2.1.1.3.0
   DISMAN-EVENT-MIB::sysUpTimeInstance = Timeticks: (301149616) 34 days, 20:31:36.16
   [Thu Aug 06 10:25:55 root@centos7~#
   ```

2. Net-SNMP SNMP V3

   ```
   [Thu Aug 06 11:56:06 root@centos7~#service snmpd  stop
   Stopping snmpd (via systemctl):                            [  OK  ]
   [Thu Aug 06 11:56:20 root@centos7~#
   [Thu Aug 06 12:00:52 root@centos7~#
   [Thu Aug 06 12:01:01 root@centos7~#net-snmp-config --create-snmpv3-user -ro -a SHA -A forti3389 -x AES -X forti5566 fortinet 
   adding the following line to /var/lib/net-snmp/snmpd.conf:
      createUser fortinet SHA "forti3389" AES forti5566
   adding the following line to /etc/snmp/snmpd.conf:
      rouser fortinet
   [Thu Aug 06 12:01:09 root@centos7~#
   [Thu Aug 06 12:01:10 root@centos7~#chkconfig snmpd on
   Note: Forwarding request to 'systemctl enable snmpd.service'.
   Created symlink from /etc/systemd/system/multi-user.target.wants/snmpd.service to /usr/lib/systemd/system/snmpd.service.
   [Thu Aug 06 12:01:30 root@centos7~#service snmpd start
   Starting snmpd (via systemctl):                            [  OK  ]
   [Thu Aug 06 12:01:36 root@centos7~#
   [Thu Aug 06 12:06:29 root@centos7~#
   [Thu Aug 06 12:06:38 root@centos7~#snmpwalk -v3 -u fortinet -l authPriv -a SHA -A forti3389 -x AES -X forti5566 192.168.90.3
   SNMPv2-MIB::sysDescr.0 = STRING: Fortinet_BeiJing_LAB_FGT-KVM
   SNMPv2-MIB::sysObjectID.0 = OID: SNMPv2-SMI::enterprises.12356.101.1.60
   DISMAN-EVENT-MIB::sysUpTimeInstance = Timeticks: (301754102) 34 days, 22:12:21.02
   SNMPv2-MIB::sysContact.0 = STRING: kmliu@fortinet.com
   SNMPv2-MIB::sysName.0 = STRING: HUB2-ShangHai
   SNMPv2-MIB::sysLocation.0 = STRING: China_BeiJing_Lab
   SNMPv2-MIB::sysServices.0 = INTEGER: 78
   SNMPv2-MIB::sysORLastChange.0 = Timeticks: (0) 0:00:00.00
   SNMPv2-MIB::sysORIndex.1 = INTEGER: 1
   SNMPv2-MIB::sysORID.1 = OID: SNMPv2-SMI::zeroDotZero.0
   SNMPv2-MIB::sysORDescr.1 = STRING: 
   SNMPv2-MIB::sysORUpTime.1 = Timeticks: (0) 0:00:00.00
   IF-MIB::ifNumber.0 = INTEGER: 19
   IF-MIB::ifIndex.1 = INTEGER: 1
   IF-MIB::ifIndex.2 = INTEGER: 2
   ^C
   [Thu Aug 06 12:06:41 root@centos7~#
   [Thu Aug 06 12:06:45 root@centos7~#
   [Thu Aug 06 12:10:23 root@centos7~#snmpwalk -v3 -u fortinet -l authPriv -a SHA -A forti3389 -x AES -X forti5566 192.168.90.3 .1.3.6.1.4.1.12356.100.1.1.1.0
   SNMPv2-SMI::enterprises.12356.100.1.1.1.0 = STRING: "FGVM04TM20003482"
   [Thu Aug 06 12:10:45 root@centos7~#
   [Thu Aug 06 12:10:46 root@centos7~#snmpwalk -v3 -u fortinet -l authPriv -a SHA -A forti3389 -x AES -X forti5566 192.168.90.3 1.3.6.1.2.1.1.3.0
   DISMAN-EVENT-MIB::sysUpTimeInstance = Timeticks: (301780209) 34 days, 22:16:42.09
   [Thu Aug 06 12:11:00 root@centos7~#
   [Thu Aug 06 12:11:12 root@centos7~#snmpwalk -v3 -u fortinet -l authPriv -a SHA -A forti3389 -x AES -X forti5566 192.168.90.3 .1.3.6.1.4.1.12356.101.4.1.1.0
   SNMPv2-SMI::enterprises.12356.101.4.1.1.0 = STRING: "v6.4.1,build1637,200604 (GA)"
   [Thu Aug 06 12:11:19 root@centos7~#
   ```

   Net-SNMP v3参考链接：https://www.thegeekdiary.com/centos-rhel-6-install-and-configure-snmpv3/

2. iReasoning MIB browser（提供免费版本）

   网址链接：https://www.ireasoning.com/

   Download：https://www.ireasoning.com/downloadmibbrowserfree.php

   <img src=".\..\..\..\images\image-20220905151100597.png" alt="image-20220905151100597" style="zoom:50%;" />

4. PRTG（提供100个传感器的免费版本）

   网址链接：https://www.cn.paessler.com/
   Download：https://www.cn.paessler.com/download/prtg-download/

   ![image-20220905151228629](.\..\..\..\images\image-20220905151228629.png)

   ![image-20220905151350829](.\..\..\..\images\image-20220905151350829.png)

   ![image-20220905151433246](.\..\..\..\images\image-20220905151433246.png)

   ![image-20220905151458820](.\..\..\..\images\image-20220905151458820.png)

   ![image-20220905151536800](.\..\..\..\images\image-20220905151536800.png)

5. Zabbix（开源免费）

   网址链接：https://www.zabbix.com/cn

   ![image-20220905151638853](.\..\..\..\images\image-20220905151638853.png)

6. Cacti（开源免费）

   网址链接：https://www.cacti.net/  

   download网址：https://www.cacti.net/download_cacti.php

   ![image-20220905151826948](.\..\..\..\images\image-20220905151826948.png)
