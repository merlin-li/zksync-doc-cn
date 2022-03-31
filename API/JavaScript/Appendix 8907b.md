# Appendix

# **附录 A: 在浏览器中使用 bundle**

可以在浏览器中直接使用 `zksync.js`。

- `zksync.js` 需要`ethers@5.0`依赖才能正常工作。

> Example with zksync@0.8.1 fetched from https://unpkg.comCDN
> 

```html
<html>
  <body>
    <script type="text/javascript" src="https://cdn.ethers.io/lib/ethers-5.0.umd.min.js"></script>
    <script type="text/javascript" src="https://unpkg.com/zksync@0.8.1/dist/main.js"></script>
    <script type="text/javascript">
      (async () => {
				const ethWallet = ethers.Wallet.createRandom();

				const zksProvider =await zksync.getDefaultProvider('rinkeby');
				const zkSyncWallet =await zksync.Wallet.fromEthSigner(ethWallet, zksProvider);
        console.log('ETH balance:', (await zkSyncWallet.getBalance('ETH')).toString());

				const privateKey =await zksync.crypto.privateKeyFromSeed(new Uint8Array(32));
				const pubkeyHash =await zksync.crypto.privateKeyToPubKeyHash(privateKey);
        console.log('PrivateKey', ethers.utils.hexlify(privateKey), 'PubkeyHash', pubkeyHash);
      })();
    </script>
  </body>
</html>
```