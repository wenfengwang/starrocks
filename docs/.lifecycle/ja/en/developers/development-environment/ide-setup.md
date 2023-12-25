---
displayed_sidebar: English
---

# StarRocks開発用のIDEセットアップ

StarRocksのコントリビューターになりたいけれど、開発環境に悩んでいる方のために、このチュートリアルを書きました。

完璧な開発ツールチェーンとは何でしょうか？

* FEとBEをワンクリックでコンパイルできる。
* ClionとIDEAでコードジャンプがサポートされている。
* IDE内の全ての変数が正常に解析され、赤い下線が表示されない。
* Clionが分析機能を正常に有効化できる。
* FEとBEのデバッグがサポートされている。

## 準備

私はローカルコーディング用にMacBook（M1）を使用し、StarRocksのコンパイルとテスト用にリモートサーバーを使用しています（リモートサーバーはUbuntu 22を使用し、**最低16GBのRAMが必要です**）。

全体的なアイディアは、MacBookでコードを書き、IDEを通じて自動的にサーバーにコードを同期し、サーバーでStarRocksをコンパイルおよび開発することです。

### MacBookのセットアップ

#### Thrift 0.13

公式のbrewリポジトリにはThriftの0.13バージョンはありませんが、私たちのコミッターの一人がインストール用のバージョンを自分のリポジトリに作成しました。

```bash
brew install alberttwong/thrift/thrift@0.13
```

以下のコマンドでThriftが正常にインストールされているか確認できます：

```bash
$ thrift -version
Thrift version 0.13.0
```

#### Protobuf

StarRocksのProtobufプロトコルのv2バージョンと互換性があるため、直接最新バージョンのv3を使用してください。

```bash
brew install protobuf
```

#### Maven

```bash
brew install maven
```

#### OpenJDK 1.8 または 11

```bash
brew install openjdk@11
```

#### Python3

MacOSには既に含まれているため、インストールの必要はありません。

#### システム環境の設定

```bash
export JAVA_HOME=xxxxx
export PYTHON=/usr/bin/python3
```

### Ubuntu22サーバーのセットアップ

#### StarRocksコードのクローン

`git clone https://github.com/StarRocks/starrocks.git`

#### コンパイルに必要なツールのインストール

```bash
sudo apt update
```

```bash
sudo apt install gcc g++ maven openjdk-11-jdk python3 python-is-python3 unzip cmake bzip2 ccache byacc ccache flex automake libtool bison binutils-dev libiberty-dev build-essential ninja-build
```

`JAVA_HOME`環境変数の設定

```bash
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
```

#### StarRocksのコンパイル

```bash
cd starrocks/
./build.sh
```

初回のコンパイルではthirdpartyをコンパイルする必要があり、時間がかかります。

**初回のコンパイルにはgccを使用する必要があります。現在、thirdpartyはclangではコンパイルに成功しません。**

## IDEのセットアップ

### FE

FEの開発は簡単です。MacOSで直接コンパイルできます。`fe`フォルダに入り、`mvn install -DskipTests`コマンドを実行します。

その後、IDEAで`fe`フォルダを直接開くことができ、問題はありません。

#### ローカルデバッグ

他のJavaアプリケーションと同様です。

#### リモートデバッグ

Ubuntuサーバーでは、`./start_fe.sh --debug`を実行し、IDEAのリモートデバッグ機能で接続します。デフォルトポートは5005ですが、`start_fe.sh`スクリプトで変更できます。

デバッグJavaパラメータ：`-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005`はIDEAからそのままコピーされています。

![IDE](../../assets/ide-1.png)

### BE

まず`fe`フォルダで`mvn install -DskipTests`を実行し、`gensrc`ディレクトリのthriftとprotobufが正しくコンパイルされていることを確認することをお勧めします。

次に、`gensrc`フォルダに入り、`make clean`と`make`コマンドをそれぞれ実行する必要があります。そうしないとClionがthriftの出力ファイルを検出できません。

`be`フォルダをClionで開きます。

`Settings`に入り、`Toolchains`を追加します。まずリモートサーバーを追加し、次にビルドツール、Cコンパイラ、C++コンパイラをそれぞれ設定します。

![IDE](../../assets/ide-2.png)

`Settings` / `Deployment`でフォルダ`mappings`を変更します。

![IDE](../../assets/ide-3.png)

`Settings` / `Cmake`で、ツールチェーンを追加したリモートツールチェーンに変更します。以下の環境変数を追加します：

```bash
JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
STARROCKS_GCC_HOME=/usr/
STARROCKS_THIRDPARTY=/root/starrocks/thirdparty
```

注意：`Include system environment variables`にチェックを入れないようにしてください。

![IDE](../../assets/ide-4.png)

![IDE](../../assets/ide-5.png)

ここから先は、すべてのセットアップが完了しています。Clionとリモートサーバーがしばらく同期された後、コードジャンプが正常に機能します。

#### デバッグ

BEのデバッグは少し難しいですが、リモートサーバーでgdbを使用する必要があります。もちろん、gdbサーバーとClionのリモートgdbを使用することもできますが、それは非常に不安定なのでお勧めしません。

`start_backend.sh`スクリプトを以下のように変更する必要があります：

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

から：

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

> レイクハウスのデバッグ時にエラー報告が出た場合は、`~/.gdbinit`に`handle SIGSEGV nostop noprint pass`を追加してください。

#### LLVM

もちろん、LLVMツールを使用してBEの開発を行うこともできます。

UbuntuでのLLVMのインストールは、https://apt.llvm.org/ を参照してください。

その後、次のコマンドを使用してBEをコンパイルします：`CC=clang-15 CXX=clang++-15 ./build.sh`。ただし、あらかじめthirdpartyがgccでコンパイルされている必要があります。

## 最後に

StarRocksへのコードの貢献をお待ちしています。🫵

## 参考文献

* [https://www.inlighting.org/archives/setup-perfect-starrocks-dev-env-en](https://www.inlighting.org/archives/setup-perfect-starrocks-dev-env-en)
* 中国語版：[https://www.inlighting.org/archives/setup-perfect-starrocks-dev-env](https://www.inlighting.org/archives/setup-perfect-starrocks-dev-env)
