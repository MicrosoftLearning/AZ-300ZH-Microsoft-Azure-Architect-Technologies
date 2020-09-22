# 实施高级虚拟网络
# 实验室：实施 Azure 负载均衡器标准
  
### 方案
  
Adatum Corporation 希望实施 Azure 负载均衡器标准来引导 Azure VM 的入站和出站流量。 


### 目标
  
完成本实验室后，你将能执行以下操作：

-  使用 Azure 负载均衡器标准实现入站负载平衡 

-  使用 Azure 负载均衡器标准配置出站 SNAT 流量 

### 实验室设置
  
预计用时：45 分钟

用户名：**学生**

密码：**Pa55w.rd**


## 练习 1：使用 Azure 负载均衡器标准实施入站负载均衡和 NAT 
  
本练习的主要任务如下：

1. 使用 Azure 资源管理器模板在可用性集中部署 Azure VM

1. 创建 Azure 负载均衡器标准的实例

1. 创建 Azure 负载均衡器标准的负载均衡规则

1. 创建 Azure 负载均衡器标准的 NAT 规则

1. Azure 负载均衡器标准的测试功能


#### 任务 1： 使用 Azure 资源管理器模板在可用性集中部署 Azure VM

1. 从实验虚拟机启动 Microsoft Edge，并浏览到 Azure 门户 [**http://portal.azure.com**](http://portal.azure.com)，并使用在目标 Azure 订阅中具有所有者角色的 Microsoft 帐户登录。

1. 在 Azure 门户的 Microsoft Edge 窗口中，在 **Cloud Shell** 内启动 **Bash** 会话。 

1. 如果显示**你没有安装存储**消息，则使用以下设置配置存储：

    - 订阅：目标 Azure 订阅的名称

    - Cloud Shell 区域：订阅中可用的 Azure 区域的名称，该区域最接近实验室位置

    - 资源组：新资源组的名称 **“az3000800-LabRG”**

    - 存储帐户：新存储帐户的名称

    - 文件共享： 新文件共享的名称

1. 在 Cloud Shell 窗格中，通过运行以下命令创建资源组（用订阅中可用且最接近实验位置的 Azure 区域的名称替换 `<Azure region>` 占位符）

   ```
   az group create --name az3000801-LabRG --location <Azure region>
   ```

1. 在 Cloud Shell 窗格中，上载 Azure 资源管理器模板 **\\allfiles\\AZ-300T03\\Module_03\\azuredeploy0801.json** 进入主目录。

1. 在 Cloud Shell 窗格中，上载参数文件 **\\allfiles\\AZ-300T03\\Module_03\\azuredeploy0801.parameters.json** 进入主目录。

1. 在 Cloud Shell 窗格中，通过运行以下命令部署一对托管 Windows Server 2016 Datacenter 的 Azure VM：

   ```
   az deployment group create --resource-group az3000801-labRG --template-file azuredeploy0801.json --parameters @azuredeploy0801.parameters.json
   ```

    > **注意**：等待部署完成，然后再继续进行下一个任务。该操作需要约 10 分钟。

1. 在 Azure 门户中，关闭 Cloud Shell 窗格。


#### 任务 2：创建 Azure 负载均衡器标准的实例

1. 在 Azure 门户中，使用以下设置创建一个新的 Azure 负载均衡器：

    - 订阅：目标 Azure 订阅的名称

    - 资源组：**az3000801-LabRG**
    
    - 名称： **az3000801-lb**

    - 区域：在本练习的上一个任务中部署 Azure VM 的 Azure 区域的名称
    
    - 类型：**公共**

    - SKU： **标准**

    - 公共 IP 地址： **新建**名为 **az3000801-lb-pip01**

    - 可用区域： **Zone-redundant**


#### 任务 3：创建 Azure 负载均衡器标准的负载均衡规则

1. 在 Azure 门户中，导航到显示新部署的 Azure 负载均衡器 **az3000801-lb** 的属性的边栏选项卡。

1. 在 **az3000801-lb** 边栏选项卡上，单击**警报规则**。

1. 在 **az3000801-lb - 后端池**边栏选项卡中，单击**+添加**。

1. 在**添加后端池**边栏选项卡上，指定以下设置并单击**添加**：

    - 名称： **az3000801-bepool**

    - 虚拟网络：**az3000801-vnet (2 VM)**

    - 虚拟机：**az3000801-vm0** IP 地址： **ipconfig1（10.0.0.4）**或 **ipconfig1（10.0.0.5）**

    - 虚拟机：**az3000801-vm1** IP 地址： **ipconfig1（10.0.0.5）**或 **ipconfig1（10.0.0.4）**

    > **注意**：可能按照相反的顺序分配虚拟机的 IP 地址。 

    > **注意**：等待部署完成。所需的时间应该不超过 1 分钟。

1. 返回 **az3000801-lb - 后端池**边栏选项卡，单击**运行状况探测**。

1. 在 **az3000801-lb - 运行状况探测**边栏选项卡中，单击**+添加**。

1. 在**添加健康探针**边栏选项卡上，指定以下设置并单击**确定**：

    - 名称： **az3000801-healthprobe**

    - 协议：**TCP**

    - 端口： **80**

    - 间隔：**5**

    - 运行不正常阈值：**2**

    > **注意**：等待部署完成。所需的时间应该不超过 1 分钟。

1. 返回 **az3000801-lb - 运行状况探测**边栏选项卡中，单击**负载均衡规则**。

1. 在 **az3000801-lb - 负载均衡规则**边栏选项卡中，单击**+添加**。

1. 在**添加负载均衡规则**边栏选项卡上，指定以下设置并单击**确定**：

    - 名称： **az3000801-lbrule01**

    - IP 版本： **IPv4**

    - 前端 IP 地址：从下拉列表中选择分配给 **LoadBalancedFrontEnd** 的公共 IP 地址

    - 协议：**TCP**

    - 端口： **80**

    - 后端端口： **80**

    - 后端池： **az3000801-bepool（2 台虚拟机）**

    - 运行状况探测：**az3000801-healthprobe (TCP:80)**

    - 会话持久性：**无**

    - 空闲超时（分钟）：**4**

    - 浮动 IP（直接服务器返回）：**禁用**

    > **注意**：等待部署完成。所需的时间应该不超过 1 分钟。


#### 任务 4：创建 Azure 负载均衡器标准的 NAT 规则

1. 在 Azure 门户中，在 **az3000801-lb** 边栏选项卡中单击**入站 NAT 规则**。

1. 在 **az3000801-lb - 入站 NAT 规则**边栏选项卡上，单击**+添加**。

1. 在**添加入站 NAT 规则**边栏选项卡上，指定以下设置并单击**确定**：

    - 名称： **az3000801-vm0-RDP**

    - 前端 IP 地址：从下拉列表中选择分配给 **LoadBalancedFrontEnd** 的公共 IP 地址

    - IP 版本： **IPv4**

    - 服务：**RDP**

    - 协议：**TCP**

    - 端口： **33890**

    - 目标虚拟机：**az3000801-vm0**

    - 网络 IP 配置： **ipconfig1 (10.0.0.4)** 或 **ipconfig1 (10.0.0.5)**

    - 端口映射： **自定义**

    - 浮动 IP（直接服务器返回）：**禁用**

    - 目标端口：**3389**

    > **注意**：等待部署完成。所需的时间应该不超过 1 分钟。

1. 返回 **az3000801-lb - 入站 NAT 规则**边栏选项卡，单击**+添加**。

1. 在**添加入站 NAT 规则**边栏选项卡上，指定以下设置并单击**确定**：

    - 名称： **az3000801-vm1-RDP**

    - 前端 IP 地址：从下拉列表中选择分配给 **LoadBalancedFrontEnd** 的公共 IP 地址

    - IP 版本： **IPv4**

    - 服务：**RDP**

    - 协议：**TCP**

    - 端口： **33891**

    - 目标虚拟机：**az3000801-vm1**

    - 网络 IP 配置： **ipconfig1 (10.0.0.5)** 或 **ipconfig1 (10.0.0.4)**

    - 端口映射： **自定义**

    - 浮动 IP（直接服务器返回）：**禁用**

    - 目标端口：**3389**

    > **注意**：等待部署完成。所需的时间应该不超过 1 分钟。


#### 任务 5：Azure 负载均衡器标准的测试功能

1. 在 Azure 门户中，导航到 **az3000801-lb** 边栏选项卡，注意**公共 IP 地址**条目的值。

1. 在实验室计算机上，启动 Microsoft Edge 并导航到你在上一步中确定的 IP 地址。

1. 验证你是否看到默认 **Internet Information Services Welcome** 页面。 

1. 在实验计算机上，右键单击**开始**，单击**运行**，然后从**打开**文本框中运行以下命令（用你在本任务中先前确定的公共 IP 地址替换 `<IP address>` 占位符）：

   ```
   mstsc /v:<IP address>:33890
   ```

1. 系统提示时，指定以下值进行身份验证：

    - 用户名：**学生**

    - 密码：**Pa55w.rd1234**

1. 在远程桌面会话中，切换到服务器管理器窗口中的**本地服务器**视图，验证你是否连接到 **az3000801-vm0** Azure VM。

1. 切换到实验计算机，右键单击**开始**，单击**运行**，然后从**打开**文本框运行以下命令（用你在本任务中先前确定的 IP 地址替换 `<IP address>` 占位符）：

   ```
   mstsc /v:<IP address>:33891
   ```

1. 系统提示时，指定以下值进行身份验证：

    - 用户名：**学生**

    - 密码：**Pa55w.rd1234**

1. 在远程桌面会话中，切换到服务器管理器窗口中的**本地服务器**视图，验证你是否连接到 **az3000801-vm1** Azure VM。

1. 在远程桌面会话中，启动 Windows PowerShell 会话并运行下列命令以确定当前的公共 IP 地址：

   ```pwsh
   Invoke-RestMethod http://ipinfo.io/json 
   ```

1. 检查 cmdlet 的输出，并验证 IP 地址条目是否与你在本任务中先前确定的公共 IP 地址匹配。

1. 将远程桌面会话保持在打开状态。你将在下一个练习中用到它们。



> **结果**：完成本练习后，你已实施并测试 Azure 负载均衡器标准入站负载均衡和 NAT 规则


## 练习2：使用 Azure 负载均衡器标准配置出站 SNAT 流量
  
本练习的主要任务如下：

1. 使用 Azure 资源管理器模板将 Azure VM 部署到现有虚拟网络中

1. 创建 Azure 标准负载均衡器并配置出站 SNAT 规则

1. 测试 Azure 标准负载均衡器的出站规则


#### 任务 1： 使用 Azure 资源管理器模板将 Azure VM 部署到现有虚拟网络中

1. 从实验虚拟机启动 Microsoft Edge，并浏览到 Azure 门户 [**http://portal.azure.com**](http://portal.azure.com)，并使用在目标 Azure 订阅中具有所有者角色的 Microsoft 帐户登录。

1. 在 Azure 门户的 Microsoft Edge 窗口中，在 **Cloud Shell** 内启动 **Bash** 会话。 

1. 在 Cloud Shell 窗格中，上载 Azure 资源管理器模板 **\\allfiles\\AZ-300T03\\Module_03\\azuredeploy0802.json** 进入主目录。

1. 在 Cloud Shell 窗格中，上载参数文件 **\\allfiles\\AZ-300T03\\Module_03\\azuredeploy0802.parameters.json** 进入主目录。

1. 在 Cloud Shell 窗格中，通过运行以下命令部署一对托管 Windows Server 2016 Datacenter 的 Azure VM：

   ```
   az deployment group create --resource-group az3000801-labRG --template-file azuredeploy0802.json --parameters @azuredeploy0802.parameters.json
   ```

    > **注意**：等待部署完成，然后再继续进行下一个任务。该操作需要约 5 分钟。

1. 在 Azure 门户中，关闭 Cloud Shell 窗格。


#### 任务 2：创建 Azure 标准负载均衡器并配置出站 SNAT 规则

1. 在 Azure 门户的 Microsoft Edge 窗口中，在 **Cloud Shell** 内启动 **Bash** 会话。 

1. 在 Azure 门户中，从 Cloud Shell 窗格中运行下列命令以创建负载均衡器的出站公共 IP 地址：

   ```
   az network public-ip create --resource-group az3000801-LabRG --name az3000802-lb-pip01 --sku standard
   ```

1. 在 Azure 门户中，从 Cloud Shell 窗格中运行下列命令以创建 Azure 负载均衡器标准：

   ```sh
   LOCATION=$(az group show --name az3000801-LabRG --query location --out tsv)
   az network lb create --resource-group az3000801-LabRG --name az3000802-lb --sku standard --backend-pool-name az3000802-bepool --frontend-ip-name loadBalancedFrontEndOutbound --location $LOCATION --public-ip-address az3000802-lb-pip01
   ```

1. 在 Cloud Shell 窗格中，运行下列命令以创建出站规则：

   ```
   az network lb outbound-rule create --resource-group az3000801-LabRG --lb-name az3000802-lb --name outboundRuleaz30000802 --frontend-ip-configs loadBalancedFrontEndOutbound --protocol All --idle-timeout 15 --outbound-ports 10000 --address-pool az3000802-bepool
   ```

    > **注意**：等待部署完成。所需的时间应该不超过 1 分钟。

1. 关闭 Cloud Shell 窗格。

1. 在 Azure 门户中，导航到显示 Azure 负载均衡器 **az3000802-lb** 的属性的边栏选项卡。

1. 在 **az3000802-lb** 边栏选项卡，单击**警报规则**。

1. 在 **az3000802-lb - 后端池**边栏选项卡，单击 **az3000802-bepool**。

1. 在 **az3000802-bepool** 边栏选项卡上，指定以下设置并单击**保存**：

    - 虚拟网络：**az3000801-vnet (4 VM)**

    - 虚拟机：**az3000802-vm0** IP 地址： **ipconfig1（10.0.1.4）** 或 **ipconfig1（10.0.1.5）**

    - 虚拟机：**az3000802-vm1** IP 地址： **ipconfig1（10.0.1.5）** 或 **ipconfig1（10.0.1.4）**

    > **注意**：等待部署完成。所需的时间应该不超过 1 分钟。


#### 任务 3：验证出站规则是否生效
 
1. 在 Azure 门户中，导航到 **az3000802-lb** 边栏选项卡，注意**公共 IP 地址**条目的值。

1. 在实验计算机上，从 **az3000801-vm0** 的远程桌面会话运行下列命令以启动 **az3000802-vm0** 的远程桌面会话。

   ```
   mstsc /v:az3000802-vm0
   ```

1. 系统提示时，指定以下值进行身份验证：

    - 用户名：**学生**

    - 密码：**Pa55w.rd1234**

1. 在 **az3000802-vm0** 的远程桌面会话中，启动 Windows PowerShell 会话并运行下列命令以确定你当前的公共 IP 地址：

   ```pwsh
   Invoke-RestMethod http://ipinfo.io/json 
   ```

1. 检查 cmdlet 的输出，并验证 IP 地址条目是否与你在本任务中先前确定的公共 IP 地址匹配。


> **结果**：完成本练习后，你已配置并测试 Azure 负载均衡器标准出站规则 


## 练习 3：删除实验室资源

#### 任务 1：打开 Cloud Shell

1. 在门户顶部，单击 **Cloud Shell** 图标以打开“Cloud Shell”窗格。

1. 如果需要，请使用“Cloud Shell”窗格左上角的下拉列表切换到 Bash Shell 会话。

1. 在 **“Cloud Shell”** 命令提示符处，键入以下命令并按 **Enter** 键列出你在本实验中创建的所有资源组：

   ```
   az group list --query "[?starts_with(name,'az30008')]".name --output tsv
   ```

1. 验证输出中是否仅包含你在本实验室中创建的资源组。这些组将在下一个任务中删除。

#### 任务 2：删除资源组

1. 在 **“Cloud Shell”** 命令提示符处，键入以下命令并按 **Enter** 键以删除你在本实验室中创建的资源组

   ```sh
   az group list --query "[?starts_with(name,'az30008')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

1. 关闭门户底部的 **“Cloud Shell”** 提示。

> **结果**：在本练习中，你删除了本实验室中使用的资源。
