# Web管理

## 需求

通过web可视化操作界面对防火墙进行配置管理，配置wan1口的管理功能。

## 组网拓扑

<img src="..\..\images\image-20220816181159088.png" alt="image-20220816181159088" style="zoom: 50%;" />

## 配置要点

1. FortiGate出厂配置默认地址为192.168.1.99，可以通过https的方式进行web管理（默认用户名admin，密码为空）。不同型号设备用于管理的接口略有不同，如:
   FortiGate-1500D：mgmt1接口
   FortiGate-60D：internal 接口，对应于交换口 1-7

   > 说明：FortiGate-60D、FortiGate-90D等所有的交换口隶属于三层口internal下，仅有internal接口可以进行三层配置，例如IP配置。
   > 具体的接口管理IP分配需要参考设备出厂包装盒里面的：QuickStart Guide和FortiGate Supplement文档。或者通过docs网站文档搜索获取具体型号的接口信息和登陆管理信息链接：
   >
   > https://docs.fortinet.com/product/fortigate/hardware
   > https://docs.fortinet.com/document/fortigate/hardware/fortigate-fortiwifi-60e-poe-qsg-supplement?model=all
   > https://docs.fortinet.com/document/fortigate/hardware/fortigate-quickstart-guide-high-end?model=all

2. 举例：FGT60E

   <img src=".\..\..\images\image-20220817103300850.png" alt="image-20220817103300850" style="zoom: 50%;" />

   <img src=".\..\..\images\image-20220817103614202.png" alt="image-20220817103614202" style="zoom: 67%;" />

3. 将电脑IP地址设置为192.168.1.2/24，并连到internal口或者MGMT口，打开Firefox/Google Chrome，输入
   https://192.168.1.99 ，用户名：admin，密码：空，进入FortiGate设备页。
   
   > 若密码修改后忘记请参照"[系统管理>密码恢复](密码恢复.md)"章节进行恢复初始密码。

4. 登陆设备后开启wan1接口的管理功能。默认其他接口是没有IP地址，也未开启https等其他管理功能。若防火墙接口地址修改过，但忘记相应地址，可进命令行查看当前配置，参见"[系统管理>设备管理>命令行管理](常用基础命令.md)"章节。

   > 说明：浏览器建议采用Firefox、Microsoft Edge、Google Chrome.

## 配置步骤

1. 在FortiGate出厂配置情况下，将电脑设置为192.168.1.1，网关设置为192.168.1.99，如下图：

   <img src=".\..\..\images\image-20220817104927339.png" alt="image-20220817104927339" style="zoom: 50%;" />

2. 在IE浏览器里输入https://192.168.1.99 将出现防火墙登陆的页面。

   <img src=".\..\..\images\image-20220817105201092.png" alt="image-20220817105201092" style="zoom: 50%;" />

3. 输入用户名admin密码默认空，即可进入防火墙首页。

   <img src=".\..\..\images\image-20220817111223244.png" alt="image-20220817111223244" style="zoom: 50%;" />

4. 配置接口wan1的地址为192.168.0.99/24，并开启internal的管理功能（系统管理--网络--接口）。

   <img src=".\..\..\images\image-20220817111324435.png" alt="image-20220817111324435" style="zoom: 50%;" />

5. 双击wan1接口进行编辑，配置接口地址为：192.168.0.99/24，管理访问：勾选 HTTPS、HTTP、PING、SSH，SNMP。

   <img src=".\..\..\images\image-20220817111444668.png" alt="image-20220817111444668" style="zoom: 50%;" />

   具体各种方式含义如下：

   > https：允许用户用https://192.168.0.99 ，进行管理。
   >
   > ping：允许用户ping此接口地址，如果不勾选，在路由可达的情况下也Ping不通。
   >
   > http：允许用户用http://192.168.0.99 ，进行管理。
   >
   > ssh：允许用户用用ssh 192.168.0.99方式管理设备。
   >
   > snmp: 允许用户通过该接口对设备进行SNMP管理。

## 验证效果

电脑直连WAN1接口，电脑网卡IP修改为192.168.0.2，在浏览器里输入：https://192.168.0.99 ，即可通过WAN1接口登陆管理防火墙。
