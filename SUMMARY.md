# Summary

* [前言](README.md)
* [产品简介](产品简介.md)

* 系统管理
    * 设备管理
        * [Web管理](系统管理/设备管理/Web管理.md)
        * [Console管理](系统管理/设备管理/Console管理.md)
        * [Telnet/SSH管理](系统管理/设备管理/Telnet_SSH管理.md)
        * [密码恢复](系统管理/设备管理/密码恢复.md)
        * [恢复出厂](系统管理/设备管理/恢复出厂.md)
        * 固件版本升级
            * [Web界面升级](系统管理/设备管理/固件版本升级/Web界面升级.md)
            * [TFTP方式升级](系统管理/设备管理/固件版本升级/TFTP方式升级.md)
        * [配置备份及恢复](系统管理/设备管理/配置备份及恢复.md)
        * [管理员设置](系统管理/设备管理/管理员设置.md)
        * [常用基础命令](系统管理/设备管理/常用基础命令.md)
        * SNMP
            * [SNMP介绍](系统管理/设备管理/SNMP/SNMP介绍.md)
            * [SNMP公共MIB和私有MIB](系统管理/设备管理/SNMP/SNMP公共MIB和私有MIB.md)
            * [FortiGate的SNMP建议监控指标](系统管理/设备管理/SNMP/FortiGate的SNMP建议监控指标.md)
            * [SNMP的常用工具](系统管理/设备管理/SNMP/SNMP的常用工具.md)
            * [SNMP的设置](系统管理/设备管理/SNMP/SNMP的设置.md)
            * [SNMP有HA独立管理的设置](系统管理/设备管理/SNMP/SNMP有HA独立管理的设置.md)
    * FortiGuard管理
        * [FortiGuard服务介绍](系统管理/FortiGuard管理/FortiGuard服务介绍.md)
        * [设备注册步骤](系统管理/FortiGuard管理/设备注册步骤.md)
        * [服务注册步骤](系统管理/FortiGuard管理/服务注册步骤.md)
        * [产品注册信息变更](系统管理/FortiGuard管理/产品注册信息变更.md)
        * [在线自动升级](系统管理/FortiGuard管理/在线自动升级.md)
        * [离线手工升级](系统管理/FortiGuard管理/离线手工升级.md)
    * FortiManager管理
        * [FortiManger功能简介](系统管理/FortiManager管理/FortiManger功能简介.md)
        * [FortiManger产品介绍](系统管理/FortiManager管理/FortiManger产品介绍.md)
        * [FortiManger网管登陆](系统管理/FortiManager管理/FortiManger网管登陆.md)
        * FortiManger ADOM
            * [ADOM简介](系统管理/FortiManager管理/FortiManger ADOM/ADOM简介.md)
            * [开启ADOM](系统管理/FortiManager管理/FortiManger ADOM/开启ADOM.md)
            * [ADOM的高级特性](系统管理/FortiManager管理/FortiManger ADOM/ADOM的高级特性.md)
        * FortiManger管理员
            * [管理员账号管理](系统管理/FortiManager管理/FortiManger管理员/管理员账号管理.md)
            * [配置远程管理员](系统管理/FortiManager管理/FortiManger管理员/配置远程管理员.md)
            * [管理员和ADOM关联](系统管理/FortiManager管理/FortiManger管理员/管理员和ADOM关联.md)
            * [ADOM的Workspace模式](系统管理/FortiManager管理/FortiManger管理员/ADOM的Workspace模式.md)
* 网络管理
    * 路由模式
        * [静态地址上网配置](网络管理/路由模式/静态地址上网方式.md)
    * 透明模式
        * 传统透明模式
            * [开启透明模式防火墙并保护上网流量](网络管理/透明模式/传统透明模式/开启透明模式防火墙并保护上网流量.md)
            * VLAN与透明模式
                * [Forward-Domain(有VLAN的场景建议配上)](网络管理/透明模式/传统透明模式/VLAN与透明模式/Forward-Domain.md)
                * [vlanforward(默认关闭，谨慎使用)](网络管理/透明模式/传统透明模式/VLAN与透明模式/vlanforward.md)
                * [Remapping VLAN ID(VLAN NAT，特殊场景使用)](网络管理/透明模式/传统透明模式/VLAN与透明模式/Remapping_VLAN_ID.md)
            * [组播(OSPF)与透明模式](网络管理/透明模式/传统透明模式/组播OSPF与透明模式.md)
            * [非IP流量(PPPOE)与透明模式](网络管理/透明模式/传统透明模式/非IP流量PPPoE与透明模式.md)
            * [生成树(STP)与透明模式](网络管理/透明模式/传统透明模式/生成树STP与透明模式.md)
            * [透明模式的带外管理](网络管理/透明模式/传统透明模式/透明模式的带外管理.md)
            * [Bypass部署](网络管理/透明模式/传统透明模式/Bypass部署.md)
            * [透明模式注意事项与排错](网络管理/透明模式/传统透明模式/透明模式注意事项与排错.md)
        * 虚拟接口对（VWP）[推荐]
            * [虚拟接口对介绍](网络管理/透明模式/虚拟接口对/虚拟接口对介绍.md)
            * [开启虚拟接口对并保护上网流量](网络管理/透明模式/虚拟接口对/开启虚拟接口对并保护上网流量.md)
            * [虚拟接口对的网络管理问题](网络管理/透明模式/虚拟接口对/虚拟接口对的网络管理问题.md)
            * [VLAN与虚拟接口对](网络管理/透明模式/虚拟接口对/VLAN与虚拟接口对.md)
            * [组播(OSPF)与虚拟接口对](网络管理/透明模式/虚拟接口对/组播OSPF与虚拟接口对.md)
            * [非IP流量(PPPOE)与虚拟接口对](网络管理/透明模式/虚拟接口对/非IP流量PPPOE与虚拟接口对.md)
            * [生成树(STP)与虚拟接口对](网络管理/透明模式/虚拟接口对/生成树STP与虚拟接口对.md)
            * [虚拟接口对最佳配置实践](网络管理/透明模式/虚拟接口对/虚拟接口对最佳配置实践.md)
    * 旁路模式
        * [旁路模式典型案例](网络管理/旁路模式/旁路模式典型案例.md)
    * DHCP配置
        * [DHCP服务器配置](网络管理/DHCP配置/DHCP服务器配置.md)
        * [DHCP静态绑定](网络管理/DHCP配置/DHCP静态绑定.md)
        * [DHCP中继配置](网络管理/DHCP配置/DHCP中继配置.md)
    * [端口聚合配置](网络管理/端口聚合配置.md)
    * [软交换配置](网络管理/软交换配置.md)
    * SD-WAN
        * SD-WAN介绍
            * [SD-WAN数据转发逻辑](网络管理/SD-WAN/SD-WAN介绍/SD-WAN数据转发逻辑.md)
            * [SD-WAN接口成员组](网络管理/SD-WAN/SD-WAN介绍/SD-WAN接口成员组.md)
            * [SD-WAN健康状态检查（Performance SLA）](网络管理/SD-WAN/SD-WAN介绍/SD-WAN健康状态检查.md)
        * SD-WAN规则
            * [Manual（手动选路）](网络管理/SD-WAN/SD-WAN规则/Manual.md)
            * [Best Quality（最佳质量）](网络管理/SD-WAN/SD-WAN规则/Best_Quality.md)
            * [Lowest Cost（SLA)](网络管理/SD-WAN/SD-WAN规则/Lowest_Cost.md)
            * Maximize Bandwidth (SLA)
                * [基于轮询的Maximize Bandwidth (SLA)](网络管理/SD-WAN/SD-WAN规则/Maximize_Bandwidth/基于轮询的Maximize_Bandwidth.md)
                * [基于带宽的Maximize Bandwidth (SLA)](网络管理/SD-WAN/SD-WAN规则/Maximize_Bandwidth/基于带宽的Maximize_Bandwidth.md)
* VDOM
    * [VDOM基本配置](VDOM/VDOM基本配置.md)
    * [VDOM使用](VDOM/VDOM使用.md)
    * [VDOM间通信](VDOM/VDOM间通信.md)
* HA双机热备
    * [HA工作模式和配置要求](HA双机热备/HA工作模式和配置要求.md)
    * [HA选取主设备的方法](HA双机热备/HA选取主设备的方法.md)
    * [HA典型基础配置](HA双机热备/HA典型基础配置.md)
    * [HA单配置同步、会话同步配置（FGSP）](HA双机热备/HA单配置同步_会话同步配置.md)
    * [HA双机配置同步说明](HA双机热备/HA双机配置同步说明.md)
    * [HA-Cluster不间断升级](HA双机热备/HA-Cluster不间断升级.md)
    * [HA的ping server配置](HA双机热备/HA的ping_server配置.md)
    * HA-Cluster的网管
        * [HA的网管介绍](HA双机热备/HA-Cluster的网管/HA的网管介绍.md)
        * [HA集群普通管理](HA双机热备/HA-Cluster的网管/HA集群普通管理.md)
        * [HA集群带外独立管理-独立管理接口](HA双机热备/HA-Cluster的网管/HA集群带外独立管理-独立管理接口.md)
        * [HA集群带外独立管理-独立管理VDOM](HA双机热备/HA-Cluster的网管/HA集群带外独立管理-独立管理VDOM.md)
        * [HA集群带外独立管理-单独指定用于更新的VDOM](HA双机热备/HA-Cluster的网管/HA集群带外独立管理-单独指定用于更新的VDOM.md)
    * [HA配置命令集](HA双机热备/HA配置命令集.md)
* 路由
    * [路由介绍](路由/路由介绍.md)
    * [静态路由](路由/静态路由.md)
    * [策略路由](路由/策略路由.md)
    * 动态路由
        * [RIP](路由/动态路由/RIP.md)
        * [OSPF](路由/动态路由/OSPF.md)
        * [动态路由协议排错](路由/动态路由/动态路由协议排错.md)
* IPV6
    * [页面开启IPv6](IPv6/页面开启IPv6.md)
    * [上网配置](IPv6/上网配置.md)
    * [NAT64](IPv6/NAT64.md)
    * [OSPFv3](IPv6/OSPFv3.md)
    * [常用命令](IPv6/常用命令.md)
* 策略与对象
    * 单线路上网配置
        * [单条ADSL线路上网配置](策略与对象/单线路上网配置/单条ADSL线路上网配置.md)
        * [静态地址线路上网配置](策略与对象/单线路上网配置/静态地址线路上网配置.md)
        * [DHCP线路上网配置](策略与对象/单线路上网配置/DHCP线路上网配置.md)
    * 多线路上网配置
        * [双线路同运营商上网配置](策略与对象/多线路上网配置/双线路同运营商上网配置.md)
        * [双线路不同运营商上网配置](策略与对象/多线路上网配置/双线路不同运营商上网配置.md)
        * [双线路ADSL](策略与对象/多线路上网配置/双线路ADSL.md)
    * 虚拟IP映射
        * [端口映射应原理](策略与对象/虚拟IP映射/端口映射应原理.md)
        * [地址映射（一对一IP映射）](策略与对象/虚拟IP映射/地址映射.md)
        * [端口映射（一对多端口映射）](策略与对象/虚拟IP映射/端口映射.md)
        * [多线路端口映射](策略与对象/虚拟IP映射/多线路端口映射.md)
        * [内网主机通过公网IP访问VIP](策略与对象/虚拟IP映射/内网主机通过公网IP访问VIP.md)
        * [VIP46映射](策略与对象/虚拟IP映射/VIP46映射.md)
        * [VIP64映射](策略与对象/虚拟IP映射/VIP64映射.md)
    * [配置session ttl](策略与对象/配置session_ttl.md)
    * DoS策略
        * [什么是DoS](策略与对象/DoS策略/什么是DoS.md)
        * [FortiGate DoS策略工作原理](策略与对象/DoS策略/FortiGate_DoS策略工作原理.md)
        * FortiGate DoS策略使用
            * [FortiGate DoS策略配置](策略与对象/DoS策略/FortiGate_DoS策略使用/FortiGate_DoS策略配置.md)
            * [FortiGate DoS策略调整](策略与对象/DoS策略/FortiGate_DoS策略使用/FortiGate_DoS策略调整.md)
        * [FortiGate DoS策略排错](策略与对象/DoS策略/FortiGate_DoS策略排错.md)
    * QoS 限速
        * [QoS介绍](策略与对象/QoS限速/QoS介绍.md)
        * 传统配置方式配置限速测试
            * [每IP带宽限速](策略与对象/QoS限速/传统配置方式配置限速测试/每IP带宽限速.md)
            * [共享带宽限速](策略与对象/QoS限速/传统配置方式配置限速测试/共享带宽限速.md)
* UTM安全应用功能
    * 入侵防御
        * [入侵防御简介](UTM安全应用功能/入侵防御/入侵防御简介.md)
        * [入侵防御配置](UTM安全应用功能/入侵防御/入侵防御配置.md)
    * 病毒防护
        * [反病毒简介](UTM安全应用功能/病毒防护/反病毒简介.md)
        * [反病毒配置](UTM安全应用功能/病毒防护/反病毒配置.md)
    * Web过滤功能
        * [Web过滤简介](UTM安全应用功能/Web过滤功能/Web过滤简介.md)
        * [Web过滤配置](UTM安全应用功能/Web过滤功能/Web过滤配置.md)
        * [URL过滤](UTM安全应用功能/Web过滤功能/URL过滤.md)
    * 应用控制
        * [应用控制简介](UTM安全应用功能/应用控制/应用控制简介.md)
        * [应用控制配置](UTM安全应用功能/应用控制/应用控制配置.md)
* VPN技术
    * IPSec VPN
        * [IPSec VPN原理](VPN技术/IPSec_VPN/IPSec_VPN原理.md)
        * 点到点VPN
            * [IPSec的两种配置模式](VPN技术/IPSec_VPN/点到点VPN/IPSec的两种配置模式.md)
            * [接口摸式](VPN技术/IPSec_VPN/点到点VPN/接口摸式.md)
            * [PKI证书认证](VPN技术/IPSec_VPN/点到点VPN/PKI证书认证.md)
            * [VPN隧道上运行OSPF或BGP](VPN技术/IPSec_VPN/点到点VPN/VPN隧道上运行OSPF或BGP.md)
            * [多路负载/主备IPSec VPN](VPN技术/IPSec_VPN/点到点VPN/多路负载_主备IPSec_VPN.md)
            * [FortiGuard DDNS方式建立IPSec VPN](VPN技术/IPSec_VPN/点到点VPN/FortiGuard_DDNS方式建立IPSec_VPN.md)
            * [GRE over IPSec](VPN技术/IPSec_VPN/点到点VPN/GRE_over_IPSec.md)
            * [IPSec部署性能优化脚本](VPN技术/IPSec_VPN/点到点VPN/IPSec部署性能优化脚本.md)
            * 和友商进行IPSec VPN对接
                * [和思科路由器建立IPSec VPN(IKE v1)](VPN技术/IPSec_VPN/点到点VPN/和友商进行IPSec_VPN对接/和思科路由器建立IPSec_VPN_IKE_v1.md)
                * [和思科路由器建立IPSec VPN(IKE v2)](VPN技术/IPSec_VPN/点到点VPN/和友商进行IPSec_VPN对接/和思科路由器建立IPSec_VPN_IKE_v2.md)
                * [和思科ASA防火墙建立IPSec VPN(IKE v1)](VPN技术/IPSec_VPN/点到点VPN/和友商进行IPSec_VPN对接/和思科ASA防火墙建立IPSec_VPN_IKE_v1.md)
                * [和思科ASA防火墙建立IPSec VPN(IKE v2)](VPN技术/IPSec_VPN/点到点VPN/和友商进行IPSec_VPN对接/和思科ASA防火墙建立IPSec_VPN_IKE_v2.md)
        * 拨号VPN
            * [HUB-and-SPOKE模式](VPN技术/IPSec_VPN/拨号VPN/HUB-and-SPOKE模式.md)
            * [拨号VPN上运行动态路由协议](VPN技术/IPSec_VPN/拨号VPN/拨号VPN上运行动态路由协议.md)
            * [ADVPN模式](VPN技术/IPSec_VPN/拨号VPN/ADVPN模式.md)
            * [野蛮模式在动态拨号VPN中的应用](VPN技术/IPSec_VPN/拨号VPN/野蛮模式在动态拨号VPN中的应用.md)
            * [FortiClient客户端-VPN拨号模式](VPN技术/IPSec_VPN/拨号VPN/FortiClient客户端-VPN拨号模式.md)
            * [PKI证书FortiClient客户端-VPN拨号](VPN技术/IPSec_VPN/拨号VPN/PKI证书FortiClient客户端-VPN拨号.md)
        * 透明模式下IPSec VPN
            * [IPSec VPN Lan-to-Lan](VPN技术/IPSec_VPN/透明模式下IPSec_VPN/IPSec_VPN_Lan-to-Lan.md)
        * IPSec VPN排错
            * [IPSEC Debug命令](VPN技术/IPSec_VPN/IPSec_VPN排错/IPSEC_Debug命令.md)
            * [IPSEC IKEv1主模式Debug分析](VPN技术/IPSec_VPN/IPSec_VPN排错/IPSEC_IKEv1主模式Debug分析.md)
            * [IPSEC IKEv1野蛮模式Debug分析](VPN技术/IPSec_VPN/IPSec_VPN排错/IPSEC_IKEv1野蛮模式Debug分析.md)
            * [IPSEC IKEv2 Debug分析](VPN技术/IPSec_VPN/IPSec_VPN排错/IPSEC_IKEv2_Debug分析.md)
            * [IPSEC协商失败的Debug举例](VPN技术/IPSec_VPN/IPSec_VPN排错/IPSEC协商失败的Debug举例.md)
            * [IPSec业务不通问题](VPN技术/IPSec_VPN/IPSec_VPN排错/IPSec业务不通问题.md)
    * SSL VPN
        * [SSL VPN的两种模式](VPN技术/SSL_VPN/SSL_VPN的两种模式.md)
        * [SSL VPN WEB代理模式配置](VPN技术/SSL_VPN/SSL_VPN_WEB代理模式配置.md)
        * [SSL VPN FortiClient隧道模式](VPN技术/SSL_VPN/SSL_VPN_FortiClient隧道模式.md)
        * [SSL VPN数子证书认证](VPN技术/SSL_VPN/SSL_VPN数子证书认证.md)
        * SSL VPN双因子认证
            * [FortiToken Mobile在FortiGate上的激活](VPN技术/SSL_VPN/SSL_VPN双因子认证/FortiToken_Mobile在FortiGate上的激活.md)
            * [FortiToken Mobile APP软件下载与安装](VPN技术/SSL_VPN/SSL_VPN双因子认证/FortiToken_Mobile_APP软件下载与安装.md)
            * [FortiToken Mobile与用户进行绑定](VPN技术/SSL_VPN/SSL_VPN双因子认证/FortiToken_Mobile与用户进行绑定.md)
            * [FortiToken Moblie与SSL VPN结合成双因子认证](VPN技术/SSL_VPN/SSL_VPN双因子认证/FortiToken_Moblie与SSL_VPN结合成双因子认证.md)
    * FortiClient VPN
        * [FortiClient下载](VPN技术/FortiClient_VPN/FortiClient下载.md)
        * [Windows FortiClient安装及使用](VPN技术/FortiClient_VPN/Windows_FortiClient安装及使用.md)
        * [Mac FortiClient安装及使用](VPN技术/FortiClient_VPN/Mac_FortiClient安装及使用.md)
        * [FortiClient运维](VPN技术/FortiClient_VPN/FortiClient运维.md)
    * 其他VPN技术
        * [GRE](VPN技术/其他VPN技术/GRE.md)
        * [L2TP VPN](VPN技术/其他VPN技术/L2TP_VPN.md)
        * [L2TP over IPSec](VPN技术/其他VPN技术/L2TP_over_IPSec.md)
        * [PPTP VPN](VPN技术/其他VPN技术/PPTP_VPN.md)
* 日志
    * [日志存储方式](日志/日志存储方式.md)
    * [日志类型](日志/日志类型.md)
    * [删除日志](日志/删除日志.md)
* 常见问题
    * [产品知识](常见问题/产品知识.md)
    * [防火墙部署](常见问题/防火墙部署.md)
    * [功能配置](常见问题/功能配置.md)
    * [运维监控](常见问题/运维监控.md)
* 故障排查
    * Debug Flow工具
        * [Debug Flow工具](故障排查/Debug_Flow工具/Debug_Flow介绍.md)
        * [Debug Flow排错举例](故障排查/Debug_Flow工具/Debug_Flow排错举例.md)
    * Session List工具
        * [查看和删除会话信息](故障排查/Session_List工具/查看和删除会话信息.md)
        * [会话状态信息查询](故障排查/Session_List工具/会话状态信息查询.md)
    * Sniffer工具
        * [sniffer工具介绍](故障排查/Sniffer工具/sniffer工具介绍.md)
        * [Sniffer过滤条件的合理使用](故障排查/Sniffer工具/Sniffer过滤条件的合理使用.md)
        * [使用wireshark打开抓包文件](故障排查/Sniffer工具/使用wireshark打开抓包文件.md)
    * [Sniffer、Debug Flow与NP加速的矛盾](故障排查/Sniffer_Debug Flow与NP加速的矛盾.md)
    * [数据包处理流程](故障排查/数据包处理流程.md)
* 开局与日常维护
    * 实施前准备
        * [收集客户信息](开局与日常维护/实施前准备/收集客户信息.md)
        * [设备软件检查](开局与日常维护/实施前准备/设备软件检查.md)
        * [设备硬件检查](开局与日常维护/实施前准备/设备硬件检查.md)
    * 实施后设备运行检查
        * [基础检查](开局与日常维护/实施后设备运行检查/基础检查.md)
        * [运行情况](开局与日常维护/实施后设备运行检查/运行情况.md)
        * [日志查看](开局与日常维护/实施后设备运行检查/日志查看.md)
* 附件
    * [FortiGate通用多运营商路由导入脚本](附件/FortiGate通用多运营商路由导入脚本.md)
    * [命令行脚本批量转换工具说明](附件/命令行脚本批量转换工具说明.md)

