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
