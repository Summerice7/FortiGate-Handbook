# SNMP公共MIB和私有MIB

## 公共MIB库

公共MIB可以认为是大家共用使用的一些MIB查询节点，比如设备名称、设备运行时间、接口MAC、接口MTU、接口描述、接口收发报文数（接口流量）、路由表等等信息。

### 举例1：设备名称

```
Name:        sysName
Type:        OBJECT-TYPE
OID:         1.3.6.1.2.1.1.5
```

<img src=".\..\..\..\images\image-20220822165133029.png" alt="image-20220822165133029" style="zoom:50%;" />

### 举例2：设备启动时间

```
Name:        sysUpTime
Type:        OBJECT-TYPE
OID:         1.3.6.1.2.1.1.3
```

<img src=".\..\..\..\images\image-20220822165305394.png" alt="image-20220822165305394" style="zoom:50%;" />

### 举例3：设备接口名称、流量统计、接口描述、接口Alias、等OID

```
Name:        ifXTable
Type:        OBJECT-TYPE
OID:         1.3.6.1.2.1.31.1.1
Full path:        iso(1).org(3).dod(6).internet(1).mgmt(2).mib-2(1).ifMIB(31).ifMIBObjects(1).ifXTable(1)
Module:        IF-MIB

Parent:        ifMIBObjects
First child:        ifXEntry
Next sibling:        ifStackTable

Numerical syntax:        Sequence
Base syntax:        SEQUENCE OF IfXEntry
Composed syntax:        SEQUENCE OF IfXEntry
Status:        current
Max access:        not-accessible
Sequences:        
       1: ifName - DisplayString(4 - octets)
       2: ifInMulticastPkts - Counter32(65 - counter (32 bit))
       3: ifInBroadcastPkts - Counter32(65 - counter (32 bit))
       4: ifOutMulticastPkts - Counter32(65 - counter (32 bit))
       5: ifOutBroadcastPkts - Counter32(65 - counter (32 bit))
       6: ifHCInOctets - Counter64(70 - counter (64 bit))
       7: ifHCInUcastPkts - Counter64(70 - counter (64 bit))
       8: ifHCInMulticastPkts - Counter64(70 - counter (64 bit))
       9: ifHCInBroadcastPkts - Counter64(70 - counter (64 bit))
       10: ifHCOutOctets - Counter64(70 - counter (64 bit))
       11: ifHCOutUcastPkts - Counter64(70 - counter (64 bit))
       12: ifHCOutMulticastPkts - Counter64(70 - counter (64 bit))
       13: ifHCOutBroadcastPkts - Counter64(70 - counter (64 bit))
       14: ifLinkUpDownTrapEnable - INTEGER(2 - integer (32 bit))
       15: ifHighSpeed - Gauge32(66 - gauge (32 bit))
       16: ifPromiscuousMode - TruthValue(2 - integer (32 bit))
       17: ifConnectorPresent - TruthValue(2 - integer (32 bit))
       18: ifAlias - DisplayString(4 - octets)
       19: ifCounterDiscontinuityTime - TimeStamp(67 - timeticks)
```

<img src=".\..\..\..\images\image-20220822170233764.png" alt="image-20220822170233764" style="zoom:50%;" />

### 举例4：设备路由表信息

```
Name:        ipRouteNextHop
Type:        OBJECT-TYPE
OID:         1.3.6.1.2.1.4.21.1.7
```

<img src=".\..\..\..\images\image-20220822170744825.png" alt="image-20220822170744825" style="zoom:50%;" />

公共的mib 库非常之多，其中有很多有用的OID：

<img src=".\..\..\..\images\image-20220822170940168.png" alt="image-20220822170940168" style="zoom:50%;" />

> 具体可以通过以下链接查看到公共MIB：
>
> Net-SNMP Distributed MIBs：http://net-snmp.sourceforge.net/docs/mibs/
>
> UCD-SNMP-MIB：http://net-snmp.sourceforge.net/docs/mibs/ucdavis.html
>
> HOST-RESOURCES-MIB：http://net-snmp.sourceforge.net/docs/mibs/host.html
>
> IF-MIB：http://net-snmp.sourceforge.net/docs/mibs/ifMIBObjects.html
>
>
> mibDepot：http://www.mibdepot.com/index.shtml
>

### 经常监视的数据（公共MIB）

| 项目   | 访问方式 | OID  | MIB  |
| ------ | -------- | ---- | ---- |
| **处理器** |
| 用户占用时间比 | GET | .1.3.6.1.4.1.2021.11.9.0 | UCD-SNMP-MIB::ssCpuUser |
| 系统占用时间比 | GET | .1.3.6.1.4.1.2021.11.10.0 | UCD-SNMP-MIB::ssCpuSystem |
| 闲置时间比 | GET | .1.3.6.1.4.1.2021.11.11.0 | UCD-SNMP-MIB::ssCpuIdle |
| 每个 Core 的用量 | WALK | .1.3.6.1.2.1.25.3.3.1.2 | HOST-RESOURCES-MIB::hrProcessorLoad |
| **存储器**  |
| 存储器容量 | GET | .1.3.6.1.4.1.2021.4.5.0 | UCD-SNMP-MIB::memTotalReal |
| 存储器消耗量 | GET | .1.3.6.1.4.1.2021.4.6.0 | UCD-SNMP-MIB::memAvailReal |
| 存储器剩余量 | GET | .1.3.6.1.4.1.2021.4.11.0 | UCD-SNMP-MIB::memTotalFree |
| 虚拟内存容量 | GET | .1.3.6.1.4.1.2021.4.3.0 | UCD-SNMP-MIB::memTotalSwap |
| 虚拟内存剩余量 | GET | .1.3.6.1.4.1.2021.4.4.0 | UCD-SNMP-MIB::memAvailSwap |
| **存储设备**  |
| 各磁盘容量 | WALK | .1.3.6.1.4.1.2021.9.1.6.1 | UCD-SNMP-MIB::dskTotal |
| 各磁盘消耗量 | WALK | .1.3.6.1.4.1.2021.9.1.7.1 | UCD-SNMP-MIB::dskAvail |
| 各磁盘消耗量百分比 | WALK | .1.3.6.1.4.1.2021.9.1.9.1 | UCD-SNMP-MIB::dskPercent |
| **网络环境**  |
| 网络设备接口名称 | WALK | .1.3.6.1.2.1.31.1.1.1.1 | IF-MIB::ifName |

## 私有MIB库

FortiGate的私有mib文件在SNMP配置界面处直接可以下载到、或support网站下载、或互联网上获取。

### FortiGate SNMP配置界面处直接下载

<img src=".\..\..\..\images\image-20220822175618629.png" alt="image-20220822175618629" style="zoom:50%;" />

<a href="../../../files/FORTINET-CORE-MIB.txt" target="_blank">FORTINET-CORE-MIB.txt</a>，<a href="../../../files/FORTINET-FORTIGATE-MIB.txt" target="_blank">FORTINET-CORE-MIB.txt</a>（基于FortiOS 7.0.6下载的私有MIB文件，可点击下载MIB文件）

### SUPPORT网站下载

<img src=".\..\..\..\images\image-20220822181045517.png" alt="image-20220822181045517" style="zoom:50%;" />

### 其他方式

1. 在很多公共的网站也都是可以查到Fortinet的mib库文件和内容的，比如：Oidview：http://www.oidview.com/mibs/12356/FORTINET-FORTIGATE-MIB.html

2. 关于私有mib的oid fortinet的私有number是12356，可通过以下链接获取私有厂家的enterprise-numbers：https://www.iana.org/assignments/enterprise-numbers/enterprise-numbers

   <img src=".\..\..\..\images\image-20220822181305089.png" alt="image-20220822181305089" style="zoom:50%;" />
