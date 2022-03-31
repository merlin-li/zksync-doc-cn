# Utils

## 代币相关的知识

代币标识

1. 标识 (e.g. "ETH", "DAI")
2. 地址 ("0x0000000000000000000000000000000000000000" for "ETH" or "0xFab46E002BbF0b4509813474841E0716E6730136" for ERC20).

要使用令牌相关的实用功能，必须使用 TokenSet 类，TokenSet 可以从 [zkSync provider](https://merlin-li.github.io/zksync/api/js/providers) 获取。

### **Resolve token ID**

使用其标识符获取数字代币 ID。

> Signature
> 

```jsx
public resolveTokenId(tokenLike: TokenLike): number;
```

### **Resolve token Address**

根据标识获取代币地址。

> Signature
> 

```jsx
public resolveTokenAddress(tokenLike: TokenLike): TokenAddress;
```

### **Resolve token Symbol**

根据标识符获取代币符号。

> Signature
> 

```jsx
public resolveTokenSymbol(tokenLike: TokenLike): TokenSymbol;
```

### **Resolve token decimals**

获取代币精度 (e.g. Ether has 18 decimals, meaning `1.0` ETH is `1e18` wei).

> Signature
> 

```jsx
public resolveTokenDecimals(tokenLike: TokenLike): number;
```

### **Format amount for token**

将 BigNumberish 量格式化为相对于给定代币的小数值的可读字符串。

> Signature
> 

```jsx
public formatToken(tokenLike: TokenLike, amount: BigNumberish): string;
```

> Example
> 

```jsx
provider.tokenSet.formatToken('ETH', '1000000000'); // "0.000000001"
provider.tokenSet.formatToken('USDC', '1000000000'); // "1000.0"
```

### **Parse amount for token**

将可读的字符串解析为与给定代币精度的 BigNumber。

> Signature
> 

```jsx
public parseToken(tokenLike: TokenLike, amount: string): BigNumber;
```

> Example
> 

```jsx
provider.tokenSet.parseToken('ETH', '0.000000001'); // '1000000000'
provider.tokenSet.parseToken('USDC', '1000.0'); // '1000000000'
```

## **Amount packing**

### **Check if amount is packable**

转账金额应可打包为 5 字节长的浮点表示。 该功能用于检查该金额是否可以作为转账金额。

> Signature
> 

```jsx
export function isTransactionAmountPackable(amount: BigNumberish):boolean;
```

### **Closest packable amount**

转账金额应可打包为 5 字节长的浮点表示。 此函数通过将最低有效数字设置为零来返回最接近的可打包数量。

> Signature
> 

```jsx
export function closestPackableTransactionAmount(amount: ethers.BigNumberish):ethers.utils.BigNumber;
```

### **Check if fee is packable**

在转账和取款中支付的所有费用都应该可以打包为 2 字节长的浮点表示。 此功能用于检查该金额是否可以用作费用。

> Signature
> 

```jsx
export function isTransactionFeePackable(amount: BigNumberish):boolean;
```

### **Closest packable fee**

在转账和取款中支付的所有费用都应该可以打包为 2 字节长的浮点表示。 此函数通过将最低有效数字设置为零来返回最接近的可打包数量。

> Signature
> 

```jsx
export function closestPackableTransactionFee(fee: ethers.BigNumberish):ethers.utils.BigNumber;
```

### **Check if formatted amount is packable for token**

转账中支付的所有金额都应可打包为 5 字节长的浮点表示。 isTokenTransferAmountPackable 函数允许检查格式化金额是否可以用作转账金额。

> Signature
> 

```jsx
public isTokenTransferAmountPackable(
    tokenLike: TokenLike,
    amount: string
): boolean;
```

### **Check if formatted fee is packable for token**

在转账和取款中支付的所有费用都应该可以打包为 2 字节长的浮点表示。 isTokenTransactionFeePackable 函数用于检查格式化金额是否可以用作费用。

> Signature
> 

```jsx
public isTokenTransactionFeePackable(
    tokenLike: TokenLike,
    amount: string
): boolean;
```

## **等待操作完成**

能够等到交易提交或验证是很有用的。 可以使用从提交交易的方法返回的对象来做到这一点。

It is possible to wait until the transactions like Transfer is either:

1. Committed (with `awaitReceipt`) when the state is updated in the zkSync network
2. Verified (with `awaitVerifyReceipt`) when the state is finalized on the Ethereum

It is possible to wait until the operations like Deposit is either:

1. Mined on the Ethereum network (with `awaitEthereumTxCommit`)
2. Committed (with `awaitReceipt`) when the state is updated in the zkSync network
3. Verified (with `awaitVerifyReceipt`) when the state is finalized on the Ethereum

首先提交，但如果您对验证感兴趣，则无需等待提交，因为等待验证意味着等待提交。

> Awaiting for transaction.
> 

```jsx
import * as zksyncfrom "zksync";
const wallet = ..; // create zkSync Wallet

// see transfer example for details
const transfer = await wallet.syncTransfer({..});

// this function will return when deposit is committed to the zkSync chain
const receiptAfterCommit = await transfer.awaitReceipt();

// this function will return when deposit is verified with ZK proof.
const receiptAfterVerify = await transfer.awaitVerifyReceipt();
```

> Awaiting for priority operation
> 

```jsx
import * as zksyncfrom "zksync";

// see deposit example for details
const deposit = await zksync.depositFromETH({..});

// this function will return when deposit request is accepted to the Ethereum.
const txMinedCommit = await deposit.awaitEthereumTxCommit();

// this function will return when deposit is committed to the zkSync chain
const receiptAfterCommit = await deposit.awaitReceipt();

// this function will return when deposit is verified with ZK proof.
const receiptAfterVerify = await deposit.awaitVerifyReceipt();
```

> Sending and awaiting for the previously signed transaction.
> 

```jsx
import * as zksyncfrom "zksync";
const provider = ..; // create zkSync Provider
const wallet = ..; // create zkSync Wallet

// sign transfer transaction
const signedTransfer = await wallet.signSyncTransfer({..});

// submit transaction to zkSync
const transfer = await zksync.wallet.submitSignedTransaction(signedTransfer, provider);

// this function will return when deposit is committed to the zkSync chain
const receiptAfterCommit = await transfer.awaitReceipt();

// this function will return when deposit is verified with ZK proof.
const receiptAfterVerify = await transfer.awaitVerifyReceipt();
```