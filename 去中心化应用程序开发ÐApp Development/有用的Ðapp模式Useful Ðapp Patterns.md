# Useful Ðapp Patterns
下面的页面是一些有用的模式集合，它们可以用于，比如可靠地与区块链交互。
示例模式可能会改变，所以不要完全依赖它们。
## 示例
* 3 ways of instantiating web3实例化web3的三种方式:  
https://gist.github.com/frozeman/fbc7465d0b0e6c1c4c23  

* Contract deployment by code合约部署代码:
(以前使用 `web3.contract(abiArray).new({}, function(e, res){...})`)   
https://gist.github.com/frozeman/655a9325a93ac198416e  

* Test a contract transaction with a call before actually sending在实际发送之前，先通过一个调用测试合约交易:   
https://gist.github.com/ethers/2d8dfaaf7f7a2a9e4eaa