# 实施和管理存储
# 实验室：实施 Azure 存储访问控制
  
### 方案
  
Adatum Corporation 希望保护 Azure 存储中的内容


### 目标
  
完成本实验室后，你将能执行以下操作：

-  创建 Azure Storage 帐户。

-  上传 VHD 至 Azure Storage。

-  实施 Azure 存储访问控制

### 实验室设置
  
预计用时：30 分钟

用户名：**学生**

密码：**Pa55w.rd**


## 练习 1：创建和配置 Azure 存储帐户
  
本练习的主要任务如下：

1. 在 Azure 中创建一个存储帐户。

1. 查看存储帐户的属性 


#### 任务 1：创建 Azure 存储帐户

1. 从实验室虚拟机启动 Microsoft Edge 并浏览 Azure 门户 [**http://portal.azure.com**](http://portal.azure.com) 并使用在目标 Azure 订阅中具有所有者角色的 Microsoft 帐户登录。
  
1. 从 Azure 门户创建具有以下设置的新存储账户： 

    - 订阅：目标 Azure 订阅名称

    - 资源组：一个名为 **az3000201-LabRG** 的新资源组

    - 存储帐户名称：由小写字母和数字组成的长度在 3 到 24 个字符之间的任何有效的唯一名称

    - 位置：订阅中可用的 Azure 区域的名称，且该区域最接近实验位置

    - 性能： **标准**

    - 帐户类型：**存储（常规用途 v1）**

    - 复制：**本地冗余存储 (LRS)**
    
    - 需要安全传输：**已禁用**

    - 网络连接：**公共终结点（所有网络）**

    - blob 软删除：**禁用**

    - DATA LAKE STORAGE Gen2： **禁用**

1. 等待配置存储帐户。这将需要约 1 分钟。


#### 任务 2：查看存储帐户的属性
  
1. 在 Azure 门户中，打开存储帐户边栏选项卡，查看**概述**部分，包括位置、复制和性能设置。

1. 显示**访问键**边栏选项卡。在访问键边栏选项卡上，请注意你可以选择复制包括 key1 和 key2 的存储帐户名称的值。你还可以重新生成两个键。

1. 显示**配置**边栏选项卡。

1. 在 **“配置”** 边栏选项卡中，请注意你可以选择升级到 **“常规用途 v2”** 帐户并更改复制设置。但是，你无法更改性能设置（只能在创建存储帐户时分配）。

> **结果**：完成本练习后，你已创建 Azure 存储并检查其属性。


## 练习2：创建和管理 blobs
  
本练习的主要任务如下：

1. 创建一个容器

1. 使用 Azure 门户将数据上传到容器

1. 使用 SAS 令牌访问 Azure 存储帐户的内容


#### 任务 1：创建一个容器
  
1. 在 Azure 门户中，导航到显示你在上一个任务中创建的存储帐户属性的边栏选项卡。

1. 在防火墙设置边栏选项卡上，创建包含以下设置的新规则：

    - 名称： **labcontainer**

    - 访问类型：**专用**


#### 任务 2：使用 Azure 门户将数据上传到容器
  
1. 在 Azure 门户中，导航至 **labcontainer** 边栏选项卡。

1. 在 **labcontainer** 边栏选项卡中，上传文件：**C:\\Windows\\ImmersiveControlPanel\\images\\splashscreen.contrast-white_scale-400.png**。  


#### 任务 3：使用 SAS 令牌访问 Azure 存储帐户的内容
  
1. 在 **labcontainer** 边栏选项卡中，识别新上传的 Blob 的 URL。 

1. 启动 Microsoft Edge 并导航到该 URL。

1. 注意 **ResourceNotFound** 错误消息。这是预期显示的消息，因为 blob 位于专用容器中，需要经过身份验证才能访问。 

1. 切换到显示 Azure 门户的 Microsoft Edge 窗口，然后在 **splashscreen.contrast-white_scale-400.png** 边栏选项卡中切换到**生成 SAS** 选项卡。 

1. 在**生成 SAS** 选项卡中，启用 **HTTP** 选项并生成 Blob SAS 令牌和相应的 URL。

1. 打开一个新的 Microsoft Edge 窗口，然后导航到上一步生成的 URL。

1. 请注意，你可以查看该图像。这是预期显示的图像，因为此时你有权根据 URL 中包含的 SAS 令牌访问该 Blob。 

1. 关闭显示该图像的 Microsoft Edge 窗口。


#### 任务 4：使用 SAS 令牌和存储的访问策略访问 Azure 存储帐户的内容。

1. 在 Azure 门户中，导航至 **labcontainer** 边栏选项卡。

1. 在 **labcontainer** 边栏选项卡中，导航到 **labcontainer - 访问策略**边栏选项卡。

1. 添加一个新的策略，设置如下:

    - 标识符：**labcontainer-read**

    - 权限：**读取**

    - 开始时间：当前日期和时间

    - 到期时间：当前日期和时间 + 24 小时

1. 单击 **“确定”** 以创建策略，然后单击 **“保存”**。

1. 在 Azure 门户的“Microsoft Edge”窗口中，启动 **Cloud Shell** 内的 **PowerShell** 会话。 

1. 如果显示**你没有装载存储**消息，使用以下设置配置存储：

    - 订阅：目标 Azure 订阅名称

    - Cloud Shell 区域：订阅中可用的 Azure 区域的名称，该区域最接近实验室位置

    - 资源组：**az3000201-LabRG**

    - 存储帐户：新存储帐户的名称

    - 文件共享： 新文件共享的名称

1. 在 Cloud Shell 窗格中，运行以下命令以标识你在本实验的第一个练习中创建的存储帐户资源，并将其存储在变量中：

   ```pwsh
   $storageAccount = (Get-AzStorageAccount -ResourceGroupName az3000201-LabRG)[0]
   ```

1. 在 Cloud Shell 窗格中，运行下列命令以建立安全上下文，授予对存储帐户的完全控制权：

   ```pwsh
   $keyContext = $storageAccount.Context
   ```

1. 在 Cloud Shell 窗格中，运行下列命令以根据你在上一个任务中创建的访问策略创建特定于 Blob 的 SAS 令牌：

   ```pwsh
   $sasToken = New-AzStorageBlobSASToken -Container 'labcontainer' -Blob 'splashscreen.contrast-white_scale-400.png' -Policy labcontainer-read -Context $keyContext
   ```

1. 在 Cloud Shell 窗格中，运行下列命令以基于新建的 SAS 令牌建立安全上下文： 

   ```pwsh
   $sasContext = New-AzStorageContext $storageAccount.StorageAccountName -SasToken $sasToken
   ```

1. 在 Cloud Shell 窗格中，运行下列命令以检索 Blob 的属性： 

   ```pwsh
   Get-AzStorageBlob -Container 'labcontainer' -Blob 'splashscreen.contrast-white_scale-400.png' -Context $sasContext
   ```

1. 验证你是否已成功访问该 Blob。

1. 最小化 Cloud Shell 窗格。


#### 任务 5：通过修改访问策略使 SAS 令牌失效。

1. 在 Azure 门户中，导航到 **labcontainer - 访问策略**边栏选项卡。

1. 通过将开始时间和到期时间设置为昨天的日期，编辑现有策略 **labcontainer-read**。 

1. 重新打开 Cloud Shell 窗格。 

1. 在 Cloud Shell 窗格中，重新运行下列命令以尝试检索 Blob 的属性： 

   ```pwsh
   Get-AzStorageBlob -Container 'labcontainer' -Blob 'splashscreen.contrast-white_scale-400.png' -Context $sasContext
   ```

1. 确认你无法再访问该 Blob。


> **结果**：完成本练习后，你已创建 Blob 容器，将文件上载到容器中，并使用 SAS 令牌和存储的访问策略测试访问控制。

## 练习 3：删除实验室资源

#### 任务 1：打开 Cloud Shell

1. 在门户顶部，单击 **Cloud Shell** 图标以打开“Cloud Shell”窗格。

1. 如果需要，请使用“Cloud Shell”窗格左上角的下拉列表切换到 Bash Shell 会话。

1. 在 **“Cloud Shell”** 命令提示符处，键入以下命令并按 **Enter** 键列出你在本实验中创建的所有资源组：

   ```
   az group list --query "[?starts_with(name,'az30002')]".name --output tsv
   ```

1. 验证输出中是否仅包含你在本实验室中创建的资源组。这些组将在下一个任务中删除。

#### 任务 2：删除资源组

1. 在 **“Cloud Shell”** 命令提示符处，键入以下命令并按 **Enter** 键以删除你在本实验室中创建的资源组

   ```sh
   az group list --query "[?starts_with(name,'az30002')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

1. 关闭门户底部的 **Cloud Shell** 提示。

> **结果**：在本练习中，你删除了本实验室中使用的资源。
