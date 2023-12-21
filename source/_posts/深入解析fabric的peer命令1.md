---
title: 深入解析fabric的peer命令(一)
date: 2023/5/30 10:24
tags: fabric区块链
categories: 区块链
top_img: ./img/41.jpg
cover: ./img/12.jpg

---

# 

## 深入解析fabric的peer命令（一）



### 一，探索思路

peer命令的源码在https://github.com/hyperledger/fabric的cmd目录下，这个目录下有

1.common
2.configtxgen
3.configtxlator
4.cryptogen
5.discover
6.ledgerutil
7.orderer
8.osnadmin
9.peer

这九个命令应该就是fabric提供给命令行的命令了

在/bin目录下有configtxgen  configtxlator  cryptogen  discover  fabric-ca-client  fabric-ca-server  ledgerutil  orderer  osnadmin  peer，说明确实是这样的。

因此如果想要探索之前文章里查询所有酒的命令

```sh
peer chaincode query -C winechannel -n mycontract -c '{"Args":["GetAllWines"]}'
```

之后到底发生了什么，就需要去分析下源码了。



### 二，上源码

```go
package main

import (
        _ "net/http/pprof"
        "os"
        "strings"

        "github.com/hyperledger/fabric/bccsp/factory"
        "github.com/hyperledger/fabric/internal/peer/chaincode"
        "github.com/hyperledger/fabric/internal/peer/channel"
        "github.com/hyperledger/fabric/internal/peer/common"
        "github.com/hyperledger/fabric/internal/peer/lifecycle"
        "github.com/hyperledger/fabric/internal/peer/node"
        "github.com/hyperledger/fabric/internal/peer/snapshot"
        "github.com/hyperledger/fabric/internal/peer/version"
        "github.com/spf13/cobra"
        "github.com/spf13/viper"
)

// The main command describes the service and
// defaults to printing the help message.
var mainCmd = &cobra.Command{Use: "peer"}

func main() {
        // For environment variables.
        viper.SetEnvPrefix(common.CmdRoot)
        viper.AutomaticEnv()
        replacer := strings.NewReplacer(".", "_")
        viper.SetEnvKeyReplacer(replacer)

        // Define command-line flags that are valid for all peer commands and
        // subcommands.
        mainFlags := mainCmd.PersistentFlags()

        mainFlags.String("logging-level", "", "Legacy logging level flag")
        viper.BindPFlag("logging_level", mainFlags.Lookup("logging-level"))
        mainFlags.MarkHidden("logging-level")

        cryptoProvider := factory.GetDefault()

        mainCmd.AddCommand(version.Cmd())
        mainCmd.AddCommand(node.Cmd())
        mainCmd.AddCommand(chaincode.Cmd(nil, cryptoProvider))
        mainCmd.AddCommand(channel.Cmd(nil))
        mainCmd.AddCommand(lifecycle.Cmd(cryptoProvider))
        mainCmd.AddCommand(snapshot.Cmd(cryptoProvider))

        // On failure Cobra prints the usage message and error string, so we only
        // need to exit with a non-0 status
        if mainCmd.Execute() != nil {
                os.Exit(1)
        }
}

```

### 三，逐行分析



1. #### 引入依赖包：
```go
import (
        _ "net/http/pprof"
        "os"
        "strings"

        "github.com/hyperledger/fabric/bccsp/factory"
        "github.com/hyperledger/fabric/internal/peer/chaincode"
        "github.com/hyperledger/fabric/internal/peer/channel"
        "github.com/hyperledger/fabric/internal/peer/common"
        "github.com/hyperledger/fabric/internal/peer/lifecycle"
        "github.com/hyperledger/fabric/internal/peer/node"
        "github.com/hyperledger/fabric/internal/peer/snapshot"
        "github.com/hyperledger/fabric/internal/peer/version"
        "github.com/spf13/cobra"
        "github.com/spf13/viper"
)
```
这部分代码导入了需要使用的包。`import` 关键字后面的括号里是导入的包的路径。



2. #### 定义 `mainCmd` 变量：
```go
var mainCmd = &cobra.Command{Use: "peer"}
```
这里创建了一个变量 `mainCmd`，它是 `cobra.Command` 类型的指针。`&cobra.Command{Use: "peer"}` 创建了一个新的 `cobra.Command` 实例，并将其赋值给 `mainCmd` 变量。`Use: "peer"` 表示命令的名称为 "peer"。



3. #### 主函数 `main()`：
```go
func main() {
        // For environment variables.
        viper.SetEnvPrefix(common.CmdRoot)
        viper.AutomaticEnv()
        replacer := strings.NewReplacer(".", "_")
        viper.SetEnvKeyReplacer(replacer)

        // Define command-line flags that are valid for all peer commands and
        // subcommands.
        mainFlags := mainCmd.PersistentFlags()

        mainFlags.String("logging-level", "", "Legacy logging level flag")
        viper.BindPFlag("logging_level", mainFlags.Lookup("logging-level"))
        mainFlags.MarkHidden("logging-level")

        cryptoProvider := factory.GetDefault()

        mainCmd.AddCommand(version.Cmd())
        mainCmd.AddCommand(node.Cmd())
        mainCmd.AddCommand(chaincode.Cmd(nil, cryptoProvider))
        mainCmd.AddCommand(channel.Cmd(nil))
        mainCmd.AddCommand(lifecycle.Cmd(cryptoProvider))
        mainCmd.AddCommand(snapshot.Cmd(cryptoProvider))

        // On failure Cobra prints the usage message and error string, so we only
        // need to exit with a non-0 status
        if mainCmd.Execute() != nil {
                os.Exit(1)
        }
}
```
这是程序的主函数 `main()`。在主函数中，首先进行了一些设置，然后添加了一些命令和子命令，最后执行了 `mainCmd`。

- `viper.SetEnvPrefix(common.CmdRoot)` 将环境变量的前缀设置为 `common.CmdRoot`。
- `viper.AutomaticEnv()` 自动加载环境变量。
- `replacer := strings.NewReplacer(".", "_")` 创建了一个 `strings.Replacer` 对象，用于将环境变量的键中的点号（`.`）替换为下划线（`_`）。
- `viper.SetEnvKeyReplacer(replacer)` 设置环境变量键的替换规则。
- `

mainFlags := mainCmd.PersistentFlags()` 创建了一个 `mainFlags` 变量，用于定义在所有 `peer` 命令和子命令中都有效的命令行标志。
- `mainFlags.String("logging-level", "", "Legacy logging level flag")` 定义了一个 `logging-level` 的命令行标志，它是一个字符串类型的标志，没有默认值，用于指定日志记录级别。

- `viper.BindPFlag("logging_level", mainFlags.Lookup("logging-level"))` 将命令行标志 `logging-level` 绑定到配置中的键 `logging_level`，以便从配置中读取对应的值。

- `mainFlags.MarkHidden("logging-level")` 将命令行标志 `logging-level` 标记为隐藏，不会在帮助信息中显示。

- `cryptoProvider := factory.GetDefault()` 获取默认的加密提供程序。

- `mainCmd.AddCommand(version.Cmd())` 添加了一个命令 `version.Cmd()` 到 `mainCmd` 中，用于显示版本信息。

- `mainCmd.AddCommand(node.Cmd())` 添加了一个命令 `node.Cmd()` 到 `mainCmd` 中，用于操作节点。

- `mainCmd.AddCommand(chaincode.Cmd(nil, cryptoProvider))` 添加了一个命令 `chaincode.Cmd(nil, cryptoProvider)` 到 `mainCmd` 中，用于操作链码。

- `mainCmd.AddCommand(channel.Cmd(nil))` 添加了一个命令 `channel.Cmd(nil)` 到 `mainCmd` 中，用于操作通道。

- `mainCmd.AddCommand(lifecycle.Cmd(cryptoProvider))` 添加了一个命令 `lifecycle.Cmd(cryptoProvider)` 到 `mainCmd` 中，用于操作生命周期。

- `mainCmd.AddCommand(snapshot.Cmd(cryptoProvider))` 添加了一个命令 `snapshot.Cmd(cryptoProvider)` 到 `mainCmd` 中，用于操作快照。

- `if mainCmd.Execute() != nil` 执行 `mainCmd`，如果执行返回错误，则退出程序并返回非零状态码。

  

#### 发现了！关键在这一句`mainCmd.AddCommand(chaincode.Cmd(nil, cryptoProvider))` 添加了一个命令 `chaincode.Cmd(nil, cryptoProvider)` 到 `mainCmd` 中，用于操作链码。有这句，导致我们在命令行使用peer命令，后面可以加chaincode



### 四，深入分析mainCmd.AddCommand(chaincode.Cmd(nil, cryptoProvider))这一句

首先，根据代码导入的包路径可以确定，`chaincode.Cmd` 函数来自于 `"github.com/hyperledger/fabric/internal/peer/chaincode"` 包。

在 Hyperledger Fabric 中，`chaincode.Cmd` 函数用于创建与链码相关的命令。它接受两个参数：`chaincodeID` 和 `cryptoProvider`。

- `chaincodeID` 参数是一个表示链码标识的字符串。在这里，传递的是 `nil`，表示没有指定特定的链码标识，即执行与链码相关的命令时不需要指定特定的链码。
- `cryptoProvider` 参数是一个加密提供程序。在这里，通过 `cryptoProvider := factory.GetDefault()` 获取默认的加密提供程序，并将其传递给 `chaincode.Cmd`。

因此，`mainCmd.AddCommand(chaincode.Cmd(nil, cryptoProvider))` 的作用是将与链码相关的命令添加到 `mainCmd` 中，其中使用了默认的加密提供程序，并且不指定特定的链码标识。

通过添加这个命令，可以在运行程序时使用 `peer chaincode` 命令执行与链码相关的操作，例如安装、实例化、升级、查询等。



### 五，分析github.com/hyperledger/fabric/internal/peer/chaincode包

https://xiutuuu.cloud/2023/05/30/%E6%B7%B1%E5%85%A5%E8%A7%A3%E6%9E%90fabric%E7%9A%84peer%E5%91%BD%E4%BB%A42/