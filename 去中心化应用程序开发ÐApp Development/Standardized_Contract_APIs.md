# Standardized_Contract_APIs
注意：代币API目前被作为ERC（Ethereme请求发表评论）讨论，可能已过期：https：//github.com/ethereum/EIPs/issues/20  
  
虽然以太坊绝对允许开发人员创建任何类型的应用程序，而不限制特定的功能类型，并且以“缺乏功能”为傲，但仍然需要对某些非常常见的用例进行标准化，以便允许用户和应用程序更容易交互。这包括发送货币单位，登记名称，在交易所上做出交易，以及其他类似的功能。 标准通常由几种方法的一组函数签名组成，例如`send`, `register`, `delete`，提供一组参数及其格式在 [Ethereum contract ABI](https://github.com/ethereum/wiki/wiki/Ethereum-Contract-ABI) 语言中。  
下面描述的标准在[这里](https://github.com/ethereum/dapp-bin/tree/master/standardized_contract_apis)提供了示例实现。  
所有函数名称均用小[骆驼拼写法](http://www.ruanyifeng.com/blog/2007/06/camelcase.html) lower camelCase（例如`sendCoin`）中，所有事件Event名称均用大骆驼拼写法 upper CamelCase（例如`CoinTransfer`）中。输入变量是下划线作前缀的小骆驼拼写法lower camelCase（例如`_offerId`），输出变量_r用于纯粹的getter（即常量）函数，`_success`（总是布尔值）用于表示成功或失败，以及其他值（例如`_maxValue`）用于执行动作但需要返回值作为标识符的方法。 当通用时，地址使用`_address`引用，如果存在更具体的描述（例如`_from`，`_to`）则反之。
## Transferable Fungibles (see [ERC 20](https://github.com/ethereum/EIPs/issues/20) for the latest)
也称为代币tokens，coins和子货币sub-currencies。
## TF Registries (see [ERC 22](https://github.com/ethereum/EIPs/issues/22) for the latest)
代币注册表包含有关代币的信息。 至少有一个全局注册表（尽管其他人可以创建更像样的全局注册表），您可以向其添加代币。 将代币添加到它将提升无论是否使用GUI客户端的用户的用户体验。
## Registries
注册管理机构（如域名系统）具有以下API：
### Methods
#### reserve
```
reserve(string _name) returns (bool _success)
```
如果尚未保留则保留名称并将其所有者设置为您。
#### owner
```
owner(string _name) constant returns (address _r)
```
获取特定名称的所有者。
#### transfer
```
transfer(string _name, address _newOwner)
```
转让一个名称的所有权。
#### setAddr
```
setAddr(string _name, address _address)
```
设置与名称相关联的主地址（类似于传统DNS中的A记录）。
#### addr
```
addr(string _name) constant returns (address _r)
```
获取与名称相关联的主地址。
#### setContent
```
setContent(string _name, bytes32 _content)
```
如果您是名称的所有者，请设置其相关内容。
#### content
```
content(string _name) constant returns (bytes32 _r)
```
获取与名称相关联的内容。
#### setSubRegistrar
```
setSubRegistrar(string _name, address _subRegistrar)
```
Records the name as referring to a sub-registrar at the given address.
#### subRegistrar
```
subRegistrar(string _name) constant returns (address _r)
```
获取与给定名称相关联的子注册商。
#### disown
```
disown(string _name)
```
放弃对您当前控制的名称的控制权。
### Events
#### Changed
```
event Changed(string name, bytes32 indexed __hash_name)
```
Triggered when changed to a domain happen.
## Data feeds
数据馈送标准是模板标准，即在下面的描述中，应该可以用任何所需的数据类型替换`<t>`，例如 `uint256`，`bytes32`，`address`，`real192x64`
### Methods
#### get
```
get(bytes32 _key) returns (<t> _r)
```
获取与键关联的值。
#### set
```
set(bytes32 _key, <t> _value)
```
如果您是所有者，请设置与键关联的值。
#### setFee
```
setFee(uint256 _fee)
```
Sets the fee.费用
#### setFeeCurrency
```
setFeeCurrency(address _feeCurrency)
```
设定支付费用的货币  
  
后两种方法是可选的; 还要注意，费用可能以以太币或子货币、代币收费; 如果合约用以太币收费，那么`setFeeCurrency`方法是不需要的。
## Forwarding contracts (eg. multisig)
根据每个合约的授权政策，转送合约的工作可能会有很大的不同。 但是，有一些标准的工作流应尽可能的使用：
### Methods
#### execute
```
execute(address _to, uint _value, bytes _data) returns (bytes32 _id)
```
创建一个包含所需收件人，值和数据的消息并返回交易的“待处理ID”。
#### confirm
```
confirm(bytes32 _id) returns (bool _success)
```
使用您的帐户确认具有特定ID的挂起消息; 返回成功或失败。如果有足够的确认则发送消息。  
  
访问权限方案可以是任何形式，例如多重签名、任意的CNF布尔公式，这取决于交易的价值或内容的方案等。