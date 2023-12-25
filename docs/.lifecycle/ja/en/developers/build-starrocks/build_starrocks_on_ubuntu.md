---
displayed_sidebar: English
---

x86_64 および aarch64 でのビルドサポート

### 前提条件

```
sudo apt-get update
```

```
sudo apt-get install automake binutils-dev bison byacc ccache flex libiberty-dev libtool maven zip python3 python-is-python3 -y
```

### コンパイラ

Ubuntuのバージョンが22.04以上の場合、以下をインストールできます。
```
sudo apt-get install cmake gcc g++ default-jdk -y
```

Ubuntuのバージョンが22.04未満の場合、以下のツールおよびコンパイラのバージョンを確認してください。

##### 1. GCC/G++

GCC/G++ のバージョンは10.3以上である必要があります。
```
gcc --version
g++ --version
```
GCC/G++ をインストールする(https://gcc.gnu.org/releases.html)

##### 2. JDK

OpenJDK のバージョンは8以上である必要があります。
```
java --version
```
OpenJDK をインストールする(https://openjdk.org/install)

##### 3. CMake

CMake のバージョンは3.20.1以上である必要があります。

```
cmake --version
```
CMake をインストールする(https://cmake.org/download)


### コンパイル速度の向上

デフォルトのコンパイル並列数は **CPUコア数 / 4** です。
コンパイル速度を向上させたい場合、並列数を増やすことができます。

1. CPUコアが32個ある場合、デフォルトの並列数は8です。

```
./build.sh
```

2. CPUコアが32個あり、24個のコアを使用してコンパイルしたい場合。

```
./build.sh -j 24
```

### FAQ

1. Ubuntu 20.04で `aws_cpp_sdk` のビルドに失敗しました。
```
Error: undefined reference to pthread_create
```
このエラーはCMakeのバージョンが古いことが原因です。CMakeのバージョンを少なくとも3.20.1にアップグレードしてください。
