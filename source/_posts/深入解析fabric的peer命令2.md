---
title: 深入解析fabric的peer命令(二)分析chaincode包
date: 2023/5/30 10:24
tags: fabric区块链
categories: 区块链
top_img: ./img/41.jpg
cover: ./img/41.jpg

---

## 分析github.com/hyperledger/fabric/internal/peer/chaincode包

### 一，探索思路

这个目录的tree是这样的

```
.
├── chaincode.go
├── common.go
├── common_test.go
├── flags_test.go
├── install.go
├── install_test.go
├── instantiate.go
├── instantiate_test.go
├── invoke.go
├── invoke_test.go
├── list.go
├── list_test.go
├── mock
│   ├── deliver_client.go
│   ├── deliver.go
│   └── signer_serializer.go
├── package.go
├── package_test.go
├── query.go
├── query_test.go
├── signpackage.go
├── signpackage_test.go
├── testdata
│   ├── connectionprofile-uneven.yaml
│   ├── connectionprofile.yaml
│   └── src
│       └── chaincodes
│           └── noop
│               └── chaincode.go
├── upgrade.go
└── upgrade_test.go

```

显而易见，我们要探索的命令：

```sh
peer chaincode query -C winechannel -n mycontract -c '{"Args":["GetAllWines"]}'
```

chaincode.go和query.go 是关键！



### 2，上源码

#### （1）chaincode.go

```go
/*
Copyright IBM Corp. All Rights Reserved.

SPDX-License-Identifier: Apache-2.0
*/

package chaincode

import (
        "fmt"
        "time"

        "github.com/hyperledger/fabric/bccsp"
        "github.com/hyperledger/fabric/common/flogging"
        "github.com/hyperledger/fabric/internal/peer/common"
        "github.com/hyperledger/fabric/internal/peer/packaging"
        "github.com/spf13/cobra"
        "github.com/spf13/pflag"
)

const (
        chainFuncName = "chaincode"
        chainCmdDes   = "Operate a chaincode: install|instantiate|invoke|package|query|signpackage|upgrade|list."
)

var logger = flogging.MustGetLogger("chaincodeCmd")

// XXX This is a terrible singleton hack, however
// it simply making a latent dependency explicit.
// It should be removed along with the other package
// scoped variables
var platformRegistry = packaging.NewRegistry(packaging.SupportedPlatforms...)

func addFlags(cmd *cobra.Command) {
        common.AddOrdererFlags(cmd)
        flags := cmd.PersistentFlags()
        flags.StringVarP(&transient, "transient", "", "", "Transient map of arguments in JSON encoding")
}

// Cmd returns the cobra command for Chaincode
func Cmd(cf *ChaincodeCmdFactory, cryptoProvider bccsp.BCCSP) *cobra.Command {
        addFlags(chaincodeCmd)

        chaincodeCmd.AddCommand(installCmd(cf, nil, cryptoProvider))
        chaincodeCmd.AddCommand(instantiateCmd(cf, cryptoProvider))
        chaincodeCmd.AddCommand(invokeCmd(cf, cryptoProvider))
        chaincodeCmd.AddCommand(packageCmd(cf, nil, nil, cryptoProvider))
        chaincodeCmd.AddCommand(queryCmd(cf, cryptoProvider))
        chaincodeCmd.AddCommand(signpackageCmd(cf, cryptoProvider))
        chaincodeCmd.AddCommand(upgradeCmd(cf, cryptoProvider))
        chaincodeCmd.AddCommand(listCmd(cf, cryptoProvider))

        return chaincodeCmd
}

// Chaincode-related variables.
var (
        chaincodeLang         string
        chaincodeCtorJSON     string
        chaincodePath         string
        chaincodeName         string
        chaincodeUsr          string // Not used
        chaincodeQueryRaw     bool
        chaincodeQueryHex     bool
        channelID             string
        chaincodeVersion      string
        policy                string
        escc                  string
        vscc                  string
        policyMarshalled      []byte
        transient             string
        isInit                bool
        collectionsConfigFile string
        collectionConfigBytes []byte
        peerAddresses         []string
        tlsRootCertFiles      []string
        connectionProfile     string
        waitForEvent          bool
        waitForEventTimeout   time.Duration
)

var chaincodeCmd = &cobra.Command{
        Use:   chainFuncName,
        Short: fmt.Sprint(chainCmdDes),
        Long:  fmt.Sprint(chainCmdDes),
        PersistentPreRun: func(cmd *cobra.Command, args []string) {
                common.InitCmd(cmd, args)
                common.SetOrdererEnv(cmd, args)
        },
}

var flags *pflag.FlagSet

func init() {
        resetFlags()
}

// Explicitly define a method to facilitate tests
func resetFlags() {
        flags = &pflag.FlagSet{}

        flags.StringVarP(&chaincodeLang, "lang", "l", "golang",
                fmt.Sprintf("Language the %s is written in", chainFuncName))
        flags.StringVarP(&chaincodeCtorJSON, "ctor", "c", "{}",
                fmt.Sprintf("Constructor message for the %s in JSON format", chainFuncName))
        flags.StringVarP(&chaincodePath, "path", "p", common.UndefinedParamValue,
                fmt.Sprintf("Path to %s", chainFuncName))
        flags.StringVarP(&chaincodeName, "name", "n", common.UndefinedParamValue,
                "Name of the chaincode")
        flags.StringVarP(&chaincodeVersion, "version", "v", common.UndefinedParamValue,
                "Version of the chaincode specified in install/instantiate/upgrade commands")
        flags.StringVarP(&chaincodeUsr, "username", "u", common.UndefinedParamValue,
                "Username for chaincode operations when security is enabled")
        flags.StringVarP(&channelID, "channelID", "C", "",
                "The channel on which this command should be executed")
        flags.StringVarP(&policy, "policy", "P", common.UndefinedParamValue,
                "The endorsement policy associated to this chaincode")
        flags.StringVarP(&escc, "escc", "E", common.UndefinedParamValue,
                "The name of the endorsement system chaincode to be used for this chaincode")
        flags.StringVarP(&vscc, "vscc", "V", common.UndefinedParamValue,
                "The name of the verification system chaincode to be used for this chaincode")
        flags.BoolVarP(&isInit, "isInit", "I", false,
                "Is this invocation for init (useful for supporting legacy chaincodes in the new lifecycle)")
        flags.BoolVarP(&getInstalledChaincodes, "installed", "", false,
                "Get the installed chaincodes on a peer")
        flags.BoolVarP(&getInstantiatedChaincodes, "instantiated", "", false,
                "Get the instantiated chaincodes on a channel")
        flags.StringVar(&collectionsConfigFile, "collections-config", common.UndefinedParamValue,
                "The fully qualified path to the collection JSON file including the file name")
        flags.StringArrayVarP(&peerAddresses, "peerAddresses", "", []string{common.UndefinedParamValue},
                "The addresses of the peers to connect to")
        flags.StringArrayVarP(&tlsRootCertFiles, "tlsRootCertFiles", "", []string{common.UndefinedParamValue},
                "If TLS is enabled, the paths to the TLS root cert files of the peers to connect to. The order and number of certs specified should match the --peerAddresses flag")
        flags.StringVarP(&connectionProfile, "connectionProfile", "", common.UndefinedParamValue,
                "Connection profile that provides the necessary connection information for the network. Note: currently only supported for providing peer connection information")
        flags.BoolVar(&waitForEvent, "waitForEvent", false,
                "Whether to wait for the event from each peer's deliver filtered service signifying that the 'invoke' transaction has been committed successfully")
        flags.DurationVar(&waitForEventTimeout, "waitForEventTimeout", 30*time.Second,
                "Time to wait for the event from each peer's deliver filtered service signifying that the 'invoke' transaction has been committed successfully")
        flags.BoolVarP(&createSignedCCDepSpec, "cc-package", "s", false,
                "create CC deployment spec for owner endorsements instead of raw CC deployment spec")
        flags.BoolVarP(&signCCDepSpec, "sign", "S", false,
                "if creating CC deployment spec package for owner endorsements, also sign it with local MSP")
        flags.StringVarP(&instantiationPolicy, "instantiate-policy", "i", "",
                "instantiation policy for the chaincode")
}

func attachFlags(cmd *cobra.Command, names []string) {
        cmdFlags := cmd.Flags()
        for _, name := range names {
                if flag := flags.Lookup(name); flag != nil {
                        cmdFlags.AddFlag(flag)
                } else {
                        logger.Fatalf("Could not find flag '%s' to attach to command '%s'", name, cmd.Name())
                }
        }
}
```



#### （2）query.go

```go
/*
Copyright IBM Corp. All Rights Reserved.

SPDX-License-Identifier: Apache-2.0
*/

package chaincode

import (
        "errors"
        "fmt"

        "github.com/hyperledger/fabric/bccsp"
        "github.com/spf13/cobra"
)

var chaincodeQueryCmd *cobra.Command

// queryCmd returns the cobra command for Chaincode Query
func queryCmd(cf *ChaincodeCmdFactory, cryptoProvider bccsp.BCCSP) *cobra.Command {
        chaincodeQueryCmd = &cobra.Command{
                Use:       "query",
                Short:     fmt.Sprintf("Query using the specified %s.", chainFuncName),
                Long:      fmt.Sprintf("Get endorsed result of %s function call and print it. It won't generate transaction.", chainFuncName),
                ValidArgs: []string{"1"},
                RunE: func(cmd *cobra.Command, args []string) error {
                        return chaincodeQuery(cmd, cf, cryptoProvider)
                },
        }
        flagList := []string{
                "ctor",
                "name",
                "channelID",
                "peerAddresses",
                "tlsRootCertFiles",
                "connectionProfile",
        }
        attachFlags(chaincodeQueryCmd, flagList)

        chaincodeQueryCmd.Flags().BoolVarP(&chaincodeQueryRaw, "raw", "r", false,
                "If true, output the query value as raw bytes, otherwise format as a printable string")
        chaincodeQueryCmd.Flags().BoolVarP(&chaincodeQueryHex, "hex", "x", false,
                "If true, output the query value byte array in hexadecimal. Incompatible with --raw")

        return chaincodeQueryCmd
}

func chaincodeQuery(cmd *cobra.Command, cf *ChaincodeCmdFactory, cryptoProvider bccsp.BCCSP) error {
        if channelID == "" {
                return errors.New("The required parameter 'channelID' is empty. Rerun the command with -C flag")
        }
        // Parsing of the command line is done so silence cmd usage
        cmd.SilenceUsage = true

        var err error
        if cf == nil {
                cf, err = InitCmdFactory(cmd.Name(), true, false, cryptoProvider)
                if err != nil {
                        return err
                }
        }

        return chaincodeInvokeOrQuery(cmd, false, cf)
}
```



### 三，逐句分析chaincode.go



1. #### 导入依赖包：
```go
import (
        "fmt"
        "time"

        "github.com/hyperledger/fabric/bccsp"
        "github.com/hyperledger/fabric/common/flogging"
        "github.com/hyperledger/fabric/internal/peer/common"
        "github.com/hyperledger/fabric/internal/peer/packaging"
        "github.com/spf13/cobra"
        "github.com/spf13/pflag"
)
```
这部分代码导入了需要使用的包。`import` 关键字后面的括号里是导入的包的路径。



2. #### 常量和全局变量：
```go
const (
        chainFuncName = "chaincode"
        chainCmdDes   = "Operate a chaincode: install|instantiate|invoke|package|query|signpackage|upgrade|list."
)

var logger = flogging.MustGetLogger("chaincodeCmd")
var platformRegistry = packaging.NewRegistry(packaging.SupportedPlatforms...)
```
这里定义了一个常量 `chainFuncName` 和一个变量 `chainCmdDes`。`logger` 和 `platformRegistry` 是全局变量。



3. #### 辅助函数：
```go
func addFlags(cmd *cobra.Command) {
        common.AddOrdererFlags(cmd)
        flags := cmd.PersistentFlags()
        flags.StringVarP(&transient, "transient", "", "", "Transient map of arguments in JSON encoding")
}
```
这是一个名为 `addFlags` 的函数。它接受一个 `*cobra.Command` 类型的参数 `cmd`。在函数内部，它调用 `common.AddOrdererFlags(cmd)` 和 `cmd.PersistentFlags()` 函数，然后使用 `flags.StringVarP()` 函数将一个标志 `transient` 添加到 `cmd` 中。



4. #### `Cmd` 函数：
```go
func Cmd(cf *ChaincodeCmdFactory, cryptoProvider bccsp.BCCSP) *cobra.Command {
        addFlags(chaincodeCmd)

        chaincodeCmd.AddCommand(installCmd(cf, nil, cryptoProvider))
        chaincodeCmd.AddCommand(instantiateCmd(cf, cryptoProvider))
        chaincodeCmd.AddCommand(invokeCmd(cf, cryptoProvider))
        chaincodeCmd.AddCommand(packageCmd(cf, nil, nil, cryptoProvider))
        chaincodeCmd.AddCommand(queryCmd(cf, cryptoProvider))
        chaincodeCmd.AddCommand(signpackageCmd(cf, cryptoProvider))
        chaincodeCmd.AddCommand(upgradeCmd(cf, cryptoProvider))
        chaincodeCmd.AddCommand(listCmd(cf, cryptoProvider))

        return chaincodeCmd
}
```
这是一个名为 `Cmd` 的函数。它接受两个参数：`cf` 是 `*ChaincodeCmdFactory` 类型，`cryptoProvider` 是 `bccsp.BCCSP` 类型。该函数返回一个 `*cobra.Command` 类型的指针。

- 在函数内部，调用 `addFlags(chaincodeCmd)` 函数为 `chaincodeCmd` 添加标志。
- 使用 `chaincodeCmd.AddCommand()` 函数将多个命令添加到 `chaincodeCmd` 中，这些命令包括 `installCmd`、`instantiateCmd`、`invokeCmd`、`packageCmd`、`queryCmd`、`signpackageCmd`、`upgradeCmd` 和 `listCmd`。
- 返回 `chaincodeCmd`。
- 

#### 5.变量定义：

```go
var (
        chaincodeLang         string
        chaincodeCtorJSON     string
        chaincodePath         string
        chaincodeName         string
        chaincodeUsr          string // Not used
        chaincodeQueryRaw     bool
        chaincodeQueryHex     bool
        channelID             string
        chaincodeVersion      string
        policy                string
        escc                  string
        vscc                  string
        policyMarshalled      []byte
        transient             string
        isInit                bool
        collectionsConfigFile string
        collectionConfigBytes []byte
        peerAddresses         []string
        tlsRootCertFiles      []string
        connectionProfile     string
        waitForEvent          bool
        waitForEventTimeout   time.Duration
)
```
这是一组变量的定义，它们将在后续的代码中使用。



6. #### `chaincodeCmd` 命令：
```go
var chaincodeCmd = &cobra.Command{
        Use:   chainFuncName,
        Short: fmt.Sprint(chainCmdDes),
        Long:  fmt.Sprint(chainCmdDes),
        PersistentPreRun: func(cmd *cobra.Command, args []string) {
                common.InitCmd(cmd, args)
                common.SetOrdererEnv(cmd, args)
        },
}
```
这是一个名为 `chaincodeCmd` 的命令。它是一个 `*cobra.Command` 类型的变量。在这个命令的定义中，我们指定了 `Use`、`Short` 和 `Long` 字段来描述命令的用法和说明。此外，我们还定义了一个 `PersistentPreRun` 回调函数，在命令运行之前会执行这个函数。



7. #### `flags` 变量和 `resetFlags` 函数：
```go
var flags *pflag.FlagSet

func init() {
        resetFlags()
}

func resetFlags() {
        flags = &pflag.FlagSet{}

        // 标志的定义
        // ...
}
```
这里定义了一个 `flags` 变量，它是 `pflag.FlagSet` 类型的指针。在 `init` 函数中，调用了 `resetFlags` 函数，该函数将 `flags` 设置为一个新的 `FlagSet` 对象。



8. #### `attachFlags` 函数：
```go
func attachFlags(cmd *cobra.Command, names []string) {
        cmdFlags := cmd.Flags()
        for _, name := range names {
                if flag := flags.Lookup(name); flag != nil {
                        cmdFlags.AddFlag(flag)
                } else {
                        logger.Fatalf("Could not find flag '%s' to attach to command '%s'", name, cmd.Name())
                }
        }
}
```
这是一个名为 `attachFlags` 的函数。它接受一个 `*cobra.Command` 类型的参数 `cmd` 和一个字符串切片 `names`。函数内部通过 `cmd.Flags()` 获取命令的标志集合，并循环遍历 `names`，然后通过 `flags.Lookup()` 函数查找对应的标志，并将其添加到命令的标志集合中。



### 四，逐句分析query.go



1. #### 导入依赖包：
```go
import (
        "errors"
        "fmt"

        "github.com/hyperledger/fabric/bccsp"
        "github.com/spf13/cobra"
)
```
这部分代码导入了需要使用的包。`import` 关键字后面的括号里是导入的包的路径。



2. #### 声明全局变量：
```go
var chaincodeQueryCmd *cobra.Command
```
这里声明了一个全局变量 `chaincodeQueryCmd`，它是一个指向 `cobra.Command` 类型的指针。



3. #### `queryCmd` 函数：
```go
func queryCmd(cf *ChaincodeCmdFactory, cryptoProvider bccsp.BCCSP) *cobra.Command {
        chaincodeQueryCmd = &cobra.Command{
                Use:       "query",
                Short:     fmt.Sprintf("Query using the specified %s.", chainFuncName),
                Long:      fmt.Sprintf("Get endorsed result of %s function call and print it. It won't generate transaction.", chainFuncName),
                ValidArgs: []string{"1"},
                RunE: func(cmd *cobra.Command, args []string) error {
                        return chaincodeQuery(cmd, cf, cryptoProvider)
                },
        }
        flagList := []string{
                "ctor",
                "name",
                "channelID",
                "peerAddresses",
                "tlsRootCertFiles",
                "connectionProfile",
        }
        attachFlags(chaincodeQueryCmd, flagList)

        chaincodeQueryCmd.Flags().BoolVarP(&chaincodeQueryRaw, "raw", "r", false,
                "If true, output the query value as raw bytes, otherwise format as a printable string")
        chaincodeQueryCmd.Flags().BoolVarP(&chaincodeQueryHex, "hex", "x", false,
                "If true, output the query value byte array in hexadecimal. Incompatible with --raw")

        return chaincodeQueryCmd
}
```
这是一个名为 `queryCmd` 的函数。它接受两个参数：`cf` 和 `cryptoProvider`，并返回一个 `*cobra.Command` 类型的指针。

- 函数内部创建了一个新的 `cobra.Command` 实例，并将其赋值给 `chaincodeQueryCmd` 变量。`Use` 字段表示命令的名称为 "query"，`Short` 字段提供了对该命令的简短描述，`Long` 字段提供了更详细的描述，`ValidArgs` 字段指定了有效的参数列表。

- 在 `RunE` 字段中定义了一个匿名函数，该函数将调用 `chaincodeQuery` 函数并返回其结果。

- 通过 `attachFlags` 函数将一组标志（flagList）附加到 `chaincodeQueryCmd` 中。

- 使用 `chaincodeQueryCmd.Flags().BoolVarP()` 函数定义了两个布尔类型的标志：`chaincodeQueryRaw` 和 `chaincodeQueryHex`。

  

4. #### `chaincodeQuery` 函数：
```go
func chaincodeQuery(cmd *cobra.Command, cf *ChaincodeCmdFactory, cryptoProvider bccsp.BCCSP) error {
        if channelID == "" {
                return errors.New("The required parameter 'channelID' is empty. Rerun the command with

 -C flag")
        }
        // Parsing of the command line is done so silence cmd usage
        cmd.SilenceUsage = true

        var err error
        if cf == nil {
                cf, err = InitCmdFactory(cmd.Name(), true, false, cryptoProvider)
                if err != nil {
                        return err
                }
        }

        return chaincodeInvokeOrQuery(cmd, false, cf)
}
```
这是一个名为 `chaincodeQuery` 的函数。它接受三个参数：`cmd` 是一个 `*cobra.Command` 类型，`cf` 是 `*ChaincodeCmdFactory` 类型，`cryptoProvider` 是 `bccsp.BCCSP` 类型。该函数返回一个 `error` 类型的值。

- 首先，检查 `channelID` 是否为空字符串，如果为空，则返回一个错误。
- 设置 `cmd.SilenceUsage` 为 `true`，以静默输出命令使用说明。
- 如果 `cf` 为 `nil`，则调用 `InitCmdFactory` 函数初始化 `cf`，并将其赋值给 `cf` 变量。
- 调用 `chaincodeInvokeOrQuery` 函数执行链码的调用或查询操作，并返回其结果。

### 五，继续分析chaincodeInvokeOrQuery函数

https://xiutuuu.cloud/2023/06/03/%E6%B7%B1%E5%85%A5%E8%A7%A3%E6%9E%90fabric%E7%9A%84peer%E5%91%BD%E4%BB%A43/