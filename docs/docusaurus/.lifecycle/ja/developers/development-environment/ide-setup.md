---
displayed_sidebar: "Japanese"
---

# StarRocksの開発環境を構築するためのIDEのセットアップ

何人かの方はStarRocksの貢献者になりたいと考えていますが、開発環境に悩んでいます。そのため、ここではそのチュートリアルを書いてみます。

完璧な開発ツールチェーンとは何でしょうか？

* FEとBEの1クリックによるコンパイルをサポートすること。
* CLionとIDEAでのコードジャンプをサポートすること。
* IDE内のすべての変数が赤い線なしで通常に解析されること。
* CLionでの解析機能が通常に有効にされること。
* FEとBEのデバッグをサポートすること。

## 準備

私はMacBook(M1)をローカルコーディングに使用し、リモートサーバーでStarRocksのコンパイルおよびテストを実行しています（リモートサーバーはUbuntu 22を使用しており、**最低16GBのRAMが必要**です）。

全体的なアイデアは、MacBookでコードを書いた後、IDEを介してコードを自動的にサーバーに同期し、サーバーを使用してStarRocksをコンパイルおよび開発することです。

### MacBookのセットアップ

#### Thrift 0.13

公式のbrewリポジトリに0.13のThriftがありません。私たちのコミッターの一人が自分のリポジトリでバージョンを作成してインストールしました。

```bash
brew install alberttwong/thrift/thrift@0.13
```

次のコマンドでThriftが正常にインストールされたかどうかを確認できます：

```bash
$ thrift -version
Thrift version 0.13.0
```

#### Protobuf

最新のバージョンv3を直接使用してください。なぜなら、最新のProtobufのバージョンはStarRocksのProtobufプロトコルのv2バージョンと互換性があるからです。

```bash
brew install protobuf
```

#### Maven

```bash
brew install maven
```

#### OpenJDK 1.8または11

```bash
brew install openjdk@11
```

#### Python3

MacOSには含まれているため、インストールは不要です。

#### システム環境のセットアップ

```bash
export JAVA_HOME=xxxxx
export PYTHON=/usr/bin/python3
```

### Ubuntu22サーバーのセットアップ

#### StarRocksのコードをクローンする

`git clone https://github.com/StarRocks/starrocks.git`

#### コンパイルに必要なツールをインストールする

```bash
sudo apt update
```

```bash
sudo apt install gcc g++ maven openjdk-11-jdk python3 python-is-python3 unzip cmake bzip2 ccache byacc ccache flex automake libtool bison binutils-dev libiberty-dev build-essential ninja-build
```

`JAVA_HOME`環境をセットアップする

```bash
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
```

#### StarRocksをコンパイルする

```bash
cd starrocks/
./build.sh
```

初回のコンパイルはthirdpartyをコンパイルする必要があり、時間がかかります。

**最初のコンパイルではgccを使用する必要があります。現在のところ、thirdpartyはclangで正常にコンパイルできません。**

## IDEのセットアップ

### FE

FEの開発は簡単です。MacOSで直接コンパイルできます。`fe`フォルダに移動して、コマンド`mvn install -DskipTests`を実行してください。

その後、IDEAを使用して`fe`フォルダを直接開くことができます。

#### ローカルデバッグ

他のJavaアプリケーションと同様です。

#### リモートデバッグ

Ubuntuサーバー上で、`./start_fe.sh --debug`を実行し、それに接続するためにIDEAのリモートデバッグを使用します。デフォルトのポート番号は5005です。`start_fe.sh`スクリプトで変更することができます。

デバッグJavaのパラメーター：`-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005` は、IDEAからコピーしたものです。

![IDE](../../assets/ide-1.png)

### BE

まず、`fe`フォルダで`mvn install -DskipTests`を実行して、gensrcディレクトリ内のthriftとprotobufが正しくコンパイルされていることを確認することをお勧めします。

その後、`gensrc`フォルダに移動し、順番に`make clean`および`make`コマンドを実行する必要があります。そうしないと、Clionはthriftの出力ファイルを検出できません。

Clionを使用して`be`フォルダを開きます。

`設定`に移動し、`ツールチェーン`を追加します。まず、リモートサーバーを追加し、その後ビルドツールをセットアップし、CおよびC++コンパイラを個別にセットアップします。

![IDE](../../assets/ide-2.png)

`設定` / `配置`に移動します。フォルダの`マッピング`を変更します。

![IDE](../../assets/ide-3.png)

`設定` / `Cmake`に移動します。ツールチェーンを追加したリモートツールチェーンに変更します。次の環境変数を追加します：

```bash
JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
STARROCKS_GCC_HOME=/usr/
STARROCKS_THIRDPARTY=/root/starrocks/thirdparty
```

注意：`システムの環境変数を含める`をチェックしないように注意してください。

![IDE](../../assets/ide-4.png)

![IDE](../../assets/ide-5.png)

ここまでで準備は完了です。Clionとリモートサーバーがしばらく同期されると、コードジャンプが通常どおり動作します。

#### デバッグ

BEのデバッグは少し難しいです。リモートサーバーでgdbを使用する必要があります。もちろん、gdbサーバー+Clionリモートgdbを使用することもできますが、お勧めしません。非常に固まります。

`start_backend.sh`スクリプトを次のように変更する必要があります：

```bash
if [ ${RUN_BE} -eq 1 ]; then
    echo "start time: "$(date) >> $LOG_DIR/be.out
    if [ ${RUN_DAEMON} -eq 1 ]; then
        nohup ${STARROCKS_HOME}/lib/starrocks_be "$@" >> $LOG_DIR/be.out 2>&1 </dev/null &
    else
        gdb -tui ${STARROCKS_HOME}/lib/starrocks_be
    fi
fi
```

その後、フラグなしで`./bin/start_be.sh`を実行します。

> lakehouseのデバッグ時にエラーレポートが発生した場合は、`~/.gdbinit`に`handle SIGSEGV nostop noprint pass`を追加してください。

#### LLVM

もちろん、LLVMツールを使用してBEを開発することもできます。

UbuntuでのLLVMのインストールは次のリンクを参照してください：https://apt.llvm.org/

その後、次のコマンドを使用して、BEをコンパイルします： `CC=clang-15 CXX=clang++-15 ./build.sh`。ただし、前提条件として、thirdpartyがgccでコンパイルされている必要があります。

## 最後に

StarRocksへのコードの貢献は自由です。 🫵

## 参考

* [https://www.inlighting.org/archives/setup-perfect-starrocks-dev-env-en](https://www.inlighting.org/archives/setup-perfect-starrocks-dev-env-en)
* 中国語版：[https://www.inlighting.org/archives/setup-perfect-starrocks-dev-env](https://www.inlighting.org/archives/setup-perfect-starrocks-dev-env)