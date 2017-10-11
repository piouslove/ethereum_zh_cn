# 网址提示协议[URL Hint Protocol](https://github.com/ethereum/wiki/wiki/URL-Hint-Protocol)
在地址`0x42`上由一个智能合约`Registry`（有关拍卖），存在一个`Registry`实现的合约接口`register`  
由长度为32的字符串查找到的`register`中的条目有几个字段，包括`content`和`register`  
`content`是当您在Mist / AZ中输入此名称时加载的内容哈希值  
`register`是递归查询的次区域合同（一些合同实施注册表）的地址
#### 例子Example
* `eth://gavofyork`导致主注册表（0x42）被查询条目`gavofyork`，此条目的`content`字段（zip文件或清单的SHA3哈希）用于显示内容（有关详细信息，请参阅内容部分）
## URL组成
所有URL都可以开始`eth：//`，并且是一些组件，每个组件都用一个`/`隔开，字段`register`用于执行递归查找。  
因此，`gavofyork/tools/site/contact`包含以下四个组件：`gavofyork`、`tools`、`site`、`contact`  
任何时候`.`都可用于在`/`以相反顺序指定组件  
所以上述地址的非规范形式是：
* tools.gavofyork/site/contact
* site.tools.gavofyork/contact
* contact.site.tools.gavofyork
注意：任何时候最左侧`/`后处理的组件与其他字母数字字符不同。
#### 例子Example
`eth://gavofyork/site`导致主`Registry`（`0x42`）被查询条目`gavofyork`。此条目的`register`字段（`register`实现的合约的地址）然后被查询作为入口站点。此条目的`register`字段用于显示内容。
## 内容Content
通常，Swarm将用于确定内容哈希中的内容。
对于1.0，这是不可能的，所以我们将回到使用HTTP分发。 为了实现这一点，我们将包括一个或多个URLHint智能合约，其中提供允许下载特定内容散列的URL的提示。 在dapp-bin中可以找到该合约。
下载的内容应该以多种方式进行处理（散列）来发现内容是什么。 可能的方法包括base64编码，十六进制编码和原始内容，以及所需的任何内容裁剪（例如，HTML页面应该具有除去body标签之外的所有内容）。
这将由dapp /内容上传者来更新URLHint条目。
URLHint合约的地址将被临时指定，用户将能够在其浏览器中输入其他地址。