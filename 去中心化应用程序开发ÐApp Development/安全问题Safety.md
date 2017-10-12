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
### 基本权衡：简单性与复杂性案例Fundamental Tradeoffs: Simplicity versus Complexity cases
在评估智能合约喜用的结构和安全性时，需要考虑多重基本权衡。对所有智能合约系统的总体建议是确保这些基本权衡的适当平衡。从软件工程思想出发，理想的智能合约系统是模块化的，要重用代码而不是重写代码，并支持可升级组件。遵照安全的架构思想一个完美的智能合约系统可能会分享这种心态，特别是那种更加复杂的智能合约系统。然而，安全和软件工程最佳做法可能不一致是一个重要例外。在每种情况下，通过根据合同系统维度确定最佳的属性组合来获得适当的平衡，例如：
* 固定性与可升级
* 整一性与模块化
* 重写与复用
#### 固定性与可升级Rigid versus Upgradeable
虽然包括这一个在内的多种资源强调可延展性特征，例如可杀死性，可升级或可修改的模式，但可延展性和安全性之间存在着基本的权衡。
按照定义，延展性模式增加了复杂性和潜在的攻击面。在智能合约系统对预定义的有限时间段执行非常有限的一组函数（例如无治理的有限时间帧代币销售合约系统）的情况下，简单性对于复杂性尤其有效。
#### 整一性与模块化Monolithic versus Modular
一个独立独用的合约保持所有信息在本地可识别且可读。虽然很少有作为巨型存在对于数据和流程的极端本地性是有争议的智能合约系统被高度重视，例如，在优化代码审查效率的情况下。就像这里考虑的其他权衡一样，安全性最佳实践从简单的短期合约的软件工程最佳实践中走出来，趋向于在更复杂的永久合约系统的情况下软件工程最佳实践。
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