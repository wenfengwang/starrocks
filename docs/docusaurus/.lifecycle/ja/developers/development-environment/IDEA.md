---
displayed_sidebar: "Japanese"
---

# IDEAでStarRocks FE開発環境をセットアップする

このチュートリアルはmacOSをベースにしており、Apple Chip(M1、M2)でテストされています。
macOSを使用していなくても、このチュートリアルを参照することができます。

## 必要なもの

### Thrift 0.13

公式のbrewリポジトリには0.13のThriftのバージョンはありません。当社のコミッターの1人が彼らのリポジトリでバージョンを作成してインストールしました。

```bash
brew install alberttwong/thrift/thrift@0.13
```

Thriftのインストールが成功したら、次のコマンドを実行して確認できます。

```bash
$ thrift -version
Thrift version 0.13.0
```

### Protobuf

StarRocksで使用されているProtobufのv2バージョンと互換性のある最新バージョンv3を使用してください。

```bash
brew install protobuf
```

### Maven

```
brew install maven
```

### Openjdk 1.8または11

```bash
brew install openjdk@11
```

### Python3

MacOSにはデフォルトでインストールされています。

ThriftとProtobufのインストールディレクトリはそれぞれ異なる場合があります。brew listコマンドを使用して調査できます。

```bash
brew list thrift@0.13.0
brew list protobuf
```

## StarRocksを構成する

### StarRocksをダウンロードする

```
git clone https://github.com/StarRocks/starrocks.git
```

### thirdpartyディレクトリをセットアップする

`thirdparty`内に`installed/bin`ディレクトリを作成します。

```bash
cd starrocks && mkdir -p thirdparty/installed/bin
```

その後、それぞれThriftとProtobufのためのシンボリックリンクを作成します。

```bash
ln -s /opt/homebrew/bin/thrift thirdparty/installed/bin/thrift
ln -s /opt/homebrew/bin/protoc thirdparty/installed/bin/protoc
```

### 環境変数を設定する

```bash
export JAVA_HOME="/opt/homebrew/Cellar/openjdk@11/11.0.15" # 注意: あなたのデスクトップでのJDKのバージョンは異なる場合があります
export PYTHON=/usr/bin/python3
export STARROCKS_THIRDPARTY=$(pwd)/thirdparty # 注意: starrocksディレクトリ内にいることを確認してください
```

## ソースコードを生成する

FE内の多くのソースファイルは手動で生成する必要があります。そうしないと、IDEAがファイルがないためエラーを報告します。次のコマンドを実行して自動的に生成します。

```bash
cd gensrc
make clean
make
```

## FEをコンパイルする

`fe`ディレクトリに移動して、Mavenを使用してコンパイルします。

```bash
cd fe
mvn install -DskipTests
```

## IDEAでStarRocksを開く

1. IDEAで`StarRocks`ディレクトリを開きます。

2. コーディングスタイルの設定を追加する
   コーディングスタイルを標準化するために、IDEAで`fe/starrocks_intellij_style.xml`コードスタイルファイルをインポートする必要があります。
   ![image-20220701193938856](../../assets/IDEA-2.png)

## MacOSでStarRocks FEを実行する

IDEAを使用して`fe`ディレクトリを開きます。

`StarRocksFE.java`でMain関数を直接実行すると、いくつかのエラーが報告されます。スムーズに実行するために、いくつかの簡単な設定を行うだけです。

**注意:** `StarRocksFE.java`は`fe/fe-core/src/main/java/com/starrocks`ディレクトリ内にあります。

1. `conf`、`bin`、`webroot`ディレクトリをStarRocksディレクトリから`fe`ディレクトリにコピーします。

```bash
cp -r conf fe/conf
cp -r bin fe/bin
cp -r webroot fe/webroot
```

2. `fe`ディレクトリに移動し、`fe`ディレクトリの下にlogとmetaフォルダを作成します。

```bash
cd fe
mkdir log
mkdir meta
```

3. 次の図に示すように、環境変数を設定します。

![image-20220701193938856](../../assets/IDEA-1.png)

```bash
export PID_DIR=/Users/smith/Code/starrocks/fe/bin
export STARROCKS_HOME=/Users/smith/Code/starrocks/fe
export LOG_DIR=/Users/smith/Code/starrocks/fe/log
```

4. `fe/conf/fe.conf`の`priority_networks`を`127.0.0.1/24`に変更して、FEが現在のコンピューターのLAN IPを使用してポートのバインドに失敗するのを防ぎます。

5. これで、StarRocks FEを成功裏に実行することができます。

## MacOSでStarRocks FEをデバッグする

デバッグオプションでFEを起動した場合、IDEAデバッガをFEプロセスにアタッチすることができます。

```
./start_fe.sh --debug
```

https://www.jetbrains.com/help/idea/attaching-to-local-process.html#attach-to-local を参照してください。