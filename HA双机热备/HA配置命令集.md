# HA配置命令集

HA配置命令config system ha，如下是常用的配置命令。

| 配置参数                                        | 参数说明                                                     |
| ----------------------------------------------- | ------------------------------------------------------------ |
| set group-id 0                                  | 配置HA机群的组ID，一个机群内的成员必须有相同的组ID。该ID会成为生成防火墙接口的的虚拟MAC的一个组成因素，因此当同一个广播域有２组以上的HA机群的时候需要配置不同的组ID,防止MAC地址冲突 |
| set group-name "FGT-HA"                         | 一个机群内的成员必须有相同的组名字                           |
| set mode standalone/a-a/a-p                     | HA工作模式，常用为a-p模式。AA模式下在HA状态中查看到HA的角色，有主设备及从设备,通常会被认为工作在主被模式下,实际上主主下设备虽然都在工作,仍会有一台作为集群的主设备用来控制和分配流量和会话给集群中的其他设备。AA模式默认情况下仅负载均衡UTM的流量,所以在下不使用UTM功能时建议使用AP模式 |
| set password                                    | 一个集群内的成员必须有相同的密码                             |
| set hbdev "port1" 50 "port2" 50                 | 配置心跳接口。其中50为优先级，优先级高的被优先使用           |
| unset session-sync-dev                          | 可以配置专门的心跳接口用于会话信息同步，默认和控制信息为同一心跳接口 |
| set route-ttl 10                                | 路由转发表的存活时间。HA设备之间只同步转发表，不同步路由表。当一个备机被选举成主机后，其原有转发表的存活时间，单位秒。随后通过静态或动态路由协议生成转发表，继续工作 |
| set route-wait 0                                | 主设备收到新的路由条目后，会等待x秒后，再同步给从设备        |
| set route-hold 10                               | 主设备进行２次路由同步之间的间隔，防止路由震荡而造成反复更新路由 |
| set sync-config enable                          | 配置文件自动同步，需要开启                                   |
| set encryption disable                          | 是否允许使用AES-128和SHA1对心跳信息进行加密和完整性验证      |
| set authentication disable                      | 是否使用SHA1算法验证心跳信息的完整性                         |
| set hb-interval 2                               | 发送心跳数据包的间隔，单位为每100ms.如配置2,则每200ms发送一个心跳信息 |
| set hb-lost-threshold 6                         | 心跳信息连续丢失６个后则认为对方不再存活                     |
| set helo-holddown 20                            | Hello状态时间。设备加入HA机群前等待的时间，防止由于未能发现所有的机群成员而造成Ha的反复协商 |
| set arps 5                                      | 设备成为主设备后，要发送免费arp来宣布自己的MAC地址，以便相连的交换机能够及时更新MAC地址表，该参数用于配置其发送的数量 |
| set arps-interval 8                             | 发送免费arp的间隔，单位秒                                    |
| set session-pickup enable/disable               | 关闭或者开启会话同步，默认为disable。一般需要开启            |
| set session-pickup-delay {enable/disable}       | 仅对存活30秒以上的会话进行同步。开启后会对性能有所优化，但小于30秒的会话在HA切换的时候会丢失。默认为关闭，谨慎使用 |
| set link-failed-signal disable                  | 防火墙上发生被监控端口失效触发HA切换的时候，是否将除心跳口外的所有端口shutdown一秒钟的时间，便于与之相连的交换机及时更新MAC表。 |
| set uninterruptable-upgrade enable              | 是否允许无中断升级OS。系统自动分别对机群内的设备升级，并自动切换，不会造成业务的中断 |
| set ha-uptime-diff-margin 300                   | 当进行HA选举时，启动时间为一个选举的一个参数，当２台设备启动时间差小于300时则将该部分差异忽略，视为相同 |
| set override disable                            | 默认为disable,ＨＡ选举按如下顺序进行比较：有效接口数量>运行时间>HA优先级>设备序列号。Enable情况下，讯据顺序改变。有效接口数量> HA优先级>运行时间>设备序列号。每次设备加入或者离开机群，都会强制整个机群重新进行主设备的选举 |
| set priority 128                                | HA优先级，为便于管理，建议主设备200,从设备100                |
| set monitor port3 port4                         | 配置需要被监控的端口，其有效数量多的设备成为主设备           |
| unset pingserver-monitor-interface              | 是否设置pingserver监控端口                                   |
| set pingserver-failover-threshold 0             | pingserver触发的阀值，0则意味着任何的pingserver失效都会触发HA的切换 |
| set pingserver-flip-timeout 60                  | 两次pingserver失效切换之间的间隔。如A发生失效，切换到B. 切过去之后发现B也是失效的，则需要等待60分钟的时间允许切换回A |
| set ha-mgmt-status  enable HA的带外管理配置命令 | set ha-mgmt-status enable<br />set ha-mgmt-interface port1<br />set ha-mgmt-interface-gateway  x.x.x.x |
