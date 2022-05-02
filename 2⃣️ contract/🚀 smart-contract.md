# 智能合约

## 官网
https://soliditylang.org/

简介：一门专为开发智能合约而设计的运行在以太网上的开发语言。

## 智能合约基础
先从智能合约的“Hello World开发”开始，以最简单的开发作为切入口。

一个数据存储小样例：

1、对变量赋值

2、对别的合约代码开放访问权限



```Solidity
pragma solidity >=0.4.16 <0.9.0;

contract SimpleStorage {
    uint storedData;
    
    function set(uint x) public{
        storedData = x;
    }
    
    function get() public view return (uint){
        return storedData;
    }
}
```

第一行指出 `Solidity` 代码最低版本支持到 0.4.16，不高于 0.9.0。

SimpleStorage 方法定义了一个 uint 型的状态变量 storedData，定义了查询和修改变量的 get() 和 set() 函数。

访问当前合约里的成员，不需要用 this. 前缀，只需要用成员名字就可以访问的到。不像其它语言，省略 this. 并不仅仅是一个风格的事情，而是一种完全不同的访问成员的方式。

合约只是存储下数据，并没有做其它什么事情，任何人都可以通过set方法改变成员的值，但是任何的改变都会被记录到区块链的历史里面。只有你可以改变成员的值。

## 子货币
继承自虚拟货币的最精简的形式。相互之间发币不需要账户和密码，只需要有一对密钥就可以。
```Solidity
pragma solidity ^0.8.4;

contract Coin{
    address public minter;
    mapping (address => uint) public balances;
    
    event Sent(address from, address to, uint amount);
    
    constructor(){
        minter = msg.sender;
    }
    
    function mint(address receiver, uint amount) public {
        require(msg.sender == minter);
        balances[receiver] += amount;
    }
    
    error InsufficientBalance(uint requested, uint available);
    
    function send(address receiver, uint amount) public{
        if(amount > balance[msg.sender])
            revert InsufficientBalance({
                requested: amount;
                available: balances[msg.sender]
            });
            
        balances[msg.sender] -= amount;
        balances[receiver] += amount;
        emit Sent(msg.sender, receiver, amount);
    }
}
```

address public minter: 声明一个地址变量，指向外部地址。public赋予外部合约访问的公开权限。下面是编译器生成的代码：

```Solidity
function minter() external view returns (address){ return minter;}
```

你完全可以自己命名一个这样的函数，但是没必要，因为编译器已经帮你做好了。

mapping (address => uint) public balances; 也是声明了一个公开的状态变量，但是是一个更加复杂的类型mapping。

通过这种方式可以查询某个账户的余额：

```Solidity
function balances(address account) external view returns (uint){
    return balances[account];
}
```

event Sent(address from, address to, uint amount); 这行定义了一个监听事件，以太坊的应用以很小的成本监听着这个事件，在最后一行会触发这个监听，事件里带的参数，from, to, amount 保证了交易的可追踪性。

web3.js定义Coin对象监听这个事件，任何用户接口调用都自动继承balances方法。

```web3.js
Coin.Sent().watch({},'',function(error,result){
  if(!error){
    console.log("Coin transfer:" + result.args.amount +
        "coins were sent from " + result.args.from +
        "to " + result.args.to + ".");
    console.log("Balances now:\n" + 
        "Sender:" + Coin.balances.call(result.args.from)+
        "Receiver: "+ Coin.balances.call(result.args.to));
  }
})
```





