---
displayed_sidebar: "Japanese"
---

# StarRocksの開発用IDEのセットアップ

一部の人々はStarRocksの貢献者になりたいと考えていますが、開発環境に悩んでいるため、ここでそのチュートリアルを書きます。

完璧な開発ツールチェーンとは何ですか？

* 1クリックでFEとBEをコンパイルできること。
* ClionとIDEAでのコードジャンプをサポートすること。
* IDEのすべての変数が赤い線なしで正常に解析できること。
* Clionが正常に解析機能を有効にできること。
* FEとBEのデバッグをサポートすること。

## 準備

MacBook(M1)を使用してローカルコーディングを行い、リモートサーバーを使用してStarRocksのコンパイルとテストを行います（リモートサーバーはUbuntu 22を使用し、**少なくとも16GB RAMが必要**です）。

全体的な考え方は、MacBookでコードを書き、IDEを通じて自動的にコードをサーバーに同期し、サーバーを使用してStarRocksをコンパイルおよび開発することです。

### MacBookのセットアップ

#### Thrift 0.13

公式のbrewリポジトリには0.13バージョンのThriftがありません。コミッターの1人が自分のリポジトリでバージョンを作成してインストールしました。

```bash
brew install alberttwong/thrift/thrift@0.13
```

次のコマンドでThriftが正常にインストールされたかどうかを確認できます。

```bash
$ thrift -version
Thrift version 0.13.0
```

#### Protobuf

最新バージョンv3を直接使用します。なぜなら、最新バージョンのProtobufはStarRocksのProtobufプロトコルのv2バージョンと互換性があるからです。

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

MacOSには標準で含まれているため、インストールは不要です。

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

#### StarRocksのコンパイルを行う

```bash
cd starrocks/
./build.sh
```

初めてのコンパイルはthirdpartyをコンパイルする必要がありますが、それには時間がかかります。

**最初のコンパイルではgccを使用する必要があります。現在、thirdpartyはclangで正常にコンパイルできません。**

## IDEのセットアップ

### FE（Frontend）

FEの開発は簡単です。MacOSで直接コンパイルできます。単に`fe`フォルダに移動し、`mvn install -DskipTests`コマンドを実行します。

その後、IDEAで`fe`フォルダを直接開くことができます。

#### ローカルデバッグ

他のJavaアプリケーションと同様です。

#### リモートデバッグ

Ubuntuサーバーで`./start_fe.sh --debug`を実行し、それに接続するためにIDEAのリモートデバッグを使用します。デフォルトのポートは5005です。`start_fe.sh`スクリプトでポートを変更することができます。

Javaデバッグパラメータ: `-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005` は、単にIDEAからコピーされたものです。

![IDE](../../assets/ide-1.png)

### BE（Backend）

まず、`fe`フォルダで`mvn install -DskipTests`を実行して、gensrcディレクトリ内のthriftとprotobufが正しくコンパイルされていることを確認することをお勧めします。

その後、`gensrc`フォルダに入り、それぞれ`make clean`と`make`コマンドを実行する必要があります。そうしないと、Clionではthriftの出力ファイルを検出できません。

Clionで`be`フォルダを開きます。

`Settings`に入り、`Toolchains`を追加します。まず、リモートサーバーを追加し、それからビルドツール、CおよびC++コンパイラをそれぞれ設定します。

![IDE](../../assets/ide-2.png)

`Settings` / `Deployment` にて、フォルダのマッピングを変更します。

![IDE](../../assets/ide-3.png)

`Settings` / `CMake` にて、ツールチェーンを追加したリモートツールチェーンに設定します。次の環境変数を追加します。

```bash
JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
STARROCKS_GCC_HOME=/usr/
STARROCKS_THIRDPARTY=/root/starrocks/thirdparty
```

注意：「システム環境変数を含める」をチェックしないように注意してください。

![IDE](../../assets/ide-4.png)

![IDE](../../assets/ide-5.png)

ここで全てのセットアップが完了しました。Clionとリモートサーバーがしばらく同期された後、コードジャンプが正常に機能します。

#### デバッグ

BEのデバッグは少し難しいです。リモートサーバーでgdbを使用する必要があります。もちろん、gdbサーバー+Clionリモートgdbを使用することもできますが、お勧めしません、非常に遅くなります。

`start_backend.sh`スクリプトを次のように修正する必要があります。（一部省略）

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

その後、単にフラグなしで`./bin/start_be.sh`を実行します。

> Lakehouseのデバッグでエラーレポートに直面した場合、`~/.gdbinit`に`handle SIGSEGV nostop noprint pass`を追加してください。

#### LLVM

もちろん、LLVMツールを使用してBEを開発することもできます。

UbuntuでのLLVMのインストールは、https://apt.llvm.org/ を参照してください。

その後、次のコマンドを使用してBEをコンパイルします：`CC=clang-15 CXX=clang++-15 ./build.sh`。ただし、前提条件として、thirdpartyがgccでコンパイルされていることが必要です。

## 最後に

気軽にStarRocksにコードを貢献してください。🫵

## 参考

* [https://www.inlighting.org/archives/setup-perfect-starrocks-dev-env-en](https://www.inlighting.org/archives/setup-perfect-starrocks-dev-env-en)
* 中国語版: [https://www.inlighting.org/archives/setup-perfect-starrocks-dev-env](https://www.inlighting.org/archives/setup-perfect-starrocks-dev-env)