---
displayed_sidebar: "Japanese"
---

# AWSでStarRocksを展開する

StarRocksとAWSは、[AWSパートナーソリューション](https://aws.amazon.com/solutions/partners)を提供しており、StarRocksを迅速にAWSに展開するための手順を提供しています。このトピックでは、StarRocksの展開とアクセスを支援する手順をステップバイステップで説明します。

## 基本概念

[AWSパートナーソリューション](https://aws-ia.github.io/content/qs_info.html)

AWSパートナーソリューションは、AWSのソリューションアーキテクトとAWSパートナーによって作成された自動化されたリファレンス展開です。AWSパートナーソリューションでは、[AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html) テンプレートが使用され、AWSリソースやStarRocksクラスターなどのサードパーティのリソースがAWS Cloud上に自動的に展開されます。

[テンプレート](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-whatis-concepts.html#w2aab5c15b7)

テンプレートは、AWSリソースやサードパーティのリソース、およびそれらのリソースのプロパティを記述するJSONまたはYAML形式のテキストファイルです。

[スタック](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-whatis-concepts.html#w2ab1b5c15b9)

スタックは、テンプレートで記述されたリソースを作成および管理するために使用されます。スタックを作成、更新、削除することで、一連のリソースを作成、更新、削除できます。

スタック内のすべてのリソースはテンプレートで定義されています。たとえば、様々なリソースを記述したテンプレートを作成した場合、これらのリソースを構成するには、作成したテンプレートを送信してスタックを作成し、AWS CloudFormationがそれらのリソースをすべて設定します。

## StarRocksクラスターの展開

1. [AWSアカウントにサインイン](https://console.aws.amazon.com/console/home)します。アカウントをお持ちでない場合は、[AWS](https://aws.amazon.com/)でサインアップしてください。

2. 上部ツールバーからAWSリージョンを選択します。

3. この[パートナーソリューション](https://aws.amazon.com/quickstart/architecture/starrocks-starrocks/)を展開する展開オプションを選択します。AWS CloudFormationコンソールが開き、プレプロビジョニングされたテンプレートが表示されます。このテンプレートは、1つのFEと3つのBEを持つStarRocksクラスターを展開するために使用されます。展開には約30分かかります。

   1. [新しいVPCにStarRocksを展開](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fus-east-1.console.aws.amazon.com%2Fcloudformation%2Fhome%3Fregion%3Dus-east-1%26state%3DhashArgs%2523%252Fstacks%252Fnew%253FstackName%253Dstarrocks-starrocks%2526templateURL%253Dhttps%253A%252F%252Faws-quickstart.s3.us-east-1.amazonaws.com%252Fquickstart-starrocks-starrocks%252Ftemplates%252Fstarrocks-entrypoint-new-vpc.template.yaml%26isauthcode%3Dtrue&client_id=arn%3Aaws%3Aiam%3A%3A015428540659%3Auser%2Fcloudformation&forceMobileApp=0&code_challenge=yo-6I1O2W0f0VcoqYOVvSwMmhRkC7Vod1M9vWbiMWUM&code_challenge_method=SHA-256)を選択します。このオプションでは、VPC、サブネット、NATゲートウェイ、セキュリティグループ、バスチョンホスト、および他のインフラ構成要素から構成される新しいAWS環境を構築し、その後StarRocksをこの新しいVPCに展開します。
   2. [既存のVPCにStarRocksを展開](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fus-east-1.console.aws.amazon.com%2Fcloudformation%2Fhome%3Fregion%3Dus-east-1%26state%3DhashArgs%2523%252Fstacks%252Fnew%253FstackName%253Dstarrocks-starrocks%2526templateURL%253Dhttps%253A%252F%252Faws-quickstart.s3.us-east-1.amazonaws.com%252Fquickstart-starrocks-starrocks%252Ftemplates%252Fstarrocks-entrypoint-existing-vpc.template.yaml%26isauthcode%3Dtrue&client_id=arn%3Aaws%3Aiam%3A%3A015428540659%3Auser%2Fcloudformation&forceMobileApp=0&code_challenge=dDa178BxB6UkFfrpADw5CIoZ4yDUNRTG7sNM1EO__eo&code_challenge_method=SHA-256)を選択します。このオプションでは、既存のAWSインフラストラクチャにStarRocksをプロビジョニングします。

4. 正しいAWSリージョンを選択します。

5. **スタックの作成**ページで、テンプレートURLのデフォルト設定を保持し、**次へ**を選択します。

6. **スタックの詳細の指定**ページ

   1. 必要に応じてスタック名をカスタマイズします。

   2. テンプレートのパラメータを構成してレビューします。

      1. 必要なパラメータを構成します。

         - StarRocksを新しいVPCに展開する場合は、次のパラメータに注意してください：

             | タイプ                 | パラメータ名              | 必要                | 説明                                                     |
             | --------------------- | ------------------------- | ------------------- | ------------------------------------------------------- |
             | ネットワーク構成       | 可用性ゾーン              | はい                | StarRocksクラスターを展開するための2つの可用性ゾーンを選択します。詳細は[リージョンとゾーン](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html#concepts-local-zones) を参照してください。 |
             | EC2構成                | キーペア名                | はい                | EC2インスタンスに接続する際に自分のアイデンティティを証明するセキュリティクレデンシャルである、公開キーと秘密キーから成るキーペアを入力します。詳細は[キーペア](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)を参照してください。 > 注意 > > キーペアを作成する必要がある場合は、[キーペアの作成](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html) を参照してください。 |
             | StarRocksクラスタ構成 | StarRocksのルートパスワード | はい                | StarRocksのルートアカウントのパスワードを入力します。ルートアカウントを使用してStarRocksクラスタに接続する際にこのパスワードが必要です。 |
             | |ルートパスワードの確認      | はい                | StarRocksのルートアカウントのパスワードを確認します。 |                                                           |

         - 既存のVPCにStarRocksを展開する場合は、次のパラメータに注意してください：

           | タイプ                 | パラメータ名              | 必要                | 説明                                                         |
           | ----------------------- | ------------------------- | ------------------- | ----------------------------------------------------------- |
           | ネットワーク構成         | VPC ID                    | はい                | 既存のVPCのIDを入力します。[AWS S3のVPCエンドポイントを構成](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-s3.html)していることを確認してください。 |
           | |プライベートサブネット1 ID | はい                | 既存のVPCの可用性ゾーン1のプライベートサブネットのIDを入力します（例：subnet-fe9a8b32）。 |                                                               |
           | |パブリックサブネット1 ID    | はい                | 既存のVPCの可用性ゾーン1のパブリックサブネットのIDを入力します。 |                                                               |
           | |パブリックサブネット2 ID    | はい                | 既存のVPCの可用性ゾーン2のパブリックサブネットのIDを入力します。 |                                                               |
           | EC2構成                  | キーペア名                | はい                | EC2インスタンスに接続する際に自分のアイデンティティを証明するセキュリティクレデンシャルである、公開キーと秘密キーから成るキーペアを入力します。詳細は[キーペア](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)を参照してください。 <br /> **注意** <br /> キーペアを作成する必要がある場合は、[キーペアの作成](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html) を参照してください。 |
           | StarRocksクラスタ構成   | StarRocksのルートパスワード | はい                | StarRocksのルートアカウントのパスワードを入力します。ルートアカウントを使用してStarRocksクラスタに接続する際にこのパスワードが必要です。 |
           | |ルートパスワードの確認    | はい                | StarRocksのルートアカウントのパスワードを確認します。       |                                                               |

      2. その他のパラメータについては、デフォルトの設定を確認し、必要に応じてカスタマイズします。

   3. パラメータの構成およびレビューが完了したら、**次へ**を選択します。

7. **スタックのオプションの構成**ページで、デフォルト設定を保持し、**次へ**をクリックします。

8. **starrocks-starrocksのレビュー**ページで、上記で構成したスタック情報（テンプレート、詳細、その他オプション）をレビューします。詳細については、[AWS CloudFormationコンソールでスタックのレビューとスタックコストの見積もり](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-using-console-create-stack-review.html)を参照してください。

    > **注意**
    >
    > パラメータを変更する必要がある場合は、関連するセクションの右上隅にある**編集**をクリックして関連ページに戻ります。

9. 次の2つのチェックボックスを選択し、**スタックの作成**をクリックします。
    ![StarRocks_on_AWS_1](../assets/StarRocks_on_AWS_1.png)

    **AWSサービスの費用およびサードパーティのライセンスの利用については、お客様の責任となります。** このパートナーソリューションを実行する際の費用の見積りについては、使用する各AWSサービスの価格ページを参照してください。

## StarRocksクラスターへのアクセス

StarRocksクラスターはプライベートサブネットに展開されているため、まずEC2バスチオンホストに接続し、その後StarRocksクラスターにアクセスする必要があります。

1. StarRocksクラスターにアクセスするために使用されるEC2バスチオンホストに接続します。

   1. AWS CloudFormationコンソールから、`BastionStack`の**Outputs**タブで、`EIP1`の値をメモしてください。
   ![StarRocks_on_AWS_2](../assets/StarRocks_on_AWS_2.png)

   2. EC2コンソールから、EC2バスチオンホストを選択します。
   ![StarRocks_on_AWS_3](../assets/StarRocks_on_AWS_3.png)

   3. EC2バスチオンホストに関連付けられたセキュリティグループのインバウンドルールを編集し、自分のマシンからEC2バスチオンホストへのトラフィックを許可します。

   4. EC2バスチオンホストに接続します。

2. StarRocksクラスターにアクセスします

   1. EC2バスチオンホストにMySQLをインストールします。

   2. 次のコマンドを使用してStarRocksクラスターに接続します:

      ```Bash
      mysql -u root -h 10.0.xx.xx -P 9030 -p
      ```

      - ホスト:
        次の手順に従って、FEのプライベートIPアドレスを見つけることができます:

        1. AWS CloudFormationコンソールから、`StarRocksClusterStack`の**Outputs**タブで、`FeLeaderInstance`の値をクリックします。
        ![StarRocks_on_AWS_4](../assets/StarRocks_on_AWS_4.png)

        2. インスタンスの概要ページから、FEのプライベートIPアドレスを見つけます。
        ![StarRocks_on_AWS_5](../assets/StarRocks_on_AWS_5.png)

      - パスワード: ステップ5で構成したパスワードを入力してください。