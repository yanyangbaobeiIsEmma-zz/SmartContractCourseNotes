# Smart Contract Course Notes
notes of 2018 smart contract course w. Lao Dong
## 第一堂课要点整理
### gas: 
1. the unit of transcation fee to miners
2. gas price: change with ethereum price

### background:
eg: 传统员工薪酬支付系统
* 问题： - 人力陈本； - 信任问题（拖欠工资）
* 目标：1. 高效低成本；2. 防止违约

### Solidity
静态系统
* Bool: true, false, !, &&, ...
* UINT/INT: <=, &, |, +, ...
* 不存在 float, 非整数类型
* 地址Address: (like list, dictionary in Python) -- remix上需要加双引号

  address.balance 
  
  address.transfer(value) 
  
  address.send(value)  (return bool value)
  
  address.call,  address.callcode,  address.delegatecall

### 设计：
1. 薪水将全部基于以太
* unit salary
* address of Frank
2. 每30天发放一次工资
* unit payDuration
* unit lastPayDay

### 全局变量：
ETHER 单位
* wei
* sazbo = 1^12 wei
* finney = 10^15 wei
* ether = 10^18 wei

### 时间单位：
 * seconds
 * minutes
 * hours
 * days
 * weeks
 * years

### 块：
 * block.blockhash(unit blockNumber) returns (bytes32)
 * block.coinbase(address) # who mined
 * block.difficulty（uint)
 * block.gaslimit(unit)
 * block.number (uint) # number id of block
 * block.timestamp (uint)
 * now (short-hand for block.timestamp)

### 消息 
 * msg.data (meta information of caller)
 * msg.gas(unit) (caller's gas)
 * msg.sender(address) (caller's address)
 * msg.sig (signature: first 4 bytes of msg.data)
 * msg.value(uint)

### 设计：如何发放薪水
* 定时器？（外部定时器) 被动调用的程序：所有函数都在一个block内完成, 本身不具备定时功能。
* 老董每月通过合约给frank打钱
* 如何防止老董拖欠frank工资
* 在合约上存钱, frank在每30天领取薪水
* function addFund
* function calculateRunway
* function hasEnoughFund
* function getPaid


### 关键词
* constant
* payable
* this
* revert

### 其他
* if, else 语句


### code Example:
```C
pragma solidity ^0.4.14; // version

contract SimpleStorage{ 
    uint salary = 1 ether;
    // constant for variable: which cannot be changed during constract
    uint constant payDuration = 10 seconds;
    uint lastPayday = now;
    address frank = 0xca35b7d915458ef540ade6068dfe2f44e8fa733c;
    
    function test() returns (bool) {
        return 1 wei == 1;
    }
    
    function addFund() payable returns (uint) {
        // "this" is the address of the contract
        return this.balance;
    }
    
    function calculateRunway() returns (uint) {
        return this.balance / salary;
    }
    
    function hasEnoughFund() returns(bool) {
        return this.balance >= salary;
        // or return this.calculateRunway() > 0;
        // note: remove this can pay less gas
        // so return calculateRunway() > 0;
    }
    
    function getPaid() {
        // msg is for function caller
        if(msg.sender != frank) {
            revert();
        }
       
        // !!! avoid repeated calculation
        uint nextPayday = lastPayday + payDuration;
        if (nextPayday > now) {
            revert();
        }

        lastPayday = nextPayday;
        frank.transfer(salary);
        // remember to change all inner variables, then send money
    }
}
```

