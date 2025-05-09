---
sidebar_position: 1
slug: protocol-buffers
---

# Protocol Buffers 编译器安装及多语言编译

## Protocol Buffers 简介
[Protocol Buffers](https://protobuf.dev/) [Protocol Buffers (中文)](https://astro-xao.github.io/protocolbuffers.github.io/) 是一种与语言无关、与平台无关的可扩展机制，用于序列化结构化数据。

## Protocol Buffers 安装

### 安装预编译二进制文件（任何操作系统）
1. 从 https://github.com/google/protobuf/releases 下载对应平台的 zip 文件(`protoc-<version>-<os>-<arch>.zip`)
    ```bash
    PB_REL="https://github.com/protocolbuffers/protobuf/releases"
    curl -LO $PB_REL/download/v30.2/protoc-30.2-linux-x86_64.zip
    ```
2. 将文件解压到 $HOME/.local 下或你选择的目录中。
    ```
    unzip protoc-30.2-linux-x86_64.zip -d $HOME/.local
    ```
3. 更新环境变量，包含 `protoc` 可执行文件。
    ```
    export PATH="$PATH:$HOME/.local/bin"
    ```
### 使用包管理器安装
- Linux
    ```
    apt install -y protobuf-compiler
    protoc --version  # Ensure compiler version is 3+
    ```
- MacOS
    ```
    brew install protobuf
    protoc --version  # Ensure compiler version is 3+
    ```
- Windows
    ```
    > winget install protobuf
    > protoc --version # Ensure compiler version is 3+
    ```

## Protocol Buffers 文件编译
定义好 `.proto` 文件后，需要将其编译成不同语言的代码。这需要不同语言的插件支持。

### C++ 编译
`.proto` 文件编译成 `cpp` 代码无需下载插件，直接运行 `protoc` 同时输入(`$SRC_DIR`)和输出(`$DST_DIR`)位置。
    ```
    protoc -I=$SRC_DIR --cpp_out=$DST_DIR $SRC_DIR/addressbook.proto
    ```

### Go 编译
1. 安装 Go protocol buffers 插件
    ```
    go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
    ```
2. 编译
    ```
    protoc -I=$SRC_DIR --go_out=$DST_DIR $SRC_DIR/addressbook.proto
    ```


