# 发送交易

本节介绍将交易发送到 zkSync 网络的原理。

提供的示例是用 JavaScript 编写的，但不依赖于任何特定的 SDK。

- [发送交易](https://merlin-li.github.io/zksync/payments/tx#sending-transactions)
    - [发送优先操作](https://merlin-li.github.io/zksync/payments/tx#sending-priority-operations)
    - [发送交易](https://merlin-li.github.io/zksync/payments/tx#sending-transactions)
    - [批量发送交易](https://merlin-li.github.io/zksync/payments/tx#sending-transaction-batches)
        - [安全注意事项](https://merlin-li.github.io/zksync/payments/tx#note-on-security)
        - [批量签名](https://merlin-li.github.io/zksync/payments/tx#ethereum-signature-for-batch)
        - [2-Factor 验证](https://merlin-li.github.io/zksync/payments/tx#_2-factor-authentication)

## **发送优先交易**

通过调用相应的智能合约方法来调用优先级操作。

对应存款方式的签名：

```jsx

/// @notice Deposit ETH to Layer 2 - transfer ether from user into contract, validate it, register deposit
/// @param _franklinAddr The receiver Layer 2 address
functiondepositETH(address _franklinAddr)externalpayablenonReentrant;

/// @notice Deposit ERC20 token to Layer 2 - transfer ERC20 tokens from user into contract, validate it, register deposit
/// @param _token Token address
/// @param _amount Token amount
/// @param _franklinAddr Receiver Layer 2 address
functiondepositERC20(IERC20 _token, uint104 _amount, address _franklinAddr)externalnonReentrant;
```

请注意，在存入 ERC20 资金之前，必须先批准相应数量的资金用于合约，例如： （在 JS 中）：

```jsx
erc20contract.approve(zkSyncContractAddress, deposit_amount);
```

要执行完全退出，用户必须依次注册退出请求（通过 fullExit 合约调用），然后完成此请求（通过 **completeWithdrawals** 调用）。

对应方法的签名：

```jsx

/// @notice Register full exit request - pack pubdata, add priority request
/// @param _accountId Numerical id of the account in the zkSync network
/// @param _token Token address, 0 address for ether
functionfullExit (uint32 _accountId, address _token)externalnonReentrant;

/// @notice executes pending withdrawals
/// @param _n The number of withdrawals to complete starting from oldest
functioncompleteWithdrawals(uint32 _n)externalnonReentrant;
```

## **发送交易**

为了发送交易，用户必须执行以下步骤：

1. 准备交易数据。
2. 将交易数据编码成字节序列。
3. 使用私钥为这些字节创建 zkSync 签名。
4. 为交易描述生成以太坊签名（请参阅下面的详细信息）或提供 EIP-1271 签名。
5. 通过相应的 [JSON RPC](https://merlin-li.github.io/api/v0.1.html#tx-submit) 方法发送交易。

有关交易数据和将其编码为字节序列的详细信息，请参见[官方协议描述](https://github.com/matter-labs/zksync/blob/master/docs/protocol.md).

要查看对签名原语的编程语言支持，请参阅 [加密相关](https://merlin-li.github.io/api/sdk/crypto)。

以太坊签名的消息取决于交易类型：

```jsx

    // Amount and fee must be encoded into formatted string, e.g. by `ethers.utils.formatUnits` method
    // with respect to the token decimals value.
    // Token must be represented as a token symbol, e.g. `ETH` or `DAI`.

    // For Transfer:
const transferEthMessage =
      `Transfer ${stringAmount} ${stringToken}\n` +
      `To: ${transfer.to.toLowerCase()}\n` +
      `Nonce: ${transfer.nonce}\n` +
      `Fee: ${stringFee} ${stringToken}\n` +
      `Account Id: ${this.accountId}`;

    // For Withdraw:
const withdrawEthMessage =
      `Withdraw ${stringAmount} ${stringToken}\n` +
      `To: ${withdraw.ethAddress.toLowerCase()}\n` +
      `Nonce: ${withdraw.nonce}\n` +
      `Fee: ${stringFee} ${stringToken}\n` +
      `Account Id: ${this.accountId}`;

    // For ChangePubKey (assuming it is a stand-alone transaction, for batch see details below):
const msgNonce = utils.hexlify(serializeNonce(nonce));
const msgAccId = utils.hexlify(serializeAccountId(accountId));
const pubKeyHashHex = pubKeyHash.replace('sync:', '').toLowerCase();
const changePubKeyEthMessage =
      `Register zkSync pubkey:\n\n` +
      `${pubKeyHashHex}\n` +
      `nonce: ${msgNonce}\n` +
      `account id: ${msgAccId}\n\n` +
      `Only sign this message for a trusted client!`;

```

请注意，由于某些以太坊签名者会在签名消息中添加前缀 x19Ethereum Signed Message: $ messageBytes.length，因此如果使用的签名者没有自动添加此前缀，则可能需要手动添加此前缀。

## **发送批量交易**

事务批处理是一组应该一起成功的事务。 如果其中一个批处理事务失败，则该批处理中的所有事务也将失败。

注意

### **安全注意事项**

在当前形式中，事务批处理是服务器端的抽象。 成功执行是在电路前检查的，有关批处理的信息不会传递到电路中。 因此，如果使用此功能以不同的令牌支付费用，建议将费用支付交易设置为最后（这样服务器即使在理论上也无法执行最后一笔交易，而忽略其他交易）。 将来，批次将在电路中强制执行，以提高此功能的整体安全性。

目前，一个批次保证能够成功处理最多 50 笔交易。

对于批量交易，不必在每个单独的交易中设置费用，唯一的要求是交易中设置的费用总和必须等于或大于单独发送的交易的费用总和。

也就是说，使用批量交易，可以使用代币支付交易费用，而不是用于转账。 为此，可以创建一个包含两个事务的批次：

- 以 **FOO** 代币转账给收件人，费用设置为 0。
- 以 **BAR** 代币（您要用来支付费用的代币）转账到自己的账户，金额设置为 0，费用设置为足以支付两次转账。

服务器将检查费用总和（第一笔交易中的 0 和第二笔交易中的 2 倍预期费用）是否足以涵盖两次转账的处理，并将执行批处理。

### **批量签名**

对于批量交易，无需为其中的每笔交易提供以太坊签名，相反，可以每批次仅提供一个签名。

要签名的批量消息应形成如下：

```jsx

    // Assuming that `transactions` variable holds an array of batch transactions, and
    // `serializeTx(...)` encodes transaction into bytes as per zkSync protocol.

    // Obtain concatenated byte representations of each transaction.
const bytes = concat(transactions.map((tx) => serializeTx(tx)));
    // Calculate `keccak256` hash of this byte sequence.
const hash = ethers.utils.keccak256(bytes).slice(2);
    // Decode it into a byte sequence.
const message = Uint8Array.from(Buffer.from(hash, 'hex'));

```

上述添加前缀的要求仍然成立。

得到的这个签名可以通过相应的 [JSON RPC](https://merlin-li.github.io/api/v0.1.html#submit-txs-batch) 方式与批量一起发送，批量交易不需要以太坊签名。

### **2-Factor Authentication**

为什么需要 2-factor Authentication?

为确保用户 zkSync 账户的安全性与以太坊钱包的安全性相同，zkSync 服务器在提交交易时需要您的第 1 层和第 2 层签名。

在一些保护您的 zkSync 第 2 层私钥的钱包客户端中，您可以选择将两者解绑，这样即使您丢失了以太坊私钥，您仍然可以访问 zkSync 资金。

2FA 默认启用，但可以通过向我们的 API 提交此类请求来关闭。