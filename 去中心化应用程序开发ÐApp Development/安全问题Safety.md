# 安全问题Safety
# 以太坊安全技术与技巧
**鼓励社区更新维基百科：随着更多的贡献的增加，它变得更加完整**  
目前，官方这篇维基是过时的，以下内容翻译自其来源["Smart Contract Best Practices](https://github.com/ConsenSys/smart-contract-best-practices)
  
This document is designed to provide a starting security baseline for intermediate Solidity programmers. It additionally includes security philosophies; bug bounty program guidelines; documentation and procedures; and tools.

Pull requests are very welcome, from small fixes, to sections, and if you've written an article or blog post, please add it to the bibliography. See our [Contribution Guidelines](https://github.com/ConsenSys/smart-contract-best-practices/blob/master/CONTRIBUTING.md).  
##### Additional Requested Content
We especially welcome content in the following areas:
* Testing Solidity code (structure, frameworks, common test idioms)
* Software engineering practices for smart contracts and/or blockchain-based programming
## 基本哲学General Philosophy
以太坊和复杂的区块链程序是新颖且高度实验性的。因此，您应该明白随着发现了新的错误和安全风险会导致安全环境的不断变化，并开发全新的最佳实践。 所以作为一个智能合约开发人员，遵循本文档中的安全措施只是安全工作的开始。
智能合约编程需要一个不同于您原来习惯的工程思维方式。失败的成本可能很高，变化可能很困难，使其在某些方面更像硬件编程或金融服务编程，而不是网络或移动开发。 因此，防范已知的漏洞是不够的。 相反，您将需要学习一种新的开发理念：
* **做好失败的准备**，任何不挑剔的合约都有错误，因此，您的代码必须能够正确地响应错误和漏洞
	* 当运行出错时暂停合约（断路器）
	* 管理风险资金数量（限速、最大使用量）
	* 设置一个有效的升级路径来修正和改进
* **小心推出**，在完整的生产版本之前捕获bug总是更好
	* 彻底测试合同，并在发现新的攻击媒介时添加测试
	* 提供从alpha测试版本开始的错误奖励
	* 逐步推出，每个阶段的使用和测试不断增加
* **保持合同简单**，复杂性会增加错误的可能性
	* 确保合约逻辑简单
	* 模块化代码以保持合约和函数足够小
	* 在可能的情况下使用已经编写的工具或代码（例如，不要roll自己的随机数生成器）
	* 尽可能地表述清晰
	* 仅在你的系统需要去中心化的部分使用区块链
* **保持最新**，使用下一节中列出的资源来跟踪新的安全措施
	* 检查您的合同是否发现了任何新的错误
	* 尽快升级到任何工具或库的最新版本
	* 采用新的适用安全技术
* **注意区块链属性**，尽管您的大部分程序设计经验与以太坊编程相似，但还是存在一些陷阱需要注意
	* 外部合约调用要非常小心，这一操作可能会执行恶意代码并改变控制流程
	* 要知道您的公开函数是公开的，可能会被恶意调用，任何人也可以看到您的私有变量数据
	* 注意gas成本和区块链gas限制
### 基本权衡(tradeoffs)：简单性与复杂性案例Fundamental Tradeoffs: Simplicity versus Complexity cases
在评估智能合约喜用的结构和安全性时，需要考虑多重基本权衡(tradeoffs)。对所有智能合约系统的总体建议是确保这些基本权衡(tradeoffs)的适当平衡。从软件工程思想出发，理想的智能合约系统是模块化的，要重用代码而不是重写代码，并支持可升级组件。遵照安全的架构思想一个完美的智能合约系统可能会分享这种心态，特别是那种更加复杂的智能合约系统。然而，安全和软件工程最佳做法可能不一致是一个重要例外。在每种情况下，通过根据合同系统维度确定最佳的属性组合来获得适当的平衡，例如：
* 固定性与可升级
* 整一性与模块化
* 重写与复用
#### 固定性与可升级Rigid versus Upgradeable
虽然包括这一个在内的多种资源强调可延展性特征，例如可杀死性，可升级或可修改的模式，但可延展性和安全性之间存在着基本的权衡(tradeoffs)。
按照定义，延展性模式增加了复杂性和潜在的攻击面。在智能合约系统对预定义的有限时间段执行非常有限的一组函数（例如无治理的有限时间帧代币销售合约系统）的情况下，简单性对于复杂性尤其有效。
#### 整一性与模块化Monolithic versus Modular
一个独立独用的合约保持所有信息在本地可识别且可读。虽然很少有作为巨型存在对于数据和流程的极端本地性是有争议的智能合约系统被高度重视，例如，在优化代码审查效率的情况下。就像这里考虑的其他权衡(tradeoffs)一样，安全性最佳实践从简单的短期合约的软件工程最佳实践中走出来，趋向于在更复杂的永久合约系统的情况下软件工程最佳实践。
#### 重写与复用
从软件工程角度看，智能合约系统希望在合理的情况下最大程度地利用重用。 在Solidity中重用合同代码有很多方法。使用您所拥有的经过验证的以前部署的合约通常是实现代码重用的最安全的方式。在自有的以前部署的合同不可用的情况下，经常依赖重写。 诸如[Live Libs](https://github.com/ConsenSys/live-libs)和[Zeppelin Solidity](https://github.com/OpenZeppelin/zeppelin-solidity)等努力寻求提供模式，使安全代码可以重复使用而不用重写。任何合约安全性分析都必须包括以前没有建立与目标智能合约系统中处于风险中的资金相称的信任级别的重用代码。
## 安全通知Security Notifications
这是一个通常会突出显示Ethereum或Solidity中发现的漏洞的资源列表。安全通知的官方来源是Ethereum Blog，但在许多情况下，漏洞将在之前的其他位置披露和讨论。
* [Ethereum Blog](https://blog.ethereum.org/)：以太坊官方博客
	* [Ethereum Blog - Security only](https://blog.ethereum.org/category/security/)：所有标记为Security的博客文章
* [Ethereum Gitter chat rooms](https://gitter.im/ethereum/home)
	* [Solidity](https://gitter.im/ethereum/solidity)
	* [Go-Ethereum](https://gitter.im/ethereum/go-ethereum)
	* [CPP-Ethereum](https://gitter.im/ethereum/cpp-ethereum)
	* [Research](https://gitter.im/ethereum/research)
* [Reddit](https://www.reddit.com/r/ethereum/)
* [Network Stats](https://ethstats.net/)
强烈建议您定期阅读所有这些来源，因为他们注意到的漏洞可能会影响您的合约。
另外，这里列出了可以写安全性的Ethereum核心开发人员列表，并从社区中查看更多的参考书目。
* Vitalik Buterin: [Twitter](https://twitter.com/vitalikbuterin), [Github](https://github.com/vbuterin), [Reddit](https://www.reddit.com/user/vbuterin), [Ethereum Blog](https://blog.ethereum.org/author/vitalik-buterin/)
* Dr. Christian Reitwiessner: [Twitter](https://twitter.com/ethchris), [Github](https://github.com/chriseth), [Ethereum Blog](https://blog.ethereum.org/author/christian_r/)
* Dr. Gavin Wood: [Twitter](https://twitter.com/gavofyork), [Blog](http://gavwood.com/), [Github](https://github.com/gavofyork)
* Vlad Zamfir: [Twitter](https://twitter.com/vladzamfir), [Github](https://github.com/vladzamfir), [Ethereum Blog](https://blog.ethereum.org/author/vlad/)
除了核心开发人员之外，参与更广泛的与区块链相关的安全社区至关重要，因为安全披露或观察将通过各种各样的方式进行。
## 关于Solidity智能合约安全的建议Recommendations for Smart Contract Security in Solidity
### 外部调用External Calls
#### 尽可能的避免外部调用Avoid external calls when possible
对不可信合约的调用可能会引起几个意外的风险或错误。外部调用可能在该合约或其所依赖的任何其他合约中执行恶意代码。 因此，每个外部调用都应被视为潜在的安全隐患，如有可能，将其删除。当无法删除外部调用时，请使用本节其余部分中的建议来尽量减少危险。
#### 注意`send()`、`transfer()`、`call.value()()`之间的权衡(tradeoffs)
当发送以太币时，请注意函数`someAddress.send()`、`someAddress.transfer()`、`someAddress.call.value()()`之间的权衡(tradeoffs)
* `x.transfer(y);`等价于`require(x.send(y));` `send`是`transfer`的低级状态，建议尽可能使用`transfer`
* `someAddress.send()`和`someAddress.transfer()`被认为是安全的，以防止重入reentrancy。虽然这些方法仍然触发代码执行，但被调用合约只收到2300 gas，这些gas目前只够记录日志。
* `someAddress.call.value()()`将发送制定数量的ether并且触发代码执行，代码可以使用所有可用的gas，使得这种类型的资金转移对重入reentrancy不安全。
使用`send()`或`transfer()`可以防止重入reentrancy，但是这样做的代价是与所有回退函数需要2300 gas以上的合约不兼容。
尝试平衡这种权衡(tradeoffs)的一种模式是实现推送和拉动(push and pull)机制，对push组件使用`send()`或`transfer()`，并为pull组件调用`call.value()()`  
值得指出的是，独自使用`send()`或`transfer()`进行价值转移(资金)本身并不能使合约对于重入reentrancy安全，而只能使这些指定的价值转移操作对于重入reentrancy安全。
#### 处理外部调用中的错误Handle errors in external calls
Solidity提供适用于原始地址的低级调用方法`address.call()`、`address.callcode()`、`address.delegatecall()`和`address.send`。这些低级方法永远不会抛出异常，但如果调用遇到异常，则返回`false`。 另一方面，contract calls合约调用（比如`ExternalContract.doSomething()`）将自动传播一个异常抛出（例如，如果`doSomething()`抛出异常，`ExternalContract.doSomething()`也将`throw`）。如果选择使用低级调用方法，请确保通过检查返回值来处理调用失败的可能性。
```
// bad
someAddress.send(55);
someAddress.call.value(55)(); // this is doubly dangerous, as it will forward all remaining gas and doesn't check for result
someAddress.call.value(100)(bytes4(sha3("deposit()"))); // if deposit throws an exception, the raw call() will only return false and transaction will NOT be reverted

// good
if(!someAddress.send(55)) {
    // Some failure code
}

ExternalContract(someAddress).deposit.value(100);
```
#### 外部调用后不要做控制流假设Don't make control flow assumptions after external calls
无论使用原始呼叫或合约呼叫，一定要意识到如果ExternalContract不可信任，恶意代码就会执行。即使ExternalContract不是恶意的，恶意代码也可以通过它调用的任何合约来执行。一个特别的危险是恶意代码可能劫持控制流，导致竞争条件race conditions。 （有关这个问题的更全面的讨论，请参阅竞争条件章节Race Conditions）。
#### 外部调用中重拉轻推模式Favor pull over push for external calls
外部调用可能会意外或故意地失败。为了最小化由此类故障引起的损坏，通常最好将每个外部调用隔离成它自己的交易，由调用的接收方发起。 这对于付款特别有用，最好是让用户自己提取资金，而不是自动向他们推送资金（这也减少了气体限制问题problems with the gas limit发生的机会），避免在单个交易中组合多个`send()`调用。
```
// bad
contract auction {
    address highestBidder;
    uint highestBid;

    function bid() payable {
        require(msg.value >= highestBid);

        if (highestBidder != 0) {
            require(highestBidder.send(highestBid)) // if this call consistently fails, no one else can bid
        }

       highestBidder = msg.sender;
       highestBid = msg.value;
    }
}

// good
contract auction {
    address highestBidder;
    uint highestBid;
    mapping(address => uint) refunds;

    function bid() payable external {
        require(msg.value >= highestBid);

        if (highestBidder != 0) {
            refunds[highestBidder] += highestBid; // record the refund that this user can claim
        }

        highestBidder = msg.sender;
        highestBid = msg.value;
    }

    function withdrawRefund() external {
        uint refund = refunds[msg.sender];
        refunds[msg.sender] = 0;
        require(msg.sender.send(refund)); // revert state if send fails
        }
    }
}
```
#### 标记不可信合约Mark untrusted contracts
当与外部合约进行交互时，通过以下方式命名你的变量，方法和合约接口来表明与之进行交互可能是不安全的。这适用于您自己的调用外部合约的函数。
```
// bad
Bank.withdraw(100); // Unclear whether trusted or untrusted

function makeWithdrawal(uint amount) { // Isn't clear that this function is potentially unsafe
    Bank.withdraw(amount);
}

// good
UntrustedBank.withdraw(100); // untrusted external call
TrustedBank.withdraw(100); // external but trusted bank contract maintained by XYZ Corp

function makeUntrustedWithdrawal(uint amount) {
    UntrustedBank.withdraw(amount);
}
```
### 用`assert()`强制执行不变量Enforce invariants with `assert()`
当断言失败（例如不变属性更改）时，断言保护将触发。 例如，在代币发行合约中的代币与以太币发行比率可能是固定的。 您可以随时使用`assert()`来验证是否是这种情况。 断言通常应与其他技术结合使用，例如暂停合约和允许升级。（否则你可能会被卡住，总是有失败的断言。）
```
contract Token {
    mapping(address => uint) public balanceOf;
    uint public totalSupply;

    function deposit() public payable {
        balanceOf[msg.sender] += msg.value;
        totalSupply += msg.value;
        assert(this.balance >= totalSupply);
    }
}
```
请注意，断言不是严格的余额数量，因为合约不需要通过`deposit()`函数就可以可以强制发送！