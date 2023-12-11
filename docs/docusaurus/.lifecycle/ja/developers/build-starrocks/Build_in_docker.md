---
displayed_sidebar: "Japanese"
---

# Dockerを使用してStarRocksをコンパイルする

このトピックでは、Dockerを使用してStarRocksをコンパイルする方法について説明します。

## 概要

StarRocksは、Ubuntu 22.04とCentOS 7.9の両方に対応した開発環境イメージを提供しています。このイメージを使用すると、Dockerコンテナを起動し、その中でStarRocksをコンパイルすることができます。

### StarRocksのバージョンと開発環境イメージ

StarRocksの異なるブランチには、[StarRocks Docker Hub](https://hub.docker.com/u/starrocks)で提供されている異なる開発環境イメージが対応しています。

- Ubuntu 22.04の場合：

  | **ブランチ名** | **イメージ名**                             |
  | --------------- | ------------------------------------------- |
  | main            | starrocks/dev-env-ubuntu:latest             |
  | branch-3.1      | starrocks/dev-env-ubuntu:3.1-latest         |
  | branch-3.0      | starrocks/dev-env-ubuntu:3.0-latest         |
  | branch-2.5      | starrocks/dev-env-ubuntu:2.5-latest         |

- CentOS 7.9の場合：

  | **ブランチ名** | **イメージ名**                                 |
  | --------------- | ---------------------------------------------- |
  | main            | starrocks/dev-env-centos7:latest               |
  | branch-3.1      | starrocks/dev-env-centos7:3.1-latest           |
  | branch-3.0      | starrocks/dev-env-centos7:3.0-latest           |
  | branch-2.5      | starrocks/dev-env-centos7:2.5-latest           |

## 前提条件

StarRocksをコンパイルする前に、次の要件を満たしていることを確認してください:

- **ハードウェア**

  お使いのマシンには少なくとも8 GBのRAMが必要です。

- **ソフトウェア**

  - お使いのマシンはUbuntu 22.04またはCentOS 7.9で実行されている必要があります。
  - お使いのマシンにはDockerがインストールされている必要があります。

## 手順1：イメージをダウンロードする

次のコマンドを実行して、開発環境イメージをダウンロードします:

```Bash
# <image_name>には、ダウンロードしたいイメージの名前を指定してください。
# 例: `starrocks/dev-env-ubuntu:latest`。
# オペレーティングシステムに適した正しいイメージを選択したことを確認してください。
docker pull <image_name>
```

Dockerは自動的にお使いのマシンのCPUアーキテクチャを識別し、マシンに適したイメージを取得します。`linux/amd64`イメージはx86ベースのCPU向けであり、`linux/arm64`イメージはARMベースのCPU向けです。

## 手順2：DockerコンテナでStarRocksをコンパイルする

開発環境Dockerコンテナをローカルホストパスをマウントして起動するか、マウントせずに起動するか選択できます。次のようにローカルホストパスをマウントしてコンテナを起動することをお勧めします。これにより、次回のコンパイル時にJava依存関係を再度ダウンロードする必要がなくなり、また、コンテナからローカルホストにバイナリファイルを手動でコピーする必要がありません。

- **ローカルホストパスをマウントしてコンテナを起動する**：

  1. StarRocksのソースコードをローカルホストにクローンします。

     ```Bash
     git clone https://github.com/StarRocks/starrocks.git
     ```

  2. コンテナを起動します。

     ```Bash
     # <code_dir>には、StarRocksのソースコードディレクトリの親ディレクトリを指定してください。
     # <branch_name>には、イメージ名に対応するブランチ名を入力してください。
     # <image_name>には、ダウンロードしたイメージの名前を指定してください。
     docker run -it -v <code_dir>/.m2:/root/.m2 \
         -v <code_dir>/starrocks:/root/starrocks \
         --name <branch_name> -d <image_name>
     ```

  3. 起動したコンテナ内でbashシェルを起動します。

     ```Bash
     # <branch_name>には、イメージ名に対応するブランチ名を入力してください。
     docker exec -it <branch_name> /bin/bash
     ```

  4. コンテナ内でStarRocksをコンパイルします。

     ```Bash
     cd /root/starrocks && ./build.sh
     ```

- **ローカルホストパスをマウントせずにコンテナを起動する**：

  1. コンテナを起動します。

     ```Bash
     # <branch_name>には、イメージ名に対忘するブランチ名を入力してください。
     # <image_name>には、ダウンロードしたイメージの名前を指定してください。
     docker run -it --name <branch_name> -d <image_name>
     ```

  2. コンテナ内でbashシェルを起動します。

     ```Bash
     # <branch_name>には、イメージ名に対忘するブランチ名を入力してください。
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

Q: StarRocks BEのビルドが失敗し、次のエラーメッセージが返されました:

```Bash
g++: fatal error: Killed signal terminated program cc1plus
compilation terminated.
```

どうすればいいですか？

A: このエラーメッセージは、Dockerコンテナ内のメモリ不足を示しています。コンテナに少なくとも8 GBのメモリリソースを割り当てる必要があります。
