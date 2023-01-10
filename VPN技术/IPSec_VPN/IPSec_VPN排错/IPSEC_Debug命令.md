# IPSEC Debug命令

开启IPSEC Debug

```
输出所有IPSEC协商信息
diagnose debug application ike -1
diagnose debug enable

如果有多个IPSEC，则使用filter过滤指定的IPSEC对端，以便查看
diagnose vpn ike log filter dst-addr4 x.x.x.x

清除过滤条件
diagnose vpn ike log filter clear
```

关闭IPSEC Debug

```
diagnose debug reset
diagnose debug disable 
```

