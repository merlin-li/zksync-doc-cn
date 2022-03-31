# GetStarted

# 入门介绍

在本教程中，我们将演示如何：

1. 连接到 zkSync 网络。
2. 将以太坊中的资产存入 zkSync。
3. 转账。
4. 将资金提取回以太坊主网。

## **添加依赖**

```bash
  yarn add zksync
  yarn add ethers # ethers is a peer dependency of zksync
```

See [Appendix A](https://merlin-li.github.io/zksync/api/js/appendix) for how to add library to web project directly from [https://unpkg.com](https://unpkg.com/)CDN.

## **Adding imports**

您可以使用以下语句导入 zkSync 库的所有内容：

```jsx
import * as zksync from 'zksync';
```

请注意，实际上并不需要导入所有库。 例如，如果你只需要 Wallet 类，你可以

```jsx
import { Wallet } from 'zksync';
```

但在本书的其余部分，我们将假设使用第一种导入库的方式来区分从 zksync 和 ethers 库导入的内容。

## **连接 zkSync 网络**

要与 zkSync 网络交互，用户需要知道 operator 节点的地址。

```jsx
const syncProvider = await zksync.getDefaultProvider('rinkeby');
```

大多数操作都需要对以太坊网络进行一些只读访问。 我们使用 `ethers` 库与以太坊交互。

```jsx
const ethersProvider = ethers.getDefaultProvider('rinkeby');
```

## **创建一个钱包**

要在 zkSync 中控制您的帐户，请使用 zksync.Wallet 对象。 它可以使用存储在 zksync.Signer 中的密钥对交易进行签名，并使用 zksync.Provider 将交易发送到 zkSync 网络。

zksync.Wallet 是 2 个对象的封装：

- `ethers.Signer` 签名以太坊交易。
- `zksync.Signer` 签名原生 zkSync 交易

zksync.Signer 使用的私钥隐含地从特殊消息的以太坊签名中派生而来。

```jsx
// Create ethereum wallet using ethers.js
const ethWallet = ethers.Wallet.fromMnemonic(MNEMONIC).connect(ethersProvider);

// Derive zksync.Signer from ethereum wallet.
const syncWallet = await zksync.Wallet.fromEthSigner(ethWallet, syncProvider);
```

## **将以太坊中的资产存入 zkSync**

我们将 `1.0 ETH` 存入到我们的 zkSync 账户.

```jsx
const deposit = await syncWallet.depositToSyncFromEthereum({
      depositTo: syncWallet.address(),
      token: 'ETH',
      amount: ethers.utils.parseEther('1.0')
});
```

“ETH”代表原生以太币。 要转移受支持的 ERC20 代币，请使用 ERC20 地址或 ERC20 符号而不是“ETH”。

将 tx 提交到以太坊节点后，我们可以使用返回的对象跟踪其状态：

```jsx

// Await confirmation from the zkSync operator
// Completes when a promise is issued to process the tx
const depositReceipt = await deposit.awaitReceipt();

// Await verification
// Completes when the tx reaches finality on Ethereum
const depositReceipt = await deposit.awaitVerifyReceipt();
```

## **解锁 zkSync 账户**

要控制 zkSync 网络中的资产，账户必须注册一次单独的公钥。

```jsx
if (!(await syncWallet.isSigningKeySet())) {
	if ((await syncWallet.getAccountId()) == undefined) {
		thrownew Error('Unknown account');
  }

  // As any other kind of transaction, `ChangePubKey` transaction requires fee.
  // User doesn't have (but can) to specify the fee amount. If omitted, library will query zkSync node for
  // the lowest possible amount.
	const changePubkey = await syncWallet.setSigningKey({
        feeToken: 'ETH',
        ethAuthType: 'ECDSA'
  });

  // Wait until the tx is committed
	await changePubkey.awaitReceipt();
}
```

## **检查 zkSync 账户余额**

```jsx
// Committed state is not final yet
const committedETHBalance = await syncWallet.getBalance('ETH');

// Verified state is final
const verifiedETHBalance = await syncWallet.getBalance('ETH', 'verified');
```

一次列出该账户下所有 token，使用 `getAccountState`:

```jsx
const state = await syncWallet.getAccountState();

const committedBalances = state.committed.balances;
const committedETHBalance = committedBalances['ETH'];

const verifiedBalances = state.verified.balances;
const committedETHBalance = verifiedBalances['ETH'];
```

## **用 zkSync 转账**

现在，让我们创建第二个钱包并将一些资金转入其中。 请注意，我们可以将资产发送到任何新的以太坊账户，无需预先注册！

```jsx
const ethWallet2 = ethers.Wallet.fromMnemonic(MNEMONIC2).connect(ethersProvider);
const syncWallet2 = await zksync.SyncWallet.fromEthSigner(ethWallet2, syncProvider);
```

我们将 0.999 ETH 转移到另一个账户并向运营商支付 0.001 ETH 作为费用（发送方的 zkSync 账户余额将减少 0.999 + 0.001 ETH）。 使用最接近的PackableTransactionAmount() 和最接近的PackableTransactionFee() 是必要的，因为在zkSync 中传输的精度是有限的（参见下面的文档）。

```jsx
const amount = zksync.utils.closestPackableTransactionAmount(ethers.utils.parseEther('0.999'));
const fee = zksync.utils.closestPackableTransactionFee(ethers.utils.parseEther('0.001'));

const transfer = await syncWallet.syncTransfer({
      to: syncWallet2.address(),
      token: 'ETH',
      amount,
      fee
});
```

请注意，不需要手动设置费用。 如果省略 `fee` 字段，SDK 将选择服务器可接受的最低费用：

```jsx
const amount = zksync.utils.closestPackableTransactionAmount(ethers.utils.parseEther('0.999'));

const transfer = await syncWallet.syncTransfer({
      to: syncWallet2.address(),
      token: 'ETH',
      amount
});
```

跟踪 transaction 状态:

```jsx
const transferReceipt = await transfer.awaitReceipt();
```

## **将资金提取回以太坊主网**

```jsx
const withdraw = await syncWallet2.withdrawFromSyncToEthereum({
      ethAddress: ethWallet2.address,
      token: 'ETH',
      amount: ethers.utils.parseEther('0.998')
});
```

此操作生成zkSync区块的零知识证明并经主网合约验证后，资产将被提取到目标钱包。

我们可以等到 ZKP 验证完成：

```jsx
await withdraw.awaitVerifyReceipt();
```