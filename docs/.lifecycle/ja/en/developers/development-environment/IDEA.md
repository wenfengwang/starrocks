---
displayed_sidebar: English
---

# IDEA上でStarRocks FEの開発環境をセットアップする

このチュートリアルはmacOSをベースにしており、Appleチップ(M1、M2)でテストされています。
macOSを使用していない場合でも、このチュートリアルを参考にできます。

## 必要条件

### Thrift 0.13

公式のbrewリポジトリにはThriftの0.13バージョンはありません。私たちのコミッターの一人が、インストール用に自分のリポジトリにバージョンを作成しました。

```bash
brew install alberttwong/thrift/thrift@0.13
```

Thriftを正常にインストールしたら、以下のコマンドを実行して確認できます。

```bash
$ thrift -version
Thrift version 0.13.0
```

### Protobuf

StarRocksで使用されているProtobufのv2バージョンと互換性があるため、最新バージョンのv3を使用してください。

```bash
brew install protobuf
```

### Maven

```
brew install maven
```

### OpenJDK 1.8 または 11

```bash
brew install openjdk@11
```

### Python3

macOSにはデフォルトでインストールされています。

皆さんのThriftとProtobufのインストールディレクトリは異なる場合があります。`brew list`コマンドを使用して確認できます。

```bash
brew list thrift@0.13.0
brew list protobuf
```

## StarRocksの設定

### StarRocksをダウンロードする

```
git clone https://github.com/StarRocks/starrocks.git
```

### thirdpartyディレクトリをセットアップする

`thirdparty`内に`installed/bin`ディレクトリを作成します。

```bash
cd starrocks && mkdir -p thirdparty/installed/bin
```

次に、ThriftとProtobufのシンボリックリンクをそれぞれ作成します。

```bash
ln -s /opt/homebrew/bin/thrift thirdparty/installed/bin/thrift
ln -s /opt/homebrew/bin/protoc thirdparty/installed/bin/protoc
```

### 環境変数の設定

```bash
export JAVA_HOME="/opt/homebrew/Cellar/openjdk@11/11.0.15" # 注意: JDKのバージョンはあなたのデスクトップと異なる場合があります
export PYTHON=/usr/bin/python3
export STARROCKS_THIRDPARTY=$(pwd)/thirdparty # 注意: StarRocksディレクトリ内にいることを確認してください
```

## ソースコードの生成

FEの多くのソースファイルは手動で生成する必要があります。そうしないと、IDEAはファイルが見つからないためのエラーを報告します。
以下のコマンドを実行して自動生成します。

```bash
cd gensrc
make clean
make
```

## FEをコンパイルする

`fe`ディレクトリに入り、Mavenを使用してコンパイルします。

```bash
cd fe
mvn install -DskipTests
```

## IDEAでStarRocksを開く

1. IDEAで`StarRocks`ディレクトリを開きます。

2. コーディングスタイルの設定を追加する
    コーディングスタイルを統一するために、IDEAに`fe/starrocks_intellij_style.xml`コードスタイルファイルをインポートする必要があります。
![image-20220701193938856](../../assets/IDEA-2.png)

## MacOSでStarRocks FEを実行する

IDEAで`fe`ディレクトリを開きます。

`StarRocksFE.java`のMain関数を直接実行すると、いくつかのエラーが報告されることがあります。スムーズに実行するためには、いくつかの簡単な設定を行うだけです。

**注意:** `StarRocksFE.java`は`fe/fe-core/src/main/java/com/starrocks`ディレクトリ内にあります。

1. StarRocksディレクトリから`fe`ディレクトリにconf、bin、webrootディレクトリをコピーします。

```bash
cp -r conf fe/conf
cp -r bin fe/bin
cp -r webroot fe/webroot
```

2. `fe`ディレクトリに入り、`fe`ディレクトリの下にlogフォルダとmetaフォルダを作成します。

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

4. `fe/conf/fe.conf`内の`priority_networks`を`127.0.0.1/24`に変更して、FEが現在のコンピュータのLAN IPを使用してポートのバインドに失敗することを防ぎます。

5. これでStarRocks FEが正常に実行されるようになりました。

## MacOSでStarRocks FEをデバッグする

デバッグオプションでFEを起動した場合、IDEAデバッガをFEプロセスにアタッチできます。

```
./start_fe.sh --debug
```

https://www.jetbrains.com/help/idea/attaching-to-local-process.html#attach-to-local を参照してください。
