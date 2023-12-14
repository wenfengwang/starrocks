---
displayed_sidebar: "中文"
---

# 在AWS上部署StarRocks

StarRocks和AWS提供[AWS合作伙伴解决方案](https://aws.amazon.com/solutions/partners) 来快速在AWS上部署StarRocks。本主题提供了分步说明，帮助您部署和访问StarRocks。

## 基本概念

[AWS合作伙伴解决方案](https://aws-ia.github.io/content/qs_info.html)

AWS合作伙伴解决方案是由AWS解决方案架构师和AWS合作伙伴自动化构建的参考部署。AWS合作伙伴解决方案使用[AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html) 模板，自动在AWS云上部署AWS资源和第三方资源，例如StarRocks集群。

[模板](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-whatis-concepts.html#w2aab5c15b7)

模板是JSON或YAML格式的文本文件，描述AWS资源和第三方资源，以及这些资源的属性。

[堆栈](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-whatis-concepts.html#w2ab1b5c15b9)

堆栈用于创建和管理模板中描述的资源。您可以通过创建、更新和删除堆栈来创建、更新和删除一组资源。

堆栈中的所有资源由模板定义。假设您已创建了描述各种资源的模板。要配置这些资源，您需要通过提交所创建的模板来创建一个堆栈，然后AWS CloudFormation会为您配置所有这些资源。

## 部署StarRocks集群

1. 登录到[您的AWS账户](https://console.aws.amazon.com/console/home)。如果您还没有账户，请在[AWS](https://aws.amazon.com/)上注册。

2. 从顶部工具栏选择AWS区域。

3. 选择启动此[合作伙伴解决方案](https://aws.amazon.com/quickstart/architecture/starrocks-starrocks/) 的部署选项。 AWS CloudFormation控制台会打开一个预填充的模板，用于部署一个包含一个FE和三个BE的StarRocks集群。部署大约需要30分钟完成。

   1. [将StarRocks部署到新VPC中](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fus-east-1.console.aws.amazon.com%2Fcloudformation%2Fhome%3Fregion%3Dus-east-1%26state%3DhashArgs%2523%252Fstacks%252Fnew%253FstackName%253Dstarrocks-starrocks%2526templateURL%253Dhttps%253A%252F%252Faws-quickstart.s3.us-east-1.amazonaws.com%252Fquickstart-starrocks-starrocks%252Ftemplates%252Fstarrocks-entrypoint-new-vpc.template.yaml%26isauthcode%3Dtrue&client_id=arn%3Aaws%3Aiam%3A%3A015428540659%3Auser%2Fcloudformation&forceMobileApp=0&code_challenge=yo-6I1O2W0f0VcoqYOVvSwMmhRkC7Vod1M9vWbiMWUM&code_challenge_method=SHA-256)。此选项会构建一个包含VPC、子网、NAT网关、安全组、跳板主机和其他基础设施组件的新AWS环境。然后将StarRocks部署到这个新VPC中。
   2. [将StarRocks部署到现有VPC中](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fus-east-1.console.aws.amazon.com%2Fcloudformation%2Fhome%3Fregion%3Dus-east-1%26state%3DhashArgs%2523%252Fstacks%252Fnew%253FstackName%253Dstarrocks-starrocks%2526templateURL%253Dhttps%253A%252F%252Faws-quickstart.s3.us-east-1.amazonaws.com%252Fquickstart-starrocks-starrocks%252Ftemplates%252Fstarrocks-entrypoint-existing-vpc.template.yaml%26isauthcode%3Dtrue&client_id=arn%3Aaws%3Aiam%3A%3A015428540659%3Auser%2Fcloudformation&forceMobileApp=0&code_challenge=dDa178BxB6UkFfrpADw5CIoZ4yDUNRTG7sNM1EO__eo&code_challenge_method=SHA-256)。此选项会在您现有的AWS基础架构中配置StarRocks。

4. 选择正确的AWS区域。

5. 在**创建堆栈**页面上，保留模板URL的默认设置，然后选择 **下一步**。

6. 在**指定堆栈详细信息**页面上

   1. 如果需要，自定义堆栈名称。

   2. 配置并审查模板的参数。

      1. 配置所需的参数。

         - 当您选择将StarRocks部署到新VPC中时，请注意以下参数：

             | 类型           | 参数                   | 是否必需                     | 描述                                                             |
             | -------------- | -------------------- | ---------------------------- | -------------------------------------------------------------------- |
             | 网络配置       | 可用区                | 是                           | 选择两个可用区以部署StarRocks集群。有关更多信息，请参见[区域和区域](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html#concepts-local-zones)。 |
             | EC2配置        | 密钥对名称           | 是                           | 输入由公钥和私钥组成的密钥对，是您在连接到EC2实例时用于验证身份的一组安全凭证。有关更多信息，请参见[密钥对](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)。<br /> **注意** <br /> 如果需要创建密钥对，请参见[创建密钥对](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html)。 |
             | StarRocks集群配置 | StarRocks的Root密码   | 是                           | 输入StarRocks根账户的密码。连接StarRocks集群时，您需要提供密码以使用根账户。 |
             | |确认Root密码          | 是                           | 确认StarRocks根账户的密码。                                        |

         - 当您选择将StarRocks部署到现有VPC中时，请注意以下参数：

             | 类型           | 参数                   | 是否必需                                           | 描述                                                             |
             | -------------- | -------------------- | -------------------------------------------------- | ---------------------------------------------------------------- |
             | 网络配置       | VPC ID                | 是                                                 | 输入现有VPC的ID。确保 [为AWS S3配置VPC端点](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-s3.html)。 |
             | |私有子网1 ID          | 是                                                 | 输入现有VPC可用区1中的私有子网ID（例如，subnet-fe9a8b32）。                 |
             | |公共子网1 ID          | 是                                                 | 输入现有VPC可用区1中的公共子网ID。                                       |
             | |公共子网2 ID          | 是                                                 | 输入现有VPC可用区2中的公共子网ID。                                       |
             | EC2配置        | 密钥对名称           | 是                                                 | 输入由公钥和私钥组成的密钥对，是您在连接到EC2实例时用于验证身份的一组安全凭证。有关更多信息，请参见[密钥对](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)。<br /> **注意** <br /> 如果需要创建密钥对，请参见[创建密钥对](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html)。 |
             | StarRocks集群配置 | StarRocks的Root密码   | 是                                                 | 输入StarRocks根账户的密码。连接StarRocks集群时，您需要提供密码以使用根账户。 |
             | |确认Root密码          | 是                                                 | 确认StarRocks根账户的密码。                                         |

      2. 对于其他参数，请检查默认设置并根据需要进行自定义。

   3. 完成配置和审查参数后，选择 **下一步**。

7. 在**配置堆栈选项**页面上，保留默认设置，并单击**下一步**。

8. 在**Review starrocks-starrocks**页面上，审查上述配置的堆栈信息，包括模板、详细信息和更多选项。有关更多信息，请参见[AWS CloudFormation控制台上审查堆栈并估算堆栈成本](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-using-console-create-stack-review.html)。

    > **注意**
    >
    > 如果需要更改任何参数，请单击相关部分右上角的**编辑**。

9. 选择以下两个复选框，然后单击**创建堆栈**。

    ![StarRocks_on_AWS_1](../assets/StarRocks_on_AWS_1.png)

    **请注意，您需要承担运行此合作伙伴解决方案时的AWS服务和任何第三方许可证的成本**。有关成本估算，请参考您使用的每个AWS服务的定价页面。

## 访问StarRocks集群

由于StarRocks集群部署在私有子网中，您需要首先连接到一个EC2堡垒主机，然后访问StarRocks集群。

1. 连接到用于访问StarRocks集群的EC2堡垒主机。

   1. 从AWS CloudFormation控制台，在`BastionStack`的**输出**选项卡上，记下`EIP1`的值。
   ![StarRocks_on_AWS_2](../assets/StarRocks_on_AWS_2.png)

   2. 从EC2控制台，选择EC2堡垒主机。
   ![StarRocks_on_AWS_3](../assets/StarRocks_on_AWS_3.png)

   3. 编辑与EC2堡垒主机关联的安全组的入站规则，以允许来自您的计算机的流量到达EC2堡垒主机。

   4. 连接到EC2堡垒主机。

2. 访问StarRocks集群

   1. 在EC2堡垒主机上安装MySQL。

   2. 使用以下命令连接到StarRocks集群：

      ```Bash
      mysql -u root -h 10.0.xx.xx -P 9030 -p
      ```

      - 主机：
        您可以按照以下步骤找到FE的私有IP地址：

        1. 从AWS CloudFormation控制台，在`StarRocksClusterStack`的**输出**选项卡上，单击`FeLeaderInstance`的值。
        ![StarRocks_on_AWS_4](../assets/StarRocks_on_AWS_4.png)

        2. 从实例摘要页面，找到FE的私有IP地址。
        ![StarRocks_on_AWS_5](../assets/StarRocks_on_AWS_5.png)

      - 密码：输入您在第5步中配置的密码。