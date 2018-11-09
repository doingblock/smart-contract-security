也被称为**没有什么秘密可言**  
_合约对block.number的年龄没有进行充分的校验，一个未知的玩家等待了256个区块直到出现了那个可预测的中奖号码，导致合约输掉了400个ETH。——Arseny Reutov_  

在以太坊中很难获得一个真正的随机性。Solidity提供了一些[函数和变量](http://solidity.readthedocs.io/en/v0.4.21/units-and-global-variables.html)可以获得似乎是难以预测的值，通常它们要么是看起来更公开，要么受到了矿工的影响。因为这些随机性相关的来源在一定程度上是可预测的，恶意用户通常可以复制它，并依靠其不可预测性来攻击该功能。

**损失**  
超过400ETH  

**现实世界影响**
* [SmartBillions Lottery](https://www.reddit.com/r/ethereum/comments/74d3dc/smartbillions_lottery_contract_just_got_hacked/)
* [TheRun](https://medium.com/@hrishiolickel/why-smart-contracts-fail-undiscovered-bugs-and-what-we-can-do-about-them-119aa2843007)

**示例**  
1. 一个`智能合约`使用区块号作为游戏随机性的来源。  
2. 攻击者创建一个`恶意合约`；其检查当前区块号是否是赢家，如果是，`恶意合约`调用`游戏合约`来赢得游戏。因为这个调用是同一个交易的一部分，因此区块号在这两个合约是保持一致的。  
3. 攻击者只需不断的调用他的`恶意合约`直到赢得游戏。  

**代码示例**  
在这个示例里，一个私有的种子`seed`结合一个迭代次数`iteration`作为`keccak256`函数的输入来决定调用者是否可以赢。尽管`seed`是`private`修饰的，但它必然是在某个时候通过一个交易来进行设定，这样的话`seed`在区块链上就是可见的。
```
uint256 private seed;

function play() public payable {
	require(msg.value >= 1 ether);
	iteration++;
	uint randomNumber = uint(keccak256(seed + iteration));
	if (randomNumber % 2 == 0) {
		msg.sender.transfer(this.balance);
	}
}

```

在第二个示例中，`block.blockhash`被用来生成一个随机数。如果使用当前的`block.number`来设置`blockNumber`，那么这个哈希值是未知的。在这个例子里，`blockNumber`如果使用过去256个区块之前的`block.number`，而它的值此时为0。最后，如果它使用之前不是很久的区块来进行设置，而其他的智能合约也可以访问到同样的数字，并且通过调用游戏合约作为相同交易的一部分。
```
function play() public payable {
	require(msg.value >= 1 ether);
	if (block.blockhash(blockNumber) % 2 == 0) {
		msg.sender.transfer(this.balance);
	}
}
```

**其他资源**  
* [Predicting Random Numbers in Ethereum Smart Contracts](https://blog.positive.com/predicting-random-numbers-in-ethereum-smart-contracts-e5358c6b8620)
* [Random in Ethereum](https://blog.otlw.co/random-in-ethereum-50eefd09d33e)
