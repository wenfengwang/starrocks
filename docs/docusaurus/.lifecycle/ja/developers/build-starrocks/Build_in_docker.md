---
displayed_sidebar: "Japanese"
---

# Dockerを使用してStarRocksをコンパイルする

このトピックでは、Dockerを使用してStarRocksをコンパイルする方法について説明します。

## 概要

StarRocksは、Ubuntu 22.04およびCentOS 7.9の両方のための開発環境イメージを提供しています。このイメージを使用して、Dockerコンテナを起動し、コンテナ内でStarRocksをコンパイルできます。

### StarRocksのバージョンとDEV ENVイメージ

StarRocksの異なるブランチには、[StarRocks Docker Hub](https://hub.docker.com/u/starrocks)で提供されている異なる開発環境イメージが対応しています。

- Ubuntu 22.04の場合：

  | **ブランチ名** | **イメージ名**              |
  | --------------- | ----------------------------------- |
  | main            | starrocks/dev-env-ubuntu:latest     |
  | branch-3.1      | starrocks/dev-env-ubuntu:3.1-latest |
  | branch-3.0      | starrocks/dev-env-ubuntu:3.0-latest |
  | branch-2.5      | starrocks/dev-env-ubuntu:2.5-latest |

- CentOS 7.9の場合：

  | **ブランチ名** | **イメージ名**                       |
  | --------------- | ------------------------------------ |
  | main            | starrocks/dev-env-centos7:latest     |
  | branch-3.1      | starrocks/dev-env-centos7:3.1-latest |
  | branch-3.0      | starrocks/dev-env-centos7:3.0-latest |
  | branch-2.5      | starrocks/dev-env-centos7:2.5-latest |

## 前提条件

StarRocksをコンパイルする前に、以下の要件を満たしていることを確認してください：

- **ハードウェア**

  あなたのマシンには少なくとも8 GBのRAMが必要です。

- **ソフトウェア**

  - あなたのマシンはUbuntu 22.04またはCentOS 7.9で実行されている必要があります。
  - あなたのマシンにはDockerがインストールされている必要があります。

## ステップ1：イメージをダウンロードする

次のコマンドを実行して、開発環境イメージをダウンロードします：

```Bash
# <image_name>に、ダウンロードしたいイメージの名前を置き換えてください。
# 例： `starrocks/dev-env-ubuntu:latest`。
# 正しいOS用のイメージを選択したことを確認してください。
docker pull <image_name>
```

Dockerは自動的にあなたのマシンのCPUアーキテクチャを識別し、あなたのマシンに適した対応するイメージを取得します。`linux/amd64`イメージはx86ベースのCPU向けであり、`linux/arm64`イメージはARMベースのCPU向けです。

## ステップ2：Dockerコンテナ内でStarRocksをコンパイルする

開発環境のDockerコンテナをローカルホストのパスをマウントして起動することも、マウントせずに起動することもできます。次のコードは、ローカルホストのパスをマウントしてコンテナを起動することをお勧めします。これにより、次回のコンパイル時にJavaの依存関係を再ダウンロードする必要がなくなり、コンテナからローカルホストにバイナリファイルを手動でコピーする必要がありません。

- **ローカルホストのパスをマウントしてコンテナを起動する**：

  1. StarRocksのソースコードをローカルホストにクローンします。

     ```Bash
     git clone https://github.com/StarRocks/starrocks.git
     ```

  2. コンテナを起動します。

     ```Bash
     # <code_dir>には、StarRocksのソースコードディレクトリの親ディレクトリの名前を置き換えてください。
     # <branch_name>には、イメージ名に対応するブランチの名前を置き換えてください。
     # <image_name>には、ダウンロードしたイメージの名前を置き換えてください。
     docker run -it -v <code_dir>/.m2:/root/.m2 \
         -v <code_dir>/starrocks:/root/starrocks \
         --name <branch_name> -d <image_name>
     ```

  3. 起動したコンテナ内でbashシェルを起動します。

     ```Bash
     # <branch_name>には、イメージ名に対応するブランチの名前を置き換えてください。
     docker exec -it <branch_name> /bin/bash
     ```

  4. コンテナ内でStarRocksをコンパイルします。

     ```Bash
     cd /root/starrocks && ./build.sh
     ```

- **ローカルホストのパスをマウントせずにコンテナを起動する**：

  1. コンテナを起動します。

     ```Bash
     # <branch_name>には、イメージ名に対志するブランチの名前を置き換えてください。
     # <image_name>には、ダウンロードしたイメージの名前を置き換えてください。
     docker run -it --name <branch_name> -d <image_name>
     ```

  2. コンテナ内でbashシェルを起動します。

     ```Bash
     # <branch_name>には、イメージ名に対応するブランチの名前を置き換えてください。
     docker exec -it <branch_name> /bin/bash
     ```

  3. StarRocksのソースコードをコンテナ内にクローンします。

     ```Bash
     git clone https://github.com/StarRocks/starrocks.git
     ```

  4. コンテナ内でStarRocksをコンパイルします。

     ```Bash
     cd starrocks && ./build.sh
     ```

## トラブルシューティング

Q: StarRocks BEのビルドに失敗し、次のエラーメッセージが返されました：

```Bash
g++: fatal error: Killed signal terminated program cc1plus
compilation terminated.
```

どうすればよいですか？

A: このエラーメッセージは、Dockerコンテナ内でメモリが不足していることを示しています。コンテナに少なくとも8 GBのメモリリソースを割り当てる必要があります。