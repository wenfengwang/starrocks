---
displayed_sidebar: Chinese
---

# StarRocksのソースコードからのビルド

この文書では、Dockerイメージを使用してStarRocksをビルドする方法について説明します。

## 前提条件

StarRocksをビルドする前に、[Docker](https://www.docker.com/get-started/)がインストールされていることを確認してください。

## イメージのダウンロード

Docker Hubから開発環境のイメージファイルをダウンロードします。このイメージには、サードパーティツールとしてLLVMとClangが統合されています。

```shell
docker pull starrocks/dev-env:{branch-name}
```

> 説明：コマンド中の `{branch-name}` を下表の対応するイメージTagで置き換えてください。

StarRocksのバージョンブランチと開発環境イメージバージョンの対応関係は以下の通りです：

| StarRocksのバージョン | イメージTag                      |
| ---------------- | ------------------------------|
| main             | starrocks/dev-env:main        |
| StarRocks-2.4.*  | starrocks/dev-env:branch-2.4  |
| StarRocks-2.3.*  | starrocks/dev-env:branch-2.3  |
| StarRocks-2.2.*  | starrocks/dev-env:branch-2.2  |
| StarRocks-2.1.*  | starrocks/dev-env:branch-2.1  |
| StarRocks-2.0.*  | starrocks/dev-env:branch-2.0  |
| StarRocks-1.19.* | starrocks/dev-env:branch-1.19 |

## StarRocksのビルド

ローカルストレージをマウントする（推奨）か、GitHubのリポジトリからコードをコピーする方法でStarRocksをビルドできます。

- ローカルストレージをマウントしてStarRocksをビルドする。

  ```shell
  mkdir {local-path}
  cd {local-path}

  git clone https://github.com/StarRocks/starrocks.git
  cd starrocks
  git checkout {branch-name}

  docker run -it -v {local-path}/.m2:/root/.m2 -v {local-path}/starrocks:/root/starrocks --name {branch-name} -d starrocks/dev-env:{branch-name}

  docker exec -it {branch-name} /root/starrocks/build.sh
  ```

  > 説明：この方法では、Dockerコンテナ内で **.m2** ディレクトリのJava依存関係を繰り返しダウンロードする必要がなく、また、Dockerコンテナから **starrocks/output** ディレクトリにあるビルド済みのバイナリパッケージをコピーする必要がありません。

- ローカルストレージを使用せずにStarRocksをビルドする。

  ```shell
  docker run -it --name {branch-name} -d starrocks/dev-env:{branch-name}
  docker exec -it {branch-name} /bin/bash
  
  # StarRocksのコードをダウンロードする。
  git clone https://github.com/StarRocks/starrocks.git
  
  cd starrocks
  sh build.sh
  ```
