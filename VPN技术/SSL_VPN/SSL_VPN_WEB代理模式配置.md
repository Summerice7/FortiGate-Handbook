# SSL VPN WEB代理模式配置

## **组网需求**

在外移动办公的工作人员需要通过SSL VPN WEB模式拨入到公司内网来对内网主机进行访问。

## 网络拓扑

```
PC1---------------Internet-------------(port2:100.1.1.2)FGT-BJ(port5:192.168.0.1/24)-----------PC2(192.168.0.10  HTTPS/SSH Service)
                                                           |-----------PC3(192.168.0.20   RPD Service)
```

## 配置步骤

1. **基本配置**

   配置接口IP和路由

   ![image-20221208113336372](../../images/image-20221208113336372.png)

   ![image-20221208113353399](../../images/image-20221208113353399.png)

2. **创建用户**

   选择“用户与认证”-->“设置用户”，点击“新建”。

   ![image-20221212151128891](../../images/image-20221212151128891.png)

   选择“本地用户”，点击“下一步”。

   ![image-20221212151157811](../../images/image-20221212151157811.png)

   输入用户名和密码，点击“下一步”。

   ![image-20221212151319210](../../images/image-20221212151319210.png)

   可根据需求选择启用。这里不启用，点击“下一步”。

   ![image-20221212151409185](../../images/image-20221212151409185.png)

   点击“提交”。

   ![image-20221212151629689](../../images/image-20221212151629689.png)

   完成用户创建。

   ![image-20221212151728884](../../images/image-20221212151728884.png)

3. **创建用户组**

   选择“用户与认证”-->“用户组”，点击“新建”。

   ![image-20221212152333684](../../images/image-20221212152333684.png)

   输入名称，即组名，并将用户加入用户组，点击“确认”。

   ![image-20221216120138991](../../images/image-20221216120138991.png)

   完成用户组创建。

   ![image-20221216120159614](../../images/image-20221216120159614.png)

4. **配置SSL-VPN门户**

   系统默认创建3个SSLVPN门户：full-access:开启了隧道模式和web代理模式；tunnel-access只开启了隧道模式；web-access：只开启web代理模式。可以根据实际需求进行修改，也可以新建新的SSL-VPN门户。

   ![image-20221216104055636](../../images/image-20221216104055636.png)

   这里使用web-access门户，新建HTTPS，SSH，RPD 3个资源供客户端拨号后访问。

   ![image-20221216165706066](../../images/image-20221216165706066.png)

   PC2-HTTPS资源配置：

   ![image-20221216164235718](../../images/image-20221216164235718.png)

   PC2-SSH资源配置：

   ![image-20221216165253610](../../images/image-20221216165253610.png)

   PC3-RDP资源配置：

   ![image-20221216165823010](../../images/image-20221216165823010.png)

3. **配置SSLVPN**

   设置提供SSLVPN服务的接口和端口，将SSLVPN-Group用户组和web-access门户关联，全部其他用户/组也要关联一个门户，用于没有配置关联的用户/用户组访问。

   ![image-20221216120825927](../../images/image-20221216120825927.png)

6. **创建策略**

   ![image-20221216164747018](../../images/image-20221216164747018.png)

## 业务测试

1. PC1使用chrome访问SSLVPN，并输入用户名和密码。

   ![image-20221216165025532](../../images/image-20221216165025532.png)

2. 登录成功后，可以看到之前创建好的3个资源。

   ![image-20221216165738619](../../images/image-20221216165738619.png)

3. 业务访问

   点击PC2-HTTPS，访问成功。

   ![image-20221216165145389](../../images/image-20221216165145389.png)

   点击PC2-SSH，访问成功。

   ![image-20221216165345941](../../images/image-20221216165345941.png)

   输入PC2 SSH的用户的密码，登录成功。

   ![image-20221216165437607](../../images/image-20221216165437607.png)

   点击PC3-RDP，访问成功。

   ![image-20221216165851295](../../images/image-20221216165851295.png)

4. FortiGate查看SSLVPN用户状态。

   ![image-20221216204714782](../../images/image-20221216204714782.png)

   ```
    # get vpn ssl monitor 
   SSL-VPN Login Users:
    Index   User    Group   Auth Type      Timeout         Auth-Timeout    From     HTTP in/out    HTTPS in/out  Two-factor Auth
    0       user1   SSLVPN-Group   1(1)      289                28788    10.1.1.5       0/0               0/0          0
   
   SSL-VPN sessions:
    Index   User    Group   Source IP      Duration        I/O Bytes       Tunnel/Dest IP 
   ```

   
