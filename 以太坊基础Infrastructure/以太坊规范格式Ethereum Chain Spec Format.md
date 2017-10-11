# 以太坊区块链规范格式Ethereum Chain Spec Format
本文介绍类以太坊的区块链的格式。它继承自genesis.json的格式，但包括更改和配置共识算法的参数，指定基础架构信息，指定引导节点以及指定任何内置智能合约及其数字货币成本。
## 格式
它是一个具有六个顶级键的对象：  
* `name`:指定区块链名字的字符串，例如"Frontier/Homestead", "Morden", "Olympic"  
* `forkName`:一个可选的字符串值，指定一个子标识符，以防两个不同的区块链具有相同的生成块  
* `engine`:指定共识算法引擎的枚举值，例如"Ethash", "Null"  
* `params`:一个指定共识算法引擎各种属性的对象，允许配置  
* `genesis`:一个指定创世块标题的对象  
* `nodes`:一个字符串数组，每个成员都是一个经过特殊编码的节点地址  
* `accounts`:一个指定创世块中帐户数据的对象，包括内置的智能合约和内容
### 子格式`engine`
有两种有效的算法引擎`engine`，`Ethash` 和 `Null`  
* `Null`:无操作引擎  
* `Ethash`:以太坊使用的共识算法引擎  
	* `params`:引擎特定参数
		* `minimumDifficulty`:指定区块可能具有的最小难度的整数
		* `gasLimitBoundDivisor`:指定符合黄皮书规定的整数
		* `difficultyBoundDivisor`:指定符合黄皮书规定的整数
		* `durationLimit`:指定难度增幅边界点的整数
		* `blockReward`:指定挖矿创建区块奖励的整数
		* `registrar`:指定这条区块链上注册智能合约的40位、`0x`前缀地址
### 子格式`params`
不同的共识引擎可能允许`params`对象中的不同键，但是所有键都有一些共同之处：
* `accountStartNonce`:整数，指定所有新创建的帐户应具有nonce值
* `frontierCompatibilityModeLimit`:指定Frontier兼容模式结束并且Homestead模式开始的区块编号的整数
* `maximumExtraDataSize`:指定标题的extra_data字段最大容量的整数（以字节为单位）
* `minGasLimit`:指定区块最小gas值的整数
* `networkID`:指定这条区块链在网络上的索引
注意：所有的整数都是`0x`前缀、十六进制的字符串
