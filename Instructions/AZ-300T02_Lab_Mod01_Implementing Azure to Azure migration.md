# 评估和执行服务器向 Azure 的迁移

# 实验：实施 Azure 向 Azure 迁移

### 方案

Adatum Corporation 希望将现有 Azure VM 迁移到其他区域

### 目标

完成本实验室后，你将能执行以下操作：

- 实现 Azure Site Recovery 保管库

- 在 Azure 区域之间迁移 Azure VM

### 实验设置

预计用时：45 分钟

用户名：**学生**

密码：**Pa55w.rd**

## 练习 1： 使用 Azure Site Recovery 实现 Azure VM 迁移的先决条件

本次练习的主要任务如下：

1. 部署要迁移的 Azure VM

1. 创建 Azure 恢复服务保管库

#### 任务 1：部署要迁移的 Azure VM

1. 从实验虚拟机中，启动 Microsoft Edge 并浏览到 Azure门户 [**http://portal.azure.com**](http://portal.azure.com)，然后使用在目标Azure订阅中具有所有者角色的 Microsoft 帐户登录。

1. 在 Azure 门户的 Microsoft Edge 窗口中，在 **Cloud Shell** 中启动 **PowerShell** 会话。

1. 如果呈现**您未装载存储器**消息，请使用以下设置配置存储器：

   - 订阅：目标 Azure 订阅的名称

   - Cloud Shell 区域：在订阅中可用且最接近实验位置的 Azure 区域的名称

   - 资源组：新资源组的名称 **az3000600-LabRG**

   - 存储帐户：新存储帐户的名称

   - 文件共享： 新文件共享的名称

1. 从 “Cloud Shell” 窗格中，通过运行以下命令创建资源组（将 `<Azure region>` 占位符替换为在订阅中可用且最接近实验室位置的 Azure 区域的名称）

   ```pwsh
   New-AzResourceGroup -Name az3000601-LabRG -Location <Azure region>
   ```

1. 从 Cloud Shell 窗格中，将 Azure 资源管理器模板 **\\allfiles\\AZ-300T02\\Module_01\\azuredeploy06.json** 上载到主目录中。

1. 从 Cloud Shell 窗格中，将参数文件 **\\allfiles\\AZ-300T02\\Module_01\\azuredeploy06.parameters.json** 上载到主目录中。

1. 在 Cloud Shell 窗格中，切换到主目录：
   ```pwsh
   cd $home
   ```

1. 从 Cloud Shell 窗格中，通过运行以下命令部署托管 Windows Server 2016 Datacenter 的 Azure VM：

   ```pwsh
   New-AzResourceGroupDeployment -ResourceGroupName az3000601-LabRG -TemplateFile azuredeploy06.json -TemplateParameterFile azuredeploy06.parameters.json
   ```

   > **注意**：不要等待部署完成，而是继续执行下一个任务。


#### 任务 2：实现 Azure 恢复服务保管库

1. 在 Azure 门户中创建**恢复服务保管库**实例，设置如下：

   - 名称： **vaultaz3000602**

   - 订阅：目标 Azure 订阅的名称

   - 资源组：新建资源组的名称 **az3000602-LabRG**

   - 位置：在订阅中可用且**不同**于您在上一个任务中部署 Azure VM 的区域的 Azure 区域的名称

1. 等待保管库预配完成。该操作需要约 1 分钟的时间。

> **结果**：完成本练习后，您即已创建要迁移的 Azure VM 以及将托管 Azure VM 的已迁移磁盘文件的 Azure Site Recovery 保管库。

## 练习2： 使用 Azure Site Recovery 在Azure 区域之间迁移 Azure VM

本练习的主要任务如下：

1. 配置 Azure VM 复制

1. 查看 Azure VM 复制设置

1. 禁用 Azure VM 的复制并删除 Azure 恢复服务保管库

#### 任务 1： 配置 Azure VM 复制

1. 在 Azure 门户中，导航到新预配的 Azure Recovery Services 保管库的边栏选项卡。

1. 在“恢复服务保管库”边栏选项卡上，单击**+复制**按钮。

1. 通过指定以下设置启用复制：

   - 源： **Azure**

   - 源位置：在本实验室的上一个练习中部署 Azure VM 的相同 Azure 区域

   - Azure 虚拟机部署模型：**资源管理器**

   - 源资源组：**az3000601-LabRG**

   - 虚拟机：**az3000601-VM**

   - 目标位置：在订阅中可用且不同于您在上一个任务中部署 Azure VM 的区域的 Azure 区域的名称

   - 目标资源组：**（新）az3000601-LabRG-asr**

   - 目标虚拟网络：**（新）az3000601-vnet-asr**

   - 缓存存储帐户：接受默认设置

   - 副本托管磁盘：**(新建）1 个高级磁盘, 0 个标准磁盘**

   - 目标可用性集：**不适用**

   - 复制策略：**新建**

   - 名称： **12-hour-retention-policy**

   - 恢复点保留时间：**12小时**

   - 应用一致的快照频率：**6小时**

   - 多VM一致性： **否**

1. 启动目标资源的创建。

1. 启用复制。

   > **注意**：请等待启用复制的操作完成。然后继续执行下一个任务。

#### 任务 2：查看 Azure VM 复制设置

1. 在 Azure 门户中，从 “Azure Site Recovery 保险库”边栏选项卡导航到代表 Azure VM **az3000601-vm** 的复制项目边栏选项卡。

2. 在复制项目边栏选项卡上，查看**运行状况和状态**， **最新的可用恢复点**和**故障转移就绪情况**部分。请注意工具栏中的**故障转移**和**测试故障转移**条目。向下滚动到**基础架构视图**。

3. 如果时间允许，请等待直至 Azure VM 的状态更改为**受保护**。该操作可能需要额外的 15-20 分钟。此时，检查值 **Crash-consistent** 和 **App-consistent** 恢复点。为了查看 **RPO**，您应该执行测试故障转移。

#### 任务 3： 禁用 Azure VM 的复制并删除 Azure 恢复服务保管库

1. 在 Azure 门户中，禁用 Azure VM **az3000601-vm** 的复制。

2. 等待直至禁用复制。

3. 在 Azure 门户中，删除恢复服务保管库。

   > **注意**：在删除保管库之前，必须确保先删除已复制的项目。

> **结果**：完成本练习后，您即已实现 Azure VM 的自动复制。

## 练习 3：删除实验室资源

#### 任务 1：打开 Cloud Shell

1. 在门户顶部，单击 **Cloud Shell** 图标以打开“Cloud Shell”窗格。

1. 如果需要，请使用“Cloud Shell”窗格左上角的下拉列表切换到 Bash Shell 会话。

1. 在 **“Cloud Shell”** 命令提示符处，键入以下命令并按 **Enter** 键列出你在本实验中创建的所有资源组：

   ```
   az group list --query "[?starts_with(name,'az30006')].name" --output tsv
   ```

1. 验证输出中是否仅包含你在本实验室中创建的资源组。这些组将在下一个任务中删除。

#### 任务 2：删除资源组

1. 在 **“Cloud Shell”** 命令提示符处，键入以下命令并按 **Enter** 键以删除你在本实验室中创建的资源组

   ```sh
   az group list --query "[?starts_with(name,'az30006')].name" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

1. 关闭门户底部的 **“Cloud Shell”** 提示。

> **结果**：在本练习中，你删除了本实验室中使用的资源。
