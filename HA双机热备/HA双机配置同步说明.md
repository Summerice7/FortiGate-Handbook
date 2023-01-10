# HA双机配置同步说明

## 同步配置说明

1. FGCP协议用联合的增量同步和周期性同步去确保所有HA Cluster中的成员与主设备的配置一致，下面的配置不会在cluster成员中同步：

   - 主机名

   - GUI 仪表盘中的微件

   - HA 中的override抢占

   - HA设备priority优先级

   - 虚拟cluster priority优先级

   - HA下的Ping server 优先级

   - HA专用管理接口的配置

   - HA专用管理接口的下的默认路由配置

2. 多数订阅服务和授权是不同步的，每一个FortiGate授权是独立的（在系统→FortiGuard中，License的到期时间只会显示较近到期的那台的时间，如果HA成员的某个License过期，那么所有HA成员的该License都会过期）。

   > FortiToken Mobile是一个例外，它们注册到主设备然后同步给备设备。

3. 主设备同步所有其它的配置给集群成员，包括HA下的一些配置。

4. 所有同步活动在HA hearbert心跳接口用TCP /703 和UDP/703。

### 禁用自动同步配置

1. 在一些情况下你可能想用下面的命令关掉自动同步主设备的配置到所有集群中的设备。

   ```
   config system ha
       set sync-config disable
   end
   ```

2. 当这个选项disable禁用后，集群不再同步配置变更。如果一个设备发生故障，新的主设备可能没有相同的配置和失败的主设备。结果，新的主设备可能处理会话不同或者不能以相同的方式工作。

3. 在绝大多数情况下不不应该disable禁用自动同步配置。然而，如果你disable 自动同步，你可以用execute ha synchronize 命令手工的同步配置。

4. 你必须在备设备上执行execute ha synchronize。用execute ha manage能切换到备设备上。

5. 例子：访问备设备然后执行HA同步，即便自动同步是禁用的。

   ```
   execute ha manage 0    //切到备设备上
   execute ha synchronize start    //执行HA同步
   ```

6. 你能用下面的命令停止在同步中的过程。

   ```
   execute ha synchronize stop
   ```

### 增量同步

1. 当你登录进集群用GUI或者CLI做配置变更时，你实际上登录在主设备上。
2. 所有的配置首先被做应用到主设备，增量同步然后立即同步这些变更给其它成员设备。
3. 当你登录备设备的CLI（例如使用execute ha manage切换登录进备设备），所有的在备设备做的配置变更也立即同步到所有集群成员，包括主设备。
4. 增量同步也同步其它动态的配置信息例如DHCP Server address lease 数据库，路由更新，IPSec SAs，MAC地址表 等等。
5. 只要在一个集群成员做配置变更，增量同步将通过心跳链接发送相同的信息到所有其它成员。
6. 运行在每一个成员HA同步进程接收配置变更并应用它。
7. HA同步进程通过输入CLI命令进行配置更改，该命令看起来像是由第一次进行配置更改的管理员输入的。
8. 同步静默的进行，没有日志记录HA同步活动。然而，当同步进程执行CLI命令，日志消息能被记录在成员设备，你能看到这些消息在成员设备如果你启用了事件日志并且日志级别设置为information 信息。
9. 你也能看到这些日志在主设备当你做了配置变更在成员设备上。

### 周期性同步

1. 增量同步确保管理员做的配置变更，这边配置变更依然是相同的在集群成员。

2. 然而，一些因素可能导致一个或多个集群成员与主设备不同步。例如，当增加一个新的成员到集群中，这个新的成员的配置没有匹配其它成员，使用增量同步配置变更对于新的成员来说是不现实的。
3. 周期性同步是一种寻找同步问题并修复它们的机制，集群每分钟都会将主设备的配置文件校验和与每个成员设备的配置文件校验和进行比较。
4. 如果所有备设备checksums校验和是与主设备一致，所有的集群成员认为是同步的。
5. 如果一个或多个备设备的checksums校验和与主设备不同，备设备的配置被认为out of sync 不同步与主设备，checksums校验和不同步的备设备每15秒检查一次，如果由于增量配置序列未完成而导致配置不同步，则需要重新检查。
6. 如果校验和在5次检查后不匹配，那么不同步的备设备将从主设备索要配置。
7. 然后，备设备重新加载其配置，并作为备设备以与主设备相同的配置继续运行。
8. 备设备的配置通过这种方式重置因为当备设备配置与主设备配置不同步时，没有有效的方法来确定配置的差异是什么并纠正它们。重置备设备配置成为重新同步备设备的最有效方法。
9. 同步要求所有集群成员运行在相同的FortiOS版本，如果一些成员运行在不同的版本，不稳定和同步错误有可能发生。

### 同步成功的Console上输出的消息

当集群最初形成时，或者当一个新成员作为从属成员添加到集群中时，CLI控制台上会出现以下消息，表明该成员加入了集群，并使其配置与主设备同步。

```
slave's configuration is not in sync with master's, sequence:0
slave's configuration is not in sync with master's, sequence:1
slave's configuration is not in sync with master's, sequence:2
slave's configuration is not in sync with master's, sequence:3
slave's configuration is not in sync with master's, sequence:4
slave starts to sync with master
logout all admin users
slave succeeded to sync with master
```

### 同步失败的Console上输出的消息

1. 如果您连接到与主设备不同步的备设备的console控制台，将显示类似如下的消息。

   ```
   slave is not in sync with master, sequence:0. (type 0x3)
   slave is not in sync with master, sequence:1. (type 0x3)
   slave is not in sync with master, sequence:2. (type 0x3)
   slave is not in sync with master, sequence:3. (type 0x3)
   slave is not in sync with master, sequence:4. (type 0x3)
   global compared not matched
   ```

2. 如果出现同步问题，Console控制台消息序列可能会一再重复。

3. 所有消息都包含一个类型值(type 0x3)。type值可以帮助诊断同步问题。

### HA对象消息和它们引用的配置对象不同步的参考

| 消息类型                               | 配置对象                              |
| -------------------------------------- | ------------------------------------- |
| HA_SYNC_SETTING_CONFIGURATION   = 0x03 | /data/config                          |
| HA_SYNC_SETTING_AV   = 0x10            |                                       |
| HA_SYNC_SETTING_VIR_DB   = 0x11        | /etc/vir                              |
| HA_SYNC_SETTING_SHARED_LIB   = 0x12    | /data/lib/libav.so                    |
| HA_SYNC_SETTING_SCAN_UNIT   = 0x13     | /bin/scanunitd                        |
| HA_SYNC_SETTING_IMAP_PRXY   = 0x14     | /bin/imapd                            |
| HA_SYNC_SETTING_SMTP_PRXY   = 0x15     | /bin/smtp                             |
| HA_SYNC_SETTING_POP3_PRXY   = 0x16     | /bin/pop3                             |
| HA_SYNC_SETTING_HTTP_PRXY   = 0x17     | /bin/thttp                            |
| HA_SYNC_SETTING_FTP_PRXY   = 0x18      | /bin/ftpd                             |
| HA_SYNC_SETTING_FCNI   = 0x19          | /etc/fcni.dat                         |
| HA_SYNC_SETTING_FDNI   = 0x1a          | /etc/fdnservers.dat                   |
| HA_SYNC_SETTING_FSCI   = 0x1b          | /etc/sci.dat                          |
| HA_SYNC_SETTING_FSAE   = 0x1c          | /etc/fsae_adgrp.cache                 |
| HA_SYNC_SETTING_IDS   = 0x20           | /etc/ids.rules                        |
| HA_SYNC_SETTING_IDSUSER_RULES   = 0x21 | /etc/idsuser.rules                    |
| HA_SYNC_SETTING_IDSCUSTOM   = 0x22     |                                       |
| HA_SYNC_SETTING_IDS_MONITOR   = 0x23   | /bin/ipsmonitor                       |
| HA_SYNC_SETTING_IDS_SENSOR   = 0x24    | /bin/ipsengine                        |
| HA_SYNC_SETTING_NIDS_LIB   = 0x25      | /data/lib/libips.so                   |
| HA_SYNC_SETTING_WEBLISTS   = 0x30      |                                       |
| HA_SYNC_SETTING_CONTENTFILTER   = 0x31 | /data/cmdb/webfilter.bword            |
| HA_SYNC_SETTING_URLFILTER   = 0x32     | /data/cmdb/webfilter.urlfilter        |
| HA_SYNC_SETTING_FTGD_OVRD   = 0x33     | /data/cmdb/webfilter.fgtd-ovrd        |
| HA_SYNC_SETTING_FTGD_LRATING   = 0x34  | /data/cmdb/webfilter.fgtd-ovrd        |
| HA_SYNC_SETTING_EMAILLISTS   = 0x40    |                                       |
| HA_SYNC_SETTING_EMAILCONTENT   = 0x41  | /data/cmdb/spamfilter.bword           |
| HA_SYNC_SETTING_EMAILBWLIST   = 0x42   | /data/cmdb/spamfilter.emailbwl        |
| HA_SYNC_SETTING_IPBWL   = 0x43         | /data/cmdb/spamfilter.ipbwl           |
| HA_SYNC_SETTING_MHEADER   = 0x44       | /data/cmdb/spamfilter.mheader         |
| HA_SYNC_SETTING_RBL   = 0x45           | /data/cmdb/spamfilter.rbl             |
| HA_SYNC_SETTING_CERT_CONF   = 0x50     | /etc/cert/cert.conf                   |
| HA_SYNC_SETTING_CERT_CA   = 0x51       | /etc/cert/ca                          |
| HA_SYNC_SETTING_CERT_LOCAL   = 0x52    | /etc/cert/local                       |
| HA_SYNC_SETTING_CERT_CRL   = 0x53      | /etc/cert/crl                         |
| HA_SYNC_SETTING_DB_VER   = 0x55        |                                       |
| HA_GET_DETAIL_CSUM   = 0x71            |                                       |
| HA_SYNC_CC_SIG   = 0x75                | /etc/cc_sig.dat                       |
| HA_SYNC_CC_OP   = 0x76                 | /etc/cc_op                            |
| HA_SYNC_CC_MAIN   = 0x77               | /etc/cc_main                          |
| HA_SYNC_FTGD_CAT_LIST   = 0x7a         | /migadmin/webfilter/ublock/ftgd/data/ |

### checksums校验和

1. 可以使用diagnose sys ha checksum show命令比较所有集群单元的配置校验和。该命令的输出显示了标记为global和all的校验和，以及每个VDOM(包括root VDOM)的校验和。可以使用get system ha-nonsync-csum命令显示类似信息（这个命令仅供FortiManager使用）。

2. 主设备和备设备用这个命令show出来的checksums校验和应该是相同的。如果他们不同你可以用 execute ha synchronize start命令做一下强制同步。

3. 下面命令的输出是主设备的（没有多个VDOM启用）。

   ```
   diagnose sys ha checksum show
   is_manage_master()=1, is_root_master()=1
   debugzone
   global: a0 7f a7 ff ac 00 d5 b6 82 37 cc 13 3e 0b 9b 77
   root: 43 72 47 68 7b da 81 17 c8 f5 10 dd fd 6b e9 57
   all: c5 90 ed 22 24 3e 96 06 44 35 b6 63 7c 84 88 d5
    
   checksum
   global: a0 7f a7 ff ac 00 d5 b6 82 37 cc 13 3e 0b 9b 77
   root: 43 72 47 68 7b da 81 17 c8 f5 10 dd fd 6b e9 57
   all: c5 90 ed 22 24 3e 96 06 44 35 b6 63 7c 84 88 d5
   ```

4. 下面命令的输出是在相同集群中的备设备的。

   ```
   diagnose sys ha checksum show
   is_manage_master()=0, is_root_master()=0
   debugzone
   global: a0 7f a7 ff ac 00 d5 b6 82 37 cc 13 3e 0b 9b 77
   root: 43 72 47 68 7b da 81 17 c8 f5 10 dd fd 6b e9 57
   all: c5 90 ed 22 24 3e 96 06 44 35 b6 63 7c 84 88 d5
    
   checksum
   global: a0 7f a7 ff ac 00 d5 b6 82 37 cc 13 3e 0b 9b 77
   root: 43 72 47 68 7b da 81 17 c8 f5 10 dd fd 6b e9 57
   all: c5 90 ed 22 24 3e 96 06 44 35 b6 63 7c 84 88 d5
   ```

5. 下面的输出是启用了VDOM（test 和Eng_vdm.）的主设备输出。

   ```
   config global
   diagnose sys ha checksum show
   is_manage_master()=1, is_root_master()=1
   debugzone
   global: 65 75 88 97 2d 58 1b bf 38 d3 3d 52 5b 0e 30 a9
   test: a5 16 34 8c 7a 46 d6 a4 1e 1f c8 64 ec 1b 53 fe
   root: 3c 12 45 98 69 f2 d8 08 24 cf 02 ea 71 57 a7 01
   Eng_vdm: 64 51 7c 58 97 79 b1 b3 b3 ed 5c ec cd 07 74 09
   all: 30 68 77 82 a1 5d 13 99 d1 42 a3 2f 9f b9 15 53
    
   checksum
   global: 65 75 88 97 2d 58 1b bf 38 d3 3d 52 5b 0e 30 a9
   test: a5 16 34 8c 7a 46 d6 a4 1e 1f c8 64 ec 1b 53 fe
   root: 3c 12 45 98 69 f2 d8 08 24 cf 02 ea 71 57 a7 01
   Eng_vdm: 64 51 7c 58 97 79 b1 b3 b3 ed 5c ec cd 07 74 09
   all: 30 68 77 82 a1 5d 13 99 d1 42 a3 2f 9f b9 15 53
   ```

6. 备设备输出。

   ```
   config global
   diagnose sys ha checksum show
   is_manage_master()=0, is_root_master()=0
   debugzone
   global: 65 75 88 97 2d 58 1b bf 38 d3 3d 52 5b 0e 30 a9
   test: a5 16 34 8c 7a 46 d6 a4 1e 1f c8 64 ec 1b 53 fe
   root: 3c 12 45 98 69 f2 d8 08 24 cf 02 ea 71 57 a7 01
   Eng_vdm: 64 51 7c 58 97 79 b1 b3 b3 ed 5c ec cd 07 74 09
   all: 30 68 77 82 a1 5d 13 99 d1 42 a3 2f 9f b9 15 53
    
   checksum
   global: 65 75 88 97 2d 58 1b bf 38 d3 3d 52 5b 0e 30 a9
   test: a5 16 34 8c 7a 46 d6 a4 1e 1f c8 64 ec 1b 53 fe
   root: 3c 12 45 98 69 f2 d8 08 24 cf 02 ea 71 57 a7 01
   Eng_vdm: 64 51 7c 58 97 79 b1 b3 b3 ed 5c ec cd 07 74 09
   all: 30 68 77 82 a1 5d 13 99 d1 42 a3 2f 9f b9 15 53
   ```

## 诊断HA out of sync 不同步问题

这部分描述怎样使用diagnose sys ha checksum show和diagnose debug命令诊断HA不同步消息。如果同步不成功，用下面的步骤在每一个成员执行发现问题。

### 确定为什么没有进行HA同步

1. 连接每一个设备通过console 口（或者SSH登录）。

2. 使用下面的debug命令启用诊断。

   ```
   diagnose debug enable
   diagnose debug console timestamp enable
   diagnose debug application hatalk -1
   diagnose debug application hasync -1
   ```

3. 收集debug输出信息，然后对比上个章节中的消息信息表格。

4. 关掉debug。

   ```
   diagnose debug disable
   diagnose debug reset
   ```

### 确定哪一部分配置导致的问题

1. 如果上面的步骤显示的消息中包含了同步对象0x30（例如，HA_SYNC_SETTING_CONFIGURATION = 0x03），则说明配置存在同步问题。使用以下步骤来确定导致问题的配置部分。

2. 如果您的集群由两个集群成员组成，则使用此过程捕获每个成员的配置校验和。如果您的集群包含两个以上的集群成员，则对所有返回包含0x30同步对象消息的集群成员重复此过程。

3. 连接每个成员通过console口（或者SSH命令行）。

4. 启用debug命令。

   ```
   diagnose debug enable
   ```

5. 停止HA同步。

   ```
   execute ha sync stop
   ```

6. 显示global下的checksum校验和。

   ```
   diagnose sys ha checksum show global
   ```

7. 把输出存为一个text文件。

8. 在所有集群设备重复这个过程。

9. 比较主设备的text文件和成员设备的，发现不匹配的部分。可以用一个文本对比器来对比。

10. 重复root vdom的校验和。

    ```
    diagnose sys ha checksum show root
    ```

11. 重复所有VDOM的校验和。

    ```
    diagnose sys ha checksum show <vdom-name>
    ```

12. 你也可以用grep选项仅仅显示部分的checksums。例如，仅仅显示和system相关的checksums在 root Vdom或者log相关的checksums在global下的，通常情况下第一个不匹配的校验和意味这个是导致不同问题的原因。

    ```
    diagnose sys ha checksum root | grep system
    diagnose sys ha chechsum global | grep log
    ```

13. 尝试删除或者改变这导致问题配置部分，你可以在主设备或者备设备上修改配置。

14. 然后用下面的命令重启开启HA同步，停掉debug。

    ```
    execute ha sync start
    diagnose debug disable
    diagnose debug reset
    ```

### 重新计算checksums校验和解决HA不同步问题

1. 有时，在集群计算校验和时可能会发生错误。由于这种计算错误，即使集群正常运行，CLI控制台也可能显示out of sync错误消息。有时，在diagnostic sys ha checksum命令的输出中也会出现校验和计算错误，原因是debug zone中的校验和与输出的校验和部分不匹配。

2. 解决这个问题的一个方法是重新计算校验和。重新计算的校验和应该匹配，不同步的错误消息应该停止出现。

3. 您可以使用以下命令重新计算HA校验和。

   ```
   diagnose sys ha checksum recalculate [<vdom-name> | global]
   ```

4. 仅仅输入不带选项的命令就会重新计算所有校验和。您可以指定一个VDOM名称来重新计算该VDOM的校验和。您也可以输入global来重新计算全局校验和。

### 确定导致配置同步问题的原因

有25个配置同步的FortiOS模块。要分析这么多数据，它是困难的找到导致同步问题的原因。您可以使用以下诊断命令更容易地找到可能导致同步问题的模块。

```
diagnose sys ha hasync-stats {all | most-recent [<seconds>] | by object [<number>]}
```

- all显示自hasync进程开始运行(通常是集群启动后)以来发生的所有模块的同步活动。

- most-recent [<seconds>]显示最近发生的同步事件。可以包括以秒为单位的时间，以显示在时间间隔内发生的最近事件。如果不包含秒数，该命令将显示最近5秒内的最新事件。此选项可用于确定当前正在同步或试图同步的一个或多个模块。如果当前没有正在同步的模块，该命令只显示最近的同步事件。

- by-object [<number>]显示指定模块的同步活动，<number>是模块号，取值范围是1 ~ 25。要显示所有25个模块及其编号，请输入：

- ```
  diagnose sys ha hasync-stats by-object ?
  ```

- 要显示最近的活动，输入：

  ```
  diagnose sys ha hasync-stats most-recent 10
  current-time/jiffies=2018-03-28 13:01:42/1148242:
  hasync-obj=2(arp):
     epoll_handler=1(ev_arp_handler): start=1522256500.354400(2018-03-28 13:01:40), end=1522256500.354406(2018-03-28 13:01:40), total=0.000006/1699
  hasync-obj=5(config):
      timer=0(check_sync_status), add=1141764(2018-03-28 13:01:26), expire=1142764(2018-03-28 13:01:36), end=1142764(25018-03-28 13:01:36), del=0(), total_call=1143
  hasync-obj=8(time):
     obj_handler=0(packet): start=1522256497.851550(2018-03-28 13:01:37), end=1522256497.851570(2018-03-28 13:01:37), total=0.000020/381
     timer=0(sync_timer), add=1140106(2018-03-28 13:01:10), expire=1143106(2018-03-28 13:01:40), end=1143106(2018-03-28 13:01:40), del=0(), total_call=381
  hasync-obj=21(hastats):
     obj_handler=0(packet): start=1522256499.760934(2018-03-28 13:01:39), end=1522256499.760936(2018-03-28 13:01:39), total=0.000002/2285
     timer=0(hastats_timer), add=1142556(2018-03-28 13:01:34), expire=1143056(2018-03-28 13:01:39), end=1143056(2018-03-28 13:01:39), del=0(), total_call=2286
  ```

- 该输出的最后几行显示了hastats模块的活动，该模块是模块21。您可以使用以下命令查看更多关于此模块的同步活动的信息:

  ```
  diagnose sys ha hasync-stats by-object 21
  ```



