# 第三堂课要点整理

## Mapping
* 类似于map (C++), dict (python)，其实是hashtable
* key (bool, int, address, string)
* value(任何类型)
* mapping(address => Employee) employees

## Mapping 底层实现
* 不使用数组+链表，不需要扩容
* hash函数为keccak256hash,在storage上存储，理论上无限大的hash表
* 无法naiive遍历整个mapping :sad:
* 赋值 employees[key] = value
* 取值 value = employees[key]
* value是引用，在storage上存储，可以直接修改
* 当key不存在，value = type's default 

## 函数参数返回进阶
* 命名参数返回
* 命名返回参数直接赋值

```C
function checkEmployee(address employeeId) returns (uint salary, uint lastPayday) {
    var employee = employees[employeeId];
    //return (employee.salary, employee.lastPayday);
    salary = employee.salary;
    lastPayday = employee.lastPayday;
}

```

## 可视度
* public: 谁都可见
* external: 只有“外部调用”可见 
* internal: 外部调用不可见，内部和子类可见, 类似protected
* private: 只有当前合约可见

### 状态变量: public, internal, private
    * 默认：internal
    * public:自动定义取值函数
    * private:不代表别的无法用肉眼看到，只代表别的区块链智能合约无法看到
    *合约的所有成员变量都是肉眼可见的！*

### 函数 public, external, internal, private
    * 默认：public

```C
pragma solidity ^0.4.14;

contract Test {
    uint a;

    function func1() external {
        
    }
    
    function func2() {
        this.func1(); //外部调用
    }
}
```


### 继承

```C
pragma solidity ^0.4.14;

contract owned {
    address owner; // internal

    function owned() {
        owner = msg.sender;
    }
}

contract Parent is owned {
    uint x;

    function Parent(uint_x) {
        x = _x;
    }

    function parentFunc1() internal {
        if (msg.sender == owner) selfdestruct(owner);
    }

    function parentFunc2() public {}
    function parentFunc3() external {}
    function parentFunc4() private {}
}

contract Child is Parent {
    uint y;
    function Child(uint _y) Parent (_y*_y) {
        y = _y;
    }

    function child() {
        parentFunc2();
        this.parentFunc3(); //external
        parentFunc1();
        //无法调用 parentFunc4(), private对继承不可见
    }
}

contract Child2 is Parent(666) {
    uint y;
    function Child2(uint _y) {
        y = _y;
    }

}
```
private对继承不可见
```C
contract Parent is owned {
    uint x;
    function Parent(uint _x) {
        x = _x;
    }
    function child() {
        parentFunc2();
        this.parentFunc3(); //external
        parentFunc1();
        //无法调用 parentFunc4(), private对继承不可见
    }
}

```

### 继承 - 抽象合约
```C
constract Parent {
    function someFunc() returns (uint); 
    // 没有定义任何函数的合约叫抽象合约， 无法部署
}

constract Child is Parent {
    // 子类去定义函数
    function someFunc() returns (uint) {
        return 1;
    }
}
```

### 继承 - interface (类似Java的interface)
```C
pragma solidity ^0.4.14;
interface Parent {
    // a framework of a contract
    // 不可继承其他合约或interface
    // 没有constructor()
    // 没有state variable
    // 没有struct
    // 没有enum
    // 简单来说，只有function定义，啥都没有！
    function someFunc() returns(uint);
}

contract Child is Parent {
    function someFunc() returns (uint) {
        return 1;
    }
}
```

### 继承 - 多继承
* 最平凡的多继承
```C
contract Base1 {
    function func1() {}
}

contract Base2 {
    function func2() {}
}

contract Final is Base1, Base2 {
}
```
* 重名函数的overide次序
```C
contract Base1 {
    function func1() {}
}

contract Base2 {
    function func1() {}
}

contract Final is Base1, Base2 { 
}

contract test {
    Final f = new Final(); 
    f.func1(); // Base2的function1被调用
}
```
* super:动态绑定上级函数
```C
contract foundation {
    function func1 {
        // do stuff
    }
}

contract Base1 is foundation {
    function func1() {
        super.func1(); //动态绑定上级函数
    }
}

contract Base2 is foundation {
    function func1() {
        super.func1();
    }
}

contract Final is Base1, Base2 {   
}

contract test {
    Final f = new Final(); 
    f.func1(); 
    // 函数调用次序: Base2.func1, Base1.func1, foundation.func1
}
```
* 假设下面所有合约都有一个同名函数，实现都为如下，函数调用的顺序是什么
```C
function foo() {
    // do something specific to each class
    super.foo();
}
contract O
contract A is O
contract B is O
contract C is O
contract D is O
contract E is O
contract K1 is C, B, A
contract K2 is E, B, D
contract K3 is A, D
contract Z is K3, K2, K1
```
* 多继承Method Resolution Order使用C3 Linearization

## modifier
类似python的decorator
```C
modifier onlyOwner {
    require(msg.sender == owner);
    _;
}

modifier employeeExist(address employeeId) {
    var employee = employees[employeeId];
    assert(employee.id != 0x0);
    _;
}
  
function addEmployee(address employeeId, uint s) onlyOwner {
    //require(msg.sender == owner);
    //var employee = employees[employeeId];
        
    // make sure it is new employee
    assert(employee.id == 0x0);
    employees[employeeId] = Employee(employeeId, s * 1 ether, now);
    totalSalary += s * 1 ether;
}

```
## Advanced modifier
```C
pragma solidity ^0.4.14;

contract Parent {
    uint public a = 2;
    modifier someModifier() {
        _;
        // 除了return外的所有语句
        a = 1;
    }
    
    function parentFunc1(uint value) someModifier public returns(uint) {
        a = value;
        // a = 1
        return a;
    }    
}
```

## Safe Math和Library
```C
pragma solidity ^0.4.14;

contract Test {
    uint8 public a = 0; // uint8:0 - 255
    
    function test() {
        a -= 100; // a = 156
    }
}
```
One possible solution:

```C
pragma solidity ^0.4.14;

contract Test {
    uint8 public a = 1;
    
    function test() {
        uint8 c = a - 100;
        assert(c < a);
        a = c;
    }
}
```
非常麻烦，调用其他库
### OpenZeppelin/zeppelin-solidity
* SafeMath
![link](https://github.com/OpenZeppelin/zeppelin-solidity/edit/master/contracts/math/SafeMath.sol)

```C
pragma solidity ^0.4.14;

import './SafeMath.sol';

contract Test {
    using SafeMath for uint8;
    uint8 public a = 101;
    
    function test() {
        a = a.sub(100);
        //a = SafeMath.sub(a, 100);
    }
}
```

* Ownable
![link](https://github.com/OpenZeppelin/zeppelin-solidity/edit/master/contracts/ownership/Ownable.sol)
```C
import './Ownable.sol';
```









