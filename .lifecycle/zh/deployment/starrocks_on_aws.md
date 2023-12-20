---
displayed_sidebar: English
---

# 在 AWS 上部署 StarRocks

StarRocks 与 AWS 提供了 [AWS 合作伙伴解决方案](https://aws.amazon.com/solutions/partners)，可以快速在 AWS 上部署 StarRocks。本主题将提供逐步指导，帮助您部署并访问 StarRocks。

## 基础概念

[AWS 合作伙伴解决方案](https://aws-ia.github.io/content/qs_info.html)

AWS 合作伙伴解决方案是由 AWS 解决方案架构师和 AWS 合作伙伴共同构建的自动化参考部署。AWS 合作伙伴解决方案使用[AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)模板，自动部署 AWS 资源和第三方资源，例如 StarRocks 集群，到 AWS 云上。

[模板](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-whatis-concepts.html#w2aab5c15b7)

模板是用 JSON 或 YAML 格式编写的文本文件，描述了 AWS 资源和第三方资源以及这些资源的属性。

[堆栈](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-whatis-concepts.html#w2ab1b5c15b9)

堆栈用于创建和管理模板中描述的资源。您可以通过创建、更新和删除堆栈来创建、更新和删除资源集合。

堆栈中的所有资源都由模板定义。假设您已经创建了描述多种资源的模板。为了配置这些资源，您需要提交您创建的模板来创建堆栈，然后 AWS CloudFormation 会帮您配置所有这些资源。

## 部署 StarRocks 集群

1. 登录到您的 [AWS 账户](https://console.aws.amazon.com/console/home)。如果您还没有账户，请在 [AWS](https://aws.amazon.com/) 上注册。

2. 从顶部工具栏选择 AWS 区域。

3. 选择一个部署选项以启动这个[合作伙伴解决方案](https://aws.amazon.com/quickstart/architecture/starrocks-starrocks/)。AWS CloudFormation 控制台将打开，并显示一个预先填充的模板，用于部署一个 StarRocks 集群，其中包括一个 FE 和三个 BE。部署大约需要 30 分钟完成。

   1. 将[StarRocks部署到新建的VPC](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fus-east-1.console.aws.amazon.com%2Fcloudformation%2Fhome%3Fregion%3Dus-east-1%26state%3DhashArgs%2523%252Fstacks%252Fnew%253FstackName%253Dstarrocks-starrocks%2526templateURL%253Dhttps%253A%252F%252Faws-quickstart.s3.us-east-1.amazonaws.com%252Fquickstart-starrocks-starrocks%252Ftemplates%252Fstarrocks-entrypoint-new-vpc.template.yaml%26isauthcode%3Dtrue&client_id=arn%3Aaws%3Aiam%3A%3A015428540659%3Auser%2Fcloudformation&forceMobileApp=0&code_challenge=yo-6I1O2W0f0VcoqYOVvSwMmhRkC7Vod1M9vWbiMWUM&code_challenge_method=SHA-256)。该选项会构建一个包含VPC、子网、NAT网关、安全组、堡垒主机等基础设施组件的新AWS环境，并将StarRocks部署到这个新VPC中。
   2. 将[StarRocks部署到现有的VPC](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fus-east-1.console.aws.amazon.com%2Fcloudformation%2Fhome%3Fregion%3Dus-east-1%26state%3DhashArgs%2523%252Fstacks%252Fnew%253FstackName%253Dstarrocks-starrocks%2526templateURL%253Dhttps%253A%252F%252Faws-quickstart.s3.us-east-1.amazonaws.com%252Fquickstart-starrocks-starrocks%252Ftemplates%252Fstarrocks-entrypoint-existing-vpc.template.yaml%26isauthcode%3Dtrue&client_id=arn%3Aaws%3Aiam%3A%3A015428540659%3Auser%2Fcloudformation&forceMobileApp=0&code_challenge=dDa178BxB6UkFfrpADw5CIoZ4yDUNRTG7sNM1EO__eo&code_challenge_method=SHA-256)。该选项会在您现有的AWS基础架构中配置StarRocks。

4. 选择正确的 AWS 区域。

5. 在**创建堆栈**页面上，保持模板 URL 的默认设置，然后选择**下一步**。

6. 在**指定堆栈详细信息**页面上

1.    如果需要，自定义堆栈名称。

2.    配置并审查模板的参数。

1.       配置所需参数。

-          当您选择将 StarRocks 部署到新建的 VPC 时，请关注以下参数：

           |类型|参数|必需|说明|
|---|---|---|---|
           |网络配置|可用区|是|选择两个可用区来部署StarRocks 集群。有关更多信息，请参阅区域和可用区。|
           |EC2 配置|密钥对名称|是|输入密钥对，由公钥和私钥组成，是一组安全凭证，用于在连接 EC2 实例时证明您的身份。有关详细信息，请参阅密钥对。 > 注意>> 如果需要创建密钥对，请参见创建密钥对。|
           |StarRocks 集群配置|Starrock 的根密码|是|输入您的 StarRocks 根帐户的密码。使用 root 帐户连接 StarRocks 集群时需要提供密码。|
           |确认 Root 密码|是|确认您的 StarRocks root 帐户的密码。|

-          当您选择将 StarRocks 部署到现有的 VPC 时，请关注以下参数：

           |类型|参数|必需|说明|
|---|---|---|---|
           |网络配置|VPC ID|是|输入您现有VPC的ID。确保您为 AWS S3 配置 VPC 终端节点。|
           |私有子网 1 ID|是|输入现有 VPC 的可用区 1 中私有子网的 ID（例如，subnet-fe9a8b32）。|
           |公有子网 1 ID|是|输入现有 VPC 可用区 1 中的公有子网 ID。|
           |公有子网 2 ID |是|输入现有 VPC 可用区 2 中公有子网的 ID。|
           |EC2 配置|密钥对名称|是|输入密钥对，由公钥和私钥组成，是一组安全凭证，用于在连接 EC2 实例时证明您的身份。有关详细信息，请参阅密钥对。说明 如果您需要创建密钥对，请参见创建密钥对。|
           |StarRocks 集群配置|Starrock 的根密码|是|输入您的 StarRocks 根帐户的密码。使用 root 帐户连接 StarRocks 集群时需要提供密码。|
           |确认 Root 密码|是|确认您的 StarRocks root 帐户的密码。|

2.       对于其他参数，请查看默认设置并根据需要进行自定义。

3.    当你完成配置和审查参数后，选择**下一步**。

7. 在**配置堆栈选项**页面上，保持默认设置，点击**下一步**。

8. 在“审查 **starrocks-starrocks**”页面上，审查您配置的堆栈信息，包括模板、详细信息及其他选项。更多信息请参见 [AWS CloudFormation 控制台创建堆栈时审查堆栈并估算成本](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-using-console-create-stack-review.html)。

      > **注意**
      > 如果您需要更改任何参数，请点击**编辑**按钮返回到相关页面。

9. 勾选以下两个复选框并点击**创建堆栈**。

   ![StarRocks_on_AWS_1](../assets/StarRocks_on_AWS_1.png)

   **请注意，您需要承担运行此合作伙伴解决方案时所使用的 AWS 服务和任何第三方许可证的费用。**成本估算请参考您所使用的每项 AWS 服务的定价页面。

## 访问 StarRocks 集群

由于 StarRocks 集群部署在私有子网中，您需要首先连接到 EC2 堡垒主机，然后才能访问 StarRocks 集群。

1. 连接到用于访问 StarRocks 集群的 EC2 堡垒主机。

1.    在 AWS CloudFormation 控制台的 **Outputs** 选项卡上，记下 `BastionStack` 的值为 `EIP1`。![StarRocks_on_AWS_2](../assets/StarRocks_on_AWS_2.png)

2.    在 EC2 控制台，选择 EC2 堡垒主机。
![StarRocks_on_AWS_3](../assets/StarRocks_on_AWS_3.png)

3.    编辑与 EC2 堡垒主机关联的安全组的入站规则，允许您的计算机到 EC2 堡垒主机的流量。

4.    连接到 EC2 堡垒主机。

2. 访问 StarRocks 集群

1.    在 EC2 堡垒主机上安装 MySQL。

2.    使用以下命令连接到 StarRocks 集群：

      ```Bash
      mysql -u root -h 10.0.xx.xx -P 9030 -p
      ```

-       host：您可以通过以下步骤找到 FE 的私有 IP 地址：

1.         从AWS CloudFormation控制台，转到`StarRocksClusterStack`的**输出**选项卡，点击`FeLeaderInstance`的值。
![StarRocks_on_AWS_4](../assets/StarRocks_on_AWS_4.png)

2.         在实例概要页面，找到 FE 的私有 IP 地址。
![StarRocks_on_AWS_5](../assets/StarRocks_on_AWS_5.png)

-       password：输入您在第 5 步中配置的密码。
