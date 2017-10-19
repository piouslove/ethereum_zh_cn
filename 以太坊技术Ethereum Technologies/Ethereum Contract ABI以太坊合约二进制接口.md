# **Ethereum Contract ABI**
此规范现在作为[Solidity文档](https://solidity.readthedocs.io/en/develop/abi-spec.html)的一部分进行维护。
# 函数Functions
## Basic design
我们假设应用程序二进制接口（ABI）是强类型的，在编译时和静态时都是已知的，不会提供反省机制。我们断言所有合约都将具有在编译期间他们调用可用的任何合约的接口定义。  
该规范不涉及接口是动态的或仅在运行时才知道的合约。如果这些情况变得重要，那么可以充分地将其作为Ethereum生态系统内建设施来处理。
## 函数选择器Function Selector
函数调用的调用数据的前四个字节指定要调用的函数。它是函数签名的Keccak（SHA-3）哈希的第一个（左，高位[大端](http://blog.csdn.net/ce123_zhouwei/article/details/6971544)）四个字节。这个签名由基本原型的规范表达式，即具有参数类型的括号列表的函数名称定义。参数类型由单个逗号分隔，不使用空格。
## 参数编码Argument Encoding
从第五个字节开始，编码参数随之而来。这种编码也用于其他地方，例如返回值和事件参数都以相同的方式编码，但是它们没有指定函数的前四个字节。
### Types类型
以下是基本类型：

* `uint<M>`：`M`位无符号整数，`0 < M <= 256`, `M % 8 == 0` 例如 `uint32`, `uint8`, `uint256`.
* `int<M>`：`M`位二进制补码有符号整数，`0 < M <= 256`, `M % 8 == 0`.
* `address`：相当于`uint160`，除了假设的解释和语言输入。
* `uint`, `int`：`uint256`，`int256`的同义词（不能用于计算函数选择器）。
* `bool`：相当于限制为`0`和`1`的值的`uint8`
* `fixed<M>x<N>`：`M`位的有符号定点十进制数，`0 < M <= 256`, `M % 8 ==0`, 并且 `0 < N <= 80`, 也就是值 `v` 表示十进制数 `v / (10 ** N)`.
* `ufixed<M>x<N>`：无符号的 `fixed<M>x<N>`.
* `fixed`, `ufixed`：`fixed128x19`, `ufixed128x19` 的同义词（不能用于计算函数选择器）。
* `bytes<M>`：`M`字节的二进制类型，`0 <M <= 32`.
* `function`：等效于`bytes24`，一个地址后面加一个函数选择器  
  
以下是（固定大小）数组类型：

* `<type>[M]`：给定固定长度的固定长度数组。
以下是动态大小类型：  
  
* `bytes`：动态大小的字节序列。
* `string`：动态大小的unicode字符串，假定为UTF-8编码。
* `<type> []`：给定固定长度类型的可变长度数组。  

可以通过用括号包围有限非负数个类型，并以逗号分隔，组合成匿名结构体：
* `(T1,T2,...,Tn)`：由类型`T1，...，Tn，n> = 0`组成的匿名结构体（有序元组）。  
  
还可以构建结构体的结构体，结构体的数组等等。
## 编码的正式规范Formal Specification of the Encoding
我们现在将正式指定编码，使其具有以下属性，如果某些参数是嵌套数组，则特别有用：  
#### 属性
1. 访问值所需的读取次数最多为参数数组结构内的值的深度，即检索`a_i[k][l][r]`需要四次读取。在以前版本的ABI中，读取次数与最坏情况下的动态参数总数呈线性关系。

2. 变量或数组元素的数据不与其他数据交错，并且它是可重定位的，即它仅使用相对的“地址”  
  
我们区分静态和动态类型。静态类型是原位编码的，动态类型在当前区块之后的单独分配的位置进行编码。  
**定义：**以下类型是动态的：  
* `bytes`
* `string`
* `T[]` for any `T`
* `T[k]` for any dynamic `T` and any `k > 0`  
  
所有其他类型都被称为“静态”(static)。  

**定义：**`len(a)`是二进制字符串`a`中的字节数，`len(a)`的类型假定为`uint256`  
我们将`enc`（实际编码）定义为ABI类型的值到二进制字符串的映射，使得`len(enc(X)`取决于`X`的值，当且仅当`X`的类型是动态时。  
  
**定义：**对于任何ABI值`X`，我们递归地定义`enc(X)`，具体取决于`X`的类型  
* `T[k]` for any `T` and `k`:  
```
enc(X) = head(X[0]) ... head(X[k-1]) tail(X[0]) ... tail(X[k-1])
```
where `head` and `tail` are defined for `X[i]` being of a static type as `head(X[i]) = enc(X[i])` and `tail(X[i]) = ""` (the empty string) and as `head(X[i]) = enc(len(head(X[0]) ... head(X[k-1]) tail(X[0]) ... tail(X[i-1])))` `tail(X[i]) = enc(X[i])` otherwise.

Note that in the dynamic case, `head(X[i])` is well-defined since the lengths of the head parts only depend on the types and not the values. Its value is the offset of the beginning of `tail(X[i])` relative to the start of `enc(X)`.

* `T[]` where `X` has `k` elements (`k` is assumed to be of type `uint256`):
```
enc(X) = enc(k) enc([X[1], ..., X[k]])
```
i.e. it is encoded as if it were an array of static size `k`, prefixed with the number of elements.

* `bytes`, of length `k` (which is assumed to be of type `uint256`):

`enc(X) = enc(k) pad_right(X)`, i.e. the number of bytes is encoded as a `uint256` followed by the actual value of `X` as a byte sequence, followed by the minimum number of zero-bytes such that `len(enc(X))` is a multiple of 32.

* `string`:

`enc(X) = enc(enc_utf8(X))`, i.e. `X` is utf-8 encoded and this value is interpreted as of `bytes` type and encoded further. Note that the length used in this subsequent encoding is the number of bytes of the utf-8 encoded string, not its number of characters.

* `uint<M>`: `enc(X)` is the big-endian encoding of `X`, padded on the higher-order (left) side with zero-bytes such that the length is a multiple of 32 bytes.

* `address`: as in the `uint160` case

* `int<M>`: `enc(X)` is the big-endian two's complement encoding of `X`, padded on the higher-order (left) side with `0xff` for negative `X` and with zero bytes for positive `X` such that the length is a multiple of 32 bytes.

* `bool`: as in the `uint8` case, where `1` is used for `true` and `0` for `false`

* `fixed<M>x<N>`: `enc(X)` is `enc(X * 2**N)` where `X * 2**N` is interpreted as a `int256`.

* `fixed`: as in the `fixed128x19` case

* `ufixed<M>x<N>`: `enc(X)` is `enc(X * 2**N)` where `X * 2**N` is interpreted as a `uint256`.

* `ufixed`: as in the `ufixed128x19` case

* `bytes<M>`: `enc(X)` is the sequence of bytes in `X` padded with zero-bytes to a length of 32.  
  
Note that for any `X`, `len(enc(X))` is a multiple of 32.
## 函数选择器与参数编码Function Selector and Argument Encoding
总而言之，对参数是`a_1，...，a_n`的函数f的调用被编码为
```
function_selector(f) enc([a_1，...，a_n])
```
`f`的返回值`v_1，...，v_k`被编码为
```
enc([v_1，...，v_k])
```
其中`[a_1，...，a_n]`和`[v_1，...，v_k]`的类型分别被假定为长度为`n`和`k`的固定大小的数组。 注意，严格来说，`[a_1，...，a_n]`可以是具有不同类型的元素的“数组”，但是编码仍然被很好地定义，因为假设的公共类型`T`（上面）并没有被实际使用。
## 示例Example
给定以下合约：
```
contract Foo {
  function bar(fixed[2] xy) {}
  function baz(uint32 x, bool y) returns (bool r) { r = x > 32 || y; }
  function sam(bytes name, bool z, uint[] data) {}
}
```
因此，对于我们的`Foo`示例，如果我们想使用参数`69`和`true`调用`baz`，那么我们将传递68个字节，这可以分为：  
* `0xcdcd77c0`：方法ID，这是派生为`baz(uint32，bool)`签名的ASCII形式的Keccak哈希的前4个字节。
* `0x000000000000000000000000000000000000000000000000000000000000004545`：第一个参数，一个uint32值`69`填充到32个字节
* `0x000000000000000000000000000000000000000000000000000000000000000001`：第二个参数 - 布尔值`true`，填充到32个字节  
  
合起来为：  
```
0xcdcd77c000000000000000000000000000000000000000000000000000000000000000450000000000000000000000000000000000000000000000000000000000000001
```
它返回一个单一的`bool`。例如，如果它返回`false`，其输出将是单字节数组`0x0000000000000000000000000000000000000000000000000000000000000000`，一个单`bool`值。

如果我们想用参数`[2.125,8.5]`来调用`bar`，那么我们将传递68个字节，分为：  
* `0xab55044d`：方法ID，这是从签名`bar(fixed128x19[2])`派生的。 请注意，`fixed`替换为其规范表示`fixed128x19`。
* `0x0000000000000000000000000000000220000000000000000000000000000000000`：第一部分第一个参数，一个`fixed128x19`的值为`2.125`。
* `0x0000000000000000000000000000000880000000000000000000000000000000000`：第二部分第一个参数，一个`fixed128x19`的值为`8.5`。  
  
合起来为：  
```
0xab55044d00000000000000000000000000000002200000000000000000000000000000000000000000000000000000000000000880000000000000000000000000000000
```
如果我们想用`"dave"`，`true`和`[1,2,3]`的参数调用`sam`，我们将传递292个字节，分为：  
* `0xa5643bf2`：方法ID，这是从签名`sam(bytes,bool,uint256 [])`导出的。请注意，`uint`被替换为其规范表示`uint256`。
* `0x0000000000000000000000000000000000000000000000000000000000000060`：第一个参数（动态类型）的数据部分的位置，以参数区块开始的字节为单位。在这种情况下，0x60。
* `0x0000000000000000000000000000000000000000000000000000000000000001`：第二个参数：`boolean true`。
* `0x00000000000000000000000000000000000000000000000000000000000000a0`：第三个参数（动态类型）的数据部分的位置，以字节为单位。在这种情况下，0xa0。
* `0x0000000000000000000000000000000000000000000000000000000000000044`：数据部分的第一个参数，它以元素中字节数组的长度开始，在这种情况下为4。
* `0x6461766500000000000000000000000000000000000000000000000000000000`：第一个参数的内容：UTF-8（在这种情况下等于ASCII）编码`"dave"`，并从右边填充到32个字节。
* `0x0000000000000000000000000000000000000000000000000000000000000003`：数据部分的第三个参数，它以元素的长度开始，在这种情况下为3。
* `0x0000000000000000000000000000000000000000000000000000000000000001`：第三个参数的第一个条目。
* `0x0000000000000000000000000000000000000000000000000000000000000002`：第三个参数的第二个条目。
* `0x0000000000000000000000000000000000000000000000000000000000000003`：第三个参数的第三个条目。  
  
合起来为：  
```
0xa5643bf20000000000000000000000000000000000000000000000000000000000000060000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000a0000000000000000000000000000000000000000000000000000000000000000464617665000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000003
```
### [Use of Dynamic Types](https://github.com/ethereum/wiki/wiki/Ethereum-Contract-ABI#use-of-dynamic-types)
# [Events事件](https://github.com/ethereum/wiki/wiki/Ethereum-Contract-ABI#events)
# [JSON](https://github.com/ethereum/wiki/wiki/Ethereum-Contract-ABI#json)