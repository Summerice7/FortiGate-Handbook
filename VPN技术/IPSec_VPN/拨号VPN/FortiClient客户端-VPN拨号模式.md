# FortiClient客户端-VPN拨号模式

## 组网需求

某公司内部有一台OA服务器，在外移动办公的工作人员需要通过IPSEC VPN拨入到公司内网来对内网服OA服务器进行访问。

## 网络拓扑

PC---------------Internet-------------(port2:100.1.1.2)FGT-BJ(port5:192.168.0.1/24)-----------OA Server(192.168.0.10)

## 配置步骤

### 配置IPSec

1. **基本配置**

   配置接口IP和路由

   ![image-20221208113336372](../../../images/image-20221208113336372.png)

   ![image-20221208113353399](../../../images/image-20221208113353399.png)

2. **创建用户**

   选择“用户与认证”-->“设置用户”，点击“新建”。

   ![image-20221212151128891](../../../images/image-20221212151128891.png)

   选择“本地用户”，点击“下一步”。

   ![image-20221212151157811](../../../images/image-20221212151157811.png)

   输入用户名和密码，点击“下一步”。

   ![image-20221212151319210](../../../images/image-20221212151319210.png)

   可根据需求选择启用。这里不启用，点击“下一步”。

   ![image-20221212151409185](../../../images/image-20221212151409185.png)

   点击“提交”。

   ![image-20221212151629689](../../../images/image-20221212151629689.png)

   完成用户创建。

   ![image-20221212151728884](../../../images/image-20221212151728884.png)

3. **创建用户组**

   选择“用户与认证”-->“用户组”，点击“新建”。

   ![image-20221212152333684](../../../images/image-20221212152333684.png)

   输入名称，即组名，并将用户加入用户组，点击“确认”。

   ![image-20221212152302590](../../../images/image-20221212152302590.png)

   完成用户组创建。

   ![image-20221212152551383](../../../images/image-20221212152551383.png)

4. **创建子网**

   ![image-20221212165733235](../../../images/image-20221212165733235.png)

5. **配置IPSEC VPN**

   选择“VPN”-->“IPsec隧道”，点击“新建”，选择“IPsec隧道”。

   ![image-20221211171718818](../../../images/image-20221211171718818.png)

   根据“VPN创建向导”进行VPN模板配置，输入名称，选择”远程接“-->"基于客户端"-->“FortiClient”，并点击下一步。

   ![image-20221212155845612](../../../images/image-20221212155845612.png)

   选择对外的接口，设置预共享秘钥和用户组，并点击下一步。

   ![image-20221212160044184](../../../images/image-20221212160044184.png)

   输入内网接口，允许访问的本地地址段，客户端拨号后获取的地址范围，并点击下一步。

   ![image-20221212165918440](../../../images/image-20221212165918440.png)

   保存密码，FortiClient第一次拨号成功后，FortiClient会显示“保存密码”的选项，默认勾选。

   自动连接，FortiClient第一次拨号成功后，FortiClient会显示“自动连接”的选项，该功能是运行FortiClient会拨入该IPSEC VPN。

   保持存活，FortiClient第一次拨号成功后，FortiClient会显示“保持连接”的选项，该功能是IPSEC VPN由于网络原因中断后，会自动重拨。

   免费的FortiClient版本不支持自动连接和保持连接，需要完整版的FortiClient，需要购买EMS license。

   ![image-20221212190846081](../../../images/image-20221212190846081.png)

   VPN创建向导提示即将创建的内容，然后点击完成。

   ![image-20221212163207898](../../../images/image-20221212163207898.png)

   VPN创建成功。

   ![image-20221212163225172](../../../images/image-20221212163225172.png)


### 查看IPSEC向导创建的配置

通过“VPN创建向导”可以很方便的配置VPN，但我们需要知道向导具体做了哪些配置。

1. **创建地址对象和地址对象组。**

   ![image-20221212170204843](../../../images/image-20221212170204843.png)

2. **创建IPSEC VPN**

   ![image-20221212163430343](../../../images/image-20221212163430343.png)

   对应的命令行

   ```
   config vpn ipsec phase1-interface
       edit "ClientDial"
           set type dynamic
           set interface "port2"
           set mode aggressive
           set peertype any
           set net-device disable
           set mode-cfg enable
           set proposal aes128-sha256 aes256-sha256 aes128-sha1 aes256-sha1
           set comments "VPN: ClientDial (Created by VPN wizard)"
           set wizard-type dialup-forticlient
           set xauthtype auto
           set authusrgrp "IPSEC_VPN_Group"
           set ipv4-start-ip 172.16.100.100
           set ipv4-end-ip 172.16.100.200
           set dns-mode auto
           set ipv4-split-include "ClientDial_split"
           set save-password enable
           set psksecret ENC cTt0jnn1i1wx0Elky0oa0pRS3I7Kb8UAUHRndju/6cI7nbcQ1IBcmXqdj9aJXjE/ZNUZgElJLwKARXoP3iJPBAkWJaYtQrEC+XfZC2QGr/Xsw72UQSL0Rll8wIKAk7vPDIQXbn/eQSYPs7CH+2r64HiKJ91+zt3LzAlASNooXPKea8EK7f9qaUTfj/p/kxqrk9aN6A==
       next
   end
   
   config vpn ipsec phase2-interface
       edit "ClientDial"
           set phase1name "ClientDial"
           set proposal aes128-sha1 aes256-sha1 aes128-sha256 aes256-sha256 aes128gcm aes256gcm chacha20poly1305
           set comments "VPN: ClientDial (Created by VPN wizard)"
       next
   end
   ```

2. **创建策略。**

   ![image-20221212170251319](../../../images/image-20221212170251319.png)

### 配置FortClient

1. 选择“Remote Access”，点击“配置VPN”。

   ![image-20221216190716426](../../../images/image-20221216190716426.png)

2. 输入连接名，远程网关，共享秘钥，以及用户名，然后点击“保存”。

   ![image-20221212184826744](../../../images/image-20221212184826744.png)

   在”高级设置“有更多的选项，如ike版本，IKE阶段一和阶段二的加密集等，可以根据FortiGate的设置做对应的修改。

   ![image-20221212184907089](../../../images/image-20221212184907089.png)

   ![image-20221212184923792](../../../images/image-20221212184923792.png)

   3. VPN配置完成

      ![image-20221212192335836](../../../images/image-20221212192335836.png)

   

## FortClient拨号测试

1. 在FortiClient中输入账号密码，点击“连接”，FortiClient拨号成功。

   ![image-20221212193033211](../../../images/image-20221212193033211.png)

2. 查看终端路由表，192.168.0.0/24指向IPSEC VPN，并能成功访问OA Server。

   ![image-20221212193221067](../../../images/image-20221212193221067.png)

   ![image-20221212193339036](../../../images/image-20221212193339036.png)

3. FortiGate查看client连接。

   ![image-20221213180114989](../../../images/image-20221213180114989.png)

4. 点击“中断连接”，FortiClient就会显示“保存密码”选项。

   ![image-20221212192829690](../../../images/image-20221212192829690.png)

