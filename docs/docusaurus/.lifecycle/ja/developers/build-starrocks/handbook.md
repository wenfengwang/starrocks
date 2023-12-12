---
displayed_sidebar: "Japanese"
---

# StarRocksのビルド方法

一般的には、StarRocksは次のコマンドを実行するだけでビルドできます。

```
./build.sh
```

このコマンドはまず、すべてのサードパーティの依存関係が準備されているかどうかをチェックします。すべての依存関係が準備されている場合、StarRocksの `Backend` と `Frontend` をビルドします。

このコマンドが正常に実行された場合、生成されたバイナリは `output` ディレクトリに保存されます。

## FE/BEを別々にビルドする方法

毎回FEとBEの両方をビルドする必要はありません。それぞれを個別にビルドすることができます。
例えば、次のようにしてBEのみをビルドすることができます。

```
./build.sh --be
```

そして、次のようにしてFEのみをビルドできます。

```
./build.sh --fe
```

# ユニットテストの実行方法

BEとFEのユニットテストは分かれています。一般的には、BEのテストは次のコマンドで実行できます。

```
./run-be-ut.sh
```

FEのテストは次のように実行できます。

```
./run-fe-ut.sh
```

## コマンドラインでBEのUTを実行する方法

現在、BEのUTは実行するためにいくつかの依存関係が必要で、`./run-be-ut.sh` がそれを手助けします。ただし、これには柔軟性がありません。コマンドラインでUTを実行したい場合は、次のように実行できます。

```
UDF_RUNTIME_DIR=./ STARROCKS_HOME=./ LD_LIBRARY_PATH=/usr/lib/jvm/java-18-openjdk-amd64/lib/server ./be/ut_build_ASAN/test/starrocks_test
```

StarRocks Backend UTはgoogle-testをベースにして構築されているため、UTの一部を実行するためのフィルタを渡すことができます。例えば、MapColumnに関連するテストのみを実行したい場合は、次のように実行できます。

```
UDF_RUNTIME_DIR=./ STARROCKS_HOME=./ LD_LIBRARY_PATH=/usr/lib/jvm/java-18-openjdk-amd64/lib/server ./be/ut_build_ASAN/test/starrocks_test --gtest_filter="*MapColumn*"
```

# ビルドオプション

## clangを使用してビルドする

StarRocksは `clang` を使用してもビルドできます。

```
CC=clang CXX=clang++ ./build.sh --be
```

その後、ビルドメッセージで以下のような似たようなメッセージが表示されます。

```
-- compiler Clang version 14.0.0
```

## 異なるリンカーを使用してビルドする

デフォルトのリンカーは遅いため、開発者はリンクを高速化するために異なるリンカーを指定することができます。
例えば、LLVMベースのリンカーである `lld` を使用することができます。

まず、`lld` をインストールする必要があります。

```
sudo apt install lld
```

その後、希望するリンカーを環境変数 STARROCKS_LINKER で設定します。
例えば：

```
STARROCKS_LINKER=lld ./build.sh --be
```

## 異なるタイプでビルドする

異なるBUILD_TYPE変数を使用して、異なるタイプでStarRocksをビルドすることができます。デフォルトのBUILD_TYPEは `RELEASE` です。例えば、次のようにして `ASAN` タイプでStarRocksをビルドすることができます。

```
BUILD_TYPE=ASAN ./build.sh --be
```