# Mac FortiClient安装及使用

## Mac FortiClient完整版安装

1. **下载FortiClient**

   这里使用FortiClient 7.0.7的版本。登录https://support.fortinet.com/Download/FirmwareImages.aspx 网站下载FortiClient_7.0.7.0245_macosx.dmg。
   
   ![image-20221228165002805](../../images/image-20221228165002805.png)

2. **安装FortiClient**

   点击FortiClient_7.0.7.0245_macosx.dmg安装，点击”Install“。
   
   ![image-20230103174611362](../../images/image-20230103174611362.png)
   
   点击“继续”。
   
   ![image-20230103174630838](../../images/image-20230103174630838.png)
   
   点击“继续”。
   
   ![image-20230103174716173](../../images/image-20230103174716173.png)
   
   点击“Agree”。
   
   ![image-20230103174729430](../../images/image-20230103174729430.png)
   
   点击“继续”。
   
   ![image-20230103174806806](../../images/image-20230103174806806.png)
   
   输入用户名和密码授权安装。
   
   ![image-20230103174829031](../../images/image-20230103174829031.png)
   
   ![image-20230103174909410](../../images/image-20230103174909410.png)
   
   选择“允许"。
   
   ![image-20230103191131556](../../images/image-20230103191131556.png)
   
   
   
   安装成功。
   
   ![image-20230103174928061](../../images/image-20230103174928061.png)
   
   打开FortiClient。
   
   ![image-20230103175308446](../../images/image-20230103175308446.png)
   
   **3.连接EMS**
   
   在连接EMS前，需要授予FortiClient磁盘访问权限。
   
   ![image-20230103175005842](../../images/image-20230103175005842.png)
   
   输入EMS IP地址，点击“连接”。
   
   ![image-20230103175443865](../../images/image-20230103175443865.png)
   
   选择“允许”
   
   ![image-20230103175516003](../../images/image-20230103175516003.png)
   
   已连接上EMS。
   
   ![image-20230103175559084](../../images/image-20230103175559084.png)

## MAC 免费版FortiClient VPN安装

1. **下载FortiClient**

   这里使用FortiClient 7.0.7的版本。登录https://support.fortinet.com/Download/FirmwareImages.aspx 网站下载FortiClientVPNSetup_7.0.7.0245_macosx.dmg。

   ![image-20230103191530184](../../images/image-20230103191530184.png)

2. **安装FortiClient**

   点击FortiClientVPNSetup_7.0.7.0245_macosx.dmg，点击“Install”。

   ![image-20230103191622704](../../images/image-20230103191622704.png)

   点击“继续”。

   ![image-20230103191721265](../../images/image-20230103191721265.png)

   点击“继续”。

   ![image-20230103191745910](../../images/image-20230103191745910.png)

   选择“Agree”。

   ![image-20230103191759540](../../images/image-20230103191759540.png)

   点击“安装”。

   ![image-20230103191822527](../../images/image-20230103191822527.png)

   输入用户名和密码授权安装。

   ![image-20230103191843864](../../images/image-20230103191843864.png)

   ![image-20230103191853015](../../images/image-20230103191853015.png)

   选择“允许”。

   ![image-20230103191915970](../../images/image-20230103191915970.png)

   安装完成。

   ![image-20230103191938341](../../images/image-20230103191938341.png)

   3. **打开FortiClient**

      勾选“I acknowledge that this free software...”，点击I accept。免费的FortiClient是不提供技术支持服务的。

      ![image-20230103191954052](../../images/image-20230103191954052.png)

      同意后FortiClient界面如下：

      ![image-20230103192127415](../../images/image-20230103192127415.png)

      授予FortiClient磁盘访问权限。

      ![image-20230103192146348](../../images/image-20230103192146348.png)

## 设置SSLVPN

1. **点击“配置VPN”**

   ![image-20230103180103911](../../images/image-20230103180103911.png)

2. **配置SSLVPN**

   连接名：SSLVPN名称

   远程网关：FortiGate开启SSLVPN服务的接口地址；

   自定义端口：SSLVPN服务的端口号；

   ![image-20230103180124620](../../images/image-20230103180124620.png)

3. **配置完成**

   ![image-20230103180235319](../../images/image-20230103180235319.png)

   4. **连接SSLVPN**

      输入账号密码后，点击“连接”，再弹出窗口选择 continue，接受FortiGate的证书。

      ![image-20230103180318897](../../images/image-20230103180318897.png)

      SSLVPN连接成功。

      ![image-20230103180425300](../../images/image-20230103180425300.png)

