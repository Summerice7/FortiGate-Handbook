# FortiGuard服务介绍

## 服务介绍

1. 设备服务是需要注册才会生效的（当然有特例，就是设备购买的前三个月，可以免费的试用所有的服务，而通常这些服务都是需要购买并且注册才生效的）。
2. 注册分为设备注册、服务注册（将服务绑定到固定的设备上，详细可见后续的设备注册、服务注册章节）。
3. 注册地址：https://support.fortinet.com
4. 设备注册指南：https://fortinet-public.s3.cn-north-1.amazonaws.com.cn/Fortinet_TAC_Doc/%E8%AE%BE%E5%A4%87%E6%B3%A8%E5%86%8C%E6%AD%A5%E9%AA%A4.pdf
5. 服务注册指南：https://fortinet-public.s3.cn-north-1.amazonaws.com.cn/Fortinet_TAC_Doc/%E6%9C%8D%E5%8A%A1License%E6%B3%A8%E5%86%8C.pdf

## 服务分类

1. FortiCare支持服务
   - RMA硬件返修服务
   - 400电话/邮件等等技术支持服务
2. 固件和通用更新
   - 设备固件下载与更新服务
   - APP Control特征库更新
   - 设备识别库更新
   - Internet service DateBase库更新

	> 以上只需要有基础的FortiCare服务即可等到以上所有服务更新。

3. 入侵防御（IPS特征库更新）
   - IPS特征库
   - IPS引擎
   - 恶意URL
   - 僵尸网络IP库
   - 僵尸网络域名库
4. 反病毒（AV特征库更新）
   - AV特征库
   - AV引擎
   - 移动端恶意软件
5. Web过滤（URL分类过滤）
   - FortiGuard URL分类
   - 列入黑名单的网址数字证书
6. 工业安全数据库（Industrial DB）
7. FortiGate Cloud/FortiSandbox Cloud（有一定空间的免费许可）
8. Anti-Spam过滤

	> 以上的NGFW类的特征服务，需要购买相应的license服务许可才可以正常的使用（设备开箱启动可以免费全功能使用3个月试用），具体的Bondle价格等等需要通过供应商处获取。关于特征库的更新和研究的网站：http://www.fortiguard.com/

9. 虚拟域许可（默认有10个VDOM的license许可）
10. 其他许可，不同的软件版本，以上内容不尽相同，但是都大致有上述的模块和内容，具体不一样的版本可能存在一些差异，以上基于FortiOS 7.0.6进行的说明。

## 服务查看

1. 通过Support网站登陆后查看：https://support.fortinet.com/asset/#/views/products 查看设备服务状态，注册设备之后才可以查看。

   <img src="..\..\images\image-20220907153605459.png" alt="image-20220907153605459" style="zoom:50%;" />

   <img src="..\..\images\image-20220907154308763.png" alt="image-20220907154308763" style="zoom:50%;" />

   <img src="..\..\images\image-20220907154358805.png" alt="image-20220907154358805" style="zoom:50%;" />

2. 通过FortiGate管理页面查看（FortiGate的服务是通过互联网进行更新，因此FortiGate需要可以连接到互联网，才可以正常的自动更新），正常情况下，网站的服务和设备上的服务将会是同步的状态。

   <img src="..\..\images\image-20220907155911915.png" alt="image-20220907155911915" style="zoom:50%;" />

## 常见问题

1. FortiGate设备服务无法正常更新。
   - FGT无法连接到互联网，包括FGT无法上网、DNS异常解析、运营商阻止了FortiGuard更新的目的端口，FGT的某些源端口工作异常。
   - 修改了FGT的DNS为内部DNS，不是FortiGuard默认的DNS，这样可能引起DNS解析FortiGuard的相关域名存在异常（服务器在国外）。
   - 服务没有在网站上注册，先到网站上确认设备服务状态。
   - 有HA的环境下，只会显示服务到期时间最近的那台设备的服务时间，如果其中有一台设备过期，那么HA中的其他设备均会显示过期。
   - FortiGate默认为2小时更新一次，有时候不是立即就生效的，需要等一等，或者手工更新一下，手工命令如下：execute update-now。
   - 拥有多个VDOM，而可以上互联网的不是管理VDOM（默认是管理VDOM是root），这样也会导致更新失败，修改管理VDOM为可以上网的VDOM，或者让root VDOM可以上互联网。
2. 排错思路和方法：
   - Troubleshooting Tip: Diagnosing FortiGuard problems of Antivirus, Intrusion Prevention, Web Filtering, Spam Filtering: https://community.fortinet.com/t5/FortiGate/Troubleshooting-Tip-Diagnosing-FortiGuard-problems-of-Antivirus/ta-p/194121
   - Technical Tip: Verifying and troubleshooting FortiGuard updates status and versions: https://community.fortinet.com/t5/FortiGate/Technical-Tip-Verifying-and-troubleshooting-FortiGuard-updates/ta-p/194931
   - Technical Note: FortiGuard Support licenses show as expired: https://community.fortinet.com/t5/FortiGate/Technical-Note-FortiGuard-Support-licenses-show-as-expired/ta-p/194093
3. FGT无法连接互联网的情况下，如何更新特征库？
   - 手工导入特征库，GUI导入特征库或者命令行导入特征库，特征库可以在https://support.fortinet.com/Download/AvNidsDownload.aspx 下载离线版本特征库（需要登陆Support帐号）。
   - 统一通过FortiManger作为FortiGuard的中转站，需要FortiManger可以上互联网即可（可以使用代理方式），FortiGate无需上互联网，FGT将会通过FMG更新服务和特征库。
