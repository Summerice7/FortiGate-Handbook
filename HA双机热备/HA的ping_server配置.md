# HA的ping server配置

## 功能介绍

1. Link Monitor功能是为了防止所谓端口“假死”的问题，链路状态正常，但链路无法工作，防火墙可以通过利用发送ping包，根据对方的响应来判断该端口链路是否可用。

2. 配置命令如下：

   ```
   config system link-monitor
       edit "wan1"
           set srcintf "wan1"
           set server "100.1.1.254"
           set failtime 3
           set ha-priority 5
       next
   end
   ```

3. 如果只配置上述的配置，ping检测失败时HA不会切换，只是去往该接口的路由不再有效。需要在ＨA配置中告诉防火墙wan1口的 pingserver会作为HA切换的触发条件。

   ```
   FGT#config system ha
   FGT(ha)#set pingserver-monitor-interface wan1　 　 //监视wan2口上的pingserver
   FGT(ha)#set pingserver-failover-threshold 0　 　  　 //ha切换的闸值,默认为0
   FGT(ha)#set pingserver-flip-timeout 60        　    　//连续2次ping server触发的HA切换的间隔
   ```

## HA相关配置

1. set ha-priority 1与pingserver-failover-threshold 0相关联。两者相比较，当wan1接口pingserver检查失效后赋值增加1，已经达到了闸值0，所以触发HA会切换。
2. 如果 pingserver-failover-threshold 2，即使该接口wan1的link-moniter检测失败，set ha-priority 1小于pingserver-failover-threshold 2, 达不到触发的闸值，HA不会切换。
