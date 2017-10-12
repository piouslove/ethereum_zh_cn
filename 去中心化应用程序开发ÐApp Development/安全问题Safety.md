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
** 当运行出错时暂停合约（断路器）
** 管理风险资金数量（限速、最大使用量）
** 设置一个有效的升级路径来修正和改进
* **小心推出**，在完整的生产版本之前捕获bug总是更好
** 彻底测试合同，并在发现新的攻击媒介时添加测试
** 提供从alpha测试版本开始的错误奖励
** 逐步推出，每个阶段的使用和测试不断增加
* **保持合同简单**，复杂性会增加错误的可能性