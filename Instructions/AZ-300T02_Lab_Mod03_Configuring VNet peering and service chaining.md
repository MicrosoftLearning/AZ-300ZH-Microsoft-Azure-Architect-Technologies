﻿# 配置和管理虚拟网络
# 实验：配置 VNet 对等互连和服务链
  
### 方案
  
ADatum 公司希望在其 Azure 订阅中实现 Azure 虚拟网络之间的服务链。 


### 目标
  
完成本实验室后，你将能执行以下操作：

- 使用 Azure 资源管理器模板部署 Azure VM。

- 配置 VNet 对等互连

- 实施路由

- 验证服务链


### 实验设置
  
预计用时：45 分钟

用户名：**学生**

密码：**Pa55w.rd**



## 练习 1：使用部署模板创建 Azure 实验环境
  
本练习的主要任务如下：

1. 使用 Azure资源管理器模板创建第一个 Azure 虚拟网络环境

1. 使用 Azure 资源管理器模板创建第二个 Azure 虚拟网络环境


#### 任务 1： 使用 Azure 资源管理器模板创建第一个 Azure 虚拟网络环境
  
1. 从实验虚拟机中，启动 Microsoft Edge 并浏览到 Azure 门户 [**http://portal.azure.com**](http://portal.azure.com)，然后使用在目标Azure订阅中具有所有者角色的 Microsoft 帐户登录。

1. 在 Azure 门户的 Microsoft Edge 窗口中，在 **Cloud Shell** 中启动 **Bash** 会话。 

1. 如果呈现**您未装载存储器**消息，请使用以下设置配置存储器：

    - 订阅：目标 Azure 订阅的名称

    - Cloud Shell 区域：在订阅中可用且最接近实验位置的 Azure 区域的名称

    - 资源组：新资源组的名称 **az3000400-LabRG**

    - 存储帐户：新存储帐户的名称

    - 文件共享： 新文件共享的名称

1. 在 Cloud Shell 窗格中，通过运行以下命令创建两个资源组（用订阅中可用且最接近实验位置的 Azure 区域的名称替换 `<Azure region>` 占位符）

   ```
   az group create --resource-group az3000401-LabRG --location <Azure region>
   ```

   ```
   az group create --resource-group az3000402-LabRG --location <Azure region>
   ```

1. 从 Cloud Shell 窗格中，将第一个 Azure 资源管理器模板 **\\allfiles\\AZ-300T02\\Module_03\\azuredeploy0401.json** 上载到主目录中。

1. 从 Cloud Shell 窗格中，将参数文件 **\\allfiles\\AZ-300T02\\Module_03\\azuredeploy04.parameters.json** 上载到主目录中。

1. 在 Cloud Shell 窗格中，通过运行以下命令将托管 Windows Server 2016 Datacenter 的两个 Azure VM 部署到第一个虚拟网络中：

   ```
   az deployment group create --resource-group az3000401-LabRG --template-file azuredeploy0401.json --parameters @azuredeploy04.parameters.json --no-wait
   ```

    > **注意**：不要等待部署完成，而是继续执行下一个任务。


#### 任务 2： 使用 Azure 资源管理器模板创建第二个 Azure 虚拟网络环境

1. 从 Cloud Shell 窗格中，将第二个 Azure 资源管理器模板 **\\allfiles\\AZ-300T02\\Module_03\\azuredeploy0402.json** 上载到主目录中。

1. 从 Cloud Shell 窗格中，通过运行以下命令将托管 Windows Server 2016 Datacenter 的 Azure VM 部署到第二个虚拟网络中：

   ```
   az deployment group create --resource-group az3000402-LabRG --template-file azuredeploy0402.json --parameters @azuredeploy04.parameters.json --no-wait
   ```

    > **注意**： 第二个模板使用同一参数文件。 

    > **注意**：不要等待部署完成，而是继续执行下一个练习。

> **结果**：完成本练习后，您应已创建用于托管运行 Windows Server 2016 Datacenter 的 Azure VM 的两个 Azure 虚拟网络。 


## 练习2： 配置 VNet 对等互连 
  
本练习的任务如下：

1. 为两个虚拟网络配置 VNet 对等互连

#### 任务 1：为两个虚拟网络配置 VNet 对等互连
  
1. 在显示Azure门户的 Microsoft Edge 窗口中，导航到 **az3000401-vnet** 虚拟网络边栏选项卡。

1. 从 **az3000401-vnet** 边栏选项卡中，创建具有以下设置的 VNet 对等互连：

    - 从第一个虚拟网络到第二个虚拟网络的对等互连的名称：**az3000401-vnet-to-az3000402-vnet**

    - 虚拟网络部署模型：**资源管理器**

    - 订阅：用于本实验的 Azure 订阅的名称

    - 虚拟网络：**az3000402-vnet**
    
    - 从第二个虚拟网络到第一个虚拟网络的对等互连的名称：**az3000402-vnet-to-az3000401-vnet**    
    
    - 允许从第一个虚拟网络到第二个虚拟网络的虚拟网络访问：**已启用**
    
    - 允许从第二个虚拟网络到第一个虚拟网络的虚拟网络访问：**已启用**    

    - 允许从第一个虚拟网络到第二个虚拟网络的转发流量：**已禁用**
    
    - 允许从第二个虚拟网络到第一个虚拟网络的转发流量：**已禁用**

    - 允许网关传输：已禁用

## 练习3：实施路由
  
本练习的主要任务如下：

1. 启用 IP 转发

1. 配置用户定义的路由 

1. 在运行 Windows Server 2016 的Azure VM上配置路由


#### 任务 1：启用 IP 转发 
  
1. 在 Microsoft Edge 中，导航到 **az3000401-nic2** 边栏选项卡（**az3000401-vm2** 的NIC）

1. 在 **az3000401-nic2** 边栏选项卡上，通过将 **IP 转发**设置为**已启用**来修改 **IP 配置**。


#### 任务 2：配置用户定义的路由 

1. 在 Azure 门户中创建具有以下参数的新路由表：

    - 名称： **az3000402-rt1**

    - 订阅：要用于本实验的订阅的名称

    - 资源组：**az3000402-LabRG**

    - 位置：您在其中创建虚拟网络的同一 Azure 区域
  
    - 虚拟网络网关路由传播：**已禁用**
    
    路由表创建完成后，单击 **“转到资源”**

1. 在 Azure 门户中，在上一步创建的路由表 az3000402-rt1 上单击 **“设置”** 下的 **“路由”**，并添加具有以下设置的路由： 

    - 路由名称： **custom-route-to-az3000401-vnet**

    - 地址前缀：**10.0.0.0/22**

    - 下一个跃点类型：**虚拟设备**

    - 下一个跃点地址：**10.0.1.4**

1. 在 Azure 门户中，将路由表与 **az3000402-vnet** 的 **subnet-1** 进行关联。


#### 任务 3：在运行 Windows Server 2016 的 Azure VM 上配置路由

1. 在实验计算机上，从 Azure 门户启动 **az3000401-vm2** Azure VM的远程桌面会话。 

1. 系统提示进行身份验证时，请指定以下凭据：

    - 用户名：**学生**

    - 密码：**Pa55w.rd1234**

1. 通过远程桌面会话连接到 az3000401-vm2 后，安装 **“远程访问”** 角色。在服务器管理器中，单击 **“管理”**，然后单击 **“添加角色和功能”**。

1. 在“开始之前”屏幕上，单击 **“下一步”**。

1. 在“安装类型”屏幕上，保留默认设置 **“基于角色或基于功能的安装”** 处于选中状态，然后单击 **“下一步”**。

1. 在“服务器选择”屏幕上，保持默认服务器处于选中状态，然后单击 **“下一步”**。

1. 在“服务器角色”屏幕上，选中 **“远程访问”** 旁边的复选标记，然后单击 **“下一步”**。

1. 在“功能”屏幕上，保留默认设置的选中状态，然后单击 **“下一步”*。

1. 在“远程访问”屏幕上，单击 **“下一步”**。

1. 在“服务”屏幕上，选中 **“路由”** 旁边的复选标记，然后单击 **“下一步”**。

1. 在“确认”屏幕上，单击 **“下一步”**。

1. 在“结果”屏幕上，单击 **“安装”**。

1. 在服务器管理器的 az3000401-vm2 的远程桌面会话中，单击 **“工具”**，然后单击 **“路由和远程访问”** 以打开 RRAS 控制台。 

1. 在 **“路由和远程访问”** 控制台中，右键单击服务器 az3000401-vm2 名称的下方，然后选择 **“配置并启用路由和远程访问”** 以运行 **“路由和远程访问服务器安装向导”**。

1. 在 **“路由和远程访问服务器安装向导”** 中，选择 **“配置”** 下的 **“自定义配置”** 并启用 **“LAN 路由”**。 

> **注意**：如果收到警告弹出窗口，请单击 **“确定”**。

1. 启动**路由和远程访问**服务。

1. 在 az3000401-vm2 的远程桌面会话中，单击“开始”，然后单击 **“Windows 管理工具”**。

1. 在管理工具中，启动 **“具有高级安全性的 Windows 防火墙”** 控制台。

1. 在控制台中，单击 **“入站规则”**。

1. 找到 **“文件和打印机共享(回显请求 - ICMPv4-In)”** 入站规则，右键单击该规则，然后单击 **“启用规则”**。

> **结果**：完成本练习后，你应已在第二个虚拟网络中配置了自定义路由。


## 练习 4：验证服务链
  
本次练习的主要任务如下：

1. 在 Azure VM 上配置高级安全 Windows 防火墙

1. 测试对等虚拟网络之间的服务链


#### 任务 1：在目标 Azure VM 上配置具有高级安全性的 Windows 防火墙
  
1. 在实验计算机上，从 Azure 门户启动 **az3000401-vm1** Azure VM 的远程桌面会话。 

1. 系统提示进行身份验证时，请指定以下凭据：

    - 用户名：**学生**

    - 密码：**Pa55w.rd1234**

1. 在 az3000401-vm1 的远程桌面会话中，启动**高级安全 Windows 防火墙**控制台并为所有配置文件启用**文件和打印机共享（回显请求 - ICMPv4-In）**入站规则。


#### 任务 2：测试对等虚拟网络之间的服务链接
  
1. 在实验计算机上，从 Azure 门户启动 **az3000402-vm1** Azure VM 的远程桌面会话。 

1. 系统提示进行身份验证时，请指定以下凭据：

    - 用户名：**学生**

    - 密码：**Pa55w.rd1234**

1. 通过远程桌面会话连接到 az3000402-vm1 后，启动 **Windows PowerShell**。

1. 在 **Windows PowerShell** 窗口中，运行以下命令：

   ```pwsh
   Test-NetConnection -ComputerName 10.0.0.4 -TraceRoute
   ```

1. 验证测试是否成功，并注意连接是否通过 10.0.1.4 进行了路由

>  **结果**：完成本练习后，您应已验证了对等虚拟网络之间的服务链。

## 练习 5：删除实验室资源

#### 任务 1：打开 Cloud Shell

1. 在门户顶部，单击 **Cloud Shell** 图标以打开 Cloud Shell 窗格。

1. 如果需要，请使用“Cloud Shell”窗格左上角的下拉列表切换到 Bash Shell 会话。

1. 在 **“Cloud Shell”** 命令提示符处，键入以下命令并按 **Enter** 键列出你在本实验中创建的所有资源组：

   ```
   az group list --query "[?starts_with(name,'az30004')]".name --output tsv
   ```

1. 验证输出中是否仅包含你在本实验室中创建的资源组。这些组将在下一个任务中删除。

#### 任务 2：删除资源组 

1. 在 **“Cloud Shell”** 命令提示符处，键入以下命令并按 **Enter** 键以删除你在本实验室中创建的资源组

   ```sh
   az group list --query "[?starts_with(name,'az30004')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

1. 关闭门户底部的 **“Cloud Shell”** 提示。

> **结果**：在本练习中，你删除了本实验室中使用的资源。