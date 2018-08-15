```solidity
pragma solidity  ^0.4.24;

contract Counter{
    uint count = 0;
    address owner;
    
    function Counter(uint a){
        count = a;
        owner = msg.sender;
    }
    
    function increment() public{
        if(owner == msg.sender){
            count = count + 10;
        }else {
            count++;
        }
    }
    
    function getCount() constant returns(uint){
        return count;
    }
    
    function  kill(){
        if(owner == msg.sender){
            selfdestruct(owner);
        }
    }
}
```

#### 版本声明

pragma solidity ^0.4.24;

#### 合约声明

contract关键字后面加合约名字，类似于Java中的class关键字。

#### 状态变量

在solidity中，状态变量是在合约内声明的共有变量。

uint变量默认是指的uint256变量，还有uint8，uint16等，占的长度不同；

address：地址变量，是solidity语言中特有的一个变量，用来表示一个帐户地址。

#### 本地变量/局部变量

超过作用域即不可被访问，等待被收回。对本地变量/局部变量而言，具有穿透作用，在其声明至函数结束，其都可以起作用，能穿透过逻辑判断，循环等代码块。

#### 构造函数（Contructor）

function d()函数名和合约相同时，此函数是合约的构造函数，当合约对象创建时，会先调用构造函数对相关数据进行初始化处理。只会在合约部署时被调用一次。

#### 成员函数

成员函数就是智能合约中为了完成某些功能而编写的方法。

#### 析构函数selfdestruct

析构函数和构造函数对应，构造函数是初始化数据，而析构函数是销毁函数。

#### 全局变量

整个合约中声明，全局都能使用的变量

#### 多继承，重写

Solidity具有多继承的特性：

```solidity
pragma solidity ^0.4.24;
contract Animal1{
    function sayHello() pure returns(string){
        return "hello";
    }
}
contract Animal2{
    function sayBye() pure returns(string){
        return "bye";
    }
}

contract Dog is Animal1, Animal2{
    //Dog 会继承 Animal 及 Animal2 两个类
}
```

Solidity中的继承原则遵循C3 Linearation约定。

#### 函数的访问权限public，internal，private，external

- private：私有函数。内部正常访问，外部无法访问，子类无法继承。

- internal：内部函数。内部正常反问，外部无法访问，子类可继承。对内部状态变量来说，内部可以正常访问，外部无法访问，子类可继承。

- public：公共函数。内部正常访问，外部正常访问，子类可继承。

- external：外部函数。内部不能访问（可通过this.()访问），外部正常访问，子类可继承。

函数默认为public类型，状态变量默认为internal类型。

#### pure，view，constant，payable的区别

以上四个关键词可以用来修饰函数。

结论：

- 只有当函数有返回值的情况下，才能使用pure，view，constant。
- pure：当函数返回值为自变量而非变量时，使用pure。
- view：当函数返回值为全局变量或属性时，使用view。
- constant：与view是等价的，constant是view的别名。
- payable：使用msg.value的所在方法若不用payable标记也能编译通过。但是msg.value有值时运行会报错，为零时不会报错。

如果一个函数有返回值，函数中正常来讲需要有pure，view或constant关键字，如果没有，在调用函数的过程中，需要主动去调用底层的call方法。

注意：如果一个函数中带了关键字view或者constant，就不能修改状态变量的值。但凡是带了这两个关键字，区块链就默认只是向区块链读取数据，读取数据不需要花费gas，但是不花gas就不能修改状态变量的值。写入数据或者是修改状态变量的值都需要花费gas。

#### memory，storage，calldata

storage是持久化存在区块链上，不会随着程序的执行而丢失；

memory类型的数据是程序执行时的临时内存，会随着程序的执行而丢失；

calldata：

又：

- 对于引用类型的变量的传递有两种类型：值传递memory（深拷贝）和指针传递的storage（浅拷贝）。

- 值类型的变量，只能深拷贝。













