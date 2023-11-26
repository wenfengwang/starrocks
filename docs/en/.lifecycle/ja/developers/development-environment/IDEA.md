---
displayed_sidebar: "Japanese"
---

# IDEAでStarRocks FE開発環境をセットアップする方法

このチュートリアルはmacOSを基にしており、Apple Chip(M1, M2)でテストされています。
macOSを使用していなくても、このチュートリアルを参考にすることができます。

## 必要なもの

### Thrift 0.13

公式のbrewリポジトリには0.13バージョンのThriftがありません。コミッターの一人がインストールするためのバージョンを作成しました。

```bash
brew install alberttwong/thrift/thrift@0.13
```

Thriftを正常にインストールした後、次のコマンドを実行して確認できます。

```bash
$ thrift -version
Thrift version 0.13.0
```

### Protobuf

最新バージョンのv3を使用してください。最新バージョンのProtobufは、StarRocksで使用されているv2バージョンのProtobufと互換性があります。

```bash
brew install protobuf
```

### Maven

```bash
brew install maven
```

### Openjdk 1.8または11

```bash
brew install openjdk@11
```

### Python3

MacOSにはデフォルトでインストールされています。

各自のThriftとProtobufのインストールディレクトリは異なる場合があります。brew listコマンドを使用して確認できます。

```bash
brew list thrift@0.13.0
brew list protobuf
```

## StarRocksの設定

### StarRocksをダウンロードする

```bash
git clone https://github.com/StarRocks/starrocks.git
```

### thirdpartyディレクトリをセットアップする

`thirdparty`ディレクトリ内に`installed/bin`ディレクトリを作成します。

```bash
cd starrocks && mkdir -p thirdparty/installed/bin
```

次に、ThriftとProtobufのそれぞれにシンボリックリンクを作成します。

```bash
ln -s /opt/homebrew/bin/thrift thirdparty/installed/bin/thrift
ln -s /opt/homebrew/bin/protoc thirdparty/installed/bin/protoc
```

### 環境変数の設定

```bash
export JAVA_HOME="/opt/homebrew/Cellar/openjdk@11/11.0.15" # 注意: jdkのバージョンはデスクトップ上で異なる場合があります
export PYTHON=/usr/bin/python3
export STARROCKS_THIRDPARTY=$(pwd)/thirdparty # 注意: starrocksディレクトリ内にいることを確認してください
```

## ソースコードの生成

FEの多くのソースファイルは手動で生成する必要があります。そうしないと、IDEAがファイルが見つからないためエラーが発生します。
次のコマンドを実行して自動的に生成します。

```bash
cd gensrc
make clean
make
```

## FEのコンパイル

`fe`ディレクトリに移動し、Mavenを使用してコンパイルします。

```bash
cd fe
mvn install -DskipTests
```

## IDEAでStarRocksを開く

1. IDEAで`StarRocks`ディレクトリを開きます。

2. コーディングスタイルの設定を追加します
    コーディングスタイルを統一するために、IDEAで`fe/starrocks_intellij_style.xml`のコードスタイルファイルをインポートする必要があります。
![image-20220701193938856](../../assets/IDEA-2.png)

## MacOSでStarRocks FEを実行する

IDEAで`fe`ディレクトリを開きます。

`StarRocksFE.java`のMain関数を直接実行すると、いくつかのエラーが報告されます。スムーズに実行するために、いくつかの簡単な設定を行うだけです。

**注意:** `StarRocksFE.java`は`fe/fe-core/src/main/java/com/starrocks`ディレクトリにあります。

1. StarRocksディレクトリからconf、bin、webrootディレクトリを`fe`ディレクトリにコピーします。

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

3. 環境変数を設定します。以下の図のように設定します。

![image-20220701193938856](../../assets/IDEA-1.png)

```bash
export PID_DIR=/Users/smith/Code/starrocks/fe/bin
export STARROCKS_HOME=/Users/smith/Code/starrocks/fe
export LOG_DIR=/Users/smith/Code/starrocks/fe/log
```

4. `fe/conf/fe.conf`のpriority_networksを`127.0.0.1/24`に変更して、現在のコンピュータのLAN IPを使用しないようにします。

5. これでStarRocks FEを正常に実行できます。

## MacOSでStarRocks FEをデバッグする

デバッグオプションでFEを起動した場合、IDEAデバッガをFEプロセスにアタッチすることができます。

```
./start_fe.sh --debug
```

詳細は、https://www.jetbrains.com/help/idea/attaching-to-local-process.html#attach-to-local を参照してください。
