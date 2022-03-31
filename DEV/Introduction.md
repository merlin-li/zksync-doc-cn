# 概览

zkSync 是以太坊的扩展和隐私引擎。 它目前的功能范围包括以太坊网络中 ETH 和 ERC20 代币的低 gas 交易、原子交换和限价订单以及原生 L2 NFT 支持。 本文档是对 zkSync 开发生态系统的说明。

zkSync 建立在 ZK Rollup 架构之上。 ZK Rollup 是一种 L2 扩展解决方案，其中所有资金都由主链上的智能合约持有，而计算和存储则在链下执行。 对于每个 Rollup 块，都会生成状态转换零知识证明 (SNARK)，并由主链合约进行验证。 这个 SNARK 包括 Rollup 块中每笔交易的有效性证明。 此外，每个区块的公共数据更新都通过主链网络以廉价的 calldata 发布。

此架构提供以下保证：

- Rollup 验证器永远不会破坏状态或窃取资金（与侧链不同）。
- 即使验证者因数据可用而停止合作，用户也始终可以从 Rollup 中检索资金（与 Plasma 不同）。
- 多亏了有效性证明，用户或任何其他受信任方都不需要在线监控 Rollup 块以防止欺诈（与支付渠道或 Optimistic Rollups 不同）。

也就是说，ZK Rollup 严格继承了底层 L1 的安全保障。

## **功能**

首先，zkSync 作为一种扩容解决方案，能够进行传输，并且可以快速且便宜地进行。本文档的[支付部分](https://merlin-li.github.io/zksync/payments)介绍了核心 zkSync 功能的接口和原理。

其次，zkSync 对智能合约比较友好。到目前为止，可以使用 Zinc、基于 Rust 的类型安全编程语言编写合约，甚至可以重用现有的 Solidity 代码。合约的互操作性在[这里](https://merlin-li.github.io/zksync/contracts)查看。

第三，zkSync 对交易所比较友好。原子交换——交换协议的重要组成部分——已经在主网上[可用](https://merlin-li.github.io/zksync/contracts)！

第四，zkSync 原生支持 NFT。您可以在我们的[钱包](https://wallet.zksync.io/)中试用。

最后，所有主要平台都实现了 zkSync 支持。查看我们的SDK文档部分，并开始使用 zkSync 进行开发！
