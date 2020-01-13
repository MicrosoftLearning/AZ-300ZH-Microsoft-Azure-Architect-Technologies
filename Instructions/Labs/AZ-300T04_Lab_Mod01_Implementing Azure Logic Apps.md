---
lab:
    title: '实施 Azure 逻辑应用'
    module: '模块 1：创建 Azure 应用服务 Web 应用'
---

# 实施和管理应用服务
# 实验室：实施 Azure 逻辑应用
  
### 场景
  
Adatum Corporation 希望实施对资源组更改的自定义监控。


### 目标
  
完成本实验室课程后，将能够：

-  创建 Azure 逻辑应用 

-  配置 Azure 逻辑应用和 Azure 事件网格的集成

### 实验室设置
  
预计用时：45 分钟

用户名：**学生**

密码：**Pa55w.rd**


## 练习 1：设置由 Azure 存储帐户和 Azure 逻辑应用组成的实验室环境 
  
本练习的主要任务如下：

1. 创建 Azure 存储帐户

1. 创建 Azure 逻辑应用 

1. 创建 Azure AD 服务主体 

1. 将读者角色分配给 Azure AD 服务主体 

1. 注册 Microsoft.EventGrid 资源提供程序 


#### 任务 1：创建 Azure 存储帐户

1. 从实验室虚拟机启动 Microsoft Edge 并浏览 Azure 门户 [**http://portal.azure.com**](http://portal.azure.com) 并使用在目标 Azure 订阅中具有所有者角色的 Microsoft 帐户登录。
  
1. 从 Azure 门户创建具有以下设置的新存储账户： 

    - 订阅：目标 Azure 订阅名称

    - 资源组：创建一个命名为 **az3000701-LabRG** 的新资源组

    - 存储帐户名称：3 到 24 个字符之间的任何有效、唯一名称、由小写字母和数字组成

    - 位置：订阅中可用 Azure 区域的名称，该区域最接近实验室位置

    - 性能：**标准**

    - 帐户类型：**存储（通用目的 v1）**

    - 复制：**本地冗余存储（LRS）**

    - 需要安全转移：**已启用**

    - 虚拟网络：**所有网络**

    - 分层命名空间：**已禁用**

    > **注**：不要等待部署完成，而是继续执行下一个任务。 


#### 任务 2：创建 Azure 逻辑应用 
 
1. 在 Azure 门户中创建一个 **Logic App** 示例，设置如下：

    - 名称：**logicapp3000701**

    - 订阅：目标 Azure 订阅名称

    - 资源组：新资源组的名称 **az3000702-LabRG**

    - 位置：你在上一个任务中部署存储帐户的相同 Azure 区域

    - 日志分析：**关闭**

1. 等待应用预配完成。该操作将需要约 1 分钟。 


#### 任务 3：创建 Azure AD 服务主体 

1. 在 Azure 门户的 Microsoft Edge 窗口中，启动 **Cloud Shell** 内的 **PowerShell** 会话。 

1. 如果显示 **你没有装载存储** 消息，使用以下设置配置存储：

    - 订阅：目标 Azure 订阅名称

    - Cloud Shell 区域：订阅中可用的 Azure 区域的名称，该区域最接近实验室位置

    - 资源组：新资源组的名称 **az3000700-LabRG**

    - 存储帐户：新存储帐户的名称

    - 文件共享：新文件共享的名称

1. 在“Cloud Shell”窗格中，运行以下命令以创建新的 Azure AD 应用程序，该应用程序将与你在此任务的后续步骤中创建的服务主体相关联：

```pwsh
$password = 'Pa55w.rd1234'
$securePassword = ConvertTo-SecureString -Force -AsPlainText -String $password
$aadApp30007 = New-AzADApplication -DisplayName 'aadApp30007' -HomePage 'http://aadApp30007' -IdentifierUris 'http://aadApp30007' -Password $securePassword
```

1. 在“Cloud Shell”窗格中，运行以下命令以创建与你在上一步中创建的应用程序关联的新 Azure AD 服务主体：

```pwsh
New-AzADServicePrincipal -ApplicationId $aadApp30007.ApplicationId.Guid
```

1. 在 **New-AzADServicePrincipal** 命令的输出中，注意 **ApplicationId** 属性的值。在本实验室的下一个练习中，你将需要这个。

1. 在 Cloud Shell 窗格中，运行以下命令以标识当前 Azure 订阅的 **Id** 属性值和与该订阅关联的 Azure AD 租户的 **TenantId** 属性值（在本实验室的下一个练习中，你也需要它们）：

```pwsh
Get-AzSubscription
```

1. 关闭 Cloud Shell 窗格。


#### 任务 4：将读者角色分配给 Azure AD 服务主体 

1. 在 Azure 门户中，导航到显示 Azure 订阅属性的边栏选项卡。 

1. 在 Azure 订阅边栏选项卡上，单击 **访问控制（IAM）**。

1. 将 Azure 订阅范围内的 **读者** 角色分配至 **aadApp30007** 服务主体。


#### 任务 5：注册 Microsoft.EventGrid 资源提供程序

1. 在 Azure 门户的 Microsoft Edge 窗口中，重新打开 **Cloud Shell** 内的 **PowerShell** 会话。 

1. 在 Cloud Shell 窗格中，运行以下命令以注册 Microsoft.EventGrid 资源提供程序：

```pwsh
Register-AzResourceProvider -ProviderNamespace Microsoft.EventGrid
```

1. 关闭 Cloud Shell 窗格。


> **结果**：完成本练习后，你已创建了一个存储帐户、将在本实验室的下一个练习中配置的逻辑应用程序以及将在该配置期间引用的 Azure AD 服务主体。


## 练习 2：配置 Azure 逻辑应用程序以监视对资源组的更改情况。
  
本练习的主要任务如下：

1. 在 Azure 逻辑应用程序中添加触发器

1. 向 Azure 逻辑应用程序添加操作

1. 标识 Azure 逻辑应用的回叫 URL

1. 配置事件订阅

1. 测试逻辑应用程序


#### 任务 1：在 Azure 逻辑应用程序中添加触发器

1. 在 Azure 门户中，导航到新预配的 Azure 逻辑应用的 **逻辑应用设计器** 边栏选项卡。

1. 点击 **空白逻辑应用程序**。这将创建一个空白预设器工作区，并显示要添加到工作区的连接器和触发器列表。

1. 搜索 **事件网格** 触发器，在结果列表中单击 **发生资源事件时（预览）** Azure 事件网格触发器将其添加到设计器工作区。

1. 在 **Azure 事件网格** 中，单击 **与服务主体连接** 链接，指定以下值，然后单击 **创建**：

    - 连接名称：**egc30007**

    - 客户端 ID：你在上一个练习中标识的 **ApplicationId** 属性

    - 客户端密钥：**Pa55w.rd1234**

    - 租户：你在上一个练习中标识的 **TenantId** 属性

1. 在 **发生资源事件时** 中，指定以下值：

    - 订阅：你在上一个练习中标识的订阅 **Id** 属性

    - 资源类型：**Microsoft.Resources.resourceGroups**

    - 资源名称：**/subscriptions/*subscriptionId*/resourceGroups/az3000701-LabRG**，其中 ***subscriptionId*** 你在上一个练习中标识的订阅 **Id** 属性

    - 事件类型项目 - 1：**Microsoft.Resources.ResourceWriteSuccess**

    - 事件类型项目 - 2：**Microsoft.Resources.ResourceDeleteSuccess**

1. 单击 **添加新参数** 并选择 **订阅名称**

1. 在 **订阅名称** 文本框中，键入 **event-subscription-az3000701** 并单击 **保存**。

#### 任务 2：向 Azure 逻辑应用程序添加操作

1. 在 Azure 门户中，在新预配的 Azure 逻辑应用的逻辑应用设计器边栏选项卡上，单击 **添加新的一步**。 

1. 在 **选择操作** 窗格，在 **搜索连接器和操作** 文本框内键入 **Outlook**。

1. 在结果列表中，单击 **Outlook.com**。 

1. 在 **Outlook.com** 操作列表中，单击 **Outlook.com - 发送电子邮件**。

1. 在 **Outlook.com** 窗格中，单击 **登录**。 

1. 出现提示时，使用你在本实验室中使用的 Microsoft 帐户进行身份验证。 

1. 当系统提示你同意授予 Azure Logic App 访问 Outlook 资源的权限时，请单击 **是**。

1. 在 **发送电子邮件** 窗格，指定以下设置并单击 **保存**：

    - 收件人：你的 Microsoft 帐户名称

    - 主题：键入 **资源更新：** 并且在 **动态内容** 栏（**发送电子邮件** 窗格右边），单击 **主题**。

    - 正文：键入 **资源组**,在 **发送电子邮件** 窗格右边的 **动态内容列**，单击 **话题**，键入 **事件类型：**， 在 **发送电子邮件** 窗格右边的 **动态内容** 列，单击 **事件类型**，键入 **事件 ID：**，在 **发送电子邮件** 窗格右边的 **动态内容** 列，单击 **ID**，键入 **事件时间：**，并在 **发送电子邮件** 窗格右边的 **动态内容** 列，单击 **事件时间**。


#### 任务 3：标识 Azure 逻辑应用的回叫 URL

1. 在 Azure 门户中，导航到 **logicapp3000701** 边栏选项卡，并且在 **摘要** 部分，单击 **查看触发历史**。忽略任何 **被禁止** 错误消息。

1. 在 **资源事件发生时** 边栏选项卡上，复制 **回叫 url[POST]** 文本框的值。


#### 任务 4：配置事件订阅

1. 在 Azure 门户中，导航到 **az3000701-LabRG** 资源组，然后在垂直菜单中单击 **事件**。

1. 在“**az3000701-LabRG - 事件**”边栏选项卡上，选择“**入门**”，然后单击“**Webhook**”。

1. 在 **创建事件订阅** 边栏选项卡上，在 **筛选到事件类型** 下拉列表中，确保只有 **资源写入成功** 和 **资源删除成功** 旁边的复选框被选中。

1. 在 **终结点类型** 下拉列表中，确保选中 **Web Hook** 并单击 **选择终结点** 链接。 

1. 在 **选择 Web Hook ** 边栏选项卡上，在 **订阅者终结点**，粘贴你在上一个任务中复制的 Azure 逻辑应用的 **回叫 url[POST]** 的值，然后单击 **确认选择**。

1. 在 **事件订阅详情** 部分内的 **名称** 文本框中，键入 **event-subscription-az3000701**。

1. 单击**创建**。


#### 任务 5：测试逻辑应用程序 

1. 在 Azure 门户中，导航到 **az3000701-LabRG** 资源组，然后在垂直菜单中单击 **概述**。 

1. 在资源列表中，单击你在第一个练习中创建的 Azure 存储帐户。

1. 在存储帐户边栏选项卡上的垂直菜单中，单击 **配置**。

1. 在配置边栏选项卡上，将“**需要安全转移**”设置为“**禁用**”并单击“**保存**”

1. 导航到 **logicapp3000701** 边栏选项卡，单击 **刷新**，并注意 **运行历史** 包括与 Azure 存储帐户的配置更改相对应的条目。

1. 导航到你在本练习中配置的电子邮件帐户的收件箱，并验证其中是否包含逻辑应用生成的电子邮件。

> **结果**：完成本练习后，你已配置了 Azure 逻辑应用程序，可用于监视资源组的更改情况。


## 练习 3：删除实验资源

#### 任务 1：打开 Cloud Shell

1. 在门户顶部，单击“**Cloud Shell**”图标以打开 Cloud Shell 窗格。

1. 如果需要，请使用“Cloud Shell”窗格左上角的下拉列表切换到 Bash Shell 会话。

1. 在“**Cloud Shell**”命令提示符处，键入以下命令并按 **Enter** 键列出你在本实验中创建的所有资源组：

```
   az group list --query "[?starts_with(name,'az30007')]".name --output tsv
```

1. 验证输出是否仅包含你在本实验中创建的资源组。这些组将在下一个任务中删除。

#### 任务 2：删除资源组

1. 在“**Cloud Shell**”命令提示符处，键入以下命令并按 **Enter** 键以删除你在本实验中创建的资源组

```sh
   az group list --query "[?starts_with(name,'az30007')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
```

1. 关闭门户底部的 **Cloud Shell** 提示。

> **结果**：在本练习中，你删除了本实验中使用的资源。
