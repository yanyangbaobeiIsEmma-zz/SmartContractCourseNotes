## 第二堂课要点整理：支持多人的员工系统

### 延伸
address frank -> address employee

function updateEmployee

### 构造函数 (Constructor)
``` C
function Payroll() {
    owner = msg.sender;
}
```

### 错误检测
* assert(bool) - 程序运行时查看数据是否错误
    assert(nextPayday < now);
* require(bool) - 要去输入值满足要求
    require(msg.sender == employee);

### 目标: 支持多人的系统

### 数组
* 固定长度（uint[5] a）
* 动态长度（uint[] a）
* 成员
    * length (both)
    * push (only dynamic)

``` C
pragma solidity ^0.4.14;

contract Test {
    uint[] a;    
    function test() returns (uint, uint) {
        a.push(1);
        a.push(2);
        return (a[0], a[1]);
    }    
}
```

### 设计
方法一：通过数组来存储多个员工的信息
* address[] employee
* uint[] salary
* unit[] lastPayday

方法二： struct
``` C
struct Employee {
    address e;
    uint s;
    uint lastPayday;
}
```

## 数据存储
* storage 区块上
* memory
* calldata 

强制
* 状态变量：storage
* function 输入参数：calldata

默认
* function 返回函数：memory
* 本地变量：storage

## 规则
* 相同存储空间变量赋值
    * 传递reference (EVM上的内存地址)
* 不同存储空间变量赋值
    * 拷贝

## 可见度
* private
* public
* ...

## Code Example
```C
pragma solidity ^0.4.14;

contract Payroll {

    uint constant payDuration = 10 seconds;
    
    struct Employee {
        address id;
        uint salary;
        uint lastPayday;
    }

    address owner;
    Employee[] employees;

    function Payroll(){ 
        owner = msg.sender;
    }
    
    // private function
    function _partialPaid(Employee employee) private {
        uint payment = employee.salary * (now - employee.lastPayday) / payDuration;
        employee.id.transfer(payment);
    }
    
    // note: Employee is only in memory, 
    // can set "storage Employee" as parameter
    function _findEmployee(address employeeId) private returns (Employee, uint) {
        for (uint i = 0; i < employees.length; i++) {
            if (employees[i].id == employeeId) {
                return (employees[i], i);
            }
        }
    }
    
    function addEmployee(address employeeId, uint salary) {
        require(msg.sender == owner);
        var (employee, index) = _findEmployee(employeeId);
        assert(employee.id == 0x0);
        
        // need to check whether 1 employee appear twice
        //for (uint i = 0; i < employees.length; i++)
        //{
        //    if (employees[i].id == employee) {
        //        revert();
        //    }
        //}
             
        employees.push(Employee(employeeId, salary, now));
    }
    
    
    function removeEmployee(address employeeId) {
        require(msg.sender == owner);
        
        var(employee, index) = _findEmployee(employeeId);
        assert(employee.id != 0x0);
        _partialPaid(employee);
        
        delete employees[index];
        employees[index] = employees[employees.length - 1];
        employees.length -= 1;
    }    

    function updateEmployee(address employeeId, uint salary) {
        require(msg.sender == owner);
        
        var(employee, index) = _findEmployee(employeeId);
        assert(employee.id != 0x0);
        _partialPaid(employee);
        
        // employee will point back to the employees[index]
        employees[index].salary = salary;
        employees[index].lastPayday = now;  
    }

    function addFund() payable returns (uint) {
        return this.balance;
    }
    
    function calculateRunway() returns (uint) {
        uint totalSalary = 0;
        for (uint i = 0; i < employees.length; i++) {
            totalSalary += employees[i].salary;
        }
        return this.balance / totalSalary;
    }

    function hasEnoughFund() returns (bool) {
        return calculateRunway() > 0;
    }

    function getPaid() {
        var(employee, index) = _findEmployee(msg.sender);
        assert(employee.id != 0x0);
        
        uint nextPayday = employee.lastPayday + payDuration;
        assert(nextPayday < now);

        employees[index].lastPayday = nextPayday;
        employeep.id.transfer(employee.salary);

    }
}
```
