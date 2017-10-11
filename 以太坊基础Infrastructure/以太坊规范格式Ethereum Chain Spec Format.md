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
### 子格式`genesis`
`genesis`中的`key`指定了区块链的创世块中的相关字段。 所有字段都是十六进制，`0x`前缀的字符串。 所需要的字段有：
* `seal`
* difficulty`
* `author`
* `timestamp`
* `parentHash`
* `extraData`
* `gasLimit`
### 子格式`accounts`
`accounts`对象将地址(不以`0x`开头的40位字符串)映射成对象，每个对象有许多允许的键：
* `balance`:以`wei`为单位指定创世时账户余额的整数
* `nonce`:指定创世时帐户nonce值的整数
* `code`:指定创世时帐户中以`0x`为前缀的十六进制代码
* `storage`:创世时帐户`storage`中存储的十六进制编码整数映射的对象
* `builtin`:替代`code`，用于指定帐户代码是本地实现的，值是一个具有更多字段的对象：
	* `name`:内置代码执行的名字字符串，例如`identity`、`ecrecover`
	* `pricing`:指定调用此智能合约花费的枚举值
		* `linear`:指定调用此合约线性花费的值，值是一个具有两个字段的对象：
		`base`是以`wei`为单位且必须要支付的基础花费；`word`是每个输入单字的花费，向上舍入
## 例子Example
这是 Morden ECS JSON 文件
```
{
	"name": "Morden",
	"engine": {
		"Ethash": {
			"params": {
				"tieBreakingGas": false,
				"gasLimitBoundDivisor": "0x0400",
				"minimumDifficulty": "0x020000",
				"difficultyBoundDivisor": "0x0800",
				"durationLimit": "0x0d",
				"blockReward": "0x4563918244F40000",
				"registrar" : "0xc6d9d2cd449a754c494264e1809c50e34d64562b"
			}
		}
	},
	"params": {
		"accountStartNonce": "0x0100000",
		"frontierCompatibilityModeLimit": "0x789b0",
		"maximumExtraDataSize": "0x20",
		"tieBreakingGas": false,
		"minGasLimit": "0x1388",
		"networkID" : "0x2"
	},
	"genesis": {
		"seal": {
			"ethereum": {
				"nonce": "0x0000000000000042",
				"mixHash": "0x0000000000000000000000000000000000000000000000000000000000000000"
			}
		},
		"difficulty": "0x20000",
		"author": "0x0000000000000000000000000000000000000000",
		"timestamp": "0x00",
		"parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
		"extraData": "0x",
		"gasLimit": "0x2fefd8"
	},
	"nodes": [
		"enode://b1217cbaa440e35ed471157123fe468e19e8b5ad5bedb4b1fdbcbdab6fb2f5ed3e95dd9c24a22a79fdb2352204cea207df27d92bfd21bfd41545e8b16f637499@104.44.138.37:30303"
	],
	"accounts": {
		"0000000000000000000000000000000000000001": { "balance": "1", "nonce": "1048576", "builtin": { "name": "ecrecover", "pricing": { "linear": { "base": 3000, "word": 0 } } } },
		"0000000000000000000000000000000000000002": { "balance": "1", "nonce": "1048576", "builtin": { "name": "sha256", "pricing": { "linear": { "base": 60, "word": 12 } } } },
		"0000000000000000000000000000000000000003": { "balance": "1", "nonce": "1048576", "builtin": { "name": "ripemd160", "pricing": { "linear": { "base": 600, "word": 120 } } } },
		"0000000000000000000000000000000000000004": { "balance": "1", "nonce": "1048576", "builtin": { "name": "identity", "pricing": { "linear": { "base": 15, "word": 3 } } } },
		"102e61f5d8f9bc71d0ad4a084df4e65e05ce0e1c": { "balance": "1606938044258990275541962092341162602522202993782792835301376", "nonce": "1048576" }
	}
}
```
注意：内置的帐户可以使用Solidity语言。 包含在链定义文件中的没有它们的运行可能会导致意外的行为