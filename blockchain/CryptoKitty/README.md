
## truffle-contract 部署链接合约
http://www.osbaike.net/article-show-id-241645.html

truffle 私有链 创建多个节点
geth 私有链 区别


# geth远程服务器启动节点

### rpcaddr default 127.0.0.1 -> 0.0.0.0

1. geth --testnet --rpc --rpcapi db,eth,net,web3,personal --cache=1024 --rpcport 8545 --rpcaddr 127.0.0.1 console
2. geth --testnet --rpc --rpcapi db,eth,net,web3,personal --cache=1024 --rpcport 8545 --rpcaddr 0.0.0.0 "*" console


$ netstat -tlnp|grep 8545
tcp        0      0 127.0.0.1:8545          0.0.0.0:*               LISTEN      16237/bin/geth
$ netstat -tlnp|grep 8545
tcp6       0      0 :::8545                 :::*                    LISTEN      16248/bin/geth



web3@latest deploy contract ok!
web3@0.20.1 connect contract ok!



## sendRawTransaction
#### 资料
[how-to-use-web3-js-to-sign-a-contract-call](https://forum.ethereum.org/discussion/5039/how-to-use-web3-js-to-sign-a-contract-call)

```

var Web3 = require("web3");
var web3 = new Web3(new Web3.providers.HttpProvider("https://ropsten.infura.io/"));
var privateKey = new Buffer('your privateKey','hex');
var from = 'your address';

var Tx = require('ethereumjs-tx');
var value = web3.toWei(0.25,'ether');
var valueHex = web3.toHex( value ) ;
var nonce = web3.eth.getTransactionCount(from);
var nonceHex = web3.toHex( nonce ) ;
var gasPrice = web3.eth.gasPrice;
var gasPriceHex = web3.toHex( gasPrice );
var gasLimitHex =  web3.toHex( 300000 );

console.log('value: ',value);
console.log('valueHex: ',valueHex);
console.log('nonce: ',nonce);
console.log('nonceHex: ',nonceHex);
console.log('gasPrice: ',gasPrice);
console.log('gasPriceHex: ',gasPriceHex);
console.log('gasLimitHex: ',gasLimitHex);

var rawTx = {
    nonce: nonceHex,
    gasPrice: gasPriceHex, 
    gasLimit: gasLimitHex,
    to: '0x7423663D15C2DBae7699f32ea6561D2ADD74b378', 
    value: valueHex, 
    data: '0x1234567890'
};

var tx = new Tx(rawTx);
tx.sign(privateKey);

var serializedTx = tx.serialize();
var serializedHex = '0x' + serializedTx.toString('hex');
console.log('serializedTx: ',serializedTx);
console.log('serializedHex: ',serializedHex);

web3.eth.sendRawTransaction(serializedHex, function(err, hash) {
    if (!err){
        console.log('交易订单请查看 \nhttps://ropsten.etherscan.io/tx/' + hash);
    } else {
        console.error(err);
    }
});

/*
value:  250000000000000000
valueHex:  0x3782dace9d90000
nonce:  5
nonceHex:  0x5
gasPrice:  BigNumber { s: 1, e: 10, c: [ 27139464130 ] }
gasPriceHex:  0x651a35bc2
gasLimitHex:  0x493e0
serializedTx:  <Buffer f8 72 05 85 06 51 a3 5b c2 83 04 93 e0 94 74 23 66 3d 15 c2 db ae 76 99 f3 2e a6 56 1d 2a dd 74 b3 78 88 03 78 2d ac e9 d9 00 00 85 12 34 56 78 90 1c ... >
serializedHex:  0xf87205850651a35bc2830493e0947423663d15c2dbae7699f32ea6561d2add74b3788803782dace9d900008512345678901ca0392751481803f2895d06f7c52a65608f85876d892c095685ade9981b85ce9db1a0123abdf988ed2c4037a180bfafb410173b68bda5144e2858a4d430e2e6724001
交易订单请查看
https://ropsten.etherscan.io/tx/0xa70a53634cc6985436bf27c301291dcbac84223b40ce21b7fe8acf9cfec334f7



value:  250000000000000000
valueHex:  0x3782dace9d90000
nonce:  4
nonceHex:  0x4
gasPrice:  BigNumber { s: 1, e: 10, c: [ 27139464130 ] }
gasPriceHex:  0x651a35bc2
gasLimitHex:  0x493e0
serializedTx:  <Buffer f8 72 04 85 06 51 a3 5b c2 83 04 93 e0 94 74 23 66 3d 15 c2 db ae 76 99 f3 2e a6 56 1d 2a dd 74 b3 78 88 03 78 2d ac e9 d9 00 00 85 12 34 56 78 90 1b ... >
serializedHex:  0xf87204850651a35bc2830493e0947423663d15c2dbae7699f32ea6561d2add74b3788803782dace9d900008512345678901ba098dee9e034dba4cefe09df4eb9f3a6ecd9c629251852fc5f7a6f2457920c8
ae7a0191e9a6d71d5a5e13f1bd3c0e2e26c059ffdbec1fdb86dbd41e5020027b5623e
交易订单请查看
https://ropsten.etherscan.io/tx/0xca6024ec428844d2e7d2fd8212a9788bffca9da7ba1834f3bbca7b701d021722

```


## 合约 Smart Contract

```
// 这个事件是在两个猫成功繁衍了一只猫的时候触发，服务器端可以监听该事件，作为回调处理服务器端数据。
//事件方法定义的所有的参数服务器端都可以拿到

event Pregnant(address owner, uint256 matronId, uint256 sireId, uint256 cooldownEndBlock)

// Emit the pregnancy event.
Pregnant(kittyIndexToOwner[_matronId], _matronId, _sireId, matron.cooldownEndBlock);


服务器端

var contract_abi = config.abi;

var rechargeContract = web3.eth.contract(contract_abi);
var instance = rechargeContract.at(config.sc_address);

//监听Transfer事件
var event = instance.Transfer({},{fromBlock: 1000000, toBlock: 'latest'});
event.watch(function(err, info){
    if(!err) {
        var orderId = info.args.orderId.toString();
        var value = info.args.value.toNumber();
    }
});

```
## Deploy CryptoKitties Smart Contract to Ropsten network

Error

#### Error: Contract transaction couldn't be found after 50 blocks

[truffle-contract-transaction-couldnt-be-found-after-50-blocks](https://ethereum.stackexchange.com/questions/26599/truffle-contract-transaction-couldnt-be-found-after-50-blocks)

#### Error encountered, bailing. Network state unknown. Review successful transactions manually.

#### Error: The contract code couldn't be stored, please check your gas amount.

gas 设置的太小了。

```js
var HDWalletProvider = require("truffle-hdwallet-provider");

let mnemonic = 'metamask mnemonic';

module.exports = {
  networks: {
    dev: {
      host: "127.0.0.1",
      port: 8545,
      network_id: "*" // Match any network id
    },
    ropsten: {
      provider: new HDWalletProvider(mnemonic, "https://ropsten.infura.io/"),
      gas: 4700000,
      gasPrice: 65000000000,
      network_id: 3
    }
  },
  solc: {
    optimizer: {
      enabled: true,
      runs: 200
    }
  }
};

//$truffle migrate --network ropsten --reset
```

