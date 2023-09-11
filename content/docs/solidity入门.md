# Solidity 入门

```solidity


//声明合约编译器
// SPDX-License-Identifier: GPL-3.0 
pragma solidity >=0.8.2<0.9.0;
contract NumberStorage{
    //构造函数
    constructor(uint _x) {
        x = _x;
    }
    //modifier 可以理解为中间件吧 大多数情况使用来自访问控制
    modifier not_toolarge(uint _x){
        if (_x >1000)
        revert("too large"); //表示如果_x 大于1000终止执行 抛出异常
        //另一种写法: require(_x <=1000,"too large"); //表示 _x 必须小于1000 否则终止执行 抛出异常
  
        _; //执行所修饰的函数体
    }
    //event 相当于其他语言的logs日志
    //event 关键字 加上事件的名称
    event Xchange(uint oldv,uint newv);
    uint public x;
    uint y;
    function getX() public view returns(uint){
        return x;
    }
    function setX(uint _x) public not_toolarge(_x){
        uint oldv = x;
        emit Xchange(oldv, _x); //触发所定义的event事件
        x = _x;
    }
    //等价
    //function setX(uint _x) public {
    //     require(_x <=1000,"too large");
    //     x = _x;
    // }
    //public 表合约外部可见
    //internal 和private 合约外部不可见
    //public internal 子和约可见
    //view 合约状态读操作
    //pure 与合约状态无关的函数，纯函数
    //纯函数，跟系统状态无关，只跟输入有关
    function add(uint _a,uint _b)internal pure returns(uint){
        //数据额类型
        // uint temp1 = type(uint).min;//0
        // uint temp2 = type(uint).max;//255
        return  _a+_b;
    }

   
}
```
