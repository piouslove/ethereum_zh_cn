# [交易客户端地址编码协议ICAP: Inter exchange Client Address Protocol](https://github.com/ethereum/wiki/wiki/ICAP:-Inter-exchange-Client-Address-Protocol)
第三方账户，特别是交易所账户之间的资金转移给用户造成了相当大的负担，而且由于客户账户的存款方式被识别，所以容易出错。 这个问题是由现有的银行业通过一个称为IBAN的通用编码来解决的。 该代码合并了机构和客户端帐户以及错误检测机制，实际上消除了微不足道的错误并为用户提供了相当的便利。 不幸的是，这是一个严格监管和集中的服务，只有大型，成熟的机构才能使用。 现行协议ICAP可能被视为一个适用于任何在Ethereum系统上设有资金的机构的IBAN去中心化版本。
## IBAN
有关IBAN系统的良好概述，请参阅[维基百科的IBAN文章](https://en.wikipedia.org/wiki/International_Bank_Account_Number)。 IBAN编码最多包含34个不区分大小写的字母数字字符。 它包含三条信息：  
* 国家编码The country code; a top-level identifier for the context of the following (ISO 3166-1 alpha-2);
* 错误检测码The error-detection code; uses the mod-97-10 checksumming protocol (ISO/IEC 7064:2003);
* 基本银行账号The basic bank account number (BBAN); an identifier of the institution, branch and client account, whose composition is dependent on the aforementioned country.
对于英国来说，BBAN由以下组成：
* Institution identifier, 4-character alphabetical, e.g. MIDL (ironically) represents HSBC bank.
* Sort-code (branch identifier within the institution), a 6-digit decimal number, e.g. 402702 would be the Lancaster branch of HSBC.
* Account number (client identifier within the branch), an 8-digit decimal number.
## Proposed Design建议设计
采用一种新的IBAN国家编码：`XE`，来自Ethereum前缀`E`以及额外的`X`，被用于非管辖的货币一样，例如XRP、XCP  
该编码将有三种BBAN可能性; 直接，基本和间接
### 直接Direct
这个编码的BBAN将直接是30个字符，并将包含一个字段：  
* 帐户标识符，30个字符的字母数字（小于155位）。这将被解释为表示160位以太坊地址最低有效位的base-36编码的[大端](http://blog.csdn.net/andkylee/article/details/5361078)整数。因此，这些Ethereum地址通常以零字节开始
例如`XE7338O073KYGTWWZN0F2WZ0R8PX5ZPPZS`对应于地址`00c5496aee77c1ba1f0854206a26dda82a81d6d8`
### 基本Basic
与直接编码相同，但代码为31个字符（使其不符合IBAN），并且构成相同的单个字段：
* 帐户标识符，31个字符的字母数字（小于161位）。这将被解释为表示160位以太坊地址最低有效位的base-36编码的[大端](http://blog.csdn.net/andkylee/article/details/5361078)整数
### 间接Indirect
间接编码的BBAN为16个字符，并将包含三个字段：
* 资产标识符Asset identifier，3个字符的字母数字（<16位）;
* 机构标识符Institution identifier，4个字符的字母数字（<21位）;
* 机构客户端标识符Institution client identifier，9个字符的字母数字（<47位）;
包括四个初始字符，这导致最终的客户端地址长度为20个字符，格式如下：  
`XE81ETHXREGGAVOFYORK`  
包括：  
`XE`：以太坊的国家代码;
`81`：校验码;
`ETH`：客户帐户中的资产标识符，此例中，“ETH”是唯一有效的资产标识符，因为Ethereum的基础注册合约仅支持该资产;
`XREG`:帐户的机构代码，此例中为Ethereum的基本注册合约;
`GAVOFYORK`:机构内的客户端标识符,此例中，直接付款，无任何主要地址的附加数据与Ethereum基本注册合约中的“GAVOFYORK”名称相关联;
## 注意
以`X`开头的机构代码保留供系统使用。
## 其他形式
### URI
通常URI格式可以通过URI方案名称iban加冒号字符，后跟20个字符的字母数字标识符，因此对于上述示例，我们将使用：  
`iban:XE81ETHXREGGAVOFYORK`
### QR Code二维码
可以使用标准QR编码直接从URI生成QR码。 例如，上面的示例`iban：XE81ETHXREGGAVOFYORK`将具有相应的QR码：  
QR code for iban:XE81ETHXREGGAVOFYORK
## 交易事务语义
指定了通过三种路由协议间接资产转移的机制，所有这些机制都是特定于以太坊域（XE的国家代码）的。 一种是直接向货币转移到包含的地址（“直接”），另一种是通过客户端ID的注册表查找系统找到系统地址的客户端，由资产类别ETH表示，而最后一种是转移到 具有指定客户端的相关数据的中介，以资产类别XET（后两者为“间接”）表示。
### 直接Direct
如果IBAN代码是34个字符，它是一个直接地址; 当base-36编码准确地给出IBAN代码的数据段（最后30个字符）时，直接转移到地址。
### 间接ETh资产：简单转账Indirect ETH Asset: Simple transfers
在以太坊国家代码（XE）的ETH资产代码中，即只要代码以XE**ETH（其中星号为有效的校验码）开始，那么我们可以将所需的交易定义为由 以机构代码表示的对注册合约的调用。 对于不以X开头的机构，这对应于与Ethereum标准名称相关联的主要地址：  
[机构代码] / [客户识别码]  
Ethereum标准名称只是在Ethereum标准接口文档中指定的正常的分层查找机制。
我们将注册合约定义为满足Ethereum标准接口文档中指定的注册表接口的合同。  
TODO：用于指定传输的JS代码。
### 间接XET资产：机构转移Indirect XET Asset: Institution transfers
对于以太坊国家代码中的XET资产代码（即代码以XE**XET开始），那么我们可以通过查找Ethereum iban注册表合约来获得必须进行的交易。 对于一个给定的机构，本合同规定了两个值：存款调用签名哈希和机构的以太坊地址。
目前，只有一个这样的存款调用被定义为：  
`function deposit(uint64 clientAccount)`  
其签名散列为`0x13765838`。转让资产的交易应按照以上所述以机构的以太坊地址调用`deposit`方法为形式，客户账户以数字机构标识符的base-36编码大端字母数字为值，字面上使用字符`0`到`9`的值，然后将`A`（或`a`）评估为`10`，将`B`（或`b`）评估为`11`等等。  
TODO：用于指定传输的JS代码。