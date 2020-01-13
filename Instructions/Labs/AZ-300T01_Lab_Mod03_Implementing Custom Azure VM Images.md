---
lab:
    title: '实施自定义 Azure VM 映像'
    module: '模块 3：部署和管理虚拟机 (VM)'
---

# 部署和管理虚拟机 (VM)

# 实验室：实施自定义 Azure VM 映像

### 方案

Adatum Corporation 希望创建自定义 Azure VM 映像

### 目标

完成本实验室后，你将能够：

- 使用 HashiCorp Packer 创建自定义 VM 映像

- 基于自定义映像部署 Azure VM

### 实验室设置

预计用时： 45 分钟

接口：**在 BASH 模式下使用 Azure Cloud Shell**

## 练习 1：创建自定义映像。

本练习的主要任务如下：

1. 配置 Packer 模板

1. 生成基于 Packer 的映像

#### 任务 1：配置 Packer 模板

1. 从实验室虚拟机启动 Microsoft Edge 并浏览 Azure 门户网站[**http://portal.azure.com**](http://portal.azure.com)， 使用在目标 Azure 订阅中具有所有者角色的 Microsoft 帐户登录。

1. 在 Azure 门户的 Microsoft Edge 窗口中，启动 **Cloud Shell** 内的 **Bash** 会话。

1. 如果呈现 **你未装载存储器** 消息，请使用以下设置配置存储器：

   - 订阅：目标 Azure 订阅的名称

   - Cloud Shell 区域：订阅中可用的 Azure 区域名称，它最接近实验室位置

   - 资源组：**az3000300-LabRG**

   - 存储帐户：新存储帐户的名称

   - 文件共享：新文件共享的名称

1. 在Cloud Shell窗格中，运行以下命令以创建资源组并将 JSON 输出存储在变量中（用订阅中可用的且最接近实验室位置的 Azure 区域名称替换 `<Azure region>` 占位符）：

```sh
   RG=$(az group create --name az3000301-LabRG --location <Azure region>)
```
   > **注意**：要列出 Azure 区域，请运行`az account list-locations --output table`

1. 在 Cloud Shell 窗格中，运行以下命令以创建将由 Packer 使用的服务主体，并将 JSON 输出存储在变量中：

```sh
   AAD_SP=$(az ad sp create-for-rbac)
```

1. 在 Cloud Shell 窗格中，运行以下命令以检索服务主体 appId 的值并将其存储在变量中

```sh
   CLIENT_ID=$(echo $AAD_SP | jq -r .appId)
```

1. 在 Cloud Shell 窗格中，运行以下命令以检索服务主体密码的值并将其存储在变量中

```sh
   CLIENT_SECRET=$(echo $AAD_SP | jq -r .password)
```

1. 在 Cloud Shell 窗格中，运行以下命令以检索服务主体租户 ID 的值并将其存储在变量中

```sh
   TENANT_ID=$(echo $AAD_SP | jq -r .tenant)
```

1. 在 Cloud Shell 窗格中，运行以下命令以检索订阅 ID 的值并将其存储在变量中：

```sh
   SUBSCRIPTION_ID=$(az account show --query id | tr -d '"')
```

1. 在 Cloud Shell 窗格中，运行以下命令以检索资源组位置的值并将其存储在变量中：

```sh
   LOCATION=$(echo $RG | jq -r .location)
```

1. 在 Cloud Shell 窗格中，上传 Packer 模板 **\\allfiles\\AZ-300T01\\Module_03\\template03.json** 进入主目录。要上传文件，请单击 Cloud Shell 窗格中具有向上和向下箭头的文档图标。 

1. 在 Cloud Shell 窗格中，运行以下命令用 Packer 模板中的变量 **\$CLIENT_ID** 值替换 **client_id** 参数值的占位符：

```sh
   sed -i.bak1 's/"$CLIENT_ID"/"'"$CLIENT_ID"'"/' ~/template03.json
```

1. 在 Cloud Shell 窗格中，运行以下命令用 Packer 模板中的变量 **\$CLIENT_SECRET** 值替换 **client_secret** 参数值的占位符：

```sh
   sed -i.bak2 's/"$CLIENT_SECRET"/"'"$CLIENT_SECRET"'"/' ~/template03.json
```

1. 在 Cloud Shell 窗格中，运行以下命令用 Packer 模板中的变量 **\$TENANT_ID** 值替换 **tenant_id** 参数值的占位符：

```sh
   sed -i.bak3 's/"$TENANT_ID"/"'"$TENANT_ID"'"/' ~/template03.json
```

1. 在 Cloud Shell窗格中，运行以下命令用 Packer 模板中的变量 **\$SUBSCRIPTION_ID** 值替换 **subscription_id** 参数值的占位符：

```sh
   sed -i.bak4 's/"$SUBSCRIPTION_ID"/"'"$SUBSCRIPTION_ID"'"/' ~/template03.json
```

1. 在 Cloud Shell 窗格中，运行以下命令用 Packer 模板中的变量 **\$LOCATION** 值替换 **location** 参数值的占位符：

```sh
   sed -i.bak5 's/"$LOCATION"/"'"$LOCATION"'"/' ~/template03.json
```

#### 任务 2：构建基于 Packer 的映像

1. 在 Cloud Shell 窗格中，运行以下命令以构建基于 packer 的映像：

```sh
   packer build template03.json
```

1. 监控构建的进度，直到完成为止。

   > **注**：构建过程可能需要大约 10 分钟。

> **结果**：完成本练习后，你已创建了一个 Packer 模板并使用它来构建自定义映像。

## 练习 2：部署自定义映像

本练习的主要任务如下：

1. 基于自定义映像部署 Azure VM

1. 验证 Azure VM部署

#### 任务 1：基于自定义映像部署 Azure VM

1. 在 Cloud Shell 窗格中，运行以下命令以基于自定义映像部署 Azure VM。

```sh
   az vm create --resource-group az3000301-LabRG --name az3000301-vm --image az3000301-image --admin-username student --generate-ssh-keys
```

1. 等待部署完成

   > **注**：部署过程可能需要大约 3 分钟。

1. 部署完成后，从 Cloud Shell 窗格中，运行以下命令以允许在 TCP 端口 80 上传输新部署的 VM 的入站流量：

```sh
   az vm open-port --resource-group az3000301-LabRG --name az3000301-vm --port 80
```

#### 任务 2：验证 Azure VM 部署

1. 在 Cloud Shell 窗格中，运行以下命令以标识与新部署的 Azure VM 关联的 IP 地址。

```sh
   az network public-ip show --resource-group az3000301-LabRG --name az3000301-vmPublicIP --query ipAddress
```

1. 启动 Microsoft Edge 并导航到你在上一步中标识的 IP 地址。

1. 验证 Microsoft Edge 是否显示 **欢迎来到 nginx！** 页面。

> **结果**：完成本练习后，你已基于自定义映像部署了Azure VM 并验证了部署。

## 练习 3：删除实验资源

#### 任务 1：打开 Cloud Shell

1. 在门户顶部，单击 **Cloud Shell** 图标以打开 Cloud Shell 窗格。

1. 在 **Cloud Shell** 命令提示符处，键入以下命令并按 **Enter** 键列出你在本实验室中创建的所有资源组：

```sh
   az group list --query "[?starts_with(name,'az3000301')]".name --output tsv
```

1. 验证输出是否仅包含你在本实验室中创建的资源组。这些组将在下一个任务中删除。

#### 任务 2：删除资源组

1. 在 **Cloud Shell** 命令提示符处，键入以下命令并按 **Enter** 键以删除你在本实验室中创建的资源组

```sh
   az group list --query "[?starts_with(name,'az3000301')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
```

1. 关闭门户底部的 **Cloud Shell** 提示符。

> **结果**：在本练习中，你删除了本实验室中使用的资源。
