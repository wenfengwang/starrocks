---
displayed_sidebar: "Japanese"
---

# AWSでStarRocksをデプロイする

StarRocksとAWSは、StarRocksをAWS上で迅速にデプロイするための[AWSパートナーソリューション](https://aws.amazon.com/solutions/partners)を提供しています。このトピックでは、StarRocksをデプロイしてアクセスするための手順をステップバイステップで説明します。

## 基本的な概念

[AWSパートナーソリューション](https://aws-ia.github.io/content/qs_info.html)

AWSパートナーソリューションは、AWSのソリューションアーキテクトとAWSパートナーによって作成された自動化されたリファレンスデプロイメントです。AWSパートナーソリューションは、[AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)テンプレートを使用して、AWSリソースやStarRocksクラスタなどのサードパーティリソースを自動的にデプロイします。

[テンプレート](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-whatis-concepts.html#w2aab5c15b7)

テンプレートは、AWSリソースやサードパーティリソースの説明、およびそれらのリソースのプロパティを記述するJSONまたはYAML形式のテキストファイルです。

[スタック](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-whatis-concepts.html#w2ab1b5c15b9)

スタックは、テンプレートで記述されたリソースを作成および管理するために使用されます。スタックを作成、更新、削除することで、一連のリソースを作成、更新、削除することができます。

スタック内のすべてのリソースは、テンプレートによって定義されます。さまざまなリソースを記述するテンプレートを作成したとします。これらのリソースを構成するには、作成したテンプレートを提出してスタックを作成する必要があります。AWS CloudFormationは、その後、すべてのリソースを構成します。

## StarRocksクラスタをデプロイする

1. [AWSアカウントにサインイン](https://console.aws.amazon.com/console/home)します。アカウントをお持ちでない場合は、[AWS](https://aws.amazon.com/)でサインアップしてください。

2. 上部のツールバーからAWSリージョンを選択します。

3. この[パートナーソリューション](https://aws.amazon.com/quickstart/architecture/starrocks-starrocks/)を起動するためのデプロイオプションを選択します。デプロイには約30分かかります。

   1. [新しいVPCにStarRocksをデプロイする](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fus-east-1.console.aws.amazon.com%2Fcloudformation%2Fhome%3Fregion%3Dus-east-1%26state%3DhashArgs%2523%252Fstacks%252Fnew%253FstackName%253Dstarrocks-starrocks%2526templateURL%253Dhttps%253A%252F%252Faws-quickstart.s3.us-east-1.amazonaws.com%252Fquickstart-starrocks-starrocks%252Ftemplates%252Fstarrocks-entrypoint-new-vpc.template.yaml%26isauthcode%3Dtrue&client_id=arn%3Aaws%3Aiam%3A%3A015428540659%3Auser%2Fcloudformation&forceMobileApp=0&code_challenge=yo-6I1O2W0f0VcoqYOVvSwMmhRkC7Vod1M9vWbiMWUM&code_challenge_method=SHA-256)。このオプションは、新しいVPC、サブネット、NATゲートウェイ、セキュリティグループ、バスティオンホスト、およびその他のインフラコンポーネントで構成される新しいAWS環境を構築し、StarRocksをこの新しいVPCにデプロイします。
   2. [既存のVPCにStarRocksをデプロイする](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fus-east-1.console.aws.amazon.com%2Fcloudformation%2Fhome%3Fregion%3Dus-east-1%26state%3DhashArgs%2523%252Fstacks%252Fnew%253FstackName%253Dstarrocks-starrocks%2526templateURL%253Dhttps%253A%252F%252Faws-quickstart.s3.us-east-1.amazonaws.com%252Fquickstart-starrocks-starrocks%252Ftemplates%252Fstarrocks-entrypoint-existing-vpc.template.yaml%26isauthcode%3Dtrue&client_id=arn%3Aaws%3Aiam%3A%3A015428540659%3Auser%2Fcloudformation&forceMobileApp=0&code_challenge=dDa178BxB6UkFfrpADw5CIoZ4yDUNRTG7sNM1EO__eo&code_challenge_method=SHA-256)。このオプションは、既存のAWSインフラストラクチャにStarRocksをプロビジョニングします。

4. 正しいAWSリージョンを選択します。

5. **スタックの作成**ページで、テンプレートURLのデフォルト設定を保持し、**次へ**を選択します。

6. **スタックの詳細の指定**ページで

   1. 必要に応じてスタック名をカスタマイズします。

   2. テンプレートのパラメータを構成して確認します。

      1. 必要なパラメータを構成します。

         - StarRocksを新しいVPCにデプロイする場合は、次のパラメータに注意してください：

             | タイプ                           | パラメータ                 | 必須                                                | 説明                                                  |
             | ------------------------------- | ------------------------- | --------------------------------------------------- | ------------------------------------------------------ |
             | ネットワーク構成                  | 可用性ゾーン              | はい                                                | StarRocksクラスタをデプロイするために2つの可用性ゾーンを選択します。詳細については、[リージョンとゾーン](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html#concepts-local-zones)を参照してください。 |
             | EC2構成                          | キーペア名                | はい                                                | EC2インスタンスに接続する際に自分のアイデンティティを証明するために使用するセキュリティ資格情報のセットである、パブリックキーとプライベートキーからなるキーペアを入力します。詳細については、[キーペア](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)を参照してください。 > 注意 > > キーペアを作成する必要がある場合は、[キーペアの作成](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html)を参照してください。 |
             | StarRocksクラスタの構成          | Starrockのルートパスワード | はい                                                | StarRocksのルートアカウントのパスワードを入力します。ルートアカウントを使用してStarRocksクラスタに接続する際に、パスワードを提供する必要があります。 |
             |  |ルートパスワードの確認          | はい                      | StarRocksのルートアカウントのパスワードを確認します。 |                                                          |

         - 既存のVPCにStarRocksをデプロイする場合は、次のパラメータに注意してください：

           | タイプ                           | パラメータ                 | 必須                                                        | 説明                                                  |
           | ------------------------------- | ------------------------- | ----------------------------------------------------------- | ------------------------------------------------------ |
           | ネットワーク構成                  | VPC ID                    | はい                                                        | 既存のVPCのIDを入力します。[AWS S3のVPCエンドポイントを設定](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-s3.html)していることを確認してください。 |
           | |プライベートサブネット1のID    | はい                      | 既存のVPCの可用性ゾーン1のプライベートサブネットのIDを入力します（例：subnet-fe9a8b32）。 |                                                          |
           | |パブリックサブネット1のID      | はい                      | 既存のVPCの可用性ゾーン1のパブリックサブネットのIDを入力します。 |                                                          |
           | |パブリックサブネット2のID      | はい                      | 既存のVPCの可用性ゾーン2のパブリックサブネットのIDを入力します。 |                                                          |
           | EC2構成                          | キーペア名                | はい                                                        | EC2インスタンスに接続する際に自分のアイデンティティを証明するために使用するセキュリティ資格情報のセットである、パブリックキーとプライベートキーからなるキーペアを入力します。詳細については、[キーペア](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)を参照してください。 <br /> **注意** <br /> キーペアを作成する必要がある場合は、[キーペアの作成](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html)を参照してください。 |
           | StarRocksクラスタの構成          | Starrockのルートパスワード | はい                                                        | StarRocksのルートアカウントのパスワードを入力します。ルートアカウントを使用してStarRocksクラスタに接続する際に、パスワードを提供する必要があります。 |
           | |ルートパスワードの確認          | はい                      | StarRocksのルートアカウントのパスワードを確認します。       |                                                          |

      2. その他のパラメータについては、デフォルト設定を確認し、必要に応じてカスタマイズします。

   3. パラメータの構成と確認が完了したら、**次へ**を選択します。

7. **スタックオプションの構成**ページで、デフォルト設定を保持し、**次へ**をクリックします。

8. **starrocks-starrocksのレビュー**ページで、テンプレート、詳細、その他のオプションなど、上記で構成したスタック情報を確認します。詳細については、[AWS CloudFormationコンソールでスタックを作成し、スタックのコストを見積もる](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-using-console-create-stack-review.html)を参照してください。

    > **注意**
    >
    > パラメータを変更する必要がある場合は、関連するセクションの右上隅にある**編集**をクリックして、関連するページに戻ることができます。

9. 次の2つのチェックボックスを選択し、**スタックの作成**をクリックします。

    ![StarRocks_on_AWS_1](../assets/StarRocks_on_AWS_1.png)

    **このパートナーソリューションを実行する際に使用するAWSサービスおよびサードパーティのライセンスにかかる費用は、お客様の責任です**。費用の見積もりについては、使用する各AWSサービスの価格ページを参照してください。

## StarRocksクラスタへのアクセス

StarRocksクラスタはプライベートサブネットにデプロイされているため、まずEC2バスティオンホストに接続し、その後StarRocksクラスタにアクセスする必要があります。

1. StarRocksクラスタにアクセスするために使用されるEC2バスティオンホストに接続します。

   1. AWS CloudFormationコンソールから、`BastionStack`の**出力**タブで、`EIP1`の値をメモします。
   ![StarRocks_on_AWS_2](../assets/StarRocks_on_AWS_2.png)

   2. EC2コンソールから、EC2バスティオンホストを選択します。
   ![StarRocks_on_AWS_3](../assets/StarRocks_on_AWS_3.png)

   3. EC2バスティオンホストに関連するセキュリティグループのインバウンドルールを編集し、自分のマシンからEC2バスティオンホストへのトラフィックを許可します。

   4. EC2バスティオンホストに接続します。

2. StarRocksクラスタにアクセスします

   1. EC2バスティオンホストにMySQLをインストールします。

   2. 次のコマンドを使用してStarRocksクラスタに接続します：

      ```Bash
      mysql -u root -h 10.0.xx.xx -P 9030 -p
      ```

      - ホスト：
        次の手順に従って、FEのプライベートIPアドレスを見つけることができます：

        1. AWS CloudFormationコンソールから、`StarRocksClusterStack`の**出力**タブで、`FeLeaderInstance`の値をクリックします。
        ![StarRocks_on_AWS_4](../assets/StarRocks_on_AWS_4.png)

        2. インスタンスの概要ページから、FEのプライベートIPアドレスを見つけます。
        ![StarRocks_on_AWS_5](../assets/StarRocks_on_AWS_5.png)

      - パスワード：ステップ5で構成したパスワードを入力します。
