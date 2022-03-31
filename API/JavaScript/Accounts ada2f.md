# Accounts

如果未设置，将从 zkSync 服务器请求尽可能低的费用。 费用与主要交易令牌相同。 要了解如何手动获取可接受的费用金额，请参阅 [从服务器获取交易费用](https://merlin-li.github.io/zksync/api/js/providers#get-transaction-fee-from-the-server)。 要查看金额是否可打包，请使用 [pack fee util](https://merlin-li.github.io/api/sdk/js/utils.html#closest-packable-fee)。

当设置为false,**ethers.Signer** 用于创建签名，否则期望用户调用 **onchainAuthSigningKey** 来授权新的 pubkey。

## **Wallet**

**Wallet** 对象用于与 zkSync 网络进行交互。钱包有一个与之关联的以太坊地址，拥有这个以太坊账户的用户拥有一个相应的 zkSync 账户。以太坊帐户的所有权是指发送以太坊交易的能力和可选的签署消息的能力。

钱包有与之关联的随机数，它用于防止交易重放。只有与钱包当前 nonce 相等的 nonce 交易才能被执行。

要在 zkSync 网络钱包中创建交易，必须有与之关联的 zkSync 密钥对。 zkSync 密钥由 Signer 对象处理，可以使用不同的方法创建，最方便的方法是通过从特定消息的以太坊签名派生这些密钥来创建这些密钥，如果用户不提供使用某些方法创建的 Signer，则默认使用此方法其他方法。

要使 zkSync 密钥成为有效用户，应使用设置签名密钥事务在 zkSync 网络中注册一次。对于不支持消息签名的以太坊钱包，需要额外的以太坊交易。 zkSync 密钥可以随时更改。

转账和取款等交易额外使用钱包的以太坊账户进行签名，这个签名用于额外的安全性，以防钱包的 zkSync 密钥被泄露。要求用户签署交易的可读表示，并在交易提交到 zkSync 时执行签名检查。

### **Creating wallet from ETH signer**

> Signature
> 

```jsx
static async fromEthSigner(
      ethWallet: ethers.Signer,
      provider: Provider,
      signer?: Signer,
      accountId?: number,
      ethSignatureType?: EthSignerType
    ): Promise<Wallet>;
```

### **Inputs and outputs**

[table1](Accounts%20ada2f/table1%20223d9.csv)

> Example
> 

```jsx
import * as zksyncfrom "zksync";
import { ethers }from "ethers";

const ethersProvider = ethers.getDefaultProvider("rinkeby");
const syncProvider =await zksync.getDefaultProvider("rinkeby");

const ethWallet = ethers.Wallet.createRandom().connect(ethersProvider);
const syncWallet =await zksync.Wallet.fromEthSigner(ethWallet, syncProvider);
```

### **Creating wallet from ETH without zkSync signer**

> Signature
> 

```jsx
static async fromEthSignerNoKeys(
      ethWallet: ethers.Signer,
      provider: Provider,
      accountId?: number,
      ethSignatureType?: EthSignerType
    ): Promise<Wallet>;
```

这种方式钱包不会包含任何有效的 zkSync 密钥来执行交易，但一些操作可以在没有它们的情况下使用，例如 Deposit、Emergency exit 和读取 account state。

### **Inputs and outputs**

[table2](Accounts%20ada2f/table2%203cb24.csv)

> Example
> 

```jsx
import * as zksyncfrom "zksync";
import { ethers }from "ethers";

const ethersProvider = ethers.getDefaultProvider("rinkeby");
const syncProvider =await zksync.getDefaultProvider("rinkeby");

const ethWallet = ethers.Wallet.createRandom().connect(ethersProvider);
const syncWallet =await zksync.Wallet.fromEthSignerNoKeys(ethWallet, syncProvider);
```

### **Creating wallet from CREATE2 data**

这样您就可以创建一个钱包，可以使用 CREATE2 操作码创建相应的 L1 帐户。 syncSigner pubKeyHash 被编码为 CREATE2 的 salt 的一部分。 请注意，您不需要手动将其添加到 create2Data 的 saltArg 中，因为它是自动完成的。

此类钱包不需要为交易提供以太坊签名。 与必须验证 ChangePubKey onchain 签名的 ECDSA 钱包不同，CREATE2 钱包只需要检查 pubKeyHash 是否包含在 CREATE2 摘要中。 因此，这种账户的 ChangePubKey 成本低于 ECDSA 账户，但这有一个限制：L2 私钥不能更改。 此外，这种类型的帐户不能用于从现有 L1 地址加入用户。

> Signature
> 

```jsx
static async fromCreate2Data(
      syncSigner: Signer,
      provider: Provider,
      create2Data: Create2Data,
      accountId?: number
    ): Promise<Wallet>;
```

### **Inputs and outputs**

[table3](Accounts%20ada2f/table3%20a344f.csv)

> Example
> 

```jsx
import * as zksyncfrom "zksync";
import { ethers }from "ethers";

const syncProvider =await zksync.getDefaultProvider("rinkeby");
const signer =await zksync.Signer.fromSeed(ethers.utils.randomBytes(32));
const randomHex = (length: number) => {
const bytes = ethers.utils.randomBytes(length);
return ethers.utils.hexlify(bytes);
    };
const create2Data = {
      creatorAddress: randomHex(20),
      saltArg: randomHex(32),
      codeHash: randomHex(32),
    };
const syncWallet =await zksync.Wallet.fromCreate2Data(signer, syncProvider, create2Data);
```

### **Get account state**

Same as [Get account state from provider](https://merlin-li.github.io/api/sdk/js/providers.html#get-account-state-by-address) but gets state of this account.

> Signature
> 

```jsx
async getAccountState(): Promise<AccountState>;
```

### **Inputs and outputs**

[table4](Accounts%20ada2f/table4%20e8088.csv)

### **Get account id**

为方便起见，可以获取帐户 ID。

> Signature
> 

```jsx
async getAccountState(): Promise<AccountState>;
```

### **Inputs and outputs**

[table5](Accounts%20ada2f/table5%203e537.csv)

### **Get account nonce**

获取此帐户的 nonce。 如果您想显式提供随机数或使用最后提交的随机数作为后备，则非常方便。

> Signature
> 

```jsx
async getAccountId(): Promise<number |undefined>;
```

### **Inputs and outputs**

[table6](Accounts%20ada2f/table6%20285bc.csv)

### **Get token balance on zkSync**

> Signature
> 

```jsx
async getBalance(
      token: TokenLike,
      type: "committed" | "verified" = "committed"
    ): Promise<ethers.utils.BigNumber>;
```

### **Inputs and outputs**

[table7](Accounts%20ada2f/table7%2059d1f.csv)

> Example
> 

```jsx
const wallet = ..; // setup wallet

// Get committed Ethereum balance.
const ethCommittedBalance =await getBalance("ETH");

// Get verified ERC20 token balance.
const erc20VerifiedBalance =await getBalance("0xFab46E002BbF0b4509813474841E0716E6730136", "verified");
```

### **Get token balance on Ethereum**

Method similar to `[syncWallet.getBalance](https://merlin-li.github.io/zksync/api/js/accounts#get-token-balance-on-zksync)`, used to query balance in the Ethereum network.

> Signature
> 

```jsx
async getEthereumBalance(token: TokenLike): Promise<utils.BigNumber>;
```

### **Inputs and outputs**

[table8](Accounts%20ada2f/table8%201d423.csv)

> Example
> 

```jsx
import *as zksyncfrom "zksync";
import { ethers }from "ethers";

// Setup zksync.Wallet with ethers signer/wallet that is connected to ethers provider
const wallet = ..;

const ethOnChainBalance =await wallet.getEthereumBalance("ETH");
```

### **Unlocking ERC20 deposits**

为方便起见，可以为 zkSync 合约批准最大可能的 ERC20 代币存款，这样用户就不需要批准每笔存款。

> Signature
> 

```jsx
async approveERC20TokenDeposits(
      token: TokenLike,
      max_erc20_approve_amount: BigNumber = MAX_ERC20_APPROVE_AMOUNT
    ): Promise<ethers.ContractTransaction>;
```

[table9](Accounts%20ada2f/table9%206ec23.csv)

> Signature
> 

```jsx
async isERC20DepositsApproved(
      token: TokenLike,
      erc20ApproveThreshold: BigNumber = ERC20_APPROVE_TRESHOLD
    ): Promise<boolean>;
```

[table10](Accounts%20ada2f/table10%201f72f.csv)

### **Deposit token to Sync**

将资金从以太坊账户转移到 zkSync 账户。

要进行 ERC20 代币转移，此代币转移应获得批准。 用户可以使用解锁 ERC20 令牌使 ERC20 存款永久批准，或者用户可以在每次存款时批准确切的金额（存款所需），但不建议这样做。

一旦操作提交到以太坊网络，我们必须等待一定数量的确认（具体数字请参见 [provider docs](https://merlin-li.github.io/zksync/api/js/providers)）,然后才能在 zkSync 网络中接受它。 交易提交到 zkSync 网络后，资金已经可供接收方使用，这意味着无需等待验证即可继续进行，除非您的应用程序需要额外确认。 要等待 zkSync 网络上的事务提交，请使用 awaitReceipt（参见 [utils](https://merlin-li.github.io/zksync/api/js/utils)）。

> Signature
> 

```jsx
async depositToSyncFromEthereum(deposit: {
        depositTo: Address;
        token: TokenLike;
        amount: BigNumberish;
        ethTxOptions?: ethers.providers.TransactionRequest;
        approveDepositAmountForERC20?: boolean;
      }
    ): Promise<ETHOperation>;
```

### **Inputs and outputs**

[table11](Accounts%20ada2f/table11%20b37f7.csv)

- : If set, the user will be asked to approve the ERC20 token spending to our account (not required if[token was unlocked](https://merlin-li.github.io/zksync/api/js/accounts#unlocking-erc20-deposits))

> Example
> 

```jsx
import * as zksync from "zksync";
import { ethers }from "ethers";

const syncWallet = ..; // Setup zksync wallet from ethers.Signer.

const depositPriorityOperation = await syncWallet.depositToSyncFromEthereum({
      depositTo: "0x2d5bf7a3ab29f0ff424d738a83f9b0588bc9241e",
      token: "ETH",
      amount: ethers.utils.parseEther("1.0"),
    });

    // Wait till priority operation is committed.
const priorityOpReceipt = await depositPriorityOperation.awaitReceipt();
```

### **Changing account public key**

In order to send zkSync transactions (transfer and withdraw) user has to associate zksync key pair with account. Every zkSync account has address which is ethereum address of the owner.

There are two ways to authorize zksync key pair.

1. Using ethereum signature of specific message. This way is preferred but can only be used if your ethereum wallet can sign messages.
2. Using ethereum transaction to zkSync smart-contract.

The account should be present in the zkSync network in order to set a signing key for it.

This function will throw an error if the account is not present in the zkSync network. Check the[account id](https://merlin-li.github.io/zksync/api/js/accounts#get-account-state) to see if account is present in the zkSync state tree.

The operators require a fee to be paid in order to process transactions.[[5]](https://merlin-li.github.io/zksync/api/js/accounts#fn5)

> Signature
> 

```jsx
async setSigningKey(changePubKey: {
        feeToken: TokenLike;
        ethAuthType: ChangePubkeyTypes;
        fee?: BigNumberish;
        nonce?: Nonce;
        validFrom?: number;
        validUntil?: number;
      }
    ): Promise<Transaction>;
```

### **Inputs and outputs**

[table12](Accounts%20ada2f/table12%20ac40e.csv)

> Example
> 

```jsx
import { ethers }from "ethers";

const wallet = ..;// setup zksync wallet

if (!await wallet.isSigningKeySet()) {
const changePubkey =await wallet.setSigningKey({
        feeToken: "FAU",
        fee: ethers.utils.parseEther("0.001"),
        ethAuthType: "ECDSA"
      });

      // Wait till transaction is committed
const receipt =await changePubkey.awaitReceipt();
    }
```

### **Sign change account public key transaction**

Signs [change public key](https://merlin-li.github.io/zksync/api/js/accounts#changing-account-public-key) transaction without sending it to the zkSync network. It is important to consider transaction fee in advance because transaction can become invalid if token price changes.

> Signature
> 

```jsx
async signSetSigningKey(changePubKey: {
      feeToken: TokenLike;
      fee: BigNumberish;
      nonce: number;
      ethAuthType: ChangePubkeyTypes;
      validFrom?: number;
      validUntil?: number;
    }): Promise<SignedTransaction>;
```

### **Inputs and outputs**

| Name | Description | | ---------------------------------- | --------------------------------------------------------------------------- | --- | | changePubKey.feeToken | Token to pay fee in.[[6:1]](https://merlin-li.github.io/zksync/api/js/accounts#fn6) | | changePubKey.fee | Amount of token to be paid as a fee for this transaction.[[5:2]](https://merlin-li.github.io/zksync/api/js/accounts#fn5) | | changePubKey.nonce | Nonce that is going to be used for this transaction.[[7:1]](https://merlin-li.github.io/zksync/api/js/accounts#fn7) | | changePubKey.ethAuthType | The type which determines how will the Ethereum signature be verified. | | | changePubKey.validFrom (optional) | Unix timestamp from which the block with this transaction can be processed | | changePubKey.validUntil (optional) | Unix timestamp until which the block with this transaction can be processed | | returns | Signed transaction |

### **Authorize new public key using ethereum transaction**

This method is used to authorize [public key change](https://merlin-li.github.io/zksync/api/js/accounts#changing-account-public-key) using ethereum transaction for wallets that don't support message signing.

> Signature
> 

```jsx
async onchainAuthSigningKey(
      nonce: Nonce = "committed",
      ethTxOptions?: ethers.providers.TransactionRequest
    ): Promise<ContractTransaction>;
```

### **Inputs and outputs**

[table13](Accounts%20ada2f/table13%20408bb.csv)

> Example
> 

```jsx
import { ethers }from "ethers";

const wallet = ..;// setup zksync wallet

if (!await wallet.isSigningKeySet()) {
const onchainAuthTransaction =await wallet.onchainAuthSigningKey();
      // Wait till transaction is committed on ethereum.
await onchainAuthTransaction.wait();

const changePubkey =await wallet.setSigningKey({
        feeToken: "ETH",
        ethAuthType: "ECDSA"
      });

      // Wait till transaction is committed
const receipt =await changePubkey.awaitReceipt();
    }
```

### **Check if current public key is set**

Checks if current signer that is associated with wallet is able to sign transactions. See[change pub key](https://merlin-li.github.io/zksync/api/js/accounts#changing-account-public-key) on how to change current public key.

> Signature
> 

```jsx
async isSigningKeySet(): Promise<boolean>;
```

### **Inputs and outputs**

[table14](Accounts%20ada2f/table14%2040a0e.csv)

### **Transfer in the zkSync**

Moves funds between accounts inside the zkSync network. Sender account should have correct public key set before sending this transaction. ( see [change pub key](https://merlin-li.github.io/zksync/api/js/accounts#changing-account-public-key))

Before sending this transaction, the user will be asked to sign a specific message with transaction details using their Ethereum account (because of the security reasons).

After the transaction is committed, funds are already usable by the recipient, so there is no need to wait for verification before proceeding unless additional confirmation is required for your application. To wait for transaction commit use `awaitReceipt`(see [utils](https://merlin-li.github.io/api/sdk/js/utils.html#awaiting-for-operation-completion)).

The operators require a fee to be paid in order to process transactions.[[5:3]](https://merlin-li.github.io/zksync/api/js/accounts#fn5)

The transfer amount and fee should have a limited number of significant digits according to spec. See utils for helping with amounts packing.

> Signature
> 

```jsx
async syncTransfer(transfer:{
      to: Address;
      token: TokenLike;
      amount: BigNumberish;
      fee?: BigNumberish;
      nonce?: Nonce;
      validFrom?: number;
      validUntil?: number;
    }): Promise<Transaction>;
```

### **Inputs and outputs**

[table15](Accounts%20ada2f/table15%20c2fcf.csv)

> Example
> 

```jsx
import { ethers }from "ethers";

const wallet = ..;// setup zksync wallet

const transferTransaction =await wallet.syncTransfer({
      to: "0x2d5bf7a3ab29f0ff424d738a83f9b0588bc9241e",
      token: "0xFab46E002BbF0b4509813474841E0716E6730136", // FAU ERC20 token address
      amount: ethers.utils.parseEther("1.0"),
      fee: ethers.utils.parseEther("0.001")
    });

    // Wait till transaction is committed
const transactionReceipt =await transferTransaction.awaitReceipt();
```

### **Sign transfer in the zkSync transaction**

Signs [transfer](https://merlin-li.github.io/zksync/api/js/accounts#transfer-in-the-zksync) transaction without sending it to the zkSync network. It is important to consider transaction fee in advance because transaction can become invalid if token price changes.

> Signature
> 

```jsx
async signSyncTransfer(transfer: {
      to: Address;
      token: TokenLike;
      amount: BigNumberish;
      fee: BigNumberish;
      nonce: number;
      validFrom?: number;
      validUntil?: number;
    }): Promise<SignedTransaction>;
```

### **Inputs and outputs**

[table16](Accounts%20ada2f/table16%205ebe5.csv)

### **Swaps in zkSync**

Performs an atomic swap between 2 existing accounts in the zkSync network. For information about swaps, see the[Swaps tutorial](https://merlin-li.github.io/dev/swaps.html).

### **Signing orders**

There are two kinds of orders:

- Swap order, where an explicit amount is set.
- Limit order, where an explicit amount is not set, and instead inferred from the balance. This order can be partially filled.

> Signature
> 

```jsx
async getOrder(order:{
      tokenSell: TokenLike;
      tokenBuy: TokenLike;
      ratio: TokenRatio | WeiRatio;
      amount: BigNumberish;
      recipient?: Address;
      nonce?: Nonce;
      validFrom?: number;
      validUntil?: number;
    }):Promise<Order>;

async getLimitOrder(order: {
      tokenSell: TokenLike;
      tokenBuy: TokenLike;
      ratio: TokenRatio | WeiRatio;
      recipient?: Address;
      nonce?: Nonce;
      validFrom?: number;
      validUntil?: number;
    }): Promise<Order>;
```

### **Inputs and outputs**

[table17](Accounts%20ada2f/table17%205639c.csv)

### **Submitting a swap**

Once two compatible signed swaps are collected, anyone can submit them.

> Signature
> 

```jsx
async syncSwap(swap: {
      orders: [Order, Order];
      feeToken: TokenLike;
      amounts?: [BigNumberish, BigNumberish];
      nonce?: number;
      fee?: BigNumberish;
    }): Promise<Transaction>;
```

### **Inputs and outputs**

[table18](Accounts%20ada2f/table18%20683fc.csv)

### **Ratios**

To construct a ratio, either `utils.tokenRatio` or `utils.weiRatio` may be used.

- `tokenRatio` constructs a ratio relevant to the tokens themselves, so `{ 'ETH': 4, 'wBTC': 1 }` would mean you want 4 ETH for each wBTC.
- `weiRatio` constructs a ratio relevant to the *lowest denomination* of the token, so `{ 'ETH': 4, 'wBTC': 1 }` would mean you want 4 wei (10 ETH) for each satoshi ( 10 wBTC).
    - 18
    - 8

> Signature
> 

```jsx
function tokenRatio(ratio: { [token: string]: string | number; [token: number]: string | number }):TokenRatio;

function weiRatio(ratio: { [token: string]: BigNumberish; [token: number]: BigNumberish }):WeiRatio;
```

Only 2 tokens should be specified in each ratio.

### **Example**

```jsx
const walletA = ..; // setup first wallet
const walletB = ..; // setup second wallet

const orderA =await walletA.getOrder({
      tokenSell: 'ETH',
      tokenBuy: 'USDT',
      amount: tokenSet.parseToken('ETH', '2'),
      ratio: utils.tokenRatio({
        ETH: 1,
        USDT: '4123.40',
      })
    });

const orderB =await walletB.getOrder({
      tokenSell: 'USDT',
      tokenBuy: 'ETH',
      amount: tokenSet.parseToken('USDT', '8000'),
      ratio: utils.tokenRatio({
        ETH: 1,
        USDT: '4123.40',
      }),
      // this makes it a swap-and-transfer
      recipient: '0x2d5bf7a3ab29f0ff424d738a83f9b0588bc9241e'
    });

const swap =await walletA.syncSwap({
      orders: [orderA, orderB],
      feeToken: 'ETH',
    });

await swap.awaitReceipt();
```

### **Batched Transfers in the zkSync**

Sends several transfers in a batch. For information about transaction batches, see the[Providers section](https://merlin-li.github.io/api/sdk/js/providers.html#submit-transactions-batch).

Note that unlike in `syncTransfer`, the fee is a required field for each transaction, as in batch wallet cannot assume anything about the fee for each individual transaction.

If it is required to send a batch that include transactions other than transfers, consider using Provider's`submitTxsBatch` method instead.

> Signature
> 

```jsx
async syncMultiTransfer(transfers: {
      to: Address;
      token: TokenLike;
      amount: BigNumberish;
      fee: BigNumberish;
      nonce?: Nonce;
      validFrom?: number;
      validUntil?: number;
    }[]): Promise<Transaction[]>;
```

### **Inputs and outputs**

[table19](Accounts%20ada2f/table19%20fc77b.csv)

For details on an individual transaction, see [Transfer in the zkSync](https://merlin-li.github.io/zksync/api/js/accounts#transfer-in-the-zksync).

> Example
> 

```jsx
import { ethers }from "ethers";

const wallet = ..;// setup zksync wallet

const transferA = {
      to: "0x2d5bf7a3ab29f0ff424d738a83f9b0588bc9241e",
      token: "0xFab46E002BbF0b4509813474841E0716E6730136", // FAU ERC20 token address
      amount: ethers.utils.parseEther("1.0"),
      fee: ethers.utils.parseEther("0.001")
    };

const transferB = {
      to: "0xaabbf7a3ab29f0ff424d738a83f9b0588bc9241e",
      token: "0xFab46E002BbF0b4509813474841E0716E6730136", // FAU ERC20 token address
      amount: ethers.utils.parseEther("5.0"),
      fee: ethers.utils.parseEther("0.001")
    }

const transferTransactions =await wallet.syncMultiTransfer([transferA, transferB]);
```

### **Withdraw token from the zkSync**

Moves funds from the zkSync account to ethereum address. Sender account should have correct public key set before sending this transaction. ( see [change pub key](https://merlin-li.github.io/zksync/api/js/accounts#changing-account-public-key))

Before sending this transaction, the user will be asked to sign a specific message with transaction details using their Ethereum account (because of the security reasons).

The operators require a fee to be paid in order to process transactions.[[5:7]](https://merlin-li.github.io/zksync/api/js/accounts#fn5)

The transaction has to be verified until funds are available on the ethereum wallet balance so it is useful to use`awaitVerifyReceipt`(see [utils](https://merlin-li.github.io/api/sdk/js/utils.html#awaiting-for-operation-completion)) before checking ethereum balance.

> Signature
> 

```jsx
async withdrawFromSyncToEthereum(withdraw: {
      ethAddress: string;
      token: TokenLike;
      amount: BigNumberish;
      fee?: BigNumberish;
      nonce?: Nonce;
      fastProcessing?: boolean;
      validFrom?: number;
      validUntil?: number;
    }): Promise<Transaction>;
```

### **Inputs and outputs**

[table20](Accounts%20ada2f/table20%20814af.csv)

> Example
> 

```jsx
import { ethers }from "ethers";

const wallet = ..;// setup zksync wallet

const withdrawTransaction =await wallet.withdrawFromSyncToEthereum({
      ethAddress: "0x9de880ac69f3ed1e4d6870fcdabf07cbbed6f85c",
      token: "FAU",
      amount: ethers.utils.parseEther("1.0"),
      fee: ethers.utils.parseEther("0.001")
    });

    // Wait wait till transaction is verified
const transactionReceipt =await withdrawTransaction.awaitVerifyReceipt();
```

### **Sign withdraw token from the zkSync transaction**

Signs [withdraw](https://merlin-li.github.io/zksync/api/js/accounts#withdraw-token-from-the-zksync) transaction without sending it to the zkSync network. It is important to consider transaction fee in advance because transaction can become invalid if token price changes.

> Signature
> 

```jsx
async signWithdrawFromSyncToEthereum(withdraw: {
      ethAddress: string;
      token: TokenLike;
      amount: BigNumberish;
      fee: BigNumberish;
      nonce: number;
      validFrom?: number;
      validUntil?: number;
    }): Promise<SignedTransaction>;
```

### **Inputs and outputs**

[table21](Accounts%20ada2f/table21%207021f.csv)

### **Initiate a forced exit for an account**

Initialize a forced withdraw of funds for an unowned account. Target account must not have a signing key set and must exist more than 24 hours. After execution of the transaction, funds will be transferred from the target zkSync wallet to the corresponding Ethereum wallet. Transaction initiator pays fee for this transaction. All the balance of requested token will be transferred.

Sender account should have correct public key set before sending this transaction. (see[change pub key](https://merlin-li.github.io/zksync/api/js/accounts#changing-account-public-key))

The operators require a fee to be paid in order to process transactions[[5:10]](https://merlin-li.github.io/zksync/api/js/accounts#fn5).

**Note:** fee is paid by the transaction initiator, not by the target account.

The transaction has to be verified until funds are available on the ethereum wallet balance so it is useful to use`awaitVerifyReceipt`(see [utils](https://merlin-li.github.io/api/sdk/js/utils.html#awaiting-for-operation-completion)) before checking ethereum balance.

> Signature
> 

```jsx
async syncForcedExit(forcedExit: {
      target: Address;
      token: TokenLike;
      fee?: BigNumberish;
      nonce?: Nonce;
      validFrom?: number;
      validUntil?: number;
    }): Promise<Transaction>;
```

### **Inputs and outputs**

[table22](Accounts%20ada2f/table22%205efa8.csv)

> Example
> 

```jsx
import { ethers }from "ethers";

const wallet = ..;// setup zksync wallet

const forcedExitTransaction =await wallet.syncForcedExit({
      target: "0x9de880ac69f3ed1e4d6870fcdabf07cbbed6f85c",
      token: "FAU",
      fee: ethers.utils.parseEther("0.001")
    });

    // Wait wait till transaction is verified
const transactionReceipt = await forcedExitTransaction.awaitVerifyReceipt();
```

### **Sign a forced exit for an account transaction**

Signs [forced exit](https://merlin-li.github.io/zksync/api/js/accounts#initiate-a-forced-exit-for-an-account) transaction without sending it to the zkSync network. It is important to consider transaction fee in advance because transaction can become invalid if token price changes.

> Signature
> 

```jsx
async signSyncForcedExit(forcedExit: {
      target: Address;
      token: TokenLike;
      fee: BigNumberish;
      nonce: number;
    }): Promise<SignedTransaction>;
```

### **Inputs and outputs**

[table23](Accounts%20ada2f/table23%20979c7.csv)

### **Emergency withdraw from Sync**

If ordinary withdraw from zkSync account is ignored by network operators user could create an emergency withdraw request using special Ethereum transaction, this withdraw request can't be ignored.

Moves the full amount of the given token from the zkSync account to the Ethereum account.

Once the operation is committed to the Ethereum network, we have to wait for a certain amount of confirmations (see[provider docs](https://merlin-li.github.io/api/sdk/js/providers.html#get-amount-of-confirmations-required-for-priority-operations) for exact number) before accepting it in the zkSync network. Operation will be processed within the zkSync network as soon as the required amount of confirmations is reached.

The transaction has to be verified until funds are available on the ethereum wallet balance so it is useful to use`awaitVerifyReceipt`(see [utils](https://merlin-li.github.io/api/sdk/js/utils.html#awaiting-for-operation-completion)) before checking ethereum balance.

> Signature
> 

```jsx
async emergencyWithdraw(withdraw: {
      token: TokenLike;
      accountId?: number;
      ethTxOptions?: ethers.providers.TransactionRequest;
    }): Promise<ETHOperation>;
```

### **Inputs and outputs**

[table24](Accounts%20ada2f/table24%209e192.csv)

> Example
> 

```jsx
import * as zksyncfrom "zksync";
import { ethers }from "ethers";

const syncWallet = ..; // Setup zksync wallet.

const emergencyWithdrawPriorityOp = await syncWallet.emergencyWithdraw({
      token: "ETH",
    });

    // Wait till priority operation is verified.
const priorityOpReceipt = await emergencyWithdrawPriorityOp.awaitVerifyReceipt();
```

### **Toggle 2FA**

Two factor authentification is an additional protection layer enforced by zkSync server. You can read more about it[here](https://merlin-li.github.io/dev/payments/sending_transactions.html#_2-factor-authentication).

```jsx
import * as zksyncfrom "zksync";
import { ethers }from "ethers";

const syncWallet = ..; // Setup zksync wallet with private key or EIP-1271 signing

await syncWallet.toggle2FA(false); // disable 2FA
await syncWallet.toggle2FA(true); // enable 2FA back
```

### **Withdraw pending balance**

Calls the `withdrawPendingBalance` function on the zkSync smart contract. This function is typically used to withdraw funds that got stuck due to out-of-gas error.

> Signature
> 

```jsx
async withdrawPendingBalance(
from: Address,
      token: TokenLike,
      amount?: BigNumberish
    ): Promise<ContractTransaction>;
```

### **Inputs and outputs**

[table25](Accounts%20ada2f/table25%203d730.csv)

> Example
> 

```jsx
import * as zksyncfrom "zksync";
import { ethers } from "ethers";

const syncWallet = ..; // Setup zksync wallet.

const withdrawPendingTx = await syncWallet.withdrawPendingBalance(
      "0x9de880ac69f3ed1e4d6870fcdabf07cbbed6f85c",
      "ETH",
      ethers.utils.parseEther("0.001")
    );

    // Wait till the transaction is complete
const txReceipt =await withdrawPendingTx.wait();
```

### **Withdraw pending balances**

Calls the `withdrawPendingBalance` function multiple times on the zkSync smart contract. To optimize gas usage, instead of doing separate calls, it aggregates them using the[multicall(opens new window)](https://github.com/makerdao/multicall)smart contract.

> Signature
> 

```jsx
async withdrawPendingBalances(
      addresses: Address[],
      tokens: TokenLike[],
      multicallParams: {
        address?: Address;
        network?: Network;
        gasLimit?: BigNumberish;
      },
      amounts?: BigNumberish[]
    ): Promise<ContractTransaction>;
```

### **Inputs and outputs**

[table26](Accounts%20ada2f/table26%20cef4f.csv)

> Example
> 

```jsx
import *as zksyncfrom "zksync";
import { ethers }from "ethers";

const syncWallet = ..; // Setup zksync wallet.

const withdrawPendingTx =await syncWallet.withdrawPendingBalances(
      ["0x9de880ac69f3ed1e4d6870fcdabf07cbbed6f85c", "0x2D9835a1C1662559975B00AEA00e326D1F9f13d0"],
      ["ETH", "DAI"],
      {},
      [ethers.utils.parseEther("0.001"), ethers.utils.parseEther("0.002")]
    );

    // Wait till the transaction is complete
const txReceipt =await withdrawPendingTx.wait();
```

## **Batch Builder**

Batch Builder allows you to create and send transaction batches in a very straightforward way, without the need to worry about managing nonce or the fee transaction. It also can improve the UX of your application as it requires the user to sign the message only once for the whole batch. You can read more about transaction batches[here](https://merlin-li.github.io/dev/payments/sending_transactions#sending-transaction-batches).

Batch Builder supports all kinds of zkSync L2 transactions, such as: `Withdraw`, `Transfer`, `ChangePubKey`, etc.

All the methods of the Batch Builder are supposed to be called in a chained manner.

Note:

- *The user still has to sign a separate message for each `ChangePubKey` in the batch.*
- *Currently, a batch is guaranteed to be able to successfully process a max of 50 transactions.*

### **Create Batch Builder**

Creating Batch Builder

> Signature
> 

```jsx
batchBuilder(nonce?: Nonce): BatchBuilder;
```

### **Inputs and outputs**

[table27](Accounts%20ada2f/table27%20826b1.csv)

> Example
> 

```jsx
const syncWallet = ..; // Setup zksync wallet.

const batchBuilder = syncWallet.batchBuilder();
```

### **Add withdraw transaction**

Adding withdraw transaction to the batch.

> Signature
> 

```jsx
    addWithdraw(withdraw: {
      ethAddress: string;
      token: TokenLike;
      amount: BigNumberish;
      fee?: BigNumberish;
      fastProcessing?: boolean;
      validFrom?: number;
      validUntil?: number;
    }): BatchBuilder;
```

### **Inputs and outputs**

[table28](Accounts%20ada2f/table28%2045412.csv)

> Example
> 

```jsx
import { ethers }from "ethers";

const syncWallet = ..; // Setup zksync wallet.
const batchBuilder = ..; // Setup batch builder.

    batchBuilder.addWithdraw({
      ethAddress: syncWallet.address(),
      token: "ETH",
      amount: ethers.utils.parseEther("0.001")
    });
```

### **Add transfer transaction**

Adding transfer transaction to the batch.

> Signature
> 

```jsx
    addTransfer(transfer: {
      to: Address;
      token: TokenLike;
      amount: BigNumberish;
      fee?: BigNumberish;
      validFrom?: number;
      validUntil?: number;
    }): BatchBuilder;
```

### **Inputs and outputs**

[table29](Accounts%20ada2f/table29%2045741.csv)

> Example
> 

```jsx
import { ethers }from "ethers";

const batchBuilder = ..; // Setup batch builder.

    batchBuilder.addTransfer({
      to: "0x2D9835a1C1662559975B00AEA00e326D1F9f13d0",
      token: "ETH",
      amount: ethers.utils.parseEther("0.001")
    });
```

### **Add change pub key transaction**

Adding change pub key transaction to the batch.

> Signature
> 

```jsx
    addChangePubKey(changePubKey:
      | {
          feeToken: TokenLike;
          ethAuthType: ChangePubkeyTypes;
          fee?: BigNumberish;
          validFrom?: number;
          validUntil?: number;
        }
      | SignedTransaction
    ): BatchBuilder;
```

### **Inputs and outputs**

[table30](Accounts%20ada2f/table30%20a0639.csv)

> Example
> 

```jsx
const batchBuilder = ..; // Setup batch builder.

    batchBuilder.addChangePubKey({
      feeToken: "ETH",
      ethAuthType: 'ECDSA'
    });
```

### **Add forced exit transaction**

Adding forced exit transaction to the batch.

> Signature
> 

```jsx
    addForcedExit(forcedExit: {
      target: Address;
      token: TokenLike;
      fee?: BigNumberish;
      validFrom?: number;
      validUntil?: number;
    }): BatchBuilder;
```

### **Inputs and outputs**

[table31](Accounts%20ada2f/table31%208d5b3.csv)

> Example
> 

```jsx
import { ethers }from "ethers";

const syncWallet = ..; // Setup zksync wallet.
const batchBuilder = ..; // Setup batch builder.

    batchBuilder.addForcedExit({
      target: syncWallet.address(),
      token: "ETH"
    });
```

### **Build batch**

Construct the batch from the given transactions.

*If feeToken was provided, the fee for the whole batch will be obtained from the server in this token.*

> Signature
> 

```jsx
    build(
        feeToken?: TokenLike;
    ): Promise<{ txs: SignedTransaction[]; signature: TxEthSignature; totalFee: TotalFee }>;
```

### **Inputs and outputs**

[table32](Accounts%20ada2f/table32%200a8dd.csv)

> Example
> 

```jsx
import { ethers } from "ethers";

const syncWallet = ..; // Setup zksync wallet.
const batchBuilder = ..; // Setup batch builder.

    batchBuilder.addForcedExit({
      target: syncWallet.address(),
      token: "ETH"
    });
    batchBuilder.addTransfer({
      to: "0x2D9835a1C1662559975B00AEA00e326D1F9f13d0",
      token: "ETH",
      amount: ethers.utils.parseEther("0.001")
    });
await batchBuilder.build("ETH");
```

## **Signer**

### **Create from private key**

> Signature
> 

```jsx
static fromPrivateKey(pk: Uint8Array): Signer;
```

### **Inputs and outputs**

[table33](Accounts%20ada2f/table33%207ac5e.csv)

### **Create from seed**

> Signature
> 

```jsx
static async fromSeed(seed: Uint8Array): Promise<Signer>;
```

### **Create from ethereum signature**

> Signature
> 

```jsx
static async fromETHSignature(
      ethSigner: ethers.Signer
    ): Promise <{
      signer: Signer;
      ethSignatureType: EthSignerType;
    }>;
```

### **Inputs and outputs**

[table34](Accounts%20ada2f/table34%20e1485.csv)

### **Get public key hash**

> Signature
> 

```jsx
async pubKeyHash(): Promise<PubKeyHash>;
```

### **Inputs and outputs**

[table35](Accounts%20ada2f/table35%2094a3a.csv)

### **Sign sync transfer**

Signs transfer transaction, the result can be submitted to the zkSync network.

> Signature
> 

```jsx
async signSyncTransfer(transfer: {
      accountId: number;
from: Address;
      to: Address;
      tokenId: number;
      amount: ethers.BigNumberish;
      fee: ethers.BigNumberish;
      nonce: number;
    }): Promise<Transfer>;
```

### **Inputs and outputs**

[table36](Accounts%20ada2f/table36%20464fc.csv)

### **Sign zkSync Withdraw**

Signs withdraw transaction, the result can be submitted to the zkSync network.

> Signature
> 

```jsx
async signSyncWithdraw(withdraw: {
      accountId: number;
from: Address;
      ethAddress: string;
      tokenId: number;
      amount: ethers.BigNumberish;
      fee: ethers.BigNumberish;
      nonce: number;
    }): Promise<Withdraw>;
```

### **Inputs and outputs**

[table37](Accounts%20ada2f/table37%2052752.csv)

### **Sign zkSync Forced Exit**

Signs forced exit transaction, the result can be submitted to the zkSync network.

> Signature
> 

```jsx
async signSyncForcedExit(forcedExit: {
      initiatorAccountId: number;
      target: Address;
      tokenId: number;
      fee: BigNumberish;
      nonce: number;
    }): Promise<ForcedExit>;
```

### **Inputs and outputs**

[table38](Accounts%20ada2f/table38%202bc26.csv)

### **Sign zkSync ChangePubKey**

Signs ChangePubKey transaction, the result can be submitted to the zkSync network.

> Signature
> 

```jsx
async signSyncChangePubKey(changePubKey: {
      accountId: number;
      account: Address;
      newPkHash: PubKeyHash;
      feeTokenId: number;
      fee: BigNumberish;
      nonce: number;
      validFrom: number;
      validUntil: number;
      ethAuthData?: ChangePubKeyOnchain | ChangePubKeyECDSA | ChangePubKeyCREATE2;
      ethSignature?: string;
    }): Promise<ChangePubKey>;
```

### **Inputs and outputs**

[table39](Accounts%20ada2f/table39%20f6c8d.csv)

---

1. If undefined, `Signer` will be derived from ethereum signature of specific message. [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref1) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref1:1)
2. If undefined, it will be queried from the server. [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref2) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref2:1) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref2:2) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref2:3)
3. If undefined, it will be deduced using the signature output. [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref3) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref3:1)
4. Returned `undefined` value means that the account does not exist in the state tree. [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref4)
5. [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref5) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref5:1) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref5:2) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref5:3) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref5:4) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref5:5) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref5:6) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref5:7) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref5:8) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref5:9) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref5:10) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref5:11) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref5:12) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref5:13) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref5:14) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref5:15) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref5:16)
6. "ETH" or address of the ERC20 token [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref6) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref6:1) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref6:2) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref6:3) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref6:4) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref6:5) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref6:6) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref6:7) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref6:8) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref6:9) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref6:10) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref6:11) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref6:12) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref6:13) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref6:14) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref6:15)
7. If undefined, it will be queried from the server. [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref7) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref7:1) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref7:2) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref7:3) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref7:4) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref7:5) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref7:6) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref7:7) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref7:8) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref7:9) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref7:10) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref7:11)
8. To see if amount is packable use [pack amount util](https://merlin-li.github.io/api/sdk/js/utils.html#closest-packable-amount) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref8) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref8:1) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref8:2) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref8:3) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref8:4) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref8:5) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref8:6) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref8:7)
9. If fee was requested manually, request has to be of "FastWithdraw" type [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref9) [↩︎](https://merlin-li.github.io/zksync/api/js/accounts#fnref9:1)