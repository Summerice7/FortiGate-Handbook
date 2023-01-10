# SNMP介绍

## 简单网络管理协议

简单网络管理协议（SNMP，Simple Network Management Protocol）构成了互联网工程工作小组（IETF，Internet Engineering Task Force）定义的Internet协议族的一部分。该协议能够支持网络管理系统，用以监测连接到网络上的设备是否有任何引起管理上关注的情况。它由一组网络管理的标准组成，包含一个应用层协议（application layer protocol）、数据库模式（database schema），和一组数据对象。 

## 概论和基础观念

在典型的SNMP用法中，有许多系统被管理，而且是有一或多个系统在管理它们。每一个被管理的系统上有运行一个叫做代理者（agent）的软件组件，且透过SNMP对管理系统报告信息。

基本上，SNMP代理者以变量呈现管理数据。管理系统透过GET，GETNEXT和GETBULK协议指令取回信息，或是代理者在没有被询问的情况下，使用TRAP或INFORM发送数据。管理系统也可以发送配置更新或控制的请求，透过SET协议指令达到主动管理系统的目的。配置和控制指令只有当网络基本结构需要改变的时候使用，而监控指令则通常是常态性的工作。

可透过SNMP访问的变量以层次结构的方式结合。这些分层和其他元数据（例如变量的类型和描述）以管理信息库（MIBs）的方式描述。 

## SNMP协议

### SNMP第一版和SMI规格的数据类型SNMP V1

SNMP第一版SMI指定许多SMI规格的数据类型，它们被分为两大类：

1. 简单数据类型
2. 泛应用数据类型

### SNMP第二版和管理信息结构SNMP V2

SNMP第二版SMI在RFC 2578之中描述，它在SNMP第一版的SMI规格数据类型上进行增加和强化，例如位串（bit strings）、网络地址（network addresses）和计数器（counters）。

SNMP协议在OSI模型的应用层（第七层）运作，在第一版中指定五种核心PDU：

```
GET REQUEST
GET NEXT REQUEST
GET RESPONSE
SET REQUEST
TRAP
```

其他PDU在SNMP第二版加入，包含：

```
GETBULK REQUEST
INFORM
```

SNMP第二版SMI也指定了信息模块来详细说明一群相关连的定义。有三种SMI信息模块：MIB模块、回应状态、能力状态。

### SNMP第三版  SNMP V3

SNMP第三版由RFC 3411-RFC 3418定义，主要增加SNMP在安全性和远程配置方面的强化。SNMP第三版提供重要的安全性功能：

1. 信息完整性：保证数据包在发送中没有被窜改。
2. 认证：检验信息来自正确的来源。
3. 数据包加密：避免被未授权的来源窥探。
