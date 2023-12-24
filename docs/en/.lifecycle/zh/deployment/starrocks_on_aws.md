---
displayed_sidebar: English
---

# 在 AWS 上部署 StarRocks

StarRocks 和 AWS 提供 [AWS 合作伙伴解决方案](https://aws.amazon.com/solutions/partners)，帮助您快速在 AWS 上部署 StarRocks。本主题提供逐步说明，帮助您部署和访问 StarRocks。

## 基本概念

[AWS 合作伙伴解决方案](https://aws-ia.github.io/content/qs_info.html)

AWS 合作伙伴解决方案是由 AWS 解决方案架构师和 AWS 合作伙伴构建的自动化参考部署。AWS 合作伙伴解决方案使用 [AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html) 模板，在 AWS 云上自动部署 AWS 资源和第三方资源，例如 StarRocks 集群。

[模板](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-whatis-concepts.html#w2aab5c15b7)

模板是 JSON 或 YAML 格式的文本文件，用于描述 AWS 资源和第三方资源，以及这些资源的属性。

[堆栈](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-whatis-concepts.html#w2ab1b5c15b9)

堆栈用于创建和管理模板中描述的资源。您可以通过创建、更新和删除堆栈来创建、更新和删除一组资源。

堆栈中的所有资源都由模板定义。假设您已经创建了一个描述各种资源的模板。要配置这些资源，您需要通过提交您创建的模板来创建堆栈，然后 AWS CloudFormation 会为您配置所有这些资源。

## 部署 StarRocks 集群

1. 登录[您的 AWS 账户](https://console.aws.amazon.com/console/home)。如果您没有账户，请在 [AWS](https://aws.amazon.com/) 上注册。

2. 从顶部工具栏中选择 AWS 区域。

3. 选择一个部署选项以启动此 [合作伙伴解决方案](https://aws.amazon.com/quickstart/architecture/starrocks-starrocks/)。AWS CloudFormation 控制台将打开一个预填充的模板，用于部署一个包含 1 个 FE 和 3 个 BE 的 StarRocks 集群。部署大约需要 30 分钟才能完成。

   1. [将 StarRocks 部署到新的 VPC 中](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fus-east-1.console.aws.amazon.com%2Fcloudformation%2Fhome%3Fregion%3Dus-east-1%26state%3DhashArgs%2523%252Fstacks%252Fnew%253FstackName%253Dstarrocks-starrocks%2526templateURL%253Dhttps%253A%252F%252Faws-quickstart.s3.us-east-1.amazonaws.com%252Fquickstart-starrocks-starrocks%252Ftemplates%252Fstarrocks-entrypoint-new-vpc.template.yaml%26isauthcode%3Dtrue&client_id=arn%3Aaws%3Aiam%3A%3A015428540659%3Auser%2Fcloudformation&forceMobileApp=0&code_challenge=yo-6I1O2W0f0VcoqYOVvSwMmhRkC7Vod1M9vWbiMWUM&code_challenge_method=SHA-256)。此选项将构建一个新的 AWS 环境，该环境由 VPC、子网、NAT 网关、安全组、堡垒主机和其他基础设施组件组成。然后，它将 StarRocks 部署到这个新 VPC 中。
   2. [将 StarRocks 部署到已有的 VPC 中](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fus-east-1.console.aws.amazon.com%2Fcloudformation%2Fhome%3Fregion%3Dus-east-1%26state%3DhashArgs%2523%252Fstacks%252Fnew%253FstackName%253Dstarrocks-starrocks%2526templateURL%253Dhttps%253A%252F%252Faws-quickstart.s3.us-east-1.amazonaws.com%252Fquickstart-starrocks-starrocks%252Ftemplates%252Fstarrocks-entrypoint-existing-vpc.template.yaml%26isauthcode%3Dtrue&client_id=arn%3Aaws%3Aiam%3A%3A015428540659%3Auser%2Fcloudformation&forceMobileApp=0&code_challenge=dDa178BxB6UkFfrpADw5CIoZ4yDUNRTG7sNM1EO__eo&code_challenge_method=SHA-256)。此选项可在您现有的 AWS 基础设施中预置 StarRocks。

4. 选择正确的 AWS 区域。

5. 在 **Create stack** 页面上，保留模板 URL 的默认设置，然后选择 **Next**。

6. 在 **Specify stack details** 页面上

   1. 如果需要，自定义堆栈名称。

   2. 配置并查看模板的参数。

      1. 配置所需参数。

         - 当您选择将 StarRocks 部署到新 VPC 中时，需要注意以下参数：

             | 类型                            | 参数                 | 必填                                             | 描述                                                  |
             | ------------------------------- | ------------------------- | ---------------------------------------------------- | ------------------------------------------------------------ |
             | 网络配置           | 可用区        | 是                                                  | 选择两个可用区部署 StarRocks 集群。有关更多信息，请参阅 [区域和可用区](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html#concepts-local-zones)。 |
             | EC2 配置               | 密钥对名称             | 是                                                  | 输入密钥对（由公有密钥和私有密钥组成）是一组安全凭证，用于在连接到 EC2 实例时证明您的身份。有关更多信息，请参阅[密钥对](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)。 >注意事项 > > 如果您需要创建密钥对，请参见创建密钥对。 [](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html) |
             | StarRocks 集群配置 | Starrock 的 root 密码 | 是                                                  | 输入 StarRocks root 账号的密码。连接 StarRocks 集群时，需要提供密码。 |
             |  |确认 Root 密码           | 是                       | 确认 StarRocks root 账号的密码。 |                                                              |

         - 当您选择将 StarRocks 部署到已有 VPC 中时，需要注意以下参数：

           | 类型                            | 参数                 | 必填                                                     | 描述                                                  |
           | ------------------------------- | ------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
           | 网络配置           | VPC ID                    | 是                                                          | 输入现有 VPC 的 ID。确保您 [为 AWS S3 配置 VPC 终端节点](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-s3.html)。 |
           | |私有子网 1 ID             | 是                       | 输入现有 VPC 的可用区 1 中的私有子网的 ID（例如，subnet-fe9a8b32）。 |                                                              |
           | |公有子网 1 ID              | 是                       | 输入现有 VPC 的可用区 1 中的公有子网的 ID。 |                                                              |
           | |公有子网 2 ID              | 是                       | 输入现有 VPC 的可用区 2 中的公有子网的 ID。 |                                                              |
           | EC2 配置               | 密钥对名称             | 是                                                          | 输入密钥对（由公有密钥和私有密钥组成）是一组安全凭证，用于在连接到 EC2 实例时证明您的身份。有关更多信息，请参阅[密钥对](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)。 <br /> **说明** <br /> 如果您需要创建密钥对，请参见创建[密钥对](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html)。 |
           | StarRocks 集群配置 | Starrock 的 root 密码 | 是                                                          | 输入 StarRocks root 账号的密码。连接 StarRocks 集群时，需要提供密码。 |
           | |确认 Root 密码           | 是                       | 确认 StarRocks root 账号的密码。         |                                                              |

      2. 对于其他参数，请查看默认设置并根据需要对其进行自定义。

   3. 完成配置和查看参数后，选择 **Next**。

7. 在 **Configure stack options** 页面上，保留默认设置，然后单击 **Next**。

8. 在 **Review starrocks-starrocks** 页面上，查看上面配置的堆栈信息，包括模板、详细信息和更多选项。有关更多信息，请参阅 [在 AWS CloudFormation 控制台上查看堆栈和估算堆栈成本](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-using-console-create-stack-review.html)。

    > **注意**
    >
    > 如果您需要更改任何参数，请单击相关版块右上角的 **Edit**，返回相关页面。

9. 选中以下两个复选框，然后单击 **Create stack**。

    ![StarRocks_on_AWS_1](../assets/StarRocks_on_AWS_1.png)

    **请注意，您需要承担** 运行此合作伙伴解决方案时使用的 AWS 服务和任何第三方许可证的费用。有关成本估算，请参阅您使用的每项 AWS 服务的定价页面。

## 访问 StarRocks 集群

由于 StarRocks 集群部署在私有子网中，因此您需要先连接到 EC2 堡垒机，然后再访问 StarRocks 集群。

1. 连接用于访问 StarRocks 集群的 EC2 堡垒机。

   1. 在 AWS CloudFormation 控制台的 **Outputs** 选项卡`BastionStack`上，记下 `EIP1` 的值。
   ![StarRocks_on_AWS_2](../assets/StarRocks_on_AWS_2.png)

   2. 从 EC2 控制台中，选择 EC2 堡垒主机。
   ![StarRocks_on_AWS_3](../assets/StarRocks_on_AWS_3.png)

   3. 编辑与 EC2 堡垒主机关联的安全组的入站规则，以允许从您的计算机到 EC2 堡垒主机的流量。

   4. 连接到 EC2 堡垒主机。

2. 访问 StarRocks 集群

   1. 在 EC2 堡垒主机上安装 MySQL。

   2. 使用以下命令连接 StarRocks 集群：

      ```Bash
      mysql -u root -h 10.0.xx.xx -P 9030 -p
      ```

      - 主机：
        您可以按照以下步骤找到 FE 的私有 IP 地址：

        1. 在 AWS CloudFormation 控制台的 **Outputs** 选项卡`StarRocksClusterStack`上，单击 `FeLeaderInstance` 的值。
        ![StarRocks_on_AWS_4](../assets/StarRocks_on_AWS_4.png)

        2. 在实例摘要页面中，找到 FE 的私有 IP 地址。
        ![StarRocks_on_AWS_5](../assets/StarRocks_on_AWS_5.png)

      - 密码：输入您在步骤 5 中配置的密码。
