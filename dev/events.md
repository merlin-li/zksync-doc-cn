# Events

- [活动](https://merlin-li.github.io/zksync/watching#watching-events)
    - [设置 yarn 项目](https://merlin-li.github.io/zksync/watching#setting-up-the-yarn-project)
    - [建立链接](https://merlin-li.github.io/zksync/watching#establishing-a-connection)
    - [过滤事件](https://merlin-li.github.io/zksync/watching#filtering-events)
    - [接收消息](https://merlin-li.github.io/zksync/watching#receiving-messages)
    - [完整示例](https://merlin-li.github.io/zksync/watching#full-example)

这是一个关于如何使用 zkSync 事件 API 的快速教程。 事件数据类型的详细描述请参考文档。

该功能目前仅在主网 Ropsten 和 Rinkeby 测试网上可用。 API 尚未完全稳定，未来可能会发生变化。

确保您为网络选择了正确的 WebSocket URL：

```

          WebSocket               Network
    wss://ropsten-events.zkscan.io  -   ropsten
    wss://events.zksync.io/         -   mainnet
    wss://rinkeby-events.zksync.io/ -   rinkeby
```

示例.

```jsx
asyncfunctionmain() {
      // Connect to the event server.
	const ws =new WebSocket("wss://ropsten-events.zkscan.io/"); // Required
}
```

## **设置 `yarn` 项目**

我们将使用 yarn 设置最小的工作 Javascript 项目：

```bash
    mkdir zksync-ws-client
    cd zksync-ws-client
    yarn init -y
    yarn add zksync ethers ws # install dependencies
```

## **建立链接**

事件服务器位于 `wss://events.zksync.io/`. 在本教程中，我们使用[ws](https://www.npmjs.com/package/ws)包， 但您可以使用任何适合您项目的 WebSocket 客户端库。

```jsx
// app.js
const WebSocket = require('ws');

const ws = new WebSocket('wss://events.zksync.io/');
```

为确保连接处于活动状态，最好定期发送 ping 指令。 它还有助于避免超时断开连接。

```jsx
setInterval(function () {
    ws.ping();
}, 10000);
```

## **过滤事件**

建立连接后，客户端需要发送一条带有他的事件兴趣的文本消息。 请注意，您只能执行一次，如果要更改过滤器，则必须创建一个新连接。有关过滤器的[详细文档](https://merlin-li.github.io/api/events.html#Filters)。

```jsx
ws.on('open',functionopen() {
      ws.send('{}');
});
```

一个空的 JSON 对象与没有过滤器相同——我们告诉服务器我们想要接收所有事件。

如果您发送无效对象，您将收到带有错误消息的关闭帧：

```jsx
// app.js
const WebSocket = require('ws');

const ws =new WebSocket('wss://events.zksync.io/');

    ws.on('open',functionopen() {
      ws.send('{"blocks": {"status": "committed"}}'); // Should be "block"
    });

    ws.on('close',functionclose(code, reason) {
      console.log(`Connection closed with code ${code}, reason: ${reason}`);
    });
```

```bash
    node app.js
    Connection closed with code 1008, reason: unknown variant `blocks`, expected one of `account`, `block`, `transaction` at line 1 column 9
```

在本教程中，我们将接受所有事件并自行过滤它们。

## **接受消息**

此时，在建立连接并成功发送我们的过滤器后，服务器将开始通知我们有关新事件的信息。 它还将忽略除控制帧（**close**、**ping** 和 **pong**）之外的任何消息。 话虽如此，您无法在不重新连接的情况下更改过滤器。

消息格式如下：

```json
    {
      "block_number": 1000,
      "type": "account, block or transaction",
      "data": {
        // Event-specific data
      }
    }
```

有关数据字段定义，请参阅[文档](https://merlin-li.github.io/api/events.html#Events)。

让我们看看如何实现消息处理程序。

```jsx
// The test account we're intrested in.
const ACCOUNT_ADDRESS = '0x7ca2113e931ada26f64da66822ece493f20059b6';

ws.on('message',function (message) {
	const event = JSON.parse(message);

  // We are looking for transfers to the specific account.
	if (event.type == 'transaction' && event.data.tx.type == 'Transfer') {
		const recipient = event.data.tx.to;
		if (recipient != ACCOUNT_ADDRESS) {
			return;
    }
		const tokenId = event.data.token_id;
		const status = event.data.status;
		const amount = event.data.tx.amount;

    console.log(`There was a transfer to ${recipient}`);
    console.log(`Token Id: ${tokenId}`);
    console.log(`Amount: ${amount}`);
    console.log(`Status: ${status}`);
  }
});
```

zkSync 提供程序可用于以人类可读的格式显示代币符号和金额。提供[程序文档](https://merlin-li.github.io/api/sdk/js/providers.html)。

## **完整示例**

```jsx
// app.js
const WebSocket = require('ws');
const zksync = require('zksync');

asyncfunctionmain() {
	// Get the provider. It's important to specify the correct network.
	const provider = await zksync.getDefaultProvider('mainnet');
	// Connect to the event server.
	const ws = new WebSocket('wss://events.zksync.io/');
	console.log('Connection established');
	// Change the address to the account you're intrested in.
	const ACCOUNT_ADDRESS = '0x7ca2113e931ada26f64da66822ece493f20059b6';

  // Once connected, start sending ping frames.
  setInterval(function () {
      ws.ping();
  }, 10000);

  // Register filters.
  ws.on('open',functionopen() {
      ws.send('{}');
  });

  ws.on('close',functionclose(code, reason) {
      console.log(`Connection closed with code ${code}, reason: ${reason}`);
  });

  ws.on('message',function (data) {
		const event = JSON.parse(data);

    // We are looking for transfers to the specific account.
		if (event.type == 'transaction' && event.data.tx.type == 'Transfer') {
			const recipient = event.data.tx.to;
			if (recipient != ACCOUNT_ADDRESS) {
				return;
			}
	    // Use the provider for formatting.
			const token = provider.tokenSet.resolveTokenSymbol(event.data.token_id);
			const amount = provider.tokenSet.formatToken(token, event.data.tx.amount);

			const status = event.data.status;
			const fromAddr = event.data.tx.from;
			const blockNumber = event.block_number;

	    console.log(`There was a transfer to ${recipient}`);
	    console.log(`Block number: ${blockNumber}`);
	    console.log(`From: ${fromAddr}`);
	    console.log(`Token: ${token}`);
	    console.log(`Amount: ${amount}`);
	    console.log(`Status: ${status}\n`);
    }
  });
}

main();
```

运行以下指令启动项目:

```bash
    node app.js
```

现在，将一些资金发送到跟踪的帐户地址，您应该会在控制台中看到已提交和已完成事件的通知：

```
    There was a transfer to 0x7ca2113e931ada26f64da66822ece493f20059b6
    Block number: 31
    From: 0x66aa7f5166ee2025781adb8582a7798f9f9e51e7
    Token: ETH
    Amount: 10.0
    Status: committed

    There was a transfer to 0x7ca2113e931ada26f64da66822ece493f20059b6
    Block number: 31
    From: 0x66aa7f5166ee2025781adb8582a7798f9f9e51e7
    Token: ETH
    Amount: 10.0
    Status: finalized
```