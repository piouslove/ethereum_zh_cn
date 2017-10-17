# JSON RPC API
[JSON](http://json.org/)是一种轻量级的数据交换格式。 它可以表示数字，字符串，有序的值序列，以及键/值对的集合。
[JSON-RPC](http://www.jsonrpc.org/specification)是无状态，轻量级的远程过程调用（RPC）协议。 这个规范首先定义了几个数据结构及其处理规则。 这是传输不可知的，因为这些概念可以在同一进程中，通过套接字，通过HTTP或在许多各种消息传递环境中使用。 它使用JSON（[RFC 4627](http://www.ietf.org/rfc/rfc4627.txt)）作为数据格式。
Geth 1.4具有实验性的pub/sub支持。 有关详细信息，请参阅[此页面](https://github.com/ethereum/go-ethereum/wiki/RPC-PUB-SUB)。
Parity 1.6具有实验性的pub/sub支持有关详细信息，请参阅[此页面](https://github.com/paritytech/parity/wiki/JSONRPC-Eth-Pub-Sub-Module)。
## JavaScript API
要从JavaScript应用程序内部与一个无关节点通话，请使用[web3.js](https://github.com/ethereum/web3.js)库，为RPC方法提供了方便的界面。 有关更多信息，请参阅[JavaScript API](https://github.com/ethereum/wiki/wiki/JavaScript-API)。
## JSON-RPC端点JSON-RPC Endpoint
默认JSON-RPC端点：
* C++	http://localhost:8545
* Go	http://localhost:8545
* Py	http://localhost:4000
* Parity	http://localhost:8545
### Go
您可以使用--rpc标志启动HTTP JSON-RPC
```
geth --rpc
```
更改默认端口（8545）和列表地址（localhost）：
```
geth --rpc --rpcaddr <ip> --rpcport <portnumber>
```
如果从浏览器访问RPC，CORS将需要启用适当的域集。 否则，JavaScript调用受同源策略的限制，请求将失败：
```
geth --rpc --rpccorsdomain "http://localhost:3000"
```
也可以使用`admin.startRPC(addr, port)`命令从[geth控制台](https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console)启动JSON RPC。
### C++
您可以通过使用`-j`选项运行`eth`应用程序来启动它：
```
./eth -j
```
您还可以指定JSON-RPC端口（默认为8545）：
```
./eth -j --json-rpc-port 8079
```
### Python
在python中，JSONRPC服务器默认启动，并在127.0.0.1:4000上监听。
您可以通过提供配置选项来更改端口和监听地址。
```
pyethapp -c jsonrpc.listen_port=4002 -c jsonrpc.listen_host=127.0.0.2 run
```
## [JSON-RPC support](https://github.com/ethereum/wiki/wiki/JSON-RPC#json-rpc-support)
## 十六进制值编码 HEX value encoding
目前有两个关键的数据类型通过JSON传递：未格式化的字节数组`unformatted byte arrays`和数量`quantities`。两者都使用十六进制编码传递，但对格式化有不同的要求：
当编码`QUANTITIES`（整数，数字）时：编码为十六进制，前缀为“0x”，最紧凑的表示（微小异常：零表示为“0x0”）。 例子：
* 0x41 (65 in decimal)
* 0x400 (1024 in decimal)
* WRONG: 0x (should always have at least one digit - zero is "0x0"至少应该有一个数字)
* WRONG: 0x0400 (no leading zeroes allowed不能在0x后前导0)
* WRONG: ff (must be prefixed 0x必须以0x开头)
当编码`UNFORMATTED DATA`（字节数组，帐户地址，散列，字节码）时：编码为十六进制，前缀为“0x”，每字节两个十六进制数。 例子：
* 0x41 (size 1, "A")
* 0x004200 (size 3, "\0B\0")
* 0x (size 0, "")
* WRONG: 0xf0f0f (must be even number of digits必须是偶数位数)
* WRONG: 004200 (must be prefixed 0x)
目前，[cpp-ethereum](https://github.com/ethereum/cpp-ethereum)，[go-hohere](https://github.com/ethereum/go-ethereum)和[parity](https://github.com/paritytech/parity)通过http和IPC（Windows上的命名管道/Linux和OSX的unix套接字）提供JSON-RPC通信。go-ethereum 1.4版本和Parity 1.6版本具有Websocket支持。
## 默认区块参数The default block parameter
以下方法有一个额外的默认区块参数：
* [eth_getBalance](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_getbalance)
* [eth_getCode](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_getcode)
* [eth_getTransactionCount](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_gettransactioncount)
* [eth_getStorageAt](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_getstorageat)
* [eth_call](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_call)  
当请求作为偏移状态时，最后的默认区块参数确定区块的高度。
`defaultBlock`参数可以使用以下选项：
* `HEX String` - an integer block number 一个整数区块号
* `String "earliest"` for the earliest/genesis block 最早/创世区块
* `String "latest"` - for the latest mined block 最新开采出来的区块
* `String "pending"` - for the pending state/transactions 待处理状态/事务
## Curl实例说明 Curl Examples Explained
下面的curl选项可能返回节点对内容类型申报异常complain的响应，这是因为`--data`选项将内容类型设置为`application/x-www-form-urlencoded`。如果您的节点complain，请在调用开始时放置`-H "Content-Type：application/json"`手动设置头部参数。
这些示例也不包括必须为curl给出的最后一个参数URL/IP和端口组合，例如`127.0.0.1:8545`。
## [JSON-RPC methods](https://github.com/ethereum/wiki/wiki/JSON-RPC#json-rpc-methods)