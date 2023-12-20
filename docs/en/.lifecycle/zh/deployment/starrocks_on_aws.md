---
displayed_sidebar: English
---

# 在 AWS 上部署 StarRocks

StarRocks 和 AWS 提供 [AWS 合作伙伴解决方案](https://aws.amazon.com/solutions/partners) 来快速在 AWS 上部署 StarRocks。本主题提供逐步指导，帮助您部署和访问 StarRocks。

## 基本概念

[AWS 合作伙伴解决方案](https://aws-ia.github.io/content/qs_info.html)

AWS 合作伙伴解决方案是由 AWS 解决方案架构师和 AWS 合作伙伴构建的自动化参考部署。AWS 合作伙伴解决方案使用 [AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html) 模板，自动部署 AWS 资源和第三方资源，例如在 AWS 云上的 StarRocks 集群。

[模板](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-whatis-concepts.html#w2aab5c15b7)

模板是描述 AWS 资源和第三方资源及其属性的 JSON 或 YAML 格式文本文件。

[堆栈](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-whatis-concepts.html#w2ab1b5c15b9)

堆栈用于创建和管理模板中描述的资源。您可以通过创建、更新和删除堆栈来创建、更新和删除资源集。

所有堆栈中的资源都由模板定义。假设您已创建了描述各种资源的模板。要配置这些资源，您需要提交您创建的模板来创建堆栈，AWS CloudFormation 会为您配置所有这些资源。

## 部署 StarRocks 集群

1. 登录到[您的 AWS 账户](https://console.aws.amazon.com/console/home)。如果您没有账户，请在 [AWS](https://aws.amazon.com/) 上注册。

2. 从顶部工具栏选择 AWS 区域。

3. 选择一个部署选项以启动此[合作伙伴解决方案](https://aws.amazon.com/quickstart/architecture/starrocks-starrocks/)。AWS CloudFormation 控制台会打开一个预填充的模板，用于部署一个包含一个 FE 和三个 BE 的 StarRocks 集群。部署大约需要 30 分钟完成。

   1. [将 StarRocks 部署到新的 VPC](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fus-east-1.console.aws.amazon.com%2Fcloudformation%2Fhome%3Fregion%3Dus-east-1%26state%3DhashArgs%2523%252Fstacks%252Fnew%253FstackName%253Dstarrocks-starrocks%2526templateURL%253Dhttps%253A%252F%252Faws-quickstart.s3.us-east-1.amazonaws.com%252Fquickstart-starrocks-starrocks%252Ftemplates%252Fstarrocks-entrypoint-new-vpc.template.yaml%26isauthcode%3Dtrue&client_id=arn%3Aaws%3Aiam%3A%3A015428540659%3Auser%2Fcloudformation&forceMobileApp=0&code_challenge=yo-6I1O2W0f0VcoqYOVvSwMmhRkC7Vod1M9vWbiMWUM&code_challenge_method=SHA-256)。此选项构建一个包含 VPC、子网、NAT 网关、安全组、堡垒主机等基础设施组件的新 AWS 环境，并将 StarRocks 部署到这个新 VPC 中。
   2. [将 StarRocks 部署到现有 VPC](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fus-east-1.console.aws.amazon.com%2Fcloudformation%2Fhome%3Fregion%3Dus-east-1%26state%3DhashArgs%2523%252Fstacks%252Fnew%253FstackName%253Dstarrocks-starrocks%2526templateURL%253Dhttps%253A%252F%252Faws-quickstart.s3.us-east-1.amazonaws.com%252Fquickstart-starrocks-starrocks%252Ftemplates%252Fstarrocks-entrypoint-existing-vpc.template.yaml%26isauthcode%3Dtrue&client_id=arn%3Aaws%3Aiam%3A%3A015428540659%3Auser%2Fcloudformation&forceMobileApp=0&code_challenge=dDa178BxB6UkFfrpADw5CIoZ4yDUNRTG7sNM1EO__eo&code_challenge_method=SHA-256)。此选项在您现有的 AWS 基础架构中部署 StarRocks。

4. 选择正确的 AWS 区域。

5. 在 **创建堆栈** 页面，保持模板 URL 的默认设置，然后选择 **下一步**。

6. 在 **指定堆栈详细信息** 页面

    1. 如有需要，自定义堆栈名称。

    2. 配置并审查模板的参数。

        1. 配置所需参数。

            - 当您选择将 StarRocks 部署到新的 VPC 时，请注意以下参数：

              |类型|参数|必需|说明|
              |---|---|---|---|
              |网络配置|可用区|是|选择两个可用区来部署 StarRocks 集群。有关更多信息，请参见[区域和可用区](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html#concepts-local-zones)。|
              |EC2 配置|密钥对名称|是|输入密钥对，由公钥和私钥组成，是一组安全凭证，用于连接 EC2 实例时证明您的身份。有关更多信息，请参见[密钥对](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)。> 注意 > 如果您需要创建密钥对，请参见[创建密钥对](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html)。|
              |StarRocks 集群配置|StarRocks 根密码|是|输入您的 StarRocks 根账户密码。使用 root 账户连接 StarRocks 集群时需要提供密码。|
              |确认根密码|是|确认您的 StarRocks 根账户密码。|

            - 当您选择将 StarRocks 部署到现有 VPC 时，请注意以下参数：

              |类型|参数|必需|说明|
              |---|---|---|---|
              |网络配置|VPC ID|是|输入您现有 VPC 的 ID。确保您[为 AWS S3 配置 VPC 终端](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-s3.html)。|
              |私有子网 1 ID|是|输入您现有 VPC 的可用区 1 中的私有子网 ID（例如，subnet-fe9a8b32）。|
              |公有子网 1 ID|是|输入您现有 VPC 的可用区 1 中的公有子网 ID。|
              |公有子网 2 ID|是|输入您现有 VPC 的可用区 2 中的公有子网 ID。|
              |EC2 配置|密钥对名称|是|输入密钥对，由公钥和私钥组成，是一组安全凭证，用于连接 EC2 实例时证明您的身份。有关更多信息，请参见[密钥对](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)。<br />**注意**<br />如果您需要创建密钥对，请参见[创建密钥对](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html)。|
              |StarRocks 集群配置|StarRocks 根密码|是|输入您的 StarRocks 根账户密码。使用 root 账户连接 StarRocks 集群时需要提供密码。|
              |确认根密码|是|确认您的 StarRocks 根账户密码。|

        2. 对于其他参数，审查默认设置并根据需要进行自定义。

        3. 完成配置和审查参数后，选择 **下一步**。

7. 在 **配置堆栈选项** 页面，保持默认设置，然后点击 **下一步**。

8. 在 **Review starrocks-starrocks** 页面，审查上述配置的堆栈信息，包括模板、详细信息和其他选项。有关更多信息，请参阅[在 AWS CloudFormation 控制台上审查堆栈并估算堆栈成本](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-using-console-create-stack-review.html)。

      > **注意**
      > 如果您需要更改任何参数，请点击相关部分右上角的 **编辑** 按钮返回相关页面。

9. 勾选以下两个复选框并点击 **创建堆栈**。

   ![StarRocks_on_AWS_1](../assets/StarRocks_on_AWS_1.png)

   **请注意**，您需要承担运行此合作伙伴解决方案时使用的 AWS 服务和任何第三方许可证的费用。有关成本估算，请参考您使用的每项 AWS 服务的定价页面。

## 访问 StarRocks 集群

由于 StarRocks 集群部署在私有子网中，您需要先连接到 EC2 Bastion 主机，然后才能访问 StarRocks 集群。

1. 连接到用于访问 StarRocks 集群的 EC2 Bastion 主机。

    1. 在 AWS CloudFormation 控制台的 `BastionStack` 的 **输出** 选项卡上，记下 `EIP1` 的值。
       ![StarRocks_on_AWS_2](../assets/StarRocks_on_AWS_2.png)

    2. 从 EC2 控制台选择 EC2 Bastion 主机。
       ![StarRocks_on_AWS_3](../assets/StarRocks_on_AWS_3.png)

    3. 编辑与 EC2 Bastion 主机关联的安全组的入站规则，允许从您的计算机到 EC2 Bastion 主机的流量。

    4. 连接到 EC2 Bastion 主机。

2. 访问 StarRocks 集群

    1. 在 EC2 Bastion 主机上安装 MySQL。

    2. 使用以下命令连接 StarRocks 集群：

       ```Bash
       mysql -u root -h 10.0.xx.xx -P 9030 -p
       ```

        - host：您可以按照以下步骤找到 FE 的私有 IP 地址：

            1. 在 AWS CloudFormation 控制台的 `StarRocksClusterStack` 的 **输出** 选项卡中，点击 `FeLeaderInstance` 的值。
               ![StarRocks_on_AWS_4](../assets/StarRocks_on_AWS_4.png)

            2. 从实例摘要页面找到 FE 的私有 IP 地址。
               ![StarRocks_on_AWS_5](../assets/StarRocks_on_AWS_5.png)

        - password：输入您在步骤 6 中配置的密码。