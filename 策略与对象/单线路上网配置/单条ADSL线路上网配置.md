# 单条ADSL线路上网配置

## 组网需求

外网接口使用ADSL拨号，内网为192.168.1.0/24网段，实现基本上网功能。

## 网络拓扑

<img src="../../images/image-20230106160351709.png" alt="image-20230106160351709" style="zoom:50%;" />

## 配置要点

- wan1（port2）口: 接ADSL的接口，务必开启自动获取网关，这样ADSL拨号成功后设备会自动生成默认路由，无需手动配置默认路由
- internal（port3）口:  IP地址设置格式为：192.168.1.99/24，可选择性地开启接口的管理功能
- 配置地址对象“lan”，内容为192.168.1.0/24
- 配置从internal（port3）到pppoe口的策略，并开启NAT

## 配置步骤

1. 配置wan1口的PPPoE，建议在CLI下配置PPPoE接口，使用此方式配置获取到IP后会自动添加默认路由到运营商。

   ```
   config system pppoe-interface
       edit "pppoe"
           set device "port2"
           set username "user1"
           set password ENC HtX/f4d8yYuG2pDN2yBhZvcexzp46jaJWXCNkKcW+d1aN+Yle7kL59Mn+7o6UX+P8EXg+34FgPLtZKT8eg1cRcFNWCD3dDBAEpFNpWCpCuwfYV8jLNyjS+wgb+yO9v07+mJNu6/yXC+qaXzVM0NstowKcRkfzR3iPsbbgsb5ZjXHv7RqOgyCNGFzTp421///X5O2uQ==
       next
   end
   
   以上命令执行后，会自动在system interface下创建同名接口，其中包含“从服务器中重新得到网关”的配置
   FortiGate # show full sys int pppoe
   config system interface
       edit "pppoe"
           set vdom "root"
           set mode pppoe
           set type tunnel
           set defaultgw enable
           set interface "port2"
       next
   end
   ```
   
2. 以上配置完成后，可以在GUI上看到wan1（port2）下多了一个PPPoE接口，并获取到了运营商的IP。可以看到“从服务器中重新得到网关”已开启。

   <img src="../../images/image-20230106164035375.png" alt="image-20230106164035375" style="zoom:50%;" />

   ```
   FortiGate # get router info routing-table all
   Codes: K - kernel, C - connected, S - static, R - RIP, B - BGP
          O - OSPF, IA - OSPF inter area
          N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
          E1 - OSPF external type 1, E2 - OSPF external type 2
          i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
          * - candidate default
   
   Routing table for VRF=0
   S*      0.0.0.0/0 [5/0] via 202.103.100.1, pppoe, [1/0]
   C       192.168.1.0/24 is directly connected, port3
   C       202.103.100.1/32 is directly connected, pppoe
   C       202.103.100.200/32 is directly connected, pppoe
   ```

3. 配置internal（port3）接口的IP为192.168.1.99/24。选择性地开启接口的管理功能，建议内部开启https ,ssh, ping服务。

   <img src="../../images/image-20230106163942687.png" alt="image-20230106163942687" style="zoom:50%;" />

4. 配置地址资源，进入防火墙&对象→地址，点击新建地址按钮。

   <img src="../../images/image-20230106164330249.png" alt="image-20230106164330249" style="zoom:50%;" />

5. 名称配置为lan，地址节点选择子网：192.168.1.0/24，点击确认。

   <img src="../../images/image-20230106164426826.png" alt="image-20230106164426826" style="zoom:50%;" />

   ```
   config firewall address
       edit "lan"
           set subnet 192.168.1.0 255.255.255.0
       next
   end
   ```

6. 配置防火墙策略，放通内网到Internet的流量，并开启NAT，点击确定按钮后，系统自动保存配置，策略生效。

   <img src="../../images/image-20230106170557500.png" alt="image-20230106170557500" style="zoom:50%;" />

   ```
   config firewall policy
       edit 1
           set name "to_Internet"
           set srcintf "port3"
           set dstintf "port2"
           set action accept
           set srcaddr "lan"
           set dstaddr "all"
           set schedule "always"
           set service "ALL"
           set nat enable
       next
   end
   ```

   - 流入接口：internal（port3）
   - 源地址：选择刚才定义的地址资源lan上网网段
   - 流出接口：pppoe接口
   - 目的地址选择: all，代表所有的地址
   - 时间表：always
   - 服务: ALL
   - 动作：ACCEPT
   - NAT：选择 "启用NAT"， 系统会自动将内网的lan地址段ip，转换为wan1接口地址，进行互联网访问
   - 注意：启用"记录允许流量（记录流日志）"将会给系统带来额外的资源消耗，所以非必要情况下请不要启用记录日志

## 结果验证

将电脑IP地址设置为192.168.1.10/24，网关设置地为192.168.1.99，DNS一般设置为当地的DNS即可，电脑可正常上网。
