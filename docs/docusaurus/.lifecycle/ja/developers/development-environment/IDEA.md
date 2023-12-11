---
displayed_sidebar: "Japanese"
---

# IDEA で StarRocks FE 開発環境をセットアップする

このチュートリアルは macOS を対象としており、Apple Chip(M1, M2) でテストされています。
macOS を使用していなくても、このチュートリアルを参照することができます。

## 必要条件

### Thrift 0.13

公式の brew リポジトリに 0.13 バージョンの Thrift がありません。私たちのコミッターの1人が、インストールするためにこのリポジトリ内のバージョンを作成しました。

```bash
brew install alberttwong/thrift/thrift@0.13
```

Thrift を正常にインストールした後は、次のコマンドを実行して確認できます：

```bash
$ thrift -version
Thrift version 0.13.0
```

### Protobuf

StarRocks で使用されている Protobuf の v2 バージョンと互換性のあるため、最新バージョン v3 を使用してください。

```bash
brew install protobuf
```

### Maven

```
brew install maven
```

### Openjdk 1.8 または 11

```bash
brew install openjdk@11
```

### Python3

MacOS にはデフォルトでインストールされています。

各自の Thrift および Protobuf のインストールディレクトリは異なる場合があります。brew list コマンドを使用して調査できます：

```bash
brew list thrift@0.13.0
brew list protobuf
```

## StarRocks を設定する

### StarRocks をダウンロードする

```
git clone https://github.com/StarRocks/starrocks.git
```

### thirdparty ディレクトリをセットアップする

`thirdparty` 内に `installed/bin` ディレクトリを作成します。

```bash
cd starrocks && mkdir -p thirdparty/installed/bin
```

次に、それぞれのために Thrift と Protobuf のシンボリックリンクを作成します。

```bash
ln -s /opt/homebrew/bin/thrift thirdparty/installed/bin/thrift
ln -s /opt/homebrew/bin/protoc thirdparty/installed/bin/protoc
```

### 環境変数を設定する

```bash
export JAVA_HOME="/opt/homebrew/Cellar/openjdk@11/11.0.15" # 注意: jdk のバージョンはデスクトップによって異なる場合があります
export PYTHON=/usr/bin/python3
export STARROCKS_THIRDPARTY=$(pwd)/thirdparty # 注意: starrocks ディレクトリ内にいることを確認してください
```

## ソースコードを生成する

FE 内の多くのソースファイルは手動で生成する必要があります。手動で生成しないと、IDEA がファイルが見つからないため、エラーが報告されます。

以下のコマンドを実行して自動的に生成します：

```bash
cd gensrc
make clean
make
```

## FE をコンパイルする

`fe` ディレクトリに移動し、Maven を使用してコンパイルします：

```bash
cd fe
mvn install -DskipTests
```

## IDEA で StarRocks を開く

1. IDEA で `StarRocks` ディレクトリを開きます。

2. コーディングスタイル設定を追加する
    コーディングスタイルを標準化するために、IDEA で `fe/starrocks_intellij_style.xml` コードスタイルファイルをインポートする必要があります。
![image-20220701193938856](../../assets/IDEA-2.png)

## macOS で StarRocks FE を実行する

IDEA を使用して `fe` ディレクトリを開きます。

`StarRocksFE.java` の Main 関数を直接実行すると、いくつかのエラーが報告されます。スムーズに実行するためには、いくつかの簡単な設定のみが必要です。

**注意:** `StarRocksFE.java` は `fe/fe-core/src/main/java/com/starrocks` ディレクトリにあります。

1. StarRocks ディレクトリから `conf`、`bin` 、`webroot` ディレクトリを `fe` ディレクトリにコピーします：

```bash
cp -r conf fe/conf
cp -r bin fe/bin
cp -r webroot fe/webroot
```

2. `fe` ディレクトリに移動し、`fe` ディレクトリの下に log と meta フォルダを作成します：

```bash
cd fe
mkdir log
mkdir meta
```

3. 次の図に示すように、環境変数を設定します：

![image-20220701193938856](../../assets/IDEA-1.png)

```bash
export PID_DIR=/Users/smith/Code/starrocks/fe/bin
export STARROCKS_HOME=/Users/smith/Code/starrocks/fe
export LOG_DIR=/Users/smith/Code/starrocks/fe/log
```

4. `fe/conf/fe.conf` 内の `priority_networks` を `127.0.0.1/24` に変更して、FE が現在のコンピュータの LAN IP を使用してポートが正常にバインドされないようにします。

5. 以上で、StarRocks FE を正常に実行できます。

## macOS で StarRocks FE をデバッグする

デバッグオプションで FE を起動した場合、IDEA デバッガを FE プロセスにアタッチできます。

```
./start_fe.sh --debug
```

https://www.jetbrains.com/help/idea/attaching-to-local-process.html#attach-to-local を参照してください。