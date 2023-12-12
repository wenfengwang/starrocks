---
displayed_sidebar: "Japanese"
---

# AWSでStarRocksを展開する

StarRocksとAWSは、StarRocksを迅速に展開するための[AWSパートナーソリューション](https://aws.amazon.com/solutions/partners)を提供しています。このトピックでは、StarRocksを展開しアクセスするための手順をステップバイステップで説明します。

## 基本的な概念

[AWSパートナーソリューション](https://aws-ia.github.io/content/qs_info.html)

AWSパートナーソリューションは、AWSのソリューションアーキテクトやAWSパートナーが作成した自動化されたリファレンス展開です。AWSパートナーソリューションは[AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)テンプレートを使用して、AWSリソースやStarRocksクラスターなどのサードパーティリソースをAWS Cloud上に自動的に展開します。

[テンプレート](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-whatis-concepts.html#w2aab5c15b7)

テンプレートは、AWSリソースやサードパーティリソース、およびそれらのリソースのプロパティを記述したJSONまたはYAML形式のテキストファイルです。

[スタック](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-whatis-concepts.html#w2ab1b5c15b9)

スタックは、テンプレートで記述されたリソースを作成および管理するために使用されます。スタックを作成、更新、削除することで、リソースのセットを作成、更新、削除できます。

スタック内のすべてのリソースはテンプレートによって定義されます。たとえば、様々なリソースを記述したテンプレートを作成した場合、これらのリソースを構成するためには、そのテンプレートを提出してスタックを作成し、AWS CloudFormationがそれらのリソースをすべて構成します。

## StarRocksクラスターを展開する

1. [AWSアカウントにサインイン](https://console.aws.amazon.com/console/home)します。アカウントをお持ちでない場合は、[AWS](https://aws.amazon.com/)でサインアップしてください。

2. 上部のツールバーからAWSリージョンを選択します。

3. この[パートナーソリューション](https://aws.amazon.com/quickstart/architecture/starrocks-starrocks/)を展開するためのデプロイオプションを選択します。AWS CloudFormationコンソールが開き、StarRocksクラスターを1つのFEと3つのBEで展開するために使用される事前設定のテンプレートが表示されます。展開には約30分かかります。

   1. [新しいVPCにStarRocksを展開](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fus-east-1.console.aws.amazon.com%2Fcloudformation%2Fhome%3Fregion%3Dus-east-1%26state%3DhashArgs%2523%252Fstacks%252Fnew%253FstackName%253Dstarrocks-starrocks%2526templateURL%253Dhttps%253A%252F%252Faws-quickstart.s3.us-east-1.amazonaws.com%252Fquickstart-starrocks-starrocks%252Ftemplates%252Fstarrocks-entrypoint-new-vpc.template.yaml%26isauthcode%3Dtrue&client_id=arn%3Aaws%3Aiam%3A%3A015428540659%3Auser%2Fcloudformation&forceMobileApp=0&code_challenge=yo-6I1O2W0f0VcoqYOVvSwMmhRkC7Vod1M9vWbiMWUM&code_challenge_method=SHA-256)を選択します。このオプションでは、新しいAWS環境が構築され、VPC、サブネット、NATゲートウェイ、セキュリティグループ、バスチョンホスト、およびその他のインフラコンポーネントから構成されます。その後、この新しいVPCにStarRocksが展開されます。
   2. [既存のVPCにStarRocksを展開](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fus-east-1.console.aws.amazon.com%2Fcloudformation%2Fhome%3Fregion%3Dus-east-1%26state%3DhashArgs%2523%252Fstacks%252Fnew%253FstackName%253Dstarrocks-starrocks%2526templateURL%253Dhttps%253A%252F%252Faws-quickstart.s3.us-east-1.amazonaws.com%252Fquickstart-starrocks-starrocks%252Ftemplates%252Fstarrocks-entrypoint-existing-vpc.template.yaml%26isauthcode%3Dtrue&client_id=arn%3Aaws%3Aiam%3A%3A015428540659%3Auser%2Fcloudformation&forceMobileApp=0&code_challenge=dDa178BxB6UkFfrpADw5CIoZ4yDUNRTG7sNM1EO__eo&code_challenge_method=SHA-256)を選択します。このオプションでは、既存のAWSインフラストラクチャにStarRocksをプロビジョニングします。

4. 正しいAWSリージョンを選択します。

5. **スタックの作成** ページで、テンプレートURLのデフォルト設定を保持し、**次へ** を選択します。

6. **スタックの詳細の指定** ページ

   1. 必要に応じてスタック名をカスタマイズします。

   2. テンプレートのパラメータを構成および確認します。

      1. 必要なパラメータを構成します。

         - StarRocksを新しいVPCに展開する場合は、以下のパラメータに注意してください：

             | タイプ                             | パラメータ                | 必須                                                 | 説明                                                       |
             | ------------------------------- | ------------------------- | ---------------------------------------------------- | ------------------------------------------------------------ |
             | ネットワーク構成                   | 可用ゾーン                   | はい                                                     | StarRocksクラスターを展開するために2つの可用ゾーンを選択します。詳細は[Regions and Zones](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html#concepts-local-zones)を参照してください。 |
             | EC2構成                            | キーペア名                  | はい                                                     | EC2インスタンスに接続する際に自分自身を証明するための一組のセキュリティ資格情報である公開キーと秘密キーからなるキーペアを入力します。詳細は[key pairs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)を参照してください。 ＞ ＞ キーペアを作成する必要がある場合は、[キーペアの作成](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html)を参照してください。 |
             | StarRocksクラスター構成      | Starrockのルートパスワード | はい                                                     | StarRocksのルートアカウントのパスワードを入力します。ルートアカウントを使用してStarRocksクラスターに接続する際にこのパスワードが必要です。 |
             |  | ルートパスワードの確認        | はい                       | StarRocksのルートアカウントのパスワードを確認します。 |                                                              |

         - 既存のVPCにStarRocksを展開する場合、以下のパラメータに注意してください：

           | タイプ                             | パラメータ                | 必須                                                         | 説明                                                       |
           | ------------------------------- | ------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
           | ネットワーク構成                   | VPC ID                      | はい                                                             | 既存のVPCのIDを入力します。AWS S3の[VPCエンドポイントの構成](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-s3.html)を行っていることを確認してください。 |
           | | プライベートサブネット1 ID   | はい                                                             | 既存のVPCの可用性ゾーン1にあるプライベートサブネットのID（例: subnet-fe9a8b32）を入力します。 |
           | | パブリックサブネット1 ID    | はい                                                             | 既存のVPCの可用性ゾーン1にあるパブリックサブネットのIDを入力します。 |
           | | パブリックサブネット2 ID    | はい                                                             | 既存のVPCの可用性ゾーン2にあるパブリックサブネットのIDを入力します。 |
           | EC2構成                            | キーペア名                 | はい                                                             | EC2インスタンスに接続する際に自分自身を証明するための一組のセキュリティ資格情報である公開キーと秘密キーからなるキーペアを入力します。詳細は[key pairs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)を参照してください。 <br /> **注意** <br /> キーペアの作成が必要な場合は、[キーペアの作成](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html)を参照してください。 |
           | StarRocksクラスター構成      | Starrockのルートパスワード | はい                                                             | StarRocksのルートアカウントのパスワードを入力します。ルートアカウントを使用してStarRocksクラスターに接続する際にこのパスワードが必要です。 |
           | | ルートパスワードの確認        | はい                       | StarRocksのルートアカウントのパスワードを確認します。     |                                                              |

      2. その他のパラメータについては、デフォルトの設定を確認し必要に応じてカスタマイズします。

   3. パラメータの構成とレビューが完了したら、**次へ** を選択します。

7. **スタックのオプションの構成** ページでは、デフォルト設定を保持して **次へ** をクリックします。

8. **starrocks-starrocksのレビュー** ページで、上記で構成したスタック情報、テンプレート、詳細、その他のオプションを確認します。詳細については[AWS CloudFormationコンソールでスタックを作成する際のスタックの見直しとスタックコストの推定](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-using-console-create-stack-review.html)を参照してください。

    > **注意**
    >
    > パラメータを変更する必要がある場合は、関連するセクションの右上隅にある **編集** をクリックして該当ページに戻ります。

9. 次の2つのチェックボックスを選択して **スタックの作成** をクリックします。
    ![StarRocks_on_AWS_1](../assets/StarRocks_on_AWS_1.png)

    **AWSサービスのコストと、このパートナーソリューションの実行中に使用されるサードパーティのライセンス料金については、お客様の責任であることに注意してください。コストの見積りについては、使用する各AWSサービスの料金ページを参照してください。**

## StarRocksクラスターへのアクセス

StarRocksクラスターはプライベートサブネットに展開されているため、まずEC2 Bastionホストに接続し、その後StarRocksクラスターにアクセスする必要があります。

1. StarRocksクラスターにアクセスするために使用されるEC2 Bastionホストに接続します。

   1. AWS CloudFormationコンソールから、 `BastionStack`の**Outputs**タブで`EIP1`の値をメモしてください。
   ![StarRocks_on_AWS_2](../assets/StarRocks_on_AWS_2.png)

   2. EC2コンソールから、EC2 Bastionホストを選択します。
   ![StarRocks_on_AWS_3](../assets/StarRocks_on_AWS_3.png)

   3. EC2 Bastionホストに関連付けられたセキュリティグループのインバウンドルールを編集し、自分のマシンからEC2 Bastionホストへのトラフィックを許可します。

   4. EC2 Bastionホストに接続します。

2. StarRocksクラスターへのアクセス

   1. EC2 BastionホストにMySQLをインストールします。

   2. 次のコマンドを使用してStarRocksクラスターに接続します：

      ```Bash
      mysql -u root -h 10.0.xx.xx -P 9030 -p
      ```

      - ホスト：
        次の手順に従って、FEのプライベートIPアドレスを見つけることができます：

        1. AWS CloudFormationコンソールから、 `StarRocksClusterStack`の**Outputs**タブで`FeLeaderInstance`の値をクリックします。
        ![StarRocks_on_AWS_4](../assets/StarRocks_on_AWS_4.png)

        2. インスタンスの概要ページから、FEのプライベートIPアドレスを見つけます。
        ![StarRocks_on_AWS_5](../assets/StarRocks_on_AWS_5.png)

      - パスワード：ステップ5で構成したパスワードを入力してください。