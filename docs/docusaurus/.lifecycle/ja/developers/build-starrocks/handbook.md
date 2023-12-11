---
displayed_sidebar: "Japanese"
---

# StarRocksのビルド方法

一般的には、以下を実行することでStarRocksをビルドすることができます

```
./build.sh
```

このコマンドはまずすべてのサードパーティ依存関係が準備できているかをチェックします。依存関係がすべて準備できている場合、StarRocksの`Backend`と`Frontend`をビルドします。

このコマンドが正常に実行された場合、生成されたバイナリは`output`ディレクトリにあります。

## FE/BEを別々にビルドする方法

毎回FEとBEの両方をビルドする必要はありません。それぞれを個別にビルドすることができます。
たとえば、BEのみをビルドする場合は次のようにします
```
./build.sh --be
```

そして、FEのみをビルドする場合は次のようにします
```
./build.sh --fe
```

# ユニットテストの実行方法

BEとFEのユニットテストは別々になっています。一般的には、BEのテストは次のように実行できます
```
./run-be-ut.sh
```

FEのテストは次のように実行できます
```
./run-fe-ut.sh
```

## コマンドラインでBE UTを実行する方法

現在、BE UTを実行するためにはいくつかの依存関係が必要であり、`./run-be-ut.sh`がそれを助けます。しかし、これでは柔軟性に欠けます。コマンドラインでUTを実行したい場合は、次のように実行します

```
UDF_RUNTIME_DIR=./ STARROCKS_HOME=./ LD_LIBRARY_PATH=/usr/lib/jvm/java-18-openjdk-amd64/lib/server ./be/ut_build_ASAN/test/starrocks_test
```

StarRocks Backend UTはgoogle-testに基づいて構築されているため、いくつかのUTを実行するためにフィルタを渡すことができます。たとえば、MapColumnに関連するテストのみを実行したい場合は、次のように実行します

```
UDF_RUNTIME_DIR=./ STARROCKS_HOME=./ LD_LIBRARY_PATH=/usr/lib/jvm/java-18-openjdk-amd64/lib/server ./be/ut_build_ASAN/test/starrocks_test --gtest_filter="*MapColumn*"
```

# ビルドオプション

## clangを使用してビルドする

`clang`を使用してStarRocksをビルドすることもできます

```
CC=clang CXX=clang++ ./build.sh --be
```

すると、ビルドメッセージに以下と類似したメッセージが表示されます

```
-- compiler Clang version 14.0.0
```

## 異なるリンカを使用してビルドする

デフォルトのリンカは遅いため、開発者はリンクを高速化するために異なるリンカを指定することができます。
たとえば、LLVMベースのリンカである`lld`を使用することができます。

まず、`lld`をインストールする必要があります。

```
sudo apt install lld
```
次に、環境変数STARROCKS_LINKERに使用したいリンカを設定します。
たとえば：

```
STARROCKS_LINKER=lld ./build.sh --be
```

## 異なるタイプでビルドする

異なるBUILD_TYPE変数を使用して異なるタイプでStarRocksをビルドすることができます。デフォルトのBUILD_TYPEは`RELEASE`です。たとえば、`ASAN`タイプでStarRocksをビルドする場合は次のようにします
```
BUILD_TYPE=ASAN ./build.sh --be
```