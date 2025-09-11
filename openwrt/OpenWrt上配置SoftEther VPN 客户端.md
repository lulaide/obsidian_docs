---
source: "https://openwrt.org/docs/guide-user/services/vpn/softethervpn/client"
tags:
  - "openwrt"
---
1. 登录到 LuCI
2. 前往“系统” → “软件”
3. 点击“更新列表...”
4. 过滤列表以查找“softether”
5. 安装“softethervpn5-libs”
6. 安装“softethervpn5-client”
7. 安装“luci-app-softether”（在此时，LuCI 界面非常有限，算是可选）
8. 重启路由器

*注意：如果您有一台 Windows 电脑，可以使用远程客户端管理器（在开始菜单中选择“管理远程计算机的 SoftEther VPN 客户端”）通过 GUI 设置所有内容，前提是在命令行客户端管理中发出“RemoteEnable”命令。也可以通过 SCP 将现有的配置文件放置到相应位置。*

这里的指南将展示如何使用 CLI/ SSH 进行配置，您需要输入以下命令：

```bash
vpncmd
```
- 将出现以下提示：
```bash
By using vpncmd program, the following can be achieved. 
1. Management of VPN Server or VPN Bridge 
2. Management of VPN Client
3. Use of VPN Tools (certificate creation and Network Traffic Speed Test Tool)
Select 1, 2 or 3:
```
- 输入“2”并按“Enter”确认
- 将出现以下提示：
```bash
Specify the host name or IP address of the computer that the destination VPN Client is operating on. 
If nothing is input and Enter is pressed, connection will be made to localhost (this computer).
Hostname of IP Address of Destination:
```
- 只需按“Enter”（因为您想管理本地主机）
- 将出现以下提示：
```bash
Connected to VPN Client "localhost".
VPN Client>
```
- 通过发出以下命令创建一个 VPN 网络设备（用您选择的名称替换 ）
```bash
NicCreate <devName>
```
- 通过发出以下命令配置 VPN 连接（用您选择的名称替换 ）
```bash
AccountCreate <accountName>
```
- 现在将提示您输入多个连接参数。请用相关信息填写 <placeholders> - 这些信息将取决于您如何配置服务器，除了您上面选择的 <devName>。


```bash
Destination VPN Server Host Name and Port Number: <server address or IP>:<server port>
Destination Virtual Hub Name: <server virtual hub>
Connecting User Name: <user name>
Used Virtual Network Adapter Name: <devName>
```

- 发出以下命令以完成配置：
```bash
AccountPasswordSet <accountName>
```
- 按照提示配置 VPN 连接的用户密码（同样取决于您如何配置服务器；对于标准配置，您希望在最后的提示中选择“standard”）
- 使用 Ctrl+C 退出 VPNcmd 环境。

1. 登录到 LuCI
2. *首先，您需要设置一个合适的接口：*
	1. 前往“网络” → “接口”
	2. 点击“添加新接口...”
	3. 在“名称”中，选择并输入一个 <ifName>（例如：“ VPN ”）
	4. 在“协议”中，选择“ DHCP 客户端”
	5. 在“设备”中，选择以“vpn\_<devName>”命名的以太网适配器（在第 2 部分中选择的名称）
	6. 点击“创建接口”
	7. 转到“高级设置”选项卡
	8. 禁用“使用默认网关”
	9. 禁用“委派 IPv6 前缀”
	10. 转到“防火墙设置”选项卡
	11. 从下拉菜单中选择“wan”
	12. 点击“保存”
	13. 点击“保存并应用”
3. *在我的设置中，以下步骤是必要的以使其正常工作，但这可能是由于服务器端的问题*
	1. 转到“设备”选项卡
	2. 对于“vpn\_<devName>”，点击“配置”
	3. 禁用“启用 IPv6 ”复选框
	4. 点击“保存”
	5. 点击“保存并应用”
4. *最后，您只需设置路由。我的设置在这里显示的特定静态路由下运行良好（即，仅在特定连接中使用 VPN ）；不过，我还没有成功实现 VPN 作为默认路由。*
	1. 前往“网络” → “静态路由”
	2. 点击“添加...”
	3. 在“接口”中，选择 <ifName>（之前创建的）
	4. 在“目标”中，指定您希望进行 VPN 流量路由的远程 IP
	5. 在“子网掩码”中，指定上述地址的远程 IP 范围
	6. 对于“网关”，指定 VPN 服务器网关 IP 。这将取决于您如何设置 VPN 服务器端 DHCP （例如，通过 Softether VPN 服务器的 SecureNAT，在这种情况下，默认值我认为是 192.168.30.1）。
	7. 点击“保存”
	8. 点击“保存并应用”
5. 重启路由器

*注意：如果您使用的是 Windows 电脑，您可以再次使用远程客户端管理器（在开始菜单中选择“管理远程计算机的 SoftEther VPN 客户端”）来进行这些操作。*

本指南将再次使用 CLI/ SSH ，您需要输入以下命令：

```bash
vpncmd
```
- 将出现以下提示：
```bash
By using vpncmd program, the following can be achieved. 
1. Management of VPN Server or VPN Bridge 
2. Management of VPN Client
3. Use of VPN Tools (certificate creation and Network Traffic Speed Test Tool)
Select 1, 2 or 3:
```
- 输入“2”并按“Enter”确认
- 将出现以下提示：
```bash
Specify the host name or IP address of the computer that the destination VPN Client is operating on. 
If nothing is input and Enter is pressed, connection will be made to localhost (this computer).
Hostname of IP Address of Destination:
```
- 只需按“Enter”（因为您想管理本地主机）
- 将出现以下提示：
```bash
Connected to VPN Client "localhost".
VPN Client>
```
- **要启动 VPN** ，请执行以下命令（将 替换为您在步骤 2 中选择的内容）
```bash
AccountConnect <accountName>
```
- **要停止 VPN** ，请执行以下命令（将 替换为您在步骤 2 中选择的内容）
```bash
AccountConnect <accountName>
```
- **要在启动时自动启动 VPN** ，请执行以下命令（将 替换为您在步骤 2 中选择的内容）
```bash
AccountStartupSet <accountName>
```
- 要稍后禁用自动启动，请发出以下命令（将 <accountName> 替换为您在第 2 步中选择的名称）
```bash
AccountStartupRemove <accountName>
```
- 使用 Ctrl+C 退出 VPNcmd 环境。

*注意：不幸的是，traceroute 在我运行 VPN 时无法正常工作。不过，您可以通过设置静态路由到 IP 地理位置服务器或类似的方式来确认路由，并以此进行检查。*

如果您安装了“luci-app-softether”包，可以在 LuCI 的系统 → Softether 中检查连接状态。如果您有一台 Windows PC，可以使用远程客户端管理器来进行此操作。或者您也可以再次使用 vpncmd（请参考官方文档）。

---
