# FortiClient安装

## Windows FortiClient完整版安装

1. **下载FortiClient**

   这里使用FortiClient 7.0.7的版本。登录https://support.fortinet.com/Download/FirmwareImages.aspx 网站下载FortiClientSetup_7.0.7.0345_x64.zip，x64表示64位系统。

   ![image-20221228150744278](../../images/image-20221228150744278.png)

2. **安装FortiClient**

   解压FortiClientSetup_7.0.7.0345_x64.zip，然后点击“FortiClient.msi”安装。

   ![image-20221228150943557](../../images/image-20221228150943557.png)

   勾选“Yes,I have read an accept the ...”，点击Next。

   ![image-20221228151150201](../../images/image-20221228151150201.png)

   选择“Secure Remote Access”，即SSLVPN和IPSEC VPN，点击Next。

   ![image-20221228151339142](../../images/image-20221228151339142.png)

   点击Next。

   ![image-20221228151444534](../../images/image-20221228151444534.png)

   点击Next。

   ![image-20221228151612744](../../images/image-20221228151612744.png)

   点击Install。

   ![image-20221228151637676](../../images/image-20221228151637676.png)

   ![image-20221228151711956](../../images/image-20221228151711956.png)

   点击“Finish”，安装完成。

   ![image-20221228151927713](../../images/image-20221228151927713.png)

3. **连接EMS**

   打开FortiClient，在没有连接EMS之前，FortiClient会显示UNLICNESED，并提示试用时间。

   ![image-20221228153901855](../../images/image-20221228153901855.png)

   输入EMS服务器的地址或者域名。

   ![image-20221228153256142](../../images/image-20221228153256142.png)

   选择”允许“

   ![image-20221228153713171](../../images/image-20221228153713171.png)

   已连接上EMS，并且不再提示UNLICNESED。

   ![image-20221228154254993](../../images/image-20221228154254993.png)

   ![image-20221228154315553](../../images/image-20221228154315553.png)

## Windows免费版FortiClient VPN安装

1. **下载FortiClient**

   这里使用FortiClient 7.0.7的版本。登录https://support.fortinet.com/Download/FirmwareImages.aspx 网站下载FortiClientVPNSetup_7.0.7.0345_x64.exe，x64表示64位系统。

   ![image-20221228160447907](../../images/image-20221228160447907.png)

2. **安装FortiClient**

   双击FortiClientVPNSetup_7.0.7.0345_x64.exe安装，勾选“Yes,I have read an accept the ...”，点击Next。

   ![image-20221228163318893](../../images/image-20221228163318893.png)

   点击Install。

   ![image-20221228163404620](../../images/image-20221228163404620.png)

   ![image-20221228163426362](../../images/image-20221228163426362.png)

   安装完成，点击“Finish”。

   ![image-20221228163750814](../../images/image-20221228163750814.png)

3. **打开FortiClient**

   勾选“I acknowledge that this free software...”，点击I accept。免费的FortiClient是不提供技术支持服务的。

   ![image-20221228163844603](../../images/image-20221228163844603.png)

   同意后显示配置界面。

   ![image-20221228163958532](../../images/image-20221228163958532.png)

   

## Windows设置SSLVPN

1. **点击“配置VPN”**

   ![image-20221228155220758](../../images/image-20221228155220758.png)

2. **设置SSLVPN连接**

   连接名：SSLVPN名称

   远程网关：FortiGate开启SSLVPN服务的接口地址；

   自定义端口：SSLVPN服务的端口号；

   ![image-20221228155358232](../../images/image-20221228155358232.png)

3. **配置完成**

   ![image-20221228155625521](../../images/image-20221228155625521.png)

4. **SSLVPN连接**

   输入账号和密码后，点击连接，再弹出窗口选择是，接受FortiGate的证书。

   ![image-20221228155716504](../../images/image-20221228155716504.png)

   连接成功。

   ![image-20221228155817041](../../images/image-20221228155817041.png)
