# Providers

JSON-RPC 协议用于与 zkSync 网络节点进行通信。 Provider 用于抽象通信细节并提供有用的 API 用于与 zkSync 网络交互。

我们支持用于 JSON-RPC 通信的 HTTP 和 WebSocket 传输协议。 尽管 WebSocket 传输支持订阅，但由于其稳定性，HTTP 传输是首选。 HTTPTransport 和 WSTransport 类用于实现通信细节，但通常不需要直接处理这些对象。

> 温馨提示：由于稳定性， Websocket 支持将会在接下来移除。
> 

## **zkSync provider**

### **获取默认网络的 provider**

```jsx
import * as zksyncfrom 'zksync';

const syncHttpProvider = await zksync.getDefaultProvider('rinkeby');
```

用于通过 HTTP 传输连接到给定网络的公共端点。

支持的网络有: "rinkeby", "ropsten", "mainnet", and "localhost".

### **创建 WebSocket provider (已启用，很快会移除)**

> 通过 WebSocket 传输创建提供程序。 此调用将创建应关闭的 WS 连接。
> 

```jsx
import * as zksyncfrom 'zksync';

const syncWSProvider = await zksync.Provider.newWebsocketProvider('wss://rinkeby-api.zksync.io/jsrpc-ws');

// ..

// Later to close connection.
await syncWSProvider.disconnect();
```

### **Create HTTP provider**

> 通过 HTTP 传输创建 Provider.
> 

```jsx
import * as zksyncfrom 'zksync';
const syncHTTPProvider = await zksync.Provider.newHttpProvider('https://rinkeby-api.zksync.io/jsrpc');
```

### **Submit transaction**

> 签名
> 

```jsx
async submitTx(tx: any, signature?: TxEthSignature, fastProcessing?: boolean): Promise<string>;
```

### **输入输出**

[table1](Providers%209e1c2/table1%20a8bb7.csv)

> Example
> 

```jsx
import *as zksyncfrom 'zksync';

const syncHttpProvider =await zksync.getDefaultProvider('rinkeby');
const signedTransferTx = {
      accountId: 13, // id of the sender account in the zkSync
      type: 'Transfer',
      from: '0x..address1',
      to: '0x..address2',
      token: 0, // id of the ETH token
      amount: '1000000000000000000', // 1 Ether in Wei
      fee: '10000000000000000', // 0.01 Ether in Wei
      nonce: 0,
      signature: {
        pubKey: 'dead..', // hex encoded packed public key of signer (32 bytes)
        signature: 'beef..' // hex encoded signature of the tx (64 bytes)
      }
    };

    // const readableTxInfo =
    //     `Transfer 1.0 ETH` +
    //     `To: 0x..address2` +
    //     `Nonce: 0` +
    //     `Fee: 0.01 ETH` +
    //     `Account Id: 13`;

const ethSignature = '0xdddaaa...1c'; // Ethereum ECDSA signature of the readableTxInfo

const transactionHash =await syncHttpProvider.submitTx(signedTransferTx, ethSignature);
    // 0x..hash (32 bytes)
```

### **提交批量交易**

有关批量交易的详细信息, 请查看 [相应的开发文档部分](https://merlin-li.github.io/zksync/payments/tx)。

> 签名
> 

```jsx
async submitTxsBatch(
      transactions: { tx: any; signature?: TxEthSignature }[],
      ethSignatures?: TxEthSignature | TxEthSignature[]
    ): Promise<string[]>;
```

### **Inputs and outputs**

[table2](Providers%209e1c2/table2%2091858.csv)

有关个别交易的详情, 请查看 [提交交易](https://merlin-li.github.io/zksync/api/js/providers#submit-transaction).

> 示例
> 

```jsx
import *as zksyncfrom 'zksync';

const syncHttpProvider =await zksync.getDefaultProvider('rinkeby');
const firstTransferTx = {
      accountId: 13, // id of the sender account in the zkSync
      type: 'Transfer',
      from: '0x..address1',
      to: '0x..address2',
      token: 0, // id of the ETH token
      amount: '1000000000000000000', // 1 Ether in Wei
      fee: '10000000000000000', // 0.01 Ether in Wei
      nonce: 0,
      signature: {
        pubKey: 'dead..', // hex encoded packed public key of signer (32 bytes)
        signature: 'beef..' // hex encoded signature of the tx (64 bytes)
      }
    };
const firstTransferEthSignature = '0xdddaaa...1c'; // Ethereum ECDSA signature for the first message

const secondTransferTx = {
      type: 'Transfer'
      // ...other fields omitted
    };
const secondTransferEthSignature = '0xaaaddd...ff'; // Ethereum ECDSA signature for the second message

const batch = [
      { tx: firstTransferTx, signature: firstTransferEthSignature },
      { tx: secondTransferTx, signature: secondTransferEthSignature }
    ];

const transactionHashes =await syncHttpProvider.submitTxsBatch(batch);
    // List of transaction hashes
```

### **Get contract addresses**

> 签名
> 

```jsx
async getContractAddress(): Promise<ContractAddress>;
```

### **Inputs and outputs**

[table3](Providers%209e1c2/table3%2011273.csv)

> 示例
> 

```jsx
import * as zksyncfrom 'zksync';

const syncHttpProvider = await zksync.getDefaultProvider('rinkeby');

const contractAddresses = await syncHttpProvider.getContractAddress();
```

> Returns
> 

```json
    {
      "mainContract": "0xab..cd",
      "govContract": "0xef..12"
    }
```

### **Get tokens**

> Signature
> 

```jsx
async getTokens(): Promise<Tokens>;
```

### **输入输出**

[table4](Providers%209e1c2/table4%2077832.csv)

> Example
> 

```jsx
import * as zksyncfrom 'zksync';

const syncHttpProvider =await zksync.getDefaultProvider('rinkeby');

const contractAddresses =await syncHttpProvider.getTokens();
```

> Returns
> 

```json
    {
      "ERC20-1": {
        "address": "0xbeeb9f55d523918f9cd2979a454610f673c2885e",
        "id": 1,
        "symbol":null
      },
      "ETH": {
        "address": "0000000000000000000000000000000000000000",
        "id": 0,
        "symbol": "ETH"
      }
    }
```

### **Get account state by address**

> Signature
> 

```jsx
async getState(address: Address): Promise<AccountState>;
```

### **Inputs and outputs**

[table5](Providers%209e1c2/table5%2060870.csv)

> Returns
> 

```json
    {
      "address": "0x2d5bf7a3ab29f0ff424d738a83f9b0588bc9241e",
      "id": 1, // optional
      "committed": {
        "balances": {
          "ETH": "1000000000000000000" // 1 Ether in Wei
        },
        "nonce": 1
      },
      "depositing": {
        "balances": {
          "FAU": {
            "amount": "9000000000000000",
            "expectedAcceptBlock": 438929
          }
        }
      },
      "verified": {
        "balances": {
          "ETH": "1000000000000000000", // 1 Ether in Wei
          // ERC20 token
          "FAU": "1000000000000000000"
        },
        "nonce": 0
      }
    }
```

For details on the `depositing` section, see the description of `AccountState` type on the `types` page.

### **Get amount of confirmations required for priority operations**

> Signature
> 

```jsx
async getConfirmationsForEthOpAmount(): Promise<number>;
```

### **Inputs and outputs**

[table6](Providers%209e1c2/table6%20c2514.csv)

> Example
> 

```jsx
import * as zksyncfrom 'zksync';

const syncHttpProvider = await zksync.getDefaultProvider('rinkeby');
const requiredConfirmationsAmount = await syncHttpProvider.getConfirmationsForEthOpAmount();
```

### **Get transaction receipt**

> Signature
> 

```jsx
async getTxReceipt(txHash: string): Promise<TransactionReceipt>;
```

### **Inputs and outputs**

[table7](Providers%209e1c2/table7%20fdc2f.csv)

> Returns
> 

```json
    // Not executed yet
    {
        "executed":false
    }

    // Success
    {
        "executed":true,
        "success":true,
        "block": {
          "blockNumber": 658,
          "committed":true,
          "verified":true
        }
    }

    // Failure
    {
        "executed":true,
        "success":true,
        "failReason": "Nonce mismatch",
        "block": {
          "blockNumber": 658,
          "committed":true,
          "verified":true
        }
    }
```

### **Wait for transaction receipt**

Similar to [Get transaction receipt](https://merlin-li.github.io/zksync/api/js/providers#get-transaction-receipt) but this method will return when a given transaction is committed or verified in the zkSync network.

> Signature
> 

```json
async notifyTransaction(
      hash: string,
      action: "COMMIT" | "VERIFY"
    ): Promise<TransactionReceipt> ;
```

### **Inputs and outputs**

[table8](Providers%209e1c2/table8%203852b.csv)

> Example
> 

```jsx
import * as zksync from 'zksync';

const syncHttpProvider = await zksync.getDefaultProvider('rinkeby');

const receipt =await syncHttpProvider.notifyTransaction(
      'sync-tx:1111111111111111111111111111111111111111111111111111111111111111',
      'COMMIT'
);
```

### **Get priority operation receipt**

> Signature
> 

```jsx
async getPriorityOpStatus(
        serialId: number
    ): Promise<PriorityOperationReceipt>;
```

### **Inputs and outputs**

[table9](Providers%209e1c2/table9%2024c2f.csv)

Serial ID of the priority operation can be found in logs of the ethereum transaction that created this operation (e.g. deposit).

> Returns
> 

```json
    {
      "executed":true,
      "block": {
        "blockNumber": 658,
        "committed":true,
        "verified":true
      }
    }
```

### **Wait for priority operation receipt**

Similar to [Get priority operation receipt](https://merlin-li.github.io/zksync/api/js/providers#get-priority-operation-receipt) but this method will return when given priority operation is committed or verified in the zkSync network.

> Signature
> 

```jsx
async notifyPriorityOp(
        serialId: number,
        action: "COMMIT" | "VERIFY"
    ): Promise<PriorityOperationReceipt>;
```

### **Inputs and outputs**

[table10](Providers%209e1c2/table10%20106d1.csv)

Serial ID of the priority operation can be found in logs of the ethereum transaction that created this operation (e.g. deposit).

> Example
> 

```jsx
import *as zksyncfrom 'zksync';

const syncHttpProvider =await zksync.getDefaultProvider('rinkeby');

const receipt =await syncHttpProvider.notifyPriorityOp(
      178, // priority op id
      'COMMIT'
);
```

### **Current token set**

Provider stores list of the available tokens with methods for working with them. (see[working with tokens](https://merlin-li.github.io/api/sdk/js/utils.html#working-with-tokens))

> Signature
> 

`public tokenSet: TokenSet;`

### **Get transaction fee from the server**

Performs a query to the server, obtaining an acceptable transaction fee for transactions. The returned value contains all the price components used for the fee calculation, and the fee itself (`totalFee` field).

**Note:** If fee is requested for a `ForcedExit` operation, corresponding `txType` will be `Withdraw`.

> Signature
> 

```jsx
async getTransactionFee(
        txType: "Withdraw" | "Transfer" | "FastWithdraw" | ChangePubKeyFee | LegacyChangePubKeyFee,
        address: Address,
        tokenLike: TokenLike
    ): Promise<Fee>;
```

Interface of `IncomingTxFeeType` type is described in the [fees](https://merlin-li.github.io/api/sdk/js/types.html#fees) section.

### **Inputs and outputs**

[table11](Providers%209e1c2/table11%202180d.csv)

### **Get transaction batch fee from the server**

Performs a query to the server, obtaining an acceptable fee for a batch transaction (multi-transfer).

The fee provided is enough to perform **all** of the transactions of the batch. Thus you usually would need to specify the fee for only one transaction and set it to zero for the other ones.

**Note:** For details about the type and amount of token for batch transaction fees, see[transaction batch docs](https://merlin-li.github.io/zksync/api/js/providers#submit-transactions-batch).

> Signature
> 

```jsx
async getTransactionsBatchFee(
        txTypes: IncomingTxFeeType[],
        addresses: Address[],
        tokenLike: TokenLike
    ): Promise<BigNumber>;
```

### **Inputs and outputs**

[table12](Providers%209e1c2/table12%208de44.csv)

### **Get token price**

Performs a query to the server, obtaining a token price in USD. Data is fetched by server using third-party API (e.g. coinmarketcap).

> Signature
> 

```jsx
async getTokenPrice(
        tokenLike: TokenLike
    ): Promise<number> ;
```

### **Inputs and outputs**

[table13](Providers%209e1c2/table13%206b843.csv)

> Example
> 

```jsx
import *as zksyncfrom 'zksync';

const syncHttpProvider =await zksync.getDefaultProvider('rinkeby');
const ethPrice =await syncHttpProvider.getTokenPrice('ETH');

console.log(`Current Ethereum price is ${ethPrice} USD`);
```

## **ETH Proxy**

`ETHProxy` class is used to simplify some communication with Ethereum network.

### **Create ETH Proxy**

> Signature
> 

```jsx
constructor(
    private ethersProvider: ethers.providers.Provider,
    private contractAddress: ContractAddress
);
```

### **Inputs and outputs**

[table14](Providers%209e1c2/table14%20a7f19.csv)

> Example
> 

```jsx
import *as zksyncfrom 'zksync';
import { ethers }from 'ethers';

const ethersProvider = ethers.getDefaultProvider('rinkeby');
const syncHttpProvider =await zksync.getDefaultProvider('rinkeby');

const ethProxy =new zksync.ETHProxy(ethersProvider, syncHttpProvider.contractAddress);
```

### **Resolve token id**

To sign zkSync transaction users have to know the unique numerical id of the given token. It can be retrieved from the zkSync network governance contract.

> Signature
> 

```jsx
async resolveTokenId(token: TokenAddress): Promise<number>;
```

### **Inputs and outputs**

[table15](Providers%209e1c2/table15%204b2ad.csv)

> Example
> 

```jsx
import * as zksyncfrom 'zksync';
import { ethers }from 'ethers';

const ethersProvider = ethers.getDefaultProvider('rinkeby');
const syncProvider = await zksync.getDefaultProvider('rinkeby');
const ethProxy = new zksync.ETHProxy(ethersProvider, syncProvider.contractAddress);

const ethId = await ethProxy.resolveTokenId('0x0000000000000000000000000000000000000000'); // ETH token address is 0x0..0

// ERC20 token if it is supported, >= 1
const erc20Id = await ethProxy.resolveTokenId('0xFab46E002BbF0b4509813474841E0716E6730136');
```