# Events

本文档以与语言无关的方式更详细地描述了使用 zkSync 事件。 有关带有 Javascript 示例的 zkSync 事件的信息，请阅读 [快速介绍](https://merlin-li.github.io/zksync/watching#example)。

- [Events](https://merlin-li.github.io/zksync/api/events#events)
    - [建立连接](https://merlin-li.github.io/zksync/api/events#establishing-the-connection)
    - [事件结构](https://merlin-li.github.io/zksync/api/events#event-structure)
        - [账户事件](https://merlin-li.github.io/zksync/api/events#account-events)
        - [Block 事件](https://merlin-li.github.io/zksync/api/events#block-events)
        - [交易事件](https://merlin-li.github.io/zksync/api/events#transacton-events)
    - [Filters](https://merlin-li.github.io/zksync/api/events#filters)
        - [过滤账户事件](https://merlin-li.github.io/zksync/api/events#filter-account-events)
        - [过滤 Block 事件](https://merlin-li.github.io/zksync/api/events#filter-block-events)
        - [过滤交易事件](https://merlin-li.github.io/zksync/api/events#filter-transaction-events)
        - [Examples](https://merlin-li.github.io/zksync/api/events#examples)
    - [注意事项](https://merlin-li.github.io/zksync/api/events#note-on-stability)

## **建立连接**

主网的事件服务器 Websocket URL 是 `wss://events.zksync.io`。 在 [here](https://merlin-li.github.io/dev/events.html)[这里](https://merlin-li.github.io/zksync/watching#example) 查看示例。

## **事件结构**

建立连接后，你的客户端将收到以下格式的有关事件的消息：:

```json
    {
      "block_number": 1100,
      "type": "account",
      "data": {
        // Event-specific data
      }
    }
```

block_number 是事件发生的 zkSync 区块的编号。 type 是事件的类型，可以是 **account**、**block**或 **transaction**。 数据是特定于事件的数据。

### **账户事件**

账户事件是发生在 zkSync 账户上的事件。 账户事件有四种类型：**create**、**delete**、**update_blance**、**update_pub_key_hash**。 **delete**当前被禁用。

每个**account**的 **data** 数据格式如下t:

```json
    {
      "update_type": "create",
      "status": "committed",
      "update_details": {
        // details about account update
      }
    }
```

当更新在链上提交时，**status**字段等于 **commitedt**，或者当具有此类事件的块已通过链上的零知识证明进行验证时，状态字段等于已完成。

完整示例:

```json
    {
      "block_number": 25,
      "type": "account",
      "data": {
        "update_type": "create",
        "status": "committed",
        "update_details": {
          "account_id": 19,
          "nonce": 0
        }
      }
    }
```

```json
    {
      "block_number": 24,
      "type": "account",
      "data": {
        "update_type": "update_balance",
        "status": "committed",
        "update_details": {
          "account_id": 18,
          "nonce": 0,
          "token_id": 0,
          "new_balance": "2000000000000000000000"
        }
      }
    }
```

```json
    {
      "block_number": 27,
      "type": "account",
      "data": {
        "update_type": "change_pub_key_hash",
        "status": "committed",
        "update_details": {
          "account_id": 20,
          "nonce": 1,
          "new_pub_key_hash": "sync:8af45346a8456d7a1fc26507ce1699329efcb4c3"
        }
      }
    }
```

### **Block 事件**

块事件是通知块是否已提交或完成的事件。 此类事件具有以下结构：

```json
    {
      "block_number": 27,
      "type": "block",
      "data": {
        "status": "committed",
        "block_number": 27,
        "new_state_root": "sync-bl:194d9ce3c118867f6132c2f92bbb602b230d9d777372f6198801b0699afecbc0",
        "block_size": 32,
        "commit_tx_hash": "0xde56e5a61e4588d9c25246a841f05675b9b10cb26def279475580fe58a0b443f",
        "committed_at": "2021-05-20T11:32:35.433853Z"
      }
    }
```

当更新在链上提交时，**status**字段等于 **committed**，或者当具有此类事件的块已通过链上的零知识证明进行验证时，状态字段等于已完成。 如果块已被还原，它也可以等于 reverted。 已完成的区块永远无法恢复。

**finalized** block 事件完整示例:

```json
    {
      "block_number": 24,
      "type": "block",
      "data": {
        "status": "finalized",
        "block_number": 24,
        "new_state_root": "sync-bl:15b15f8c1b20aff7da841a77250c3ada37997e79beded3ea65641f7e0db7239a",
        "block_size": 32,
        "commit_tx_hash": "0x274e78480e51b8eb4186effab2f1c940914984eecd224675155eec26e359820d",
        "verify_tx_hash": "0xb670313796467bd97d8ed40071da5c67bc2ac123d435c0131ea9fec2894c03c0",
        "committed_at": "2021-05-20T11:32:26.432639Z",
        "verified_at": "2021-05-20T11:32:37.443933Z"
      }
    }
```

### **交易事件**

交易事件通知正在提交或完成的交易。 它们通知 L2 和 L1 的交易优先级操作。 每个事件的**data**如下所示：

```json
    {
      "tx_hash": "0xf62c8a0bc8d0ead1abbd3b8772bb27973e40e42027f06ab3952e7dc29cb69c27",
      "account_id": 18,
      "token_id": 0,
      "block_number": 24,
      "tx": {
        // Tx data unique for each transaction type
      },
      "status": "committed",
      "created_at": "2021-05-20T11:32:26.232439Z"
    }
```

**status** 字段等于交易在链上提交时已提交或在具有此类事件的块已通过链上零知识证明验证时完成。 如果交易无效，它也可以等于 reverted。

以下是交易事件的一些示例：

### **Transfer**

```json
      {
        "block_number": 25,
        "type": "transaction",
        "data": {
          "tx_hash": "sync-tx:5237455cb49ef063a81b77f1848f4ed46196ff2f8b55faa945c117d7d5282159",
          "account_id": 18,
          "token_id": 0,
          "block_number": 25,
          "tx": {
            "to": "0x675c03cadf1528af6260f1f1be1f557bbcefbb05",
            "fee": "31000000000000",
            "from": "0xdeaafff966f14498d1c9cd5561c92c914564aef9",
            "type": "Transfer",
            "nonce": 1,
            "token": 0,
            "amount": "10000000000000000000",
            "accountId": 18,
            "signature": {
              "pubKey": "4a2ef46ea7765c77c8ca84cfb0597a527985ee51b5e5be28531dbade85ee1498",
              "signature": "366ccd69f9183c2b78d0448bf0e552b95a4f47f343d9dff7ba60537c50efab9b8d18b4693f8f8ef9787e6e1d60a6cecdc21c9871d59d8a38935458ba12495f00"
            },
            "validFrom": 0,
            "validUntil": 4294967295
          },
          "status": "finalized",
          "created_at": "2021-05-20T11:32:28.840918Z"
        }
      }
```

### **ChangePubKey**

```json
    {
      "block_number": 27,
      "type": "transaction",
      "data": {
        "tx_hash": "sync-tx:b78eaacff2c86ec9fbf6e4e11d8a911cb3c1c8be41b3af14eeec1fb242d34f7e",
        "account_id": 20,
        "token_id": 0,
        "block_number": 27,
        "tx": {
          "fee": "0",
          "type": "ChangePubKey",
          "nonce": 0,
          "account": "0xf32b6fb3332b7b3edf30e1e469f581974bd8b378",
          "feeToken": 0,
          "accountId": 20,
          "newPkHash": "sync:8af45346a8456d7a1fc26507ce1699329efcb4c3",
          "signature": {
            "pubKey": "0732cb8a917b3c48fbaa39a6da760c50e972ee014033eb511237a43a221ef480",
            "signature": "b7ea639a0924bc62437e9732956b90531b0f1dee52d17e6b06dd56bf6c9df80ee968b73b55ed350579e298d0d5d5018bc9bd5238fe1a7aca91c6a63ba0b9cf04"
          },
          "validFrom": 0,
          "validUntil": 4294967295,
          "ethAuthData": {
            "type": "Onchain"
          },
          "ethSignature":null
        },
        "status": "finalized",
        "created_at": "2021-05-20T11:32:35.279076Z"
      }
    }
```

### **Withdraw**

```json
    {
      "block_number": 28,
      "type": "transaction",
      "data": {
        "tx_hash": "sync-tx:37f1adc19bd78cecc56bcdb3453d7cfd340434757f1d9bbe56dddb0d2de66047",
        "account_id": 22,
        "token_id": 0,
        "block_number": 28,
        "tx": {
          "to": "0x1cf9517f403235c7e871c5d96e224326a404e12d",
          "fee": "141400000000000",
          "fast":false,
          "from": "0x1cf9517f403235c7e871c5d96e224326a404e12d",
          "type": "Withdraw",
          "nonce": 1,
          "token": 0,
          "amount": "10000000000000000000",
          "accountId": 22,
          "signature": {
            "pubKey": "71b2ec034927306a016374419f806442a84af73e175865961f6b627adbc51b24",
            "signature": "ae923a804ef5999dba0d0c017aa4417b93b36fe24e1fe5d394b2a8b390e45b98f5e08dce47c7a1b675db9c0fab5bf4ab1b0c6059bb9d9fe7cb134deb398e2705"
          },
          "validFrom": 0,
          "validUntil": 4294967295
        },
        "status": "committed",
        "created_at": "2021-05-20T11:32:36.873472Z"
      }
    }
```

### **ForcedExit**

```json
    {
      "block_number": 29,
      "type": "transaction",
      "data": {
        "tx_hash": "sync-tx:e43867aaa909ea52879428e28836c9e162951a60bb9ad94d1c03c432017bbbb6",
        "account_id": 20,
        "token_id": 0,
        "block_number": 29,
        "tx": {
          "fee": "204300000000000",
          "type": "ForcedExit",
          "nonce": 6,
          "token": 0,
          "target": "0x3a9d7cced6e600eacf5e70a7f55b5d458e1ea5f8",
          "signature": {
            "pubKey": "0732cb8a917b3c48fbaa39a6da760c50e972ee014033eb511237a43a221ef480",
            "signature": "52af07b1d5ca8a61dd916ae31ac199f8e645d7f2be696c151a96debed0429a8476047fe4f93390ac6825b6fef816268b24fc1aee07562659fa7dc0c5bb6edc04"
          },
          "validFrom": 0,
          "validUntil": 4294967295,
          "initiatorAccountId": 20
        },
        "status": "committed",
        "created_at": "2021-05-20T11:32:38.657075Z"
      }
    }
```

### **Deposit**

```json
    {
      "block_number": 27,
      "type": "transaction",
      "data": {
        "tx_hash": "0x4bde9a0ab7eff4850799a0961afadad2d4914cfa332a5a7a140a6a9b8b5c57fc",
        "account_id": 22,
        "token_id": 2,
        "block_number": 27,
        "tx": {
          "type": "Deposit",
          "account_id": 22,
          "priority_op": {
            "to": "0x1cf9517f403235c7e871c5d96e224326a404e12d",
            "from": "0x36615cf349d7f6344891b1e7ca7c72883f5dc049",
            "token": 2,
            "amount": "2000000000000000000000"
          }
        },
        "status": "finalized",
        "created_at": "2021-05-20T11:32:33.232676Z"
      }
    }
```

### **FullExit**

```json
    {
      "block_number": 41,
      "type": "transaction",
      "data": {
        "tx_hash": "0x7d83f56d243630afd40292e2d8ca307ca5287d82d1324e2795d85a28a05694b6",
        "account_id": 145,
        "token_id": 2,
        "block_number": 41,
        "tx": {
          "type": "FullExit",
          "priority_op": {
            "token": 2,
            "account_id": 145,
            "eth_address": "0x905ef38b8b2fdedfab9a77fdde94c577d66fd91d"
          },
          "withdraw_amount":null
        },
        "status": "committed",
        "created_at": "2021-05-20T11:34:45.232211Z"
      }
    }
```

## **过滤器**

建立连接后，客户端需要发送事件过滤器。 过滤器是以下类型的 JSON 对象：

```json
    {
      "account": {
        // `account` events filter
      },
      "block": {
        // `block` events filter
      },
      "transaction": {
        // `transaction` events filter
      }
    }
```

某些字段可能会被省略。 例如，以下过滤器告诉客户端只想接收某种类型的**account**事件。

```json
    {
      "account": {
        // account filter
      }
    }
```

如果过滤器对象为空，则表示“不过滤”。 这里有一些例子：

```json
    // Here we receive all the `account` events from zkSync
    {
      "account": {}
    }
```

```json
    // Here we receive all the `account` and `block` events from zkSync
    {
      "account": {},
      "block": {}
    }
```

```json
    // Here we receive all the `account`, `block`, and `transaction` events
    // This is the same as receiving all events
    {
      "account": {},
      "block": {},
      "transaction": {}
    }
```

```json

    // Empty object means we want to receive all events
    {}
```

### **Filter account events**

**accounts**事件可以通过几个参数过滤：

```json
    {
      "accounts": [],
      "tokens": [],
      "status": "committed"
    }
```

**accounts** 参数是一个帐户 ID 数组，有关其的事件应发送到客户端。 如果未指定，将发送有关任何帐户的事件。

**tokens** 参数是一个token id 的数组，在帐户更新中涉及。

**status** 是需要发送的更新的状态。

### **Filter block events**

`block` 事件只能通过块的状态过滤：

```json
    {
      "status": "finalized"
    }
```

### **Filter transaction events**

**transaction** 事件可以通过以下几个参数过滤:

```jsx
    "types": [],
    "accounts": [],
    "tokens": [],
    "status": "finalized",
```

The **accounts**, **tokens**, and **status** parameters have the same meaning as for the **account** events filters.

**types** 是你希望接收的交易类型的数组。 支持以下交易类型: **Transfer**, **Withdraw**, **ChangePubKey** , **ForcedExit**, **FullExit**, **Deposit**。

### **示例**

一些其他示例:

```json
    // Here we are only interested in the committed events about
    // accounts with ids 1,2,3 which change their balance of token with id 0 (ETH)
    {
      "account": {
        "status": "committed",
        "accounts": [1, 2, 3],
        "tokens": [0]
      }
    }
```

```json
    // Here we are only interested in the finalized events of any account
    // and any block verification events any transaction events (both committed and finalized)
    {
      "account": {
        "status": "finalized"
      },
      "block": {
        "status": "reverted"
      },
      "transaction": {}
    }
```

## **注意事项**

从技术上讲，已提交的块可能会被还原，其中的交易也将被还原。 这种情况很少发生，但如果您的客户依赖于已提交的事件，则应采取一些预防措施。 如果一个块被恢复，客户端将收到一个适当的**block**事件通知，但它不会被通知有关单个**transaction**事件和要恢复的**account**事件。 建议如果您依赖已提交的事件，那么您还可以跟踪**block**事件以了解块是否被还原。

如果您只想接收有关最终交易的事件，那么您应该只接受最终事件。 最终确定的区块永远无法恢复，因为它是由以太坊智能合约强制执行的。