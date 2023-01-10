# 配置session ttl

## 功能介绍

1. 会话生存时间，即会话建立后无任何数据传送情况下的存活时间，默认为3600秒，当该会话在超时之前有任何数据匹配该会话，则该会话ttl计时器复位到该数值，如3600秒。有的服务如数据库。需要长连接，尽可能能将其TCP的会话超时时间设置得较大。
2. session-ttl配置有4种不同的类型，分为 全局session-ttl 、全局指定服务端口session-ttl 、策略session-ttl 、服务对象session-ttl。除了策略session-ttl可以通过web界面配置之外，其他的都必须在命令行下配置。

## 配置方法

1. 配置全局session-ttl。

   ```
   FortiGate #config system session-ttl
   FortiGate (session-ttl) # set default 604800    //300-604800秒(最大为7天)
   FortiGate (session-ttl) #end
   ```

2. 基于策略的session-ttl。

   ```
   FortiGate # config firewall policy
   FortiGate (policy) # edit 1    
   FortiGate (1) #set srcintf internal
   FortiGate (1) #set dstintf wan1
   FortiGate (1) #set srcaddr all
   FortiGate (1) #set dstaddr all
   FortiGate (1) #set action accept
   FortiGate (1) #set schedule always
   FortiGate (1) #set service ANY
   FortiGate (1) # set session-ttl 604800
   FortiGate (1) #set nat enable
   FortiGate (1) #next
   FortiGate (policy) # end
   ```

3. 基于服务对象的session-ttl，定义不同的对象，虽然同为23端口，可以配置指定不同的session-ttl 时间。

   ```
   FortiGate #config firewall service custom  
   FortiGate (custom) # edit telnetedit telnet
   FortiGate (telnet) #set protocol TCP/UDP/SCT
   FortiGate (telnet) #set tcp-portrange 23
   FortiGate (telnet) # set session-ttl 7200
   FortiGate (telnet) #next
   FortiGate (custom) #edit "telnetnew"
   FortiGate (telnetnew) #set protocol TCP/UDP/SCTP
   FortiGate (telnetnew) #set tcp-portrange 23
   FortiGate (telnetnew) # set session-ttl 3600
   FortiGate (telnetnew) #next
   FortiGate (custom) #end
   ```

## session ttl优先级

1. 全局session-ttl（低）< 全局指定服务端口session-ttl < 策略session-ttl < 服务对象session-ttl（高）。
2. 优先级高的session-ttl优先被使用。
