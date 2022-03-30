# 代币转换和限价订单

- [代币转换和限价订单](https://merlin-li.github.io/zksync/swaps#swaps-and-limit-orders)
    - [原子转换](https://merlin-li.github.io/zksync/swaps#atomic-swaps)
        - [订单签名](https://merlin-li.github.io/zksync/swaps#signing-an-order)
        - [提交转换](https://merlin-li.github.io/zksync/swaps#submitting-a-swap)
    - [限价订单](https://merlin-li.github.io/zksync/swaps#limit-orders)
        - [交易账户](https://merlin-li.github.io/zksync/swaps#trading-accounts)
        - [限价订单签名](https://merlin-li.github.io/zksync/swaps#signing-limit-orders)
        - [填写限价单](https://merlin-li.github.io/zksync/swaps#filling-a-limit-order)
        - [限价单撮合注意事项](https://merlin-li.github.io/zksync/swaps#note-on-limit-order-matching)
        - [建议](https://merlin-li.github.io/zksync/swaps#suggestions)
        - [例子](https://merlin-li.github.io/zksync/swaps#example)
    - [实用工具](https://merlin-li.github.io/zksync/swaps#utils)

## **原子转换**

原子交换让您可以安全、廉价地与现有 zkSync 账户交换资金。

成功进行交换需要 3 个步骤：

- 签署确认您要执行特定交换的订单
- 从您要交换的账户中获取相同格式的已签名订单
- 向 zkSync 服务器提交两个有偿订单

### **订单签名**

要给一个订单签名，您需要以下信息：

- 你想交换的代币
- 你想换取的代币
- 您要交换的代币数量
- 交换代币的比率，彼此相关

比率是 15 字节整数，表示交换令牌的比例。

要签名一个订单, 需要使用 Wallet 的 `[getOrder](https://merlin-li.github.io/api/sdk/js/accounts.html#signing-orders)` 函数:

```jsx
const order = await wallet.getOrder({
    tokenSell: 'ETH',
    tokenBuy: 'USDT',
    amount: tokenSet.parseToken('ETH', '2.5'),
    ratio: utils.tokenRatio({
        ETH: 1,
        USDT: '4234.5'
    })
});
```

订单还可以包括：

- `recipient` - 现有帐户的地址，如果您想要执行交换和转移，则应将交换结果转移到该地址。 默认为自己
- `validFrom` and `validUntil` Unix 时间戳，它们限制了可以处理带有交换的块的时间跨度
- `nonce` 将用于交换的随机数

### **发送转换**

如果满足以下限制，任何人都可以提交 2 个转换订单：

- 订单有匹配的令牌：如果 orderA 指定 tokenA -> tokenB，那么 orderB 应该指定 tokenB -> tokenA
- 订单中的比率兼容：1/orderB.ratio <= orderA.amount/orderB.amount >= orderA.ratio
- 如果订单有收件人，他们的账户已经存在于 zkSync 中

费用由提交者支付，并注明支付的代币。 执行交换后，交换账户和提交者的随机数都会增加。 如果从交换帐户之一提交交换，则 nonce 仅增加一次

如果用户希望取消尚未提交的交换，他们只需增加他们的随机数（例如发送零转账）。

发送转换订单, 使用 Wallet 的 `[syncSwap](https://merlin-li.github.io/api/sdk/js/accounts.html#submitting-a-swap)` 函数:

```jsx
const swap =await wallet.syncSwap({
    orders: [orderA, orderB],
    feeToken: 'wBTC'
});
```

## **限价订单**

限价单提供了一种以特定价格将特定代币兑换成另一种代币的方式。 它们旨在主要由希望提供去信任和可扩展交换服务的其他平台使用。

原子互换和限价单之间的区别是：

- 限价单从余额中推断出可以直接兑换的金额
- 限价单可以部分成交
- 限价单在部分成交时不会增加账户的 nonce

这意味着一旦为某个代币签署了限价订单，整个余额可能会被换成另一个代币（以指定的比率）。 除了使用特殊交易账户外，没有办法限制要兑换的金额。 :::

### **交易账户**

交易账户是可以用来签署限价单的普通账户。 它的功能是限制用户想要交换的某个令牌的数量。

为此，用户必须：

1. 将所需数量的所需令牌转移到新帐户。
2. 为帐户设置签名密钥。
3. 签署限价单。

这样，限价单将最多兑换您转入交易账户的金额。 主账户的余额将保持不变。

### **限价订单签名**

签名需要使用 Wallet 的 `[getLimitOrder](https://merlin-li.github.io/api/sdk/js/accounts.html#signing-orders)` 函数：

```jsx
const order =await wallet.getLimitOrder({
      tokenSell: 'ETH',
      tokenBuy: 'USDT',
      ratio: utils.tokenRatio({
        ETH: 1,
        USDT: 3900
      })
});
```

### **填写限价单**

限价单本身只代表转换操作的一部分。 为了填写，必须满足以下条件：

- 存在与原始订单的代币和买入/卖出比率相匹配的对应订单（普通订单或限价订单）。
- 有人愿意将两个订单合并到一个交换操作中并提交。

应在交换操作中指定正在填充的数量。 限价单可以部分成交，因此金额可能与实际余额不同，但必须与订单中指定的比率兼容。 有关详细信息，请参见[示例](https://merlin-li.github.io/zksync/swaps#example)。

```jsx
const swap =await wallet.syncSwap({
    orders: [orderA, orderB],
    amounts: [tokenSet.parseToken('ETH', '2'), tokenSet.parseToken('UDST', '7800')],
    feeToken: 'wBTC'
});
```

### **限价单撮合注意事项**

执行原子交换就像与要交换的一方共享已签名的订单消息一样简单。

另一方面，收集和匹配限价单可能应该由已经存在用户群的平台通过某种匹配引擎来执行。 zkSync 尽量通用，所以不考虑创建匹配引擎——我们只会提供其他平台可以集成的 L2 框架。

### **建议**

交易账户可以创建为 CREATE2 账户。这种方法有以下好处：

- 在 CREATE2 帐户上设置签名密钥更便宜
- CREATE2 中的 salt 参数可用于确定性地为某个主账户生成交易账户地址
- 如果需要，所有交易账户和主账户都可以使用相同的 L2 私钥。尽管这会带来一些风险（损害单个帐户将意味着损害所有帐户），但在某些情况下密钥管理可能会很不方便。

如果平台决定将 CREATE2 用于交易账户，则必须选择用于地址计算的合约字节码。该合约应该是开源的，并具有完整的退出和退出功能，因为在极少数审查的情况下，用户将不得不部署它来挽救他们的资金。

还建议重复使用已执行或取消订单的交易账户，因为这样就不必再次设置签名密钥。

### **示例**

本节提供了一个示例，说明订单中指定的比率如何影响账户余额。

一个转换包含 2 个限价订单:

```jsx

// walletA wants to swap wBTC for ETH at a ratio 2:5
const orderA =await walletA.getLimitOrder({
        tokenSell: 'wBTC',
        tokenBuy: 'ETH',
        ratio: utils.tokenRatio({
          wBTC: 2,
          ETH: 5
        })
      });

// walletB wants to swap ETH for wBTC at a ratio 4:1
const orderB =await walletB.getLimitOrder({
        tokenSell: 'ETH',
        tokenBuy: 'wBTC',
        ratio: utils.tokenRatio({
          wBTC: 1,
          ETH: 4
        })
      });
```

指定的比率意味着：

- walletA 预计每个 wBTC（或更多）可以获得 2.5 ETH
- walletB 预计每个 ETH（或更多）可以获得 0.25 wBTC

比率是兼容的，因为在任何一个比率（或介于两者之间），双方都会很高兴：

- 按订单 B 的比例，钱包 A 将获得每 wBTC 4 ETH，这比预期的要多
- 按订单 A 的比例，钱包 B 将每 ETH 获得 0.4 wBTC，超出预期

现在让我们实际提交交换，并在两者之间选择一个比率 - 每个 wBTC 3 ETH：

```jsx
const swap =await walletC.syncSwap({
      orders: [orderA, orderB],
      amounts: [tokenSet.parseToken('wBTC', '100'), tokenSet.parseToken('ETH', '300')],
      feeToken: 'USDT'
    });
```

这将把 Wallet A 中的 100 wBTC 兑换成 Wallet B 中的 300 ETH。 有关详细信息，请参见下表。

[Utils](swaps/swaps.csv)

## **实用工具**

要构建比率，请使用以下两个效用函数之一：

- **tokenRatio** 构造了一个与代币本身相关的比率，因此 { ETH: 4, wBTC: 1 } 意味着您需要每个 wBTC 4 ETH。
- **weiRatio** 构造了一个与代币最低面额相关的比率，因此 { ETH: 4, wBTC: 1 } 意味着您需要每个 satoshi (10-8 wBTC) 4 wei (10-18 ETH)。

如果标记符号或 ID 包含在变量中，请使用以下语法将它们传递给比率对象：

```jsx
    utils.tokenRatio({
      [tokenA]: valueA,
      [tokenB]: valueB
    });
```

更多信息请参考 [API reference](https://merlin-li.github.io/api/sdk/js/accounts.html#ratios).
