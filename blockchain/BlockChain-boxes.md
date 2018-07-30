## BlockChain-boxes
### MetaCoin
MetaCoin 是一个示例Demo
**过程**
```
truffle unbox metacoin
contacts/: solidity合约目录
migrations/: 部署文件位置
test/: 测试应用和合约的目录
truffle.js/: truffle 配置文件

配置文件: 稍后详解

```
**Event**
Events are convenience interfaces with the EVM logging facilities
Event可以方便地使用EVM日志记录工具，而这些工具又可以在一个Dapp的用户界面中“调用”JavaScript callbacks，这些JavaScript callbacks是用来listen for these events的。

事件是以太坊虚拟机(EVM)日志基础设施提供的一个便利接口。当被发送事件（调用）时，会触发参数存储到交易的日志中（一种区块链上的特殊数据结构）。这些日志与合约的地址关联，并记录到区块链中.
来捋这个关系：区块链是打包一系列交易的区块组成的链条，每一个交易“收据”会包含0到多个日志记录，日志代表着智能合约所触发的事件。

就是一个事件函数，用来触发对界面UI的刷新等操作

view 修饰的函数 ，只能读取storage变量的值，不能写入。

pure 修饰的函数 ， 不能对storage变量进行读写。



**源码**
```python
ConvertLib.sol

library ConvertLib{
	function convert(uint amount,uint conversionRate) public pure returns (uint convertedAmount)
	{
		return amount * conversionRate;
	}
}

MetaCoib.sol
contract MetaCoin {
	mapping (address => uint) balances;

	event Transfer(address indexed _from, address indexed _to, uint256 _value);

	constructor() public {
		balances[tx.origin] = 10000;
	}

	function sendCoin(address receiver, uint amount) public returns(bool sufficient) {
		if (balances[msg.sender] < amount) return false;
		balances[msg.sender] -= amount;
		balances[receiver] += amount;
		emit Transfer(msg.sender, receiver, amount);
		return true;
	}

	function getBalanceInEth(address addr) public view returns(uint){
		return ConvertLib.convert(getBalance(addr),2);
	}

	function getBalance(address addr) public view returns(uint) {
		return balances[addr];
	}
}

msg.sender (address): sender of the message (current call)
tx.origin (address): sender of the transaction (full call chain)

一个合约，对于合约本身来说，是同一个地址，测试之后也是一样的
	 owner = msg.sender;
    sender = tx.origin;
    在构造函数中，地址是一样的
tx.origin: 是初始地址。
比如合约（或外账户地址）A去调用合约B，合约B调用合约C。此时在合约C中读取到的 msg.sender 即为合约B的地址，tx.origin 即为A的地址。
若从一个地址直接对合约发起调用，那么 msg.sender 和 tx.origin 是一样的，否则这两个地址就不一样。由于合约无法决定外部调用的关系，开发人员又往往容易混淆两者的含义，进而留下隐患。

```
![Markdown](http://i4.bvimg.com/654958/c52b58954757fe49.png)


```
Migrations.sol
constructor() public {
    owner = msg.sender;
  }

  function setCompleted(uint completed) public restricted {
    last_completed_migration = completed;
  }

  function upgrade(address new_address) public restricted {
    Migrations upgraded = Migrations(new_address);
    upgraded.setCompleted(last_completed_migration);
  }

测试:需要升级truffle
truffle test ./test/TestMetacoin.sol

import "truffle/Assert.sol";

contract TestHooks{
  uint someValue;
  address owner;
  function beforeEach() public {
    someValue =5;

  }

  function beforeEachAgain() public {
    someValue +=1;
  }

  function testSomeValueIsSix() public  {
    uint expected =6;

    Assert.equal(someValue,expected,"someValue should have been 6");
  }
}
这个测试是简单的测试过程，测试合约Test T大写，测试函数test小写，引入Assert.sol断言函数，

在TestMetaCoin 中有如下

MetaCoin meta = MetaCoin(DeployedAddresses.MetaCoin());
MetaCoin meta = new MetaCoin();
注意 为了我们测试合约中的函数，我们必须新建一个合约实例。

function testAddress() public {
    Hello hello =new Hello();

    Assert.equal(hello.getOwner(),DeployedAddresses.Hello(),"someValue should have been 6");
  }


First: Assert.sol 很多有用的断言函数
Second: DeployedAddresses.sol 给单元测试提供一个干净的部署，这个包提供函数
  DeployedAddresses.<contract name>(); 将会返回一个地址去获取合约
second: 引入合约内容
Forth: 所有的测试合约名必须以Test开头
Fifth: 所有的测试函数以test开头，每一个测试函数作为一个单一交易执行
sixth: 有许多钩子函数提供，

```

**WRITING TESTS IN JAVASCRIPT**
js合约测试
```
1: 使用 artifacts.require(contract_name) 导入合约内容,不用定义目录递增新式
2: 测试结构
The contract() function provides a list of accounts made available by your Ethereum client which you can use to write tests.

在js中测试可以方便的使用console.log()

  	  console.log(account_one);
      console.log(account_two);
      console.log(account_one_starting_balance);
      console.log(account_two_starting_balance);


contract("TestName",function(accounts)){
//测试1
it(xxx);
//测试2
it(xxx);

}


```

**一个完整的js测试过程，包含很多有用的想法**
MetaCoin.deployed().then(function(instance)): 这个instance的地址是accounts[0]

```js
var MetaCoin = artifacts.require("MetaCoin");

contract("MetaCoin First",function(accounts){
  it("should put 10000 MetaCoin in the firtst account",function(){
    return MetaCoin.deployed().then(function(instance){
      return instance.getBalance.call(accounts[0]);
    }).then(function(balance){
      assert.equal(balance.valueOf(),10000,"10000 wasn't in the first account");
    });
  });

  it("error test",function(){
    return MetaCoin.deployed().then(function(instance){
      return instance.getBalance.call(accounts[0]);
    }).then(function(balance){
      assert.equal(balance.valueOf(),10000,"10000 wasn't in the first account");
    });
  });

  it("second test",function(){
    var meta;
    var metaCoinBalance;
    var metaCoinEthBalance;
    return MetaCoin.deployed().then(function(instance){
      meta = instance;
      return meta.getBalance.call(accounts[0]);
    }).then(function(balance){
      metaCoinBalance = balance.toNumber();
      return meta.getBalanceInEth.call(accounts[0]);
    }).then(function(outCoinBalanceEth) {
      metaCoinEthBalance = outCoinBalanceEth.toNumber();
    }).then(function() {
      assert.equal(metaCoinEthBalance, 2 * metaCoinBalance, "Library function returned unexpected function, linkage may be broken");
    });
  });

//   it("should send coin correctly", function(){
//     var meta;
//
//     // Get initial balances of first and second account.
//     var account_one = accounts[0];
//     var account_two = accounts[1];
//
//     var account_one_starting_balance;
//     var account_two_starting_balance;
//     var account_one_ending_balance;
//     var account_two_ending_balance;
//
//     var amount = 10;
// /*
// meta = metaCoin的实例，这个测试中只要一个实例
//
// */
//
//     return MetaCoin.deployed().then(function(instance) {
//       meta = instance;
//       return meta.getBalance.call(account_one);
//     }).then(function(balance) {
//       account_one_starting_balance = balance.toNumber();
//       return meta.getBalance.call(account_two);
//     }).then(function(balance) {
//       account_two_starting_balance = balance.toNumber();
//       return meta.sendCoin(account_two, amount, {from: account_one});
//     }).then(function() {
//       return meta.getBalance.call(account_one);
//     }).then(function(balance) {
//       account_one_ending_balance = balance.toNumber();
//       return meta.getBalance.call(account_two);
//     }).then(function(balance) {
//       account_two_ending_balance = balance.toNumber();
//
//       assert.equal(account_one_ending_balance, account_one_starting_balance - amount, "Amount wasn't correctly taken from the sender");
//       assert.equal(account_two_ending_balance, account_two_starting_balance + amount, "Amount wasn't correctly sent to the receiver");
//     });
//   });

  it("Test CoinBase ",function(){
    var meta;
    var amount=10;
    var account_one = accounts[0];
    var account_two = accounts[1];

    return MetaCoin.deployed().then(function(instance) {
      meta = instance;
      return meta.getBalance.call(account_one)
    }).then(function(balance) {
      account_one_starting_balance = balance.toNumber();
      return meta.getBalance.call(account_two);
    }).then(function(balance) {
      account_two_starting_balance = balance.toNumber();
      console.log(account_one);
      console.log(account_two);
      console.log(account_one_starting_balance);
      console.log(account_two_starting_balance);
      return meta.sendCoin(account_two,amount);
    }).then(function(){
      return meta.getBalance.call(account_one);
    }).then(function(balance) {
          account_one_ending_balance = balance.toNumber();
          return meta.getBalance.call(account_two);
        }).then(function(balance) {
          account_two_ending_balance = balance.toNumber();
          console.log(account_one_ending_balance);
          console.log(account_two_ending_balance);
          assert.equal(account_one_ending_balance, account_one_starting_balance - amount, "Amount wasn't correctly taken from the sender");
          assert.equal(account_two_ending_balance, account_two_starting_balance + amount, "Amount wasn't correctly sent to the receiver");

  });
});

注意这里的sendCoin 可以后面跟随 {from:account},也可以不跟。
```


**Configuration**
```
通过配置文件连接的 ethereum 客户端，在Windows cmd下运行失败，在powershell上成功

module.exports = {
  networks: {
    development: {
      host: "127.0.0.1",
      port: 8545,
      network_id: "*" // Match any network id
    }
  }
}，
live: {
    host: "178.25.19.88", // Random IP for example purposes (do not use)
    port: 80,
    network_id: 1,        // Ethereum public network
    // optional config values:
    // gas
    // gasPrice
    // from - default address to use for any transaction Truffle makes during migrations
    // provider - web3 provider instance Truffle should use to talk to the Ethereum network.
    //          - function that returns a web3 provider instance (see below.)
    //          - if specified, host and port are ignored.
  }

还可以增加其他网络，以便环境切换
truffle migrate --network live

```

## Pet-shop
