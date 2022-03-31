# NFTs

此 API 参考提供了 zkSync 1.x 中有关 NFT 的所有功能的描述。 建议您从我们的 NFT 教程开始，然后回到这里参考具体功能。

- [连接到 Rinkeby 测试网络](https://merlin-li.github.io/zksync/api/js/nfts#connect-to-the-rinkeby-testnet)
- [Mint NFT](https://merlin-li.github.io/zksync/api/js/nfts#mint-nft)
- [Transfer NFT](https://merlin-li.github.io/zksync/api/js/nfts#transfer-nft)
- [Swap NFT](https://merlin-li.github.io/zksync/api/js/nfts#swap-nft)
- [Withdraw NFT](https://merlin-li.github.io/zksync/api/js/nfts#withdraw-nft)
    - [Emergency Withdraw](https://merlin-li.github.io/zksync/api/js/nfts#emergency-withdraw)
- [公用函数](https://merlin-li.github.io/zksync/api/js/nfts#utility-functions)
    - [Calculate Transaction Fee](https://merlin-li.github.io/zksync/api/js/nfts#calculate-transaction-fee)
    - [View NFT](https://merlin-li.github.io/zksync/api/js/nfts#view-an-nft)
    - [Get NFT](https://merlin-li.github.io/zksync/api/js/nfts#get-an-nft)
    - [Get a receipt](https://merlin-li.github.io/zksync/api/js/nfts#get-a-receipt)

## **连接到 Rinkeby 测试网络**

主网和 ropsten 网络都支持 NFT。 出于本教程的目的，我们将使用 rinkeby 测试网。

```jsx
const syncProvider =await zksync.getDefaultProvider('rinkeby');
```

## **Mint NFT**

你可以通过调用**Wallet**类的**mintNFT**函数来铸造一个NFT。

> Signature
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

[mint](NFTs%200fc6d/mint%2007e2f.csv)

Example:

```jsx
const contentHash = '0xbd7289936758c562235a3a42ba2c4a56cbb23a263bb8f8d27aead80d74d9d996';
const nft =await syncWallet.mintNFT({
      recipient: syncWallet.address(),
      contentHash,
      feeToken: 'ETH',
      fee
});
```

铸造 NFT 后，它可以处于两种状态：已提交和已验证。 如果 NFT 已包含在汇总块中，则提交 NFT，并在为该块生成零知识证明并且汇总块的根哈希已包含在以太坊主网上的智能合约中时进行验证。

## **Transfer NFT**

NFT 只能在其铸币交易的区块经过验证后才能转移。 这意味着新铸造的 NFT 可能需要等待几个小时才能转移。 这仅适用于第一次转移； 以下所有转移都可以不受限制地完成。

你可以通过 `syncTransferNFT` 函数转移 NFT:

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

[transfer](NFTs%200fc6d/transfer%204a985.csv)

syncTransferNFT 函数在后台作为批处理交易工作，因此它将返回一个交易数组，其中第一个句柄是 NFT 传输，第二个是交易费用。

```jsx
const handles =await sender.syncTransferNFT({
      to: receiver.address(),
      feeToken,
      token: nft,
      fee
});
```

## **Swap NFT**

转换 NFT 与可替代代币的功能相同。 有关详细信息，请参阅 [API](https://merlin-li.github.io/zksync/api/js/accounts) 。

## **Withdraw NFT**

在正常情况下，使用第 2 层操作，withdrawNFT，来提取 NFT。

> Signature
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

[withdraw](NFTs%200fc6d/withdraw%201c61c.csv)

```jsx
const withdraw = await wallet.withdrawNFT({
      to,
      token,
      feeToken,
      fee,
      fastProcessing
  });
```

### **Emergency Withdraw**

在审查的情况下，用户可以要求紧急退出。 注意：这是第 1 层操作，类似于我们的 fullExit 机制。

> Signature
> 

```jsx
async emergencyWithdraw(withdraw: {
      token: TokenLike;
      accountId?: number;
      ethTxOptions?: ethers.providers.TransactionRequest;
    }): Promise<ETHOperation>
```

[emergency](NFTs%200fc6d/emergency%20f0e8e.csv)

## **Utility Functions**

### 计算交易费用

为了计算铸造 NFT 的交易费用，Provider 类的 getTransactionFee 方法现在支持交易类型“MintNFT”。

> Signature
> 

```jsx
async getTransactionFee(
        txType: 'Withdraw' | 'Transfer' | 'FastWithdraw' | 'MintNFT' | ChangePubKeyFee | LegacyChangePubKeyFee,
        address: Address,
        tokenLike: TokenLike
    ): Promise<Fee>
```

示例:

```jsx
const { totalFee: fee } = await syncProvider.getTransactionFee('MintNFT', syncWallet.address(), feeToken);
```

### 查看 **NFT**

查看一个账户的 NFT:

```jsx
// Get state of account
const state = await syncWallet.getAccountState('<account-address>');
// View committed NFTs
console.log(state.committed.nfts);
// View verified NFTs
console.log(state.verified.nfts);
```

### **Get an NFT**

您可能还会发现 Wallet 类中的 getNFT 函数很有用。

> Signature
> 

```jsx
async getNFT(tokenId: number, type: 'committed' | 'verified' = 'committed'): Promise<NFT>
```

### 获取凭证

获得不同操作的收据略有不同。

要获得铸造 NFT 的收据：

```jsx
// mint nft
const nft =await wallet.mintNFT({...});
// get receipt
const receipt =await nft.awaitReceipt();
```

To get a receipt for a transfer:

```jsx
// transfer nft
const handles = await sender.syncTransferNFT({...});
// get receipt
const receipt = await handles[0].awaitReceipt();
```

To get a receipt for a swap:

```jsx
// swap nft
const swap = await submitter.syncSwap({...});
// get receipt
const receipt = await swap.awaitReceipt();
```

To get a receipt for withdrawal:

```jsx
// normal withdraw
const withdrawal = await wallet.withdrawNFT({...});
const receipt = await withdrawal.awaitReceipt();
// emergency withdraw
const emergencyWithdraw = await wallet.emergencyWithdraw({...});
const receipt = await emergencyWithdraw.awaitReceipt();
```