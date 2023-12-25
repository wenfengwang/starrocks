---
displayed_sidebar: Chinese
---

# Docker を使用して StarRocks をコンパイルする

この文書では、Docker を使用して StarRocks をコンパイルする方法について説明します。

## 概要

StarRocks は Ubuntu 22.04 と CentOS 7.9 の開発環境イメージを提供しています。これらのイメージを使用して、Docker コンテナ内で StarRocks をコンパイルできます。

### StarRocks のバージョンと開発環境イメージ

StarRocks の異なるバージョンは、[StarRocks Docker Hub](https://hub.docker.com/u/starrocks) で提供されている異なる開発環境イメージに対応しています。

- Ubuntu 22.04:

  | **ブランチ名** | **イメージ名**                          |
  | -------------- | --------------------------------------- |
  | main           | starrocks/dev-env-ubuntu:latest         |
  | branch-3.1     | starrocks/dev-env-ubuntu:3.1-latest     |
  | branch-3.0     | starrocks/dev-env-ubuntu:3.0-latest     |
  | branch-2.5     | starrocks/dev-env-ubuntu:2.5-latest     |

- CentOS 7.9 の場合:

  | **ブランチ名** | **イメージ名**                           |
  | -------------- | ---------------------------------------- |
  | main           | starrocks/dev-env-centos7:latest         |
  | branch-3.1     | starrocks/dev-env-centos7:3.1-latest     |
  | branch-3.0     | starrocks/dev-env-centos7:3.0-latest     |
  | branch-2.5     | starrocks/dev-env-centos7:2.5-latest     |

## 前提条件

StarRocks をコンパイルする前に、以下の要件を満たしていることを確認してください：

- **ハードウェア**

  マシンは 8 GB 以上のメモリを持っている必要があります

- **ソフトウェア**

  - マシンは Ubuntu 22.04 または CentOS 7.9 を実行している必要があります
  - マシンには Docker がインストールされている必要があります

## 第一歩：イメージをダウンロードする

以下のコマンドを実行して開発環境イメージをダウンロードします：

```Bash
# <image_name> をダウンロードしたいイメージの名前に置き換えてください。例：`starrocks/dev-env-ubuntu:latest`。
# ご使用の OS に適したイメージを選択してください。
docker pull <image_name>
```

Docker はマシンの CPU アーキテクチャを自動的に認識し、対応するイメージをダウンロードします。`linux/amd64` イメージは x86 アーキテクチャの CPU に適しており、`linux/arm64` イメージは ARM アーキテクチャの CPU に適しています。

## 第二歩：Docker コンテナ内で StarRocks をコンパイルする

開発環境 Docker コンテナを起動する際に、ローカルパスをマウントするかどうかを選択できます。Java の依存関係を次回コンパイル時に再ダウンロードすることを避け、コンテナ内のバイナリファイルを手動でローカルマシンにコピーする必要がないため、ローカルホストパスをマウントすることをお勧めします。

- **Docker コンテナを起動し、ローカルパスをマウントする:**

  1. StarRocks のソースコードをローカルマシンにクローンします。

     ```Bash
     git clone https://github.com/StarRocks/starrocks.git
     ```

  2. コンテナを起動します。

     ```Bash
     # <code_dir> を StarRocks ソースコードディレクトリの親ディレクトリに置き換えてください。
     # <branch_name> をイメージ名に対応するブランチ名に置き換えてください。
     # <image_name> をダウンロードしたイメージの名前に置き換えてください。
     docker run -it -v <code_dir>/.m2:/root/.m2 \
         -v <code_dir>/starrocks:/root/starrocks \
         --name <branch_name> -d <image_name>
     ```

  3. コンテナ内で bash シェルを起動します。

     ```Bash
     # <branch_name> をイメージ名に対応するブランチ名に置き換えてください。
     docker exec -it <branch_name> /bin/bash
     ```

  4. コンテナ内で StarRocks をコンパイルします。

     ```Bash
     cd /root/starrocks && ./build.sh
     ```

- **Docker コンテナを起動し、ローカルパスをマウントしない:**

  1. コンテナを起動します。

     ```Bash
     # <branch_name> をイメージ名に対応するブランチ名に置き換えてください。
     # <image_name> をダウンロードしたイメージの名前に置き換えてください。
     docker run -it --name <branch_name> -d <image_name>
     ```

  2. コンテナ内で bash シェルを起動します。

     ```Bash
     # <branch_name> をイメージ名に対応するブランチ名に置き換えてください。
     docker exec -it <branch_name> /bin/bash
     ```

  3. StarRocks のソースコードをコンテナにクローンします。

     ```Bash
     git clone https://github.com/StarRocks/starrocks.git
     ```

  4. コンテナ内で StarRocks をコンパイルします。

     ```Bash
     cd starrocks && ./build.sh
     ```

## トラブルシューティング

Q: StarRocks BE のコンパイルに失敗し、以下のエラーメッセージが表示されました：

```Plain
g++: fatal error: Killed signal terminated program cc1plus
compilation terminated.
```

どうすればいいですか？

A: このエラーメッセージは Docker コンテナのメモリが不足していることを意味します。コンテナに少なくとも 8 GB のメモリリソースを割り当てる必要があります。
