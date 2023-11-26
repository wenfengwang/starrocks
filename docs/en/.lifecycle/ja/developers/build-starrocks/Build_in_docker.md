---
displayed_sidebar: "Japanese"
---

# Dockerを使用してStarRocksをコンパイルする

このトピックでは、Dockerを使用してStarRocksをコンパイルする方法について説明します。

## 概要

StarRocksは、Ubuntu 22.04とCentOS 7.9の両方の開発環境イメージを提供しています。このイメージを使用して、Dockerコンテナを起動し、コンテナ内でStarRocksをコンパイルすることができます。

### StarRocksのバージョンとDEV ENVイメージ

StarRocksの異なるブランチには、[StarRocks Docker Hub](https://hub.docker.com/u/starrocks)で提供されている異なる開発環境イメージが対応しています。

- Ubuntu 22.04の場合:

  | **ブランチ名** | **イメージ名**                          |
  | --------------- | --------------------------------------- |
  | main            | starrocks/dev-env-ubuntu:latest         |
  | branch-3.1      | starrocks/dev-env-ubuntu:3.1-latest     |
  | branch-3.0      | starrocks/dev-env-ubuntu:3.0-latest     |
  | branch-2.5      | starrocks/dev-env-ubuntu:2.5-latest     |

- CentOS 7.9の場合:

  | **ブランチ名** | **イメージ名**                             |
  | --------------- | ------------------------------------------ |
  | main            | starrocks/dev-env-centos7:latest           |
  | branch-3.1      | starrocks/dev-env-centos7:3.1-latest       |
  | branch-3.0      | starrocks/dev-env-centos7:3.0-latest       |
  | branch-2.5      | starrocks/dev-env-centos7:2.5-latest       |

## 前提条件

StarRocksをコンパイルする前に、以下の要件を満たしていることを確認してください:

- **ハードウェア**

  マシンには少なくとも8 GBのRAMが必要です。

- **ソフトウェア**

  - マシンがUbuntu 22.04またはCentOS 7.9で実行されている必要があります。
  - マシンにDockerがインストールされている必要があります。

## ステップ1: イメージのダウンロード

次のコマンドを実行して、開発環境イメージをダウンロードします:

```Bash
# <image_name>をダウンロードしたいイメージの名前に置き換えてください。例: `starrocks/dev-env-ubuntu:latest`。
# 正しいOS用のイメージを選択していることを確認してください。
docker pull <image_name>
```

Dockerは自動的にマシンのCPUアーキテクチャを識別し、マシンに適したイメージを取得します。`linux/amd64`イメージはx86ベースのCPU用であり、`linux/arm64`イメージはARMベースのCPU用です。

## ステップ2: Dockerコンテナ内でStarRocksをコンパイルする

ローカルホストのパスをマウントしているかどうかに関係なく、開発環境のDockerコンテナを起動することができます。次の手順では、ローカルホストのパスをマウントしてコンテナを起動することをおすすめします。これにより、次回のコンパイル時にJavaの依存関係を再ダウンロードする必要がなくなり、コンテナからローカルホストにバイナリファイルを手動でコピーする必要もありません。

- **ローカルホストのパスをマウントしてコンテナを起動する**:

  1. StarRocksのソースコードをローカルホストにクローンします。

     ```Bash
     git clone https://github.com/StarRocks/starrocks.git
     ```

  2. コンテナを起動します。

     ```Bash
     # <code_dir>をStarRocksのソースコードディレクトリの親ディレクトリに置き換えてください。
     # <branch_name>をイメージ名に対応するブランチ名に置き換えてください。
     # <image_name>をダウンロードしたイメージの名前に置き換えてください。
     docker run -it -v <code_dir>/.m2:/root/.m2 \
         -v <code_dir>/starrocks:/root/starrocks \
         --name <branch_name> -d <image_name>
     ```

  3. 起動したコンテナ内でbashシェルを起動します。

     ```Bash
     # <branch_name>をイメージ名に対応するブランチ名に置き換えてください。
     docker exec -it <branch_name> /bin/bash
     ```

  4. コンテナ内でStarRocksをコンパイルします。

     ```Bash
     cd /root/starrocks && ./build.sh
     ```

- **ローカルホストのパスをマウントせずにコンテナを起動する**:

  1. コンテナを起動します。

     ```Bash
     # <branch_name>をイメージ名に対応するブランチ名に置き換えてください。
     # <image_name>をダウンロードしたイメージの名前に置き換えてください。
     docker run -it --name <branch_name> -d <image_name>
     ```

  2. コンテナ内でbashシェルを起動します。

     ```Bash
     # <branch_name>をイメージ名に対応するブランチ名に置き換えてください。
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

どうすればよいですか？

A: このエラーメッセージは、Dockerコンテナ内のメモリが不足していることを示しています。コンテナに少なくとも8 GBのメモリリソースを割り当てる必要があります。
