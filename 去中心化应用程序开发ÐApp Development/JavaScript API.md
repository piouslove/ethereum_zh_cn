# JavaScript API
## Web3 JavaScript app API
要使您的应用程序在Ethereum上工作，您可以使用[web3.js](https://github.com/ethereum/web3.js)库提供的`web3`对象。 在引擎盖下，它通过RPC调用与一个本地节点进行通信。 web3.js与任何展示RPC层的Ethereum节点一起使用。  
`web3`包含`eth`对象 - `web3.eth`（特别是Ethereum区块链接口）和`shh`对象 - `web3.shh`（用于Whisper交互）。 随着时间的推移，我们将介绍每个其他web3协议对象，这里可以找[实例](https://github.com/ethereum/web3.js/tree/master/example)。  
如果你想看一些更复杂的例子，使用`web3.js`，看看这些[useful app patterns](https://github.com/ethereum/wiki/wiki/Useful-%C3%90app-Patterns)
### Getting Started
* Adding web3
* Using Callbacks
* Batch requests
* A note on big numbers in web3.js
* [-> API Reference](https://github.com/ethereum/wiki/wiki/JavaScript-API#web3js-api-reference)
#### Adding web3
首先，您需要将web3.js导入到您的项目中。 这可以使用以下方法完成：
* npm: `npm install web3`
* bower: `bower install web3`
* meteor: `meteor add ethereum:web3`
* vanilla: `link the dist./web3.min.js`
那么你需要创建一个web3实例，设置一个提供者。为了确保在mist中你不要覆盖已经设置的提供程序，请先检查web3是否可用：
```
if (typeof web3 !== 'undefined') {
  web3 = new Web3(web3.currentProvider);
} else {
  // set the provider you want from Web3.providers
  web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));
}
```
之后，您可以使用`web3`对象的[API](https://github.com/ethereum/wiki/wiki/web3js-api-reference)
#### Using callbacks
由于此API旨在与本地RPC节点配合使用，因此其所有功能默认使用同步HTTP请求。
如果要进行异步请求，可以将可选回调作为最后一个参数传递给大多数函数。 所有回调都使用[错误第一回调 error first callback](http://fredkschott.com/post/2014/03/understanding-error-first-callbacks-in-node-js/)样式：
```
web3.eth.getBlock(48, function(error, result){
    if(!error)
        console.log(result)
    else
        console.error(error);
})
```
#### Batch requests批量请求
批量请求允许排队请求并立即处理它们。
注意：批量请求不会更快！实际上，在某些情况下异步处理请求会更快。批量请求主要用于确保请求的串行处理。
```
var batch = web3.createBatch();
batch.add(web3.eth.getBalance.request('0x0000000000000000000000000000000000000000', 'latest', callback));
batch.add(web3.eth.contract(abi).at(address).balance.request(address, callback2));
batch.execute();
```
#### A note on big numbers in web3.js 关于`web3.js`中大数字的注释
所有的数字都被获取为一个BigNumber对象，因为JavaScript无法正确处理大数字。 看下面的例子：
```
"101010100324325345346456456456456456456"
// "101010100324325345346456456456456456456"
101010100324325345346456456456456456456
// 1.0101010032432535e+38
```
`web3.js`依赖于[BigNumber](https://github.com/MikeMcl/bignumber.js/)库并自动添加。
```
var balance = new BigNumber('131242344353464564564574574567456');
// or var balance = web3.eth.getBalance(someAddress);

balance.plus(21).toString(10); // toString(10) converts it to a number string
// "131242344353464564564574574567477"
```
下一个示例将不起作用，因为我们有超过20个浮点，因此建议始终坚持用`wei`表示您的余额，并在向用户呈现时将其转换为其他单位：
```
var balance = new BigNumber('13124.234435346456466666457455567456');

balance.plus(21).toString(10); // toString(10) converts it to a number string, but can only show max 20 floating points 
// "13145.23443534645646666646" // you number would be cut after the 20 floating point
```
### [Web3.js API Reference](https://github.com/ethereum/wiki/wiki/JavaScript-API#web3js-api-reference)