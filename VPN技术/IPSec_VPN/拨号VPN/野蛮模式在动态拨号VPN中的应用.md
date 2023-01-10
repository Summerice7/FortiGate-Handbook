# 野蛮模式在动态拨号VPN中的应用

## 关于主模式中的Peer ID问题

在主模式中，第3、4个包交互之后（DH公钥和随机数交换），需要使用到Pre-shared key（管理员配置的预共享密钥）值用于生成SHEYID（预共享密钥方式认证：SKEYID=hash (pre-shared key ,Ni | Nr) ），此时是无法通过Peer-ID查找到，因为Peer-ID参数在第5、6个包中，而且还是被加密的，解密需要使用到SHEYID为材料生成的密钥SHEYID_e，因此这个时候只能通过对方发起IKE协商的公网IP地址去查找Peer的Pre-shared key以确认其身份。

如此将引出一个问题，在对方（分部）为PPPOE/DHCP等动态IP场景中时，总部无法配置明确的对方IP和Pre-shared key，而只能配置动态拨号方式的IPsec VPN，如果我们选择第一阶段为主模式，因为Peer-ID在第5、6包携带，并且是加密的，因此FGT在主模式中是无法提前获知到Peer-ID信息的，也没办法根据Peer-ID信息去查找到底用哪一个Pre-shared key，一般动态IP场景中主模式使用较少，而且就算使用主模式的话的Peer-ID配置为any的较多。

## **什么时候我们需要使用到野蛮模式**

1. 存在多条动态拨号的IPsec VPN，此时由于拨号VPN的目的IP都是any（0.0.0.0），此时根本无法区分这两条IPsec VPN隧道，不知道拨号用户到底是匹配Dia_VPN1还是匹配Dia_VPN2（FGT的处理是优先匹配靠前的Dia_VPN），因此为了区分本地的多条动态拨号VPN，则需要配置Peer ID参数，以便区分这本地的多条动态IPsec VPN隧道，例如：

   Dia_VPN1：动态拨号VPN 野蛮模式 Local ID BJ **Peer ID**：South-FGT；

   Dia_VPN2：动态拨号VPN 野蛮模式 Local ID BJ **Peer ID**：North-FGT；

   如果对方拨号用户1填写的Local ID是South-FGT则连接到Dia_VPN1，如果对方拨号用户2填写的Local ID是North-FGT则连接到Dia_VPN2。这样就可以将不通的拨号用户进行区分和分离，方便进行独立的VPN隧道、独立的策略进行控制。在这种需求的情况下：首先需要舍弃掉主模式，用野蛮模式，同时使用不同的Peer-ID来区分不同的动态VPN隧道。

2. Forticlient等IPsec VPN客户端拨号时候需要使用野蛮模式，一般按照模板进行配置即可，模板自动会调整为野蛮模式。
