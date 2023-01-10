# Telnet/SSH管理

## 需求

当需要进设备的命令行进行配置或收集信息，但没有配置线，或不在设备旁边时，可通过远程telnet或则ssh对设备进行管理。

## 组网拓扑

<img src=".\..\..\images\image-20220817150957255.png" alt="image-20220817150957255" style="zoom: 50%;" />

## 配置要点

使用 telnet，ssh方式管理首先要保证管理主机到设备的接口地址的连通性是好的，可把接口的ping功能勾选，若能ping通管理接口则说明连通性没问题：

1. 开启接口 telnet、ssh功能。
2. telnet管理设备。
3. SSH管理设备。

## 操作步骤

1. 开启接口telnet，ssh管理功能，进入 "系统管理"--"网络"----"接口"， 编辑internal（双击internal接口）接口。如下图所示：

   <img src=".\..\..\images\image-20220817151436535.png" alt="image-20220817151436535" style="zoom: 50%;" />

2. 在管理方式中，勾选 SSH、TELNET 两项（接口的telnet功能默认是没有开启的）点击确定。

## 配置验证

### Telnet管理

1. 在管理端的电脑上，点击"开始"--"运行"--输入cmd--回车，在DOS 窗口内  输入：telnet 192.168.1.99（192.168.1.99为internal的接口地址），回车。

   <img src=".\..\..\images\image-20220817151804037.png" alt="image-20220817151804037" style="zoom: 80%;" />
   
   > 注意：Win7以上系统的电脑Telnet客户端默认为关闭状态，需要管理员手工开启才可以正常使用，点击电脑的"控制面板"--"程序"--"打开或关闭Windows功能"--勾选"Telnet客户端"，确认。

2. 输入用户名和密码（用户名密码与web登录的一致，密码输入不显示，输完回车即可）。

   <img src=".\..\..\images\image-20220817151915647.png" alt="image-20220817151915647" style="zoom: 80%;" />

### SSH管理

1. 使用终端软件(如SecureCRT)新建一个ssh连接 ，输入用户名密码（用户名密码与web登录的一致）对设备进行管理。以下以Secure CRT软件为例：打开Secure CRT软件，点击"文件"--"快速链接"。

   <img src=".\..\..\images\image-20220817152254939.png" alt="image-20220817152254939" style="zoom:150%;" />

2. 弹出如下框，协议选择ssh2,主机名输入设备的管理地址192.168.1.99（即internal对应的接口IP地址），端口号22，其他保持默认值：点击连接。

   <img src=".\..\..\images\image-20220817152336568.png" alt="image-20220817152336568" style="zoom: 150%;" />

3. 输入用户名admin（与web登录的用户名一致）：点击确定。

   <img src=".\..\..\images\image-20220817152413402.png" alt="image-20220817152413402" style="zoom:150%;" />

4. 输入口令密码默认为空（与web登录的密码一致）：点击确定。

   <img src=".\..\..\images\image-20220817152444305.png" alt="image-20220817152444305" style="zoom:150%;" />

5. 即可登录设备进行命令行管理配置。

   <img src=".\..\..\images\image-20220817152513341.png" alt="image-20220817152513341" style="zoom:150%;" />
