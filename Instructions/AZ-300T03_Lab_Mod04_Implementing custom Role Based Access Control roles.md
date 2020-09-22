# 获得身份
# 实验室：实现基于自定义角色的访问控制（RBAC）角色
  
### 方案
  
Adatum Corporation 希望实现自定义 RBAC 角色，以委派启动和停止（取消分配）Azure VM 的权限。


### 目标
  
完成本实验室后，你将能执行以下操作：

-  定义自定义 RBAC 角色 

-  分配自定义 RBAC 角色

### 实验室设置
  
预计用时：30 分钟

用户名：**学生**

密码：**Pa55w.rd**


## 练习 1：定义自定义 RBAC 角色
  
本练习的主要任务如下：

1. 使用 Azure 资源管理器模板部署 Azure VM

1. 确定通过 RBAC 委派的操作

1. 在 Azure AD 租户中创建自定义 RBAC 角色


#### 任务 1：使用 Azure 资源管理器模板部署 Azure VM

1. 从实验虚拟机启动 Microsoft Edge，并浏览到 Azure 门户[**http://portal.azure.com**](http://portal.azure.com)，并使用在目标 Azure 订阅中具有所有者角色的 Microsoft 帐户登录。

1. 在 Azure 门户的 Microsoft Edge 窗口中，在 **Cloud Shell** 内启动 **PowerShell** 会话。 

1. 如果显示**你没有安装存储**消息，则使用以下设置配置存储：

    - 订阅：目标 Azure 订阅的名称

    - Cloud Shell 区域：订阅中可用的 Azure 区域的名称，该区域最接近实验室位置

    - 资源组：新资源组的名称 **“az3000900-LabRG”**

    - 存储帐户：新存储帐户的名称

    - 文件共享：新文件共享的名称

1. 在 Cloud Shell 窗格中，通过运行以下命令创建资源组（用订阅中可用且最接近实验位置的 Azure 区域的名称替换 `<Azure region>` 占位符）

   ```pwsh
   New-AzResourceGroup -Name az3000901-LabRG -Location <Azure region>
   ```

1. 在 Cloud Shell 窗格中，上载 Azure 资源管理器模板 **\\allfiles\\AZ-300T03\\Module_04\\azuredeploy09.json** 进入主目录。

1. 在 Cloud Shell 窗格中，上载参数文件 **\\allfiles\\AZ-300T03\\Module_04\\azuredeploy09.parameters.json** 进入主目录。

1. 在 Cloud Shell 窗格中，通过运行以下命令部署托管 Ubuntu 的 Azure VM：

   ```pwsh
   New-AzResourceGroupDeployment -ResourceGroupName az3000901-LabRG -TemplateFile $home/azuredeploy09.json -TemplateParameterFile $home/azuredeploy09.parameters.json
   ```

    > **注意**：不要等待部署完成，而是继续执行下一个任务。 

1. 在 Azure 门户中，关闭 Cloud Shell 窗格。


#### 任务 2：识别通过 RBAC 委派的操作

1. 在 Azure 门户中，导航到 **az3000901-LabRG** 边栏选项卡。

1. 在 **az3000901-LabRG** 边栏选项卡中，单击**访问控制（IAM）**。

1. 在 **az3000901-LabRG - 访问控制（IAM）**边栏选项卡中，单击**角色**。

1. 在**角色**边栏选项卡中，单击**所有者**。

1. 在**所有者**边栏选项卡中，单击**权限**。

1. 在**权限（预览**边栏选项卡中，单击**微软计算**。

1. 在 **Microsoft Compute** 边栏选项卡中，单击**虚拟机**。

1. 在**虚拟机**边栏选项卡中，查看可通过 RBAC 委派的管理操作列表。请注意，它们包括**取消分配虚拟机**和**启动虚拟机**操作。


#### 任务 3：在 Azure AD 租户中创建自定义 RBAC 角色

1. 在实验计算机上，打开文件 **\\allfiles\\AZ-300T03\\Module_04\\customRoleDefinition09.json** 并审查其内容：

   ```json
   {
      "Name": "Virtual Machine Operator (Custom)",
      "Id": null,
      "IsCustom": true,
      "Description": "Allows to start and stop (deallocate) Azure VMs",
      "Actions": [
          "Microsoft.Compute/*/read",
          "Microsoft.Compute/virtualMachines/deallocate/action",
          "Microsoft.Compute/virtualMachines/start/action"
      ],
      "NotActions": [
      ],
      "AssignableScopes": [
          "/subscriptions/SUBSCRIPTION_ID"
      ]
   }
   ```

1. 在 Azure 门户的 Microsoft Edge 窗口中，在 **Cloud Shell** 内启动 **PowerShell** 会话。 

1. 在 Cloud Shell 窗格中，上载 Azure 资源管理器模板 **\\allfiles\\AZ-300T03\\Module_04\\customRoleDefinition09.json** 进入主目录。

1. 在 Cloud Shell 窗格中，运行以下命令以替换 **$订阅\_ID**占位符为 Azure 订阅的 ID 值：

   ```pwsh
   $subscription_id = (Get-AzContext).Subscription.id
   (Get-Content -Path $HOME/customRoleDefinition09.json) -Replace 'SUBSCRIPTION_ID', "$subscription_id" | Set-Content -Path $HOME/customRoleDefinition09.json
   ```
 
1. 在 Cloud Shell 窗格中，运行下列命令以创建自定义角色定义：

   ```pwsh
   New-AzRoleDefinition -InputFile $HOME/customRoleDefinition09.json
   ```

1. 在 Cloud Shell 窗格中，运行下列命令以验证是否已成功创建角色：

   ```pwsh
   Get-AzRoleDefinition -Name 'Virtual Machine Operator (Custom)'
   ```

1. 关闭“Cloud Shell”窗格。


> **结果**：完成本练习后，你已定义自定义 RBAC 角色


## 练习2：分配并测试自定义 RBAC 角色
  
本练习的主要任务如下：

1. 创建一个 Azure AD 用户

1. 创建 RBAC 角色分配

1. 测试 RBAC 角色分配


#### 任务 1：创建 Azure AD 用户

1. 在 Azure 门户的 Microsoft Edge 窗口中，在 **Cloud Shell** 内启动 **PowerShell** 会话。 

1. 在“Cloud Shell”窗格中运行以下命令，向目标 Azure AD 租户进行显式身份验证：

   ```pwsh
   Connect-AzureAD
   ```
   
1. 在 Cloud Shell 窗格中，运行以下命令以标识 Azure AD DNS 域名：

   ```pwsh
   $domainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   ```

1. 在 Cloud Shell 窗格中，运行下列命令以创建新的 Azure AD 用户：

   ```pwsh
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = 'Pa55w.rd1234'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   New-AzureADUser -AccountEnabled $true -DisplayName 'lab user0901' -PasswordProfile $passwordProfile -MailNickName 'labuser0901' -UserPrincipalName "labuser0901@$domainName"
   ```

1. 在 Cloud Shell 窗格中，运行下列命令以标识新创建的 Azure AD 用户的用户主体名称：

   ```pwsh
   (Get-AzureADUser -Filter "MailNickName eq 'labuser0901'").UserPrincipalName
   ```

1. 关闭“Cloud Shell”窗格。


#### 任务 2：创建 RBAC 角色分配
 
1. 在 Azure 门户中，导航到 **az3000901-LabRG** 边栏选项卡。

1. 在 **az3000901-LabRG** 边栏选项卡中，单击**访问控制（IAM）**。

1. 在 **az3000901-LabRG - 访问控制（IAM）**边栏选项卡中，单击**+添加**并选择**添加角色分配**选项。

1. 在**边栏选项卡上**，指定以下设置并单击 **Save**：

    - 角色：**虚拟机操作员（自定义）**

    - 分配对下列对象的访问：**Azure AD 用户、组或服务主体**

    - 选择：**lab user0901**


#### 任务 3：测试 RBAC 角色分配

1. 启动一个新的私有 Microsoft Edge 窗口，浏览到 Azure 门户网站 [**http://portal.azure.com**](http://portal.azure.com)，并使用新创建的用户帐户登录：

    - 用户名：你在本练习的第一个任务中确定的用户主体名称

    - 密码：**Pa55w.rd1234**

1. 在 Azure 门户中，导航到**资源组**边栏选项卡。请注意，你无法看到任何资源组。 

1. 在 Azure 门户中，导航到**所有资源**边栏选项卡。请注意，你只能看到 **az3000901-vm** 及其托管磁盘。

1. 在 Azure 门户中，导航到 **“az3000901-vm”** 边栏选项卡。尝试重新启动虚拟机。查看通知区域中的错误消息，并注意到此操作失败了，因为当前用户无权执行此操作。

1. 停止虚拟机并验证操作是否已成功完成。

> **结果**：完成本练习后，你已分配并测试自定义 RBAC 角色

## 练习 3：删除实验室资源

#### 任务 1：打开 Cloud Shell

1. 在门户顶部，单击 **Cloud Shell** 图标以打开“Cloud Shell”窗格。

1. 如果需要，请使用“Cloud Shell”窗格左上角的下拉列表切换到 Bash Shell 会话。

1. 在 **“Cloud Shell”** 命令提示符处，键入以下命令并按 **Enter** 键列出你在本实验中创建的所有资源组：

   ```
   az group list --query "[?starts_with(name,'az30009')]".name --output tsv
   ```

1. 验证输出中是否仅包含你在本实验室中创建的资源组。这些组将在下一个任务中删除。

#### 任务 2：删除资源组

1. 在 **“Cloud Shell”** 命令提示符处，键入以下命令并按 **Enter** 键以删除你在本实验室中创建的资源组

   ```sh
   az group list --query "[?starts_with(name,'az30009')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

1. 关闭门户底部的 **“Cloud Shell”** 提示。

> **结果**：在本练习中，你删除了本实验室中使用的资源。
