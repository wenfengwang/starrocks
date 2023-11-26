---
displayed_sidebar: "Japanese"
---

# StarRocksの開発環境のセットアップ

StarRocksのコントリビュータになりたい人もいますが、開発環境に悩んでいる人もいます。そこで、ここではそのチュートリアルを書いてみました。

完璧な開発ツールチェーンとは何でしょうか？

* 1クリックでFEとBEをコンパイルできること。
* ClionとIDEAでのコードジャンプをサポートすること。
* IDE内のすべての変数が正常に解析され、赤い下線が表示されないこと。
* Clionで解析機能を正常に有効にできること。
* FEとBEのデバッグをサポートすること。

## 準備

ローカルでのコーディングにはMacBook(M1)を使用し、StarRocksのコンパイルとテストにはリモートサーバーを使用します（リモートサーバーはUbuntu 22を使用し、**少なくとも16GBのRAMが必要です**）。

全体的なアイデアは、MacBookでコードを書き、IDEを介してコードを自動的にサーバーに同期し、サーバーを使用してStarRocksをコンパイルおよび開発することです。

### MacBookのセットアップ

#### Thrift 0.13

公式のbrewリポジトリには0.13バージョンのThriftがありません。コミッターの一人が自分のリポジトリでバージョンを作成し、インストールすることができます。

```bash
brew install alberttwong/thrift/thrift@0.13
```

以下のコマンドでThriftが正常にインストールされたかどうかを確認できます。

```bash
$ thrift -version
Thrift version 0.13.0
```

#### Protobuf

最新バージョンのv3を直接使用します。最新バージョンのProtobufは、StarRocksのProtobufプロトコルのv2バージョンと互換性があります。

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

MacOSにはすでにインストールされていますので、インストールは不要です。

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

`JAVA_HOME`環境変数を設定する

```bash
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
```

#### StarRocksをコンパイルする

```bash
cd starrocks/
./build.sh
```

初回のコンパイルでは、thirdpartyをコンパイルする必要がありますので、時間がかかります。

**最初のコンパイルではgccを使用する必要があります。現在、thirdpartyはclangで正常にコンパイルできません。**

## IDEのセットアップ

### FE

FEの開発は簡単です。MacOSで直接コンパイルできます。`fe`フォルダに移動し、`mvn install -DskipTests`コマンドを実行してください。

その後、IDEAで`fe`フォルダを直接開くことができます。

#### ローカルデバッグ

他のJavaアプリケーションと同じです。

#### リモートデバッグ

Ubuntuサーバーで`./start_fe.sh --debug`と実行し、それに接続するためにIDEAのリモートデバッグを使用します。デフォルトのポートは5005ですが、`start_fe.sh`スクリプトで変更することができます。

デバッグのJavaパラメータ:`-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005`は、IDEAからコピーされたものです。

![IDE](../../assets/ide-1.png)

### BE

まず、`fe`フォルダで`mvn install -DskipTests`を実行して、gensrcディレクトリ内のthriftとprotobufが正しくコンパイルされていることを確認することをおすすめします。

その後、`gensrc`フォルダに移動し、`make clean`と`make`のコマンドを順番に実行する必要があります。これにより、Clionがthriftの出力ファイルを検出できるようになります。

Clionを使用して`be`フォルダを開きます。

`Settings`に移動し、`Toolchains`を追加します。まず、リモートサーバーを追加し、それからBuild Tool、C、C++ Compilerを個別に設定します。

![IDE](../../assets/ide-2.png)

`Settings` / `Deployment`に移動します。フォルダのマッピングを変更します。

![IDE](../../assets/ide-3.png)

`Settings` / `Cmake`に移動します。Toolchainを追加したリモートツールチェーンに変更します。以下の環境変数を追加します。

```bash
JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
STARROCKS_GCC_HOME=/usr/
STARROCKS_THIRDPARTY=/root/starrocks/thirdparty
```

注意：「Include system environment variables」のチェックを外さないように注意してください。

![IDE](../../assets/ide-4.png)

![IDE](../../assets/ide-5.png)

ここまでで、すべてのセットアップが完了しました。Clionとリモートサーバーがしばらく同期された後、コードジャンプが正常に機能します。

#### デバッグ

BEのデバッグは少し難しいです。リモートサーバーでgdbを使用する必要があります。もちろん、gdbサーバー+Clionリモートgdbを使用することもできますが、おすすめしません。非常に重くなります。

`start_backend.sh`スクリプトを次のように変更します。

```bash
if [ ${RUN_BE} -eq 1 ]; then
    echo "start time: "$(date) >> $LOG_DIR/be.out
    if [ ${RUN_DAEMON} -eq 1 ]; then
        nohup ${STARROCKS_HOME}/lib/starrocks_be "$@" >> $LOG_DIR/be.out 2>&1 </dev/null &
    else
        ${STARROCKS_HOME}/lib/starrocks_be "$@" >> $LOG_DIR/be.out 2>&1 </dev/null
    fi
fi
```

次に、フラグなしで`./bin/start_be.sh`を実行します。

> Lakehouseのデバッグ時にエラーレポートが表示される場合は、`~/.gdbinit`に`handle SIGSEGV nostop noprint pass`を追加してください。

#### LLVM

もちろん、LLVMツールを使用してBEを開発することもできます。

UbuntuでのLLVMのインストール方法については、https://apt.llvm.org/を参照してください。

次のコマンドを使用して、beをコンパイルします：`CC=clang-15 CXX=clang++-15 ./build.sh`。ただし、前提条件として、thirdpartyがgccでコンパイルされている必要があります。

## 最後に

StarRocksへのコードの貢献は自由です。 🫵

## 参考

* [https://www.inlighting.org/archives/setup-perfect-starrocks-dev-env-en](https://www.inlighting.org/archives/setup-perfect-starrocks-dev-env-en)
* 中国語版: [https://www.inlighting.org/archives/setup-perfect-starrocks-dev-env](https://www.inlighting.org/archives/setup-perfect-starrocks-dev-env)
