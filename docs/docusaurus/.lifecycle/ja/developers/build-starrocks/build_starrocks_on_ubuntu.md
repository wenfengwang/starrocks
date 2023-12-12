---
displayed_sidebar: "Japanese"
---

x86_64およびaarch64でサポートを構築

### 必要条件

```
sudo apt-get update
```

```
sudo apt-get install automake binutils-dev bison byacc ccache flex libiberty-dev libtool maven zip python3 python-is-python3 -y
```

### コンパイラ

Ubuntuバージョンが22.04以上の場合は、次のように実行できます
```
sudo apt-get install cmake gcc g++ default-jdk -y
```

Ubuntuバージョンが22.04未満の場合。
以下のツールとコンパイラのバージョンを確認します。

##### 1. GCC/G++

GCC/G++のバージョンは10.3以上である必要があります
```
gcc --version
g++ --version
```
GCC/G++をインストールします(https://gcc.gnu.org/releases.html)

##### 2. JDK

OpenJDKのバージョンは8以上である必要があります
```
java --version
```
OpenJdkをインストールします(https://openjdk.org/install)

##### 3. CMake

cmakeのバージョンは3.20.1以上である必要があります

```
cmake --version
```
cmakeをインストールします(https://cmake.org/download)


### コンパイル速度を向上させる

デフォルトのコンパイル並列度は**CPUコア数 / 4**となります。
コンパイル速度を向上させたい場合は、並列度を向上させることができます。

1. CPUコアが32あるとして、デフォルトの並列度は8です。

```
./build.sh
```

2. CPUコアが32あるとして、24コアを使用してコンパイルしたい場合は、以下のように実行します。

```
./build.sh -j 24
```

### FAQ

1. Ubuntu 20.04で`aws_cpp_sdk`の構築に失敗しました。
```
Error: undefined reference to pthread_create
```
エラーは、CMakeのバージョンが低いことから発生しております。CMakeのバージョンを少なくとも3.20.1にアップグレードしてください