# 智能合约

zkSync 有望推出高效、安全、图灵完备的多语言智能合约。

- [智能合约](https://merlin-li.github.io/zksync/contracts#smart-contracts)
    - [编程模型](https://merlin-li.github.io/zksync/contracts#programming-model)
    - [可组合性](https://merlin-li.github.io/zksync/contracts#composability)
    - [zkEVM](https://merlin-li.github.io/zksync/contracts#zkevm)
    - [Solidity](https://merlin-li.github.io/zksync/contracts#solidity)
        - [移植智能合约](https://merlin-li.github.io/zksync/contracts#porting-smart-contracts)
        - [UI 交互](https://merlin-li.github.io/zksync/contracts#ui-interaction)
        - [提交交易](https://merlin-li.github.io/zksync/contracts#submitting-transactions)
        - [部署](https://merlin-li.github.io/zksync/contracts#deployment)
    - [Zinc](https://merlin-li.github.io/zksync/contracts#zinc)
    - [获取帮助](https://merlin-li.github.io/zksync/contracts#getting-help)

## **编程模型**

zkSync 智能合约编程模型继承了以太坊的模型。

将首先支持 Solidity，因此您可以使用循环、递归、任意长度的向量和映射等。 局部变量存储在堆栈或堆上，而合约存储是全局访问的。 合约通过强类型接口相互调用，并且可以访问公共存储字段。

## **可组合性**

zkSync 智能合约能够像以太坊智能合约一样相互调用。 每个调用交易树都是原子的，无论涉及多少合约实例。

任何 DeFi 项目都可以迁移到 zkSync，因为大多数现有的 Solidity 代码都可以在不更改的情况下部署。

## **zkEVM**

zkEVM 是一个高效、图灵完备、对 SNARK 友好的虚拟机，用于执行智能合约。

最先进的优化应用于智能合约字节码，而虚拟机本身针对高负载进行了优化，使其能够在眨眼之间执行交易。

该机器对 SNARK 友好； 也就是说，可以在 SNARK 中证明执行跟踪。 但是，每个程序不需要一个电路。 可以改为使用单个电路，只需审核一次。

sync VM 的目标证明系统是[PLONK](https://eprint.iacr.org/2019/953).

## **Solidity**

### **部署智能合约**

大多数 DeFi 和 NFT 项目无需更改代码即可运行。 但是，在第一个版本中，对 SHA256 和 Keccak256 的调用将由编译器自动替换为电路友好的哈希函数。 目前也不支持其他一些加密原语，例如 ecrecover 和加密预编译。

### **UI 交互**

您可以通过我们的 Web3 API 和 Ethers SDK 与智能合约和 zkSync 网络进行完全交互：

- 对于读取请求：任何语言的任何兼容 web3 的框架都可以开箱即用，并具有额外的可选 zkSync L2 特定功能。
- 对于写请求（发送交易）：由于 L1 和 L2 的根本区别，您将不得不编写一些额外的代码（例如，zkSync 支持以任何代币支付费用，因此发送交易将涉及选择代币支付费用）。

您可以在我们的 [Discord](https://discord.gg/nMaPGrDDwk) 上提出问题并获得帮助

### **发送交易**

对于与智能合约的交互，用户将使用 calldata 的哈希签署 EIP712 消息。 由于 EIP712 基于原生以太坊签名，所有钱包，甚至硬件钱包，都无需任何扩展即可工作。

### 部署

我们将很快发布测试网! 在[这里](https://forms.gle/jQQnJJeuVSVcmkqj9)登记。

## **Zinc**

[Zinc](https://github.com/matter-labs/zinc) 是在 zkSync 平台上开发智能合约和 SNARK 电路的新兴框架。

现有的 ZKP 框架缺乏特定于智能合约的功能。 由于智能合约处理有价值的金融资产，因此安全性和安全性至关重要。 这就是为什么现代智能合约语言（如 Simplicity 或 Libra's Move）的设计者更倾向于代码的安全性和形式可验证性而非表达性。

Zinc 旨在通过提供一种简单、可靠的智能合约语言来填补这两个世界之间的空白，该语言针对 ZKP 电路进行了优化，并且易于开发人员学习。

该框架包括一个简单的、图灵完备的、以安全为中心的通用语言，专为开发智能合约和零知识证明电路而设计，学习曲线平坦。 语法和语义紧跟[Rust](https://www.rust-lang.org/)。

Zinc 编译器使用 LLVM 作为其中端和后端，为代码优化提供了一套强大的解决方案。

**我们目前完全专注于[Solidity 优先的方案](https://medium.com/matter-labs/unisync-a-port-of-uniswap-v2-on-the-zkevm-b12954748504). Solidity 发布后，我们将恢复 Zinc 的工作！**

## **获得帮助**

大多数问题都在我们的 [zkEVM 常见问题](https://merlin-li.github.io/zkevm/) 中得到解答，主要概念在我们的 [Medium](https://medium.com/matter-labs) 上进行了概述。

您可以在我们的 [Discord](https://discord.gg/5b6s7VTC) 开发者聊天室提出问题并获得帮助。