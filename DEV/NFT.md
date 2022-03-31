# NFT

# **zkSync 1.x 中的 NFT**

zkSync 1.x 上的 NFT 支持就在这里！ 功能包括铸造、转移和原子交换 NFT。 用户还可以将 NFT 赎回到主网。

本页演示了如何在 zkSync 1.x 中实现 NFT，并为您提供将 NFT 集成到项目中的教程。

- [概述](https://merlin-li.github.io/zksync/nft#overview)
- [设置](https://merlin-li.github.io/zksync/nft#setup)
- [铸造](https://merlin-li.github.io/zksync/nft#mint)
- [转移](https://merlin-li.github.io/zksync/nft#transfer)
- [交换](https://merlin-li.github.io/zksync/nft#swap)
- [赎回到 Layer 1](https://merlin-li.github.io/zksync/nft#withdrawal-to-layer-1)

## **概述**

NFT 地址将对 NFT 内容和元数据进行如下编码：

```jsx
  address = truncate_to_20_bytes(rescue_hash(creator_account_id || serial_id || content_hash));
```

这以加密方式确保了两个不变量：

- NFT 地址是对创建者、NFT 序列号及其内容哈希的唯一承诺。
- NFT 地址不能被任何人控制或在主网上有智能合约代码。

注意：在 zkSync 1.x 中，可以使用相同的内容哈希铸造多个 NFT。

## **设置**

在开始本教程之前，请阅读我们的[入门指南](https://merlin-li.github.io/api/sdk/js/tutorial.html#getting-started)。

### **安装 zkSync 库**

```bash
yarn add zksync
```

### **连接 zkSync 网络**

对于本教程，让我们连接到 Rinkeby 测试网。 主网和 Ropsten 的步骤是相同的。

```jsx
const syncProvider =await zksync.getDefaultProvider('rinkeby');
```

## **铸造**

为了铸造 NFT，我们将引入一个带有参数的新操作码 MINT_NFT：

- creator_account_id
- content_hash
- recipient_account_id

通过传入 recipient_account_id，我们允许创建者选择是给自己铸造还是直接铸造给他人。

### **强制唯一性**

为了强制 NFT 代币 ID 的唯一性，我们使用 zkSync 余额树中的最后一个帐户来跟踪代币 ID。 这个帐户，我们将其称为 SpecialNFTAccount，将有一个 SPECIAL_NFT_TOKEN 余额，代表最新铸币厂的 token_id。

```jsx
  // token ID is represented by:
  SpecialNFTAccount[SPECIAL_NFT_TOKEN];
  // for every mint, we increment the token ID of the NFT account
  SpecialNFTAccount[SPECIAL_NFT_TOKEN] += 1;
```

为了强制 NFT 令牌地址的唯一性，调用 serial_id 是生成地址的哈希中的输入。 创建者帐户将有一个 SPECIAL_NFT_TOKEN 余额，代表 serial_id，即创建者铸造的 NFT 数量。

```jsx
  // serial ID is represented by:
  CreatorAccount[SPECIAL_NFT_TOKEN];
  //for every mint, we increment the serial ID of the creator account
  CreatorAccount[SPECIAL_NFT_TOKEN] += 1;
```

zkSync 服务器将维护 NFT 代币地址到代币 ID 的映射。

### **计算交易费用**

要计算铸造 NFT 的交易费用，您可以使用 Provider 类中的 getTransactionFee 方法。

> 签名
>

```jsx
  async getTransactionFee(
    txType: 'Withdraw' | 'Transfer' | 'FastWithdraw' | 'MintNFT' | ChangePubKeyFee | LegacyChangePubKeyFee,
    address: Address,
    tokenLike: TokenLike
  ): Promise<Fee>
```

例子:

```jsx
  const { totalFee: fee } = await syncProvider.getTransactionFee('MintNFT', syncWallet.address(), feeToken);
```

### **铸造 NFT**

您可以通过从 Wallet 类调用 mintNFT 函数来铸造 NFT。

> 签名
>

```jsx
  async mintNFT(mintNft: {
      recipient: string;
      contentHash: string;
      feeToken: TokenLike;
      fee?: BigNumberish;
      nonce?: Nonce;
  }): Promise<Transaction>
```

|字段         |描述                                       |
|-----------|-----------------------------------------|
|recipient  |以十六进制字符串表示的收件人地址                         |
|contentHash|NFT 的标识符，表示为 32 字节的十六进制字符串（例如 IPFS 内容标识符）|
|feeToken   |要支付费用的代币名称（通常是 ETH）手续费交易费                |
|fee        |交易手续费        

示例:

```jsx
  const contentHash = '0xbd7289936758c562235a3a42ba2c4a56cbb23a263bb8f8d27aead80d74d9d996';
  const nft = await syncWallet.mintNFT({
    recipient: syncWallet.address(),
    contentHash,
    feeToken: 'ETH',
    fee
  });
```

### **获取 Receipt**

要获得铸造 NFT 的收据：

```jsx
  const receipt = await nft.awaitReceipt();
```

### **View the NFT**

铸造 NFT 后，它可以处于两种状态：已提交和已验证。 如果 NFT 已包含在汇总块中，则提交 NFT，并在为该块生成零知识证明并且汇总块的根哈希已包含在以太坊主网上的智能合约中时进行验证。

查看某个账户的 NFTs:

```jsx
// Getstate of account
conststate = await syncWallet.getAccountState();
// View committed NFTs
console.log(state.committed.nfts);
// View verified NFTs
console.log(state.verified.nfts);
```

您可能还会发现 Wallet 类中的 **getNFT** 函数很有用。

> 签名
>

```jsx
async getNFT(tokenId: number, type: 'committed' | 'verified' = 'committed'): Promise<NFT>
```

## **转移**

用户可以将 NFT 转移到现有账户，也可以转移到尚未注册 zkSync 账户的地址。**TRANSFER** 和 **TRANSFER_TO_NEW** 操作码的工作方式相同。

NFT 只能在其铸币交易的区块经过验证后才能转移。 这意味着新铸造的 NFT 可能需要等待几个小时才能转移。 这仅适用于第一次转移； 以下所有转移都可以不受限制地完成。

您可以通过调用 **syncTransferNFT** 函数来转移 NFT：

```jsx
  async syncTransferNFT(transfer: {
      to: Address;
      token: NFT;
      feeToken: TokenLike;
      fee?: BigNumberish;
      nonce?: Nonce;
      validFrom?: number;
      validUntil?: number;
  }): Promise<Transaction[]>
```

|字段         |说明                                       |
|-----------|-----------------------------------------|
|to         |以十六进制字符串表示的收件人地址                         |
|feeToken   |支付费用的代币名称（通常为 ETH）                       |
|token      |NFT 对象                                   |
|fee        |交易手续费                                    |


**syncTransferNFT** 函数在后台作为批处理交易工作，因此它将返回一个交易数组，其中第一个句柄是 NFT 传输，第二个是费用。

```jsx
  const handles = await sender.syncTransferNFT({
    to: receiver.address(),
    feeToken,
    token: nft,
    fee
  });
```

### **获取 Receipt**

要获得转账收据：

```jsx
  const receipt = await handles[0].awaitReceipt();
```

## **转换**

转换函数可用于原子交换：

1. 一个 NFT 换成另外一个 NFT
2. 一个 NFT 可替代代币 (购买 NFT)

### **转换 NFTs**

为了转换 2 个 NFT，各方将签署一份订单，指定他们出售的 NFT 和他们购买的 NFT 的 NFT id。

使用 `[getOrder](https://merlin-li.github.io/api/sdk/js/accounts.html#signing-orders)` 函数:

```jsx
  const order = await wallet.getOrder({
    tokenSell: myNFT.id,
    tokenBuy: anotherNFT.id,
    amount: 1,
    ratio: utils.tokenRatio({
      [myNFT.id]: 1,
      [anotherNFT.id]: 1
    })
  });
```

注意：在执行 NFT 到 NFT 交换时，比率将始终设置为 1。

2个订单签署后，任何人都可以通过调用 `[syncSwap](https://merlin-li.github.io/api/sdk/js/accounts.html#submitting-a-swap)` 函数发起交换：

```jsx
  // whoever initiates the swap pays the fee
  const swap = await submitter.syncSwap({
    orders: [orderA, orderB],
    feeToken: 'ETH'
  });
```

获取 receipt:

```jsx
  const receipt = await swap.awaitReceipt();
```

### **购买 / 出售 NFTs**

要购买或出售可替代代币的 NFT，各方将签署一份订单，指定 NFT id 和他们正在花费/接收的代币名称。 在示例中，请特别注意 ratio 参数。 您可以在我们的[浏览器](https://zkscan.io/explorer/tokens)中找到可用代币及其符号的列表。

```jsx
  const buyingNFT = await walletA.getOrder({
    tokenBuy: nft.id,
    tokenSell: 'USDT',
    amount: tokenSet.parseToken('USDT', '100'),
    ratio: utils.tokenRatio({
      USDT: 100,
      [nft.id]: 1
    })
  });

  const sellingNFT = await walletB.getOrder({
    tokenBuy: 'USDT',
    tokenSell: nft.id,
    amount: 1,
    ratio: utils.tokenRatio({
      USDT: 100,
      [nft.id]: 1
    })
  });
```

## **提现到 Layer 1**

提现到 L1 需要 3 名参与者：

- Factory: 可以铸造 L1 NFT 代币的 L1 合约
- Creator: 在 L2 上铸造 NFT 的用户
- NFTOwner: 在 L2 上拥有 NFT 的用户

本指南将演示 2 种类型的提款：正常和紧急，并解释每种类型应在什么条件下使用。 它还解释了 zkSync 和 L1 之间的 NFT 代币桥的架构，以及如果协议想要在 L1 上实现自己的 NFT 工厂合约需要什么。

### **提取 NFT**

在正常情况下，使用第 2 层操作，**withdrawNFT**，来提取 NFT。

> 签名
>

```jsx
withdrawNFT(withdrawNFT: {
      to: string;
      token: number;
      feeToken: TokenLike;
      fee?: BigNumberish;
      nonce?: Nonce;
      fastProcessing?: boolean;
      validFrom?: number;
      validUntil?: number;
  }): Promise<Transaction>;
```

|字段         |描述                                       |
|-----------|-----------------------------------------|
|to         |以十六进制字符串表示的 L1 收件人地址                     |
|feeToken   |要支付费用的代币名称（通常是 ETH）                      |
|token      |NFT 的 ID                                 |
|fee        |交易手续费                                    |
|fastProcessing|支付额外费用立即完成区块，跳过等待其他交易填满区块                |


```jsx
  const withdraw = await wallet.withdrawNFT({
    to,
    token,
    feeToken,
    fee,
    fastProcessing
  });
```

获取 receipt:

```jsx
  const receipt = await withdraw.awaitReceipt();
```

### **紧急提取**

在审查的情况下，用户可以要求紧急退出。 注意：这是1层网络的操作，类似于我们的 [fullExit 机制](https://merlin-li.github.io/dev/payments/basic.html#withdrawing-funds)。

> 签名
>

```jsx
async emergencyWithdraw(withdraw: {
          token: TokenLike;
          accountId?: number;
          ethTxOptions?: ethers.providers.TransactionRequest;
      }): Promise<ETHOperation>
```

|字段         |描述                                       |
|-----------|-----------------------------------------|
|token      |NFT id                                   |
|accountId (Optional)|account id for fullExit                  |


```jsx
  const emergencyWithdrawal = await wallet.emergencyWithdraw({ token, accountId });
  const receipt = await emergencyWithdrawal.awaitReceipt();
```

### **Factory 与 zkSync 智能合约交互**

我们有一个默认的工厂合约，它将为不想实施自己的铸币合约的项目在 L1 上处理铸币 NFT。 拥有自己铸币合约的项目只需要实现一个铸币功能：**mintNFTFromZkSync**。

```jsx
  mintNFTFromZkSync(creator_address: address, creator_id: uint32, serial_id: uint32, content_hash: bytes, recipient_address: address, token_id: uint32)
```

zkSync Governance 合约将实现一个函数 **registerFactory**，它将创建者注册为工厂合约的 L2 上的受信任铸币者。

```jsx
  registerFactory(creator_address: address, signature: bytes)
```

要取款，用户使用 token_id 调用 **withdrawNFT()**。 zkSync 智能合约会验证所有权，在 L2 上销毁代币，并在创建者对应的工厂上调用 **mintNFTFromZkSync**。

### **Factory 注册**

1. 要注册工厂，创建者将使用数据 **factory_address** 和 **creator_address** 签署以下消息。

```jsx
  "Ethereum Signed Message:\n141",
  "\nCreator's account ID in zkSync: {creatorIdInHex}",
  "\nCreator: {CreatorAddressInHex}",
  "\nFactory: {FactoryAddressInHex}"
```

1. 工厂合约在带有签名的 zkSync L1 智能合约上调用 **registerFactory**。
2. zkSync 智能合约验证签名并发出带有 **factory_address** 和 **creator_address** 的事件。
