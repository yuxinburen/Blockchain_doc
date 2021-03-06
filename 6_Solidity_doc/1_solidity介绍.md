#### Solidity

Solidity是以太坊智能合约的编程语言。

Solidity语法接近于Javasrcipt，是一种面相对象的语言，文件扩展名以.sol结尾。其用于智能合约的开发，并能编译成以太坊虚拟机（EVM）字节码，部署到以太坊底层区块网络上。使用它很容易创建用于投票，众筹，拍卖等的合约。

#### EVM

EVM即以太坊虚拟机，全称是Ethereum Virtual Machine，智能合约的运行环境。

- EVM是由以太坊节点提供，每个以太坊节点中都包含EVM。

- Solidity至于EVM，就如同Java之于JVM一样的关系。

- 以太坊虚拟机是一个虚拟的隔离的环境。

#### 智能合约

区块链上运行的程序，被称为“智能合约”。在Solidity中，一个合约由一组代码（函数）和数据（合约的状态）组成，合约位于以太坊区块链上的一个特殊地址。在以太坊网络中运行智能合约需要几个步骤：

1. 合约编写
2. 合约编译
3. 合约部署
4. 合约运行

总结来说就是大概需要经过：coding，compile，deploy，running四个阶段。其中，编译过程是把智能合约程序编译成Bytecode和ABI。

#### ABI

ABI是Application Binary Interface的缩写，翻译为应用程序二进制接口，可以通俗的理解为合约的接口说明。当合约编译后，即ABI就是编译产物之一，并被唯一确定，在部署合约时需要用到ABI内容。