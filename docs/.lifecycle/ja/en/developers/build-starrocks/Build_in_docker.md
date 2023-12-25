---
displayed_sidebar: English
---

# Dockerを使用してStarRocksをコンパイルする

このトピックでは、Dockerを使用してStarRocksをコンパイルする方法について説明します。

## 概要

StarRocksは、Ubuntu 22.04とCentOS 7.9の両方の開発環境イメージを提供しています。このイメージを使用すると、Dockerコンテナを起動し、そのコンテナ内でStarRocksをコンパイルできます。

### StarRocksのバージョンとDEV ENVイメージ

StarRocksの異なるブランチは、[StarRocks Docker Hub](https://hub.docker.com/u/starrocks)で提供されている異なる開発環境イメージに対応しています。

- Ubuntu 22.04用:

  | **ブランチ名** | **イメージ名**                        |
  | --------------- | ----------------------------------- |
  | main            | starrocks/dev-env-ubuntu:latest     |
  | branch-3.1      | starrocks/dev-env-ubuntu:3.1-latest |
  | branch-3.0      | starrocks/dev-env-ubuntu:3.0-latest |
  | branch-2.5      | starrocks/dev-env-ubuntu:2.5-latest |

- CentOS 7.9用:

  | **ブランチ名** | **イメージ名**                         |
  | --------------- | ------------------------------------ |
  | main            | starrocks/dev-env-centos7:latest     |
  | branch-3.1      | starrocks/dev-env-centos7:3.1-latest |
  | branch-3.0      | starrocks/dev-env-centos7:3.0-latest |
  | branch-2.5      | starrocks/dev-env-centos7:2.5-latest |

## 前提条件

StarRocksをコンパイルする前に、以下の要件が満たされていることを確認してください:

- **ハードウェア**

  あなたのマシンは少なくとも8GBのRAMを持っている必要があります。

- **ソフトウェア**

  - あなたのマシンはUbuntu 22.04またはCentOS 7.9で動作している必要があります。
  - あなたのマシンにはDockerがインストールされている必要があります。

## ステップ1: イメージをダウンロードする

以下のコマンドを実行して、開発環境イメージをダウンロードします:

```Bash
# <image_name>をダウンロードしたいイメージの名前に置き換えてください。
# 例: `starrocks/dev-env-ubuntu:latest`。
# あなたのOSに適した正しいイメージを選択していることを確認してください。
docker pull <image_name>
```

Dockerは自動的にあなたのマシンのCPUアーキテクチャを識別し、あなたのマシンに適した対応するイメージをプルします。`linux/amd64`イメージはx86ベースのCPU用で、`linux/arm64`イメージはARMベースのCPU用です。

## ステップ2: Dockerコンテナ内でStarRocksをコンパイルする

開発環境Dockerコンテナは、ローカルホストパスがマウントされているかどうかにかかわらず起動できます。次回のコンパイル時にJava依存関係を再ダウンロードする必要がなく、コンテナからローカルホストにバイナリファイルを手動でコピーする必要がないため、ローカルホストパスをマウントした状態でコンテナを起動することを推奨します。

- **ローカルホストパスがマウントされた状態でコンテナを起動する**:

  1. StarRocksのソースコードをローカルホストにクローンします。

     ```Bash
     git clone https://github.com/StarRocks/starrocks.git
     ```

  2. コンテナを起動します。

     ```Bash
     # <code_dir>をStarRocksソースコードディレクトリの親ディレクトリに、
     # <branch_name>をイメージ名に対応するブランチ名に、
     # <image_name>をダウンロードしたイメージの名前にそれぞれ置き換えてください。
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

- **ローカルホストパスをマウントせずにコンテナを起動する**:

  1. コンテナを起動します。

     ```Bash
     # <branch_name>をイメージ名に対応するブランチ名に、
     # <image_name>をダウンロードしたイメージの名前にそれぞれ置き換えてください。
     docker run -it --name <branch_name> -d <image_name>
     ```

  2. コンテナ内でbashシェルを起動します。

     ```Bash
     # <branch_name>をイメージ名に対応するブランチ名に置き換えてください。
     docker exec -it <branch_name> /bin/bash
     ```

  3. コンテナにStarRocksのソースコードをクローンします。

     ```Bash
     git clone https://github.com/StarRocks/starrocks.git
     ```

  4. コンテナ内でStarRocksをコンパイルします。

     ```Bash
     cd starrocks && ./build.sh
     ```

## トラブルシューティング

Q: StarRocks BEのビルドが失敗し、以下のエラーメッセージが表示されました。

```Bash
g++: fatal error: Killed signal terminated program cc1plus
compilation terminated.
```

どうすればいいですか？

A: このエラーメッセージは、Dockerコンテナのメモリが不足していることを示しています。コンテナには少なくとも8GBのメモリリソースを割り当てる必要があります。

