## **组网需求**

在外移动办公的工作人员需要通过SSL VPN 并且使用数字证书认证的方式，拨入到公司内网来对内网主机进行访问。

## **网络拓扑**

```
PC1---------------Internet-------------(port2:100.1.1.2)FGT-BJ(port5:192.168.0.1/24)-----------PC2(192.168.0.10  HTTPS/SSH Service)
                                                           |-----------PC3(192.168.0.20   RPD Service)
```

## 配置步骤

### 使用FortiAuthenticator为FGT-BJ和终端颁发证书

FortiAuthenticator可以做为证书服务器，RootCA是已创建好的根证书，这里使用RootCA为这两台FortiGate签发证书。（也可以使用其他证书服务器）![image-20221208151721419](../../images/image-20221208151721419.png)

1. **为FGT-BJ颁发证书**

   为FGT-BJ创建证书。选择“End Entities”-->"User"，点击“Create New”，输入证书相关信息，并点击完成。![image-20221208152741420](../../images/image-20221208152741420.png)![image-20221208153601437](../../images/image-20221208153601437.png)点击“Export Key and Cert”导出FGTBJ证书的私钥和公钥。![image-20221208154345603](../../images/image-20221208154345603.png)输入一个密码保护私钥![image-20221208154402928](../../images/image-20221208154402928.png)下载证书![image-20221208154418425](../../images/image-20221208154418425.png)

2. **为终端颁发证书**

   为终端颁发证书和上述步骤是一样的。

   ![image-20221213171235357](../../images/image-20221213171235357.png)

   ![image-20221213171759834](../../images/image-20221213171759834.png)

### FortiGate导入证书

将根证书RootCA.crt和证书FGTBJ.p12导入到FGT_BJ。

在数字证书认证中，通信双方使用CA证书进行数字证书合法性验证，双方各有自己的公钥（网络上传输）和私钥（本地存储）。发送方对报文进行Hash计算，并用自己的私钥对报文计算结果进行加密，生成数字签名。接收方使用发送方的公钥对数字签名进行解密，并对报文进行Hash计算，判断计算结果与解密后的结果是否相同。如果相同，则认证通过；否则认证失败。

![image-20221213171652452](../../images/image-20221213171652452.png)

1. 导入CA证书

   选择“系统管理”-->“证书”，点击“Create/Import”，选择CA证书。![image-20221208155330877](../../images/image-20221208155330877.png)![image-20221208160106684](../../images/image-20221208160106684.png)查看导入的CA证书![image-20221208160158659](../../images/image-20221208160158659.png)

2. 导入本地证书

   选择“系统管理”-->“证书”，点击“Create/Import”，选择“证书”。

   ![image-20221208160323059](../../images/image-20221208160323059.png)

   点击导入证书

   ![image-20221208160414617](../../images/image-20221208160414617.png)

   输入证书的密码，点击“创建”

   ![image-20221208160501558](../../images/image-20221208160501558.png)

   提示证书已成功导入

   ![image-20221208160539608](../../images/image-20221208160539608.png)

   查看导入的本地证书

   ![image-20221208160615799](../../images/image-20221208160615799.png)

### 终端导入证书

将根证书RootCA.crt和证书user1.p12导入到终端。

![image-20221213171953792](../../images/image-20221213171953792.png)

1. 导入根证书RootCA.crt

   双击RootCA.crt，点击“安装证书”。

   ![image-20221213173524039](../../images/image-20221213173524039.png)

   ![image-20221213173551319](../../images/image-20221213173551319.png)

   证书存入“受信任的根证书颁发机构”

   ![image-20221213173617236](../../images/image-20221213173617236.png)

   ![image-20221213173702669](../../images/image-20221213173702669.png)

2. 导入个人证书user1.p12

   双击user1.p12，然后安装证书

   ![image-20221213173832526](../../images/image-20221213173832526.png)

   输入证书的密码

   ![image-20221213173906247](../../images/image-20221213173906247.png)

   证书存入“个人”

   ![image-20221213173921102](../../images/image-20221213173921102.png)

   ![image-20221213173953806](../../images/image-20221213173953806.png)

### 配置SSL VPN

1. **基本配置**

   配置接口IP和路由

   ![image-20221208113336372](../../images/image-20221208113336372.png)

   ![image-20221208113353399](../../images/image-20221208113353399.png)

2. **创建用户，并将用户加入到用户组。**

   ![image-20221216195448684](../../images/image-20221216195448684.png)

   ![image-20221216195539180](../../images/image-20221216195539180.png)

3. **配置SSL-VPN门户**

   ![image-20221216203330106](../../images/image-20221216203330106.png)

   ![image-20221216203418968](../../images/image-20221216203418968.png)

4. **配置SSLVPN**

   服务器证书选择“FGTBJ”，FortiGate会使用FGTBJ证书与终端交互；勾选“需要客户端证书”，那么认证过程中FortiGate会要求客户端提供证书。

   ![image-20221216203512730](../../images/image-20221216203512730.png)

5. **创建策略**

   当客户端SSLVPN拨号成功后，将会使用获取的地址（10.200.1.10-10.200.1.200）访问内部主机，因此内部网络需要增加到10.200.1.0/24网段的回程路由指向FortiGate 或者 可以在策略中开启NAT，那么源地址将被转换为FortiGate接口地址，则不用考虑回程路由。![image-20221216180923214](../../images/image-20221216180923214.png)

### 配置FortiClient

1. 选择“Remote Access”，点击“配置VPN”。

   ![image-20221216190716426](../../images/image-20221216190716426.png)

2. 选择SSL VPN，设置连接名，远程网关，SSLVPN端口，用户名以及选择终端证书，然后点击保存。

   ![image-20221216201030529](../../images/image-20221216201030529.png)

3. 配置完成。

   ![image-20221216201210723](../../images/image-20221216201210723.png)

## FortiClient拨号测试

1. 输入密码，点击连接。

   ![image-20221216202930581](../../images/image-20221216202930581.png)

   查看终端获取到的路由

   ![image-20221216191836232](../../images/image-20221216191836232.png)

2. 访问内网主机HTTPS，SSH，RDP服务都正常

   ![image-20221216192109274](../../images/image-20221216192109274.png)

3. FortiGate查看SSLVPN连接

   ![image-20221216203151250](../../images/image-20221216203151250.png)

   ```
   # get vpn ssl monitor 
   SSL-VPN Login Users:
    Index   User    Group   Auth Type      Timeout         Auth-Timeout    From     HTTP in/out    HTTPS in/out    Two-factor Auth
    0       user1   SSLVPN-Group   1(1)             189    28625    10.1.1.5       0/0     0/0     0
   
   SSL-VPN sessions:
    Index   User    Group   Source IP      Duration        I/O Bytes       Tunnel/Dest IP 
    0       user1   SSLVPN-Group   10.1.1.5         175     27388/14075    10.200.1.10
   ```

## 浏览器拨号测试

1. PC1使用chrome访问SSLVPN，在弹出的证书选择框中，使用user1证书。

   ![image-20221216204003533](../../images/image-20221216204003533.png)

2. 跳转到输入账号密码界面。

   ![image-20221216204125049](../../images/image-20221216204125049.png)

3. 登录成功。

   ![image-20221216204151133](../../images/image-20221216204151133.png)

4. 访问webserver成功。

   ![image-20221216204201295](../../images/image-20221216204201295.png)

5. FortiGate查看SSLVPN用户状态。

   ![image-20221216204601292](../../images/image-20221216204601292.png)

   ```
    # get vpn ssl monitor 
   SSL-VPN Login Users:
    Index   User    Group   Auth Type      Timeout         Auth-Timeout    From     HTTP in/out    HTTPS in/out  Two-factor Auth
    0       user1   SSLVPN-Group   1(1)      289                28788    10.1.1.5       0/0               0/0          0
   
   SSL-VPN sessions:
    Index   User    Group   Source IP      Duration        I/O Bytes       Tunnel/Dest IP 
   ```

   