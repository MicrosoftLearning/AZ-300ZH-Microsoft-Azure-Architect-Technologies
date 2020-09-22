# 管理标识

# 实验室：为 Azure 资源实施用户分配的托管标识

### 方案

Adatum Corporation 希望使用托管标识对 Azure VM 中运行的应用程序进行身份验证

### 目标

完成本实验室后，你将能执行以下操作：

- 创建和配置用户分配的托管标识

- 验证用户分配的托管标识的功能

### 实验室设置

预计用时：30 分钟

用户名：**学生**

密码：**Pa55w.rd**

## 练习 1：创建和配置用户分配的托管标识。

本练习的主要任务如下：

1. 部署运行 Windows Server 2016 Datacenter 的 Azure VM

1. 创建用户分配的托管标识。

1. 将用户分配的托管标识分配给 Azure VM。

1. 为用户分配的托管标识授予基于 RBAC 的权限。

#### 任务 1：部署运行 Windows Server 2016 Datacenter 的 Azure VM

1. 从实验室虚拟机启动 Microsoft Edge 并浏览 Azure 门户网站 [** http://portal.azure.com**](http://portal.azure.com)，使用在目标 Azure 订阅中具有所有者角色的 Microsoft 帐户登录。

1. 在 Azure 门户的 Microsoft Edge 窗口中，启动** Cloud Shell** 内的 **Bash** 会话。

1. 如果呈现**你未装载存储器**消息，请使用以下设置配置存储器：

   - 订阅：目标 Azure 订阅的名称

   - Cloud Shell 区域：订阅中可用的 Azure 区域的名称，该区域最接近实验室位置

   - 资源组：**az3000500-LabRG**

   - 存储帐户：新存储帐户的名称

   - 文件共享： 新文件共享的名称

1. 从“Cloud Shell”窗格中，通过运行以下命令创建资源组（将 `<Azure region>` 占位符替换为在订阅中可用且最接近实验室位置的 Azure 区域的名称）

   ```
   az group create --resource-group az3000501-LabRG --location <Azure region>
   ```

1. 在 Cloud Shell 窗格中，上传 Azure 资源管理器模板 **\\allfiles\\AZ-300T01\\Module_05\\azuredeploy05.json** 到主目录。

1. 在 Cloud Shell 窗格中，上传参数文件 **\\allfiles\\AZ-300T01\\Module_05\\azuredeploy05.parameters.json** 到主目录。

1. 在 Cloud Shell 窗格中，通过运行以下命令将托管 Windows Server 2016 Datacenter 的 Azure VM 部署到第一个虚拟网络中：

   ```
   az deployment group create --resource-group az3000501-LabRG --template-file azuredeploy05.json --parameters @azuredeploy05.parameters.json
   ```

   > **注意**：等待部署完成。该操作需要约 5 分钟。

#### 任务 2：创建用户分配的托管标识并将其分配给 Azure VM。

1. 在“Cloud Shell”窗格中，运行以下命令以创建用户分配的托管标识：

   ```
   az identity create --resource-group az3000501-LabRG --name az3000501-mi
   ```

1. 在 Cloud Shell 窗格中，运行以下命令以将用户分配的托管标识分配给 Azure VM：

   ```
   az vm identity assign --resource-group az3000501-LabRG --name az3000501-vm --identities az3000501-mi
   ```

#### 任务 3：引用用户分配的托管标识配置 RBAC。

1. 在“Cloud Shell”窗格中，运行以下命令以创建资源组（将 `<Azure region>` 占位符替换为其中已部署本练习中的 Azure VM 的 Azure 区域的名称）：

   ```
   az group create --resource-group az3000502-LabRG --location <Azure region>
   ```

1. 在 Azure 门户中，导航到 **az3000502-LabRG - 访问控制（IAM）** 边栏选项卡。

1. 在 **az3000502-LabRG - 访问控制（IAM）** 边栏选项卡中，将所有者角色分配给新创建的用户分配的托管标识。

> **结果**：完成本练习后，你已创建并配置了用户分配的托管标识。

## 练习2：验证用户分配的托管标识的功能

本练习的主要任务如下：

1. 配置 Azure VM 以通过用户分配的托管标识进行身份验证。

1. 在 Azure VM 验证用户分配的托管标识的功能。

#### 任务 1： 配置 Azure VM 以通过用户分配的托管标识进行身份验证。

1. 在 Azure 门户中，导航到 **“az3000501-vm”** 边栏选项卡。

1. 使用远程桌面连接到 Azure VM，并通过提供以下凭据进行身份验证：

   - 用户名：**学生**

   - 密码：**Pa55w.rd1234**

1. 建立远程桌面会话后，便将为你呈现**管理员：C:\\Windows\\system32\\cmd.exe**窗口。要启动 PowerShell 会话，请在命令提示符处键入 `PowerShell` 并按 Enter 键。

1. 在 PowerShell 提示符处，运行以下命令以安装最新版本的 PowerShellGet 模块（如果提示确认，请按 Enter）：

   ```pwsh
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force -SkipPublisherCheck
   ```

1. 在 PowerShell 提示符下，运行以下命令以安装最新版本的 Az 模块（键入 **Y** 并在提示确认时按 Enter 键）：

   ```pwsh
   Install-Module -Name Az -AllowClobber -SkipPublisherCheck
   ```

1. 通过键入 `exit` 并按 Enter 键退出当前 PowerShell 会话，然后通过在命令提示符处键入 `PowerShell` 并按 Enter 键再次将其启动。

1. 在 PowerShell 提示符处，运行以下命令以安装 AzureRM.ManagedServiceIdentity 模块：

   ```pwsh
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   ```

1. 在 PowerShell 提示符下，运行以下命令以安装 AzureRM.ManagedServiceIdentity 模块（键入 **“Y”** 并在提示确认时按 Enter 键）：

   ```pwsh
   Install-Module -Name Az.ManagedServiceIdentity
   ```

#### 任务 2： 在 Azure VM 验证用户分配的托管标识的功能。

1. 在 PowerShell 提示符处，运行以下命令以用户分配的托管标识登录：

   ```pwsh
   Add-AzAccount -Identity
   ```

1. 在 PowerShell 提示符处，运行以下命令以尝试检索当前使用的托管标识：

   ```pwsh
   (Get-AzVM -ResourceGroupName az3000501-LabRG -Name az3000501-vm).Identity
   ```

1. 请注意错误信息。如消息所述，当前安全上下文未向目标资源授予足够权限。要解决此问题，请切换到 Azure 门户，导航到 **“az3000501-LabRG - 访问控制 (IAM)”** 边栏选项卡。

1. 在 **“az3000501-LabRG - 访问控制(IAM)”** 边栏选项卡中，将参与者角色分配给用户分配的托管标识 **az3000501-mi**。

1. 切换回远程桌面会话，并从 PowerShell 提示符运行以下命令，以尝试检索当前使用的托管标识：

   ```pwsh
   (Get-AzVM -ResourceGroupName az3000501-LabRG -Name az3000501-vm).Identity
   ```

     > **注**：如果收到指示权限不够的错误消息，请从 PowerShell 提示符处运行
   
   ```pwsh
   Remove-AzAccount
   ```
   
   然后是：
   
   ```pwsh
   Add-AzAccount -Identity
   ```
      
1. 在 PowerShell 提示符下，运行以下命令以在变量中存储位置：

   ```pwsh
   $location = (Get-AzResourceGroup -Name az3000502-LabRG).Location
   ```

1. 在 PowerShell 提示符下，运行以下命令以创建公共 IP 地址资源：

   ```pwsh
   New-AzPublicIpAddress -Name az3000502-pip -ResourceGroupName az3000502-LabRG -AllocationMethod Dynamic -Location $location
   ```

1. 验证命令是否成功完成。

> **结果**：完成本练习后，你已验证用户定义的托管标识的功能。

## 练习 3：删除实验室资源

#### 任务 1：打开 Cloud Shell

1. 在门户顶部，单击 **Cloud Shell** 图标以打开“Cloud Shell”窗格。

1. 在 **“Cloud Shell”** 命令提示符处，键入以下命令并按 **Enter** 键列出你在本实验中创建的所有资源组：

   ```
   az group list --query "[?starts_with(name,'az30005')].name" --output tsv
   ```

1. 验证输出中是否仅包含你在本实验室中创建的资源组。这些组将在下一个任务中删除。

#### 任务 2：删除资源组

1. 在 **“Cloud Shell”** 命令提示符处，键入以下命令并按 **Enter** 键以删除你在本实验室中创建的资源组

   ```sh
   az group list --query "[?starts_with(name,'az30005')].name" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

1. 关闭门户底部的 **“Cloud Shell”** 提示。

> **结果**：在本练习中，你删除了本实验室中使用的资源。
