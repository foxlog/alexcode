---
layout: post
title:  " 以太坊编程接口 "
date:   2018-07-18 11:37:18 +0800
categories: 以太坊 100days
typora-copy-images-to: ../assets/images/2018-07
typora-root-url: ../../blog_alexcode
---
以太坊有三种方式交互

* TOC
{:toc}
## 0

搭建完go-etherum私有链后， 我们有三种方式与区块链交互

1. JavaScript console： 在geth控制台与以太坊交互， 前面已经用过。 
2. JSON-RPC： 轻量级的过程调用协议， 可以跨语言调用
3. web3.js 是以太坊提供的一个javaScript库， 它封装了以太坊的JSON-RPC API， 提供了一系列相关函数



## 编译部署智能合约

后面要用到智能合约， 我们先看下如何编译和部署智能合约



智能合约部署流程如下：

1. 使用solc编译智能合约
2. 启动一个以太坊节点(geth或testrpc)
3. 将编译好的合约发布到以太坊的网络上
4. 用web3.js api调用部署好的合约



### 安装solc编译工具

```bash
brew update
brew upgrade
brew tap ethereum/ethereum
brew install solidity
```



检查：

```bash
solc --version
```



### 编写智能合约

```js
pragma solidity ^0.4.24;

contract Storage {
    uint256 storeData;

    function set(uint256 data){
        storeData = data;
    }


    function get() constant returns (uint256){
        return storeData;
    }
}
```



### 编译合约

```bash
echo "var storageOutput = `solc --optimize --combined-json abi,bin,interface /tmp/Storage.sol` > /tmp/storage.js"
```



```bash
cat storage.js
```

```js
var storageOutput = {"contracts":{"/tmp/Storage.sol:Storage":{"abi":"[{\"constant\":false,\"inputs\":[{\"name\":\"data\",\"type\":\"uint256\"}],\"name\":\"set\",\"outputs\":[],\"payable\":false,\"stateMutability\":\"nonpayable\",\"type\":\"function\"},{\"constant\":true,\"inputs\":[],\"name\":\"get\",\"outputs\":[{\"name\":\"\",\"type\":\"uint256\"}],\"payable\":false,\"stateMutability\":\"view\",\"type\":\"function\"}]","bin":"608060405234801561001057600080fd5b5060bf8061001f6000396000f30060806040526004361060485763ffffffff7c010000000000000000000000000000000000000000000000000000000060003504166360fe47b18114604d5780636d4ce63c146064575b600080fd5b348015605857600080fd5b5060626004356088565b005b348015606f57600080fd5b506076608d565b60408051918252519081900360200190f35b600055565b600054905600a165627a7a723058201361c3ffa243868d8cae5dc25b55124b5dfb9ae56b9b0f27dfb634a43f66e6fb0029"}},"version":"0.4.24+commit.e67f0147.Darwin.appleclang"}
```





### 加载合约

```bash
loadScript('/tmp/storage.js')
```

返回true， 表示成功



执行storageOutput命令：

```json
{
  contracts: {
    /tmp/Storage.sol:Storage: {
      abi: "[{\"constant\":false,\"inputs\":[{\"name\":\"data\",\"type\":\"uint256\"}],\"name\":\"set\",\"outputs\":[],\"payable\":false,\"stateMutability\":\"nonpayable\",\"type\":\"function\"},{\"constant\":true,\"inputs\":[],\"name\":\"get\",\"outputs\":[{\"name\":\"\",\"type\":\"uint256\"}],\"payable\":false,\"stateMutability\":\"view\",\"type\":\"function\"}]",
      bin: "608060405234801561001057600080fd5b5060bf8061001f6000396000f30060806040526004361060485763ffffffff7c010000000000000000000000000000000000000000000000000000000060003504166360fe47b18114604d5780636d4ce63c146064575b600080fd5b348015605857600080fd5b5060626004356088565b005b348015606f57600080fd5b506076608d565b60408051918252519081900360200190f35b600055565b600054905600a165627a7a723058201361c3ffa243868d8cae5dc25b55124b5dfb9ae56b9b0f27dfb634a43f66e6fb0029"
    }
  },
  version: "0.4.24+commit.e67f0147.Darwin.appleclang"
}
```



### 部署智能合约

```bash
personal.unlockAccount(eth.accounts[0])
```

![](/assets/images/2018-07/2018-07-20-032309.jpg)



发送部署合约的交易

```bash
var storageContractAbi = storageOutput.contracts['/tmp/Storage.sol:Storage'].abi
var storageContract = eth.contract(JSON.parse(storageContractAbi))
var storageBinCode = "0x" + storageOutput.contracts['/tmp/Storage.sol:Storage'].bin
var deployTrasactionObject = {from: eth.accounts[0], data: storageBinCode, gas: 1000000}

var storageInstance = storageContract.new(deployTrasactionObject)
```



查询状态：

![](/assets/images/2018-07/2018-07-20-033017.jpg)



> 我这里是自动挖矿， 可以人工开启: miner.start(1);admin.sleepBlocks(1);miner.stop();



storageInstance:

```json
{
  abi: [{
      constant: false,
      inputs: [{...}],
      name: "set",
      outputs: [],
      payable: false,
      stateMutability: "nonpayable",
      type: "function"
  }, {
      constant: true,
      inputs: [],
      name: "get",
      outputs: [{...}],
      payable: false,
      stateMutability: "view",
      type: "function"
  }],
  address: "0xd181c3e83f2724946975da123adb24f23f151442",
  transactionHash: "0xe43d13af1b21f3925f105fff2300728823f7986787ee4eaf1e49498615b81b8c",
  allEvents: function(),
  get: function(),
  set: function()
}
```

该合约交易地址为： `0xe43d13af1b21f3925f105fff2300728823f7986787ee4eaf1e49498615b81b8c`



通过eth.getTransactionReceipt获取合约地址

```bash
var storageAddress = eth.getTransactionReceipt(storageInstance.transactionHash).contractAddress
```



```bash
> storageAddress
"0xd181c3e83f2724946975da123adb24f23f151442"
```



### 调用合约

```bash
var storage = storageContract.at(storageAddress)

> storage
{
  abi: [{
      constant: false,
      inputs: [{...}],
      name: "set",
      outputs: [],
      payable: false,
      stateMutability: "nonpayable",
      type: "function"
  }, {
      constant: true,
      inputs: [],
      name: "get",
      outputs: [{...}],
      payable: false,
      stateMutability: "view",
      type: "function"
  }],
  address: "0xd181c3e83f2724946975da123adb24f23f151442",
  transactionHash: null,
  allEvents: function(),
  get: function(),
  set: function()
}
```

```bash
storage.get.call()
0
```



调用合约set方法， 向以太坊网络发送一条合约调用交易：

```bash
> personal.unlockAccount(eth.accounts[0])
Unlock account 0x8c82498549c86043f97712be1081cf750eb25ca9
Passphrase:
true

> storage.set.sendTransaction(42, {from: eth.accounts[0], gas: 1000000})
"0x1bf307431f8b8f9c1ad37d7795c5844351c6631250c3710e782b73b5ebf737e4"

> txpool.status
{
  pending: 1,
  queued: 0
}

> txpool.status
{
  pending: 0,
  queued: 0
}

> storage.get.call()
42
```



成功。 



## install testrpc

前面的以太坊testrpc安装过， 不过当时直接安装的是testrpc， 实际安装包已经renamed to <u>ganache-cli</u>

```bash
npm install -g ganache-cli
```

![](/assets/images/2018-07/2018-07-19-214319.png)



运行：

```bash
ganache-cli
```

 

![](/assets/images/2018-07/2018-07-19-213953.png) 

> 这个也是运行在8545端口， 是在内存模拟的一个以太坊环境， 对开发来说比较方便。 



## web3.js API



### 安装web3.js

create a directory, then run command: 

```bash
npm init
```



这个会创建一个package.json文件：

```json
{
  "name": "emptyhtml",
  "version": "1.0.0",
  "description": "",
  "main": "demo.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "alex",
  "license": "ISC"
}
```



安装web3.js

```bash
npm install ethereum/web3.js — save
```

报错：

![](/assets/images/2018-07/2018-07-19-215218.png)



看来ethereum/web3.js不好使， 换一个正常的：

```bash
npm install --save web3@0.20.6
```

> 这里web3指定最新版本 0.20.6, 可以在 [github 官方](https://github.com/ethereum/web3.js/releases)查看

![](/assets/images/2018-07/2018-07-19-220903.png)



这次正常了。 





### 使用：

 ```js

var Web3 = require('web3');

var web3 = new Web3();

//连接到以太坊节点
web3.setProvider(new Web3.providers.HttpProvider("http://127.0.0.1:8545"))


var accounts = web3.eth.accounts

console.info(accounts)

//查看余额
var balance_1 = web3.eth.getBalance(web3.eth.accounts[0]);

console.info(balance_1.toString())
 ```



输出结果：

![](/assets/images/2018-07/2018-07-19-221449.png)



我们将ganache-cli这个模拟环境切换到geth环境：

```bash
geth --identity "TestNode" --rpc --rpcaddr=0.0.0.0 --rpccorsdomain "*" --rpcport "8545" --datadir "./db" --port "30303" --rpcapi "eth,net,web3,personal,admin,shh,txpool,debug,miner" --nodiscover --maxpeers 30 --networkid 1981 --mine --minerthreads 1 --etherbase "0xeb680f30715f347d4eb5cd03ac5eced297ac5046"  console

```

![](/assets/images/2018-07/2018-07-19-221729.png)



再跑一下web3.js：



![](/assets/images/2018-07/2018-07-19-221811.png)



没错， 就是之前我们在geth环境里创建的两个账户， 运行正确。 



## JSON-RPC

JSON-RPC是一个远程过程调用协议， 可以跨语言调用， 下面我们直接用curl命令来访问暴露RPC协议的服务， 可以理解为比web3.js更通用的访问协议， 因为不用固定在js环境中也可以访问。 



### 查看所有可用帐号

```bash
curl -X POST \
	http://localhost:8545 \
	-d '{
        "jsonrpc":"2.0",
        "method":"eth_accounts",
        "id":"1"
	}'
```



执行后报错：

![](/assets/images/2018-07/2018-07-19-222929.png)





google下发现是curl里需要加入参数：Content-Type, see [here](https://ethereum.stackexchange.com/questions/30651/geth-response-invalid-content-type-only-application-json-is-supported)

```bash
curl -X POST \
	http://localhost:8545 \
	-H "Content-Type: application/json" \
	-d '{
        "jsonrpc":"2.0",
        "method":"eth_accounts",
        "id":"1"
	}'
```



运行结果：

![](/assets/images/2018-07/2018-07-19-223821.png)





### 查看账户余额

```bash
curl -X POST \
	http://localhost:8545 \
	-H "Content-Type: application/json" \
	-d '{
        "jsonrpc":"2.0",
        "method":"eth_getBalance",
        "params":["0x58eb51c51864dbfb82db3a9ae0632f18a8d6f716"],
        "id":1
	}'
```



也报错:

![](/assets/images/2018-07/2018-07-19-230259.jpg)



google下， 说需要增加一个参数latest , see [here](https://github.com/ethereum/go-ethereum/issues/2472)

```bash

curl -X POST \
        http://localhost:8545 \
        -H "Content-Type: application/json" \
        -d '{
        "jsonrpc":"2.0",
        "method":"eth_getBalance",
        "params":["0x58eb51c51864dbfb82db3a9ae0632f18a8d6f716","latest"],
        "id":1
        }'
```



正常了：

![](/assets/images/2018-07/2018-07-19-230405.jpg)



### 账户之间转账交易

先估算需要多少gas

```bash
curl -X POST \
        http://localhost:8545 \
        -H "Content-Type: application/json" \
        -d '{
        "jsonrpc":"2.0",
        "method":"eth_estimateGas",
        "params":[{
            "from":"0xfd7f76276d184a4b692dd66e6c3b8f153eadaef6",
            "to":"0x367312f4c8bd5ed2edab205ef59e13959aa66f0b",
            "value":"0xde0b6b3a7640000"
        }],
        "id":1
        }'
```



返回结果：

```json
{"id":1,"jsonrpc":"2.0","result":"0x5208"}
```



转账：

```bash
curl -X POST \
        http://localhost:8545 \
        -H "Content-Type: application/json" \
        -d '{
        "jsonrpc":"2.0",
        "method":"eth_sendTransaction",
        "params":[{
            "from":"0xfd7f76276d184a4b692dd66e6c3b8f153eadaef6",
            "to":"0x367312f4c8bd5ed2edab205ef59e13959aa66f0b",
            "value":"0xde0b6b3a7640000",
            "gas":"0x5208"
        }],
        "id":1
        }'
```

返回结果：

```json
{"id":1,"jsonrpc":"2.0","result":"0x4e836f6e37aad4c6cf892b69b4b6273cf3cca16c0f851fbb1a231c2608616967"}
```



查看该交易详情:

```bash
curl -X POST \
        http://localhost:8545 \
        -H "Content-Type: application/json" \
        -d '{
        "jsonrpc":"2.0",
        "method":"eth_getTransactionByHash",
        "params":["0x4e836f6e37aad4c6cf892b69b4b6273cf3cca16c0f851fbb1a231c2608616967"],
        "id":1
        }'
```



返回结果：

```json
{"id":1,"jsonrpc":"2.0","result":{"hash":"0x4e836f6e37aad4c6cf892b69b4b6273cf3cca16c0f851fbb1a231c2608616967","nonce":"0x0","blockHash":"0xcb28050aceeb20346562c474e8d8227236c27bd9bd33a9470cbedb78a751415e","blockNumber":"0x01","transactionIndex":"0x0","from":"0xfd7f76276d184a4b692dd66e6c3b8f153eadaef6","to":"0x367312f4c8bd5ed2edab205ef59e13959aa66f0b","value":"0x0de0b6b3a7640000","gas":"0x5208","gasPrice":"0x01","input":"0x0"}}
```





交易被成功打包后， 可查看该交易在区块中的详细信息：

```bash
curl -X POST \
        http://localhost:8545 \
        -H "Content-Type: application/json" \
        -d '{
        "jsonrpc":"2.0",
        "method":"eth_getTransactionReceipt",
        "params":["0x4e836f6e37aad4c6cf892b69b4b6273cf3cca16c0f851fbb1a231c2608616967"],
        "id":1
        }'
```



### 调用合约

计算get()函数签名， 将合约的get()方法先经过sha3计算：

```bash
curl -X POST \
	http://localhost:8545 \
	-H "Content-Type: application/json" \
	-d '{
        "jsonrpc":"2.0",
        "method":"web3_sha3",
        "params":["get()"],
        "id":"1"
	}'
```



报错：

```json
{"jsonrpc":"2.0","id":"1","error":{"code":-32602,"message":"invalid argument 0: json: cannot unmarshal hex string without 0x prefix into Go value of type hexutil.Bytes"}}
```



说参数没有0x前缀， 加上0x：



先将get()转换为hex： 6765742829



```bash
curl -X POST \
	http://localhost:8545 \
	-H "Content-Type: application/json" \
	-d '{
        "jsonrpc":"2.0",
        "method":"web3_sha3",
        "params":["0x6765742829"],
        "id":"1"
	}'
```

返回结果：

```json
{"jsonrpc":"2.0","id":"1","result":"0x6d4ce63caa65600744ac797760560da39ebd16e8240936b51f53368ef9e0e01f"}
```

记下result的除0x之外的前八位： `6d4ce63c`



使用同样的方法， 计算set(uint256)方法的签名：

先将set(uint256)方法转换为hex: `7365742875696e7432353629` 

```bash
curl -X POST \
	http://localhost:8545 \
	-H "Content-Type: application/json" \
	-d '{
        "jsonrpc":"2.0",
        "method":"web3_sha3",
        "params":["0x7365742875696e7432353629"],
        "id":"1"
	}'
```

返回结果:

```json
{"jsonrpc":"2.0","id":"1","result":"0x60fe47b16ed402aae66ca03d2bfc51478ee897c26a1158669c7058d5f24898f4"}
```

同样记下前八位: `60fe47b1`



开始调用合约的get方法：

```bash
curl -X POST \
	http://localhost:8545 \
	-H "Content-Type: application/json" \
	-d '{
        "jsonrpc":"2.0",
        "method":"eth_call",
        "params":[
            {
                "from":"0x8c82498549c86043f97712be1081cf750eb25ca9",
                "to":"0xd181c3e83f2724946975da123adb24f23f151442",
                "data":"0x6d4ce63c0000000000000000000000000000000000000000000000000000000000000000"
            },
            "latest"
        ],
        "id":"1"
	}'
```

> eth.accounts[0]: 0x8c82498549c86043f97712be1081cf750eb25ca9
>
> to: 0xd181c3e83f2724946975da123adb24f23f151442
>
> data: 前缀是0x+get签名十六进制前八位； 参数：这里因为get方法没有参数， 所以传的是0



返回结果：

```json
{"jsonrpc":"2.0","id":"1","result":"0x000000000000000000000000000000000000000000000000000000000000002a"}
```

2a对应的十进制是42， 确实是我之前执行合约时候测试的结果， 正确。 



下面调用set

```bash
curl -X POST \
	http://localhost:8545 \
	-H "Content-Type: application/json" \
	-d '{
        "jsonrpc":"2.0",
        "method":"eth_sendTransaction",
        "params":[
            {
                "from":"0x8c82498549c86043f97712be1081cf750eb25ca9",
                "to":"0xd181c3e83f2724946975da123adb24f23f151442",
                "gas":"0xd450",
                "gaslimit":"0xd450",
             "data":"0x60fe47b1000000000000000000000000000000000000000000000000000000000000002c"
            },
            "latest"
        ],
        "id":"1"
	}'
```

> data: 前缀是set签名前八位， 后面是传进去的参数



结果：

```json
{"jsonrpc":"2.0","id":"1","error":{"code":-32602,"message":"too many arguments, want at most 1"}}
```



参数过多， google结果是参数太多， 去掉latest：

```bash
curl -X POST \
	http://localhost:8545 \
	-H "Content-Type: application/json" \
	-d '{
        "jsonrpc":"2.0",
        "method":"eth_sendTransaction",
        "params":[
            {
                "from":"0x8c82498549c86043f97712be1081cf750eb25ca9",
                "to":"0xd181c3e83f2724946975da123adb24f23f151442",
                "gas":"0xd450",
                "gaslimit":"0xd450",
             "data":"0x60fe47b1000000000000000000000000000000000000000000000000000000000000002c"
            }
        ],
        "id":"1"
	}'
```

继续报错：

```json
{"jsonrpc":"2.0","id":"1","error":{"code":-32000,"message":"authentication needed: password or unlock"}}
```

帐户需要解锁， 之前是在geth console做的， 再做一次吧。 至于在rpc里如何做还不清楚。 

```bash
> personal.unlockAccount(eth.accounts[0])
Unlock account 0x8c82498549c86043f97712be1081cf750eb25ca9
Passphrase:
true
```



重复执行代码：

```bash
curl -X POST \
	http://localhost:8545 \
	-H "Content-Type: application/json" \
	-d '{
        "jsonrpc":"2.0",
        "method":"eth_sendTransaction",
        "params":[
            {
                "from":"0x8c82498549c86043f97712be1081cf750eb25ca9",
                "to":"0xd181c3e83f2724946975da123adb24f23f151442",
                "gas":"0xd450",
                "gaslimit":"0xd450",
             "data":"0x60fe47b1000000000000000000000000000000000000000000000000000000000000002c"
            }
        ],
        "id":"1"
	}'
```



成功！

```json
{"jsonrpc":"2.0","id":"1","result":"0x67cfcb6c407cc324be07714f3610fff042a2393e9531886941366d2c21d6c24d"}
```



我们调用get方法看看结果是否变成2c了:

```bash
curl -X POST \
	http://localhost:8545 \
	-H "Content-Type: application/json" \
	-d '{
        "jsonrpc":"2.0",
        "method":"eth_call",
        "params":[
            {
                "from":"0x8c82498549c86043f97712be1081cf750eb25ca9",
                "to":"0xd181c3e83f2724946975da123adb24f23f151442",
                "data":"0x6d4ce63c0000000000000000000000000000000000000000000000000000000000000000"
            },
            "latest"
        ],
        "id":"1"
	}'
```



bingo！

```json
{"jsonrpc":"2.0","id":"1","result":"0x000000000000000000000000000000000000000000000000000000000000002c"}
```

