DASP Top 10
---

这个项目由NCC集团发起。它是一个开发合作项目，力求集合安全社区的努力来发现智能合约中的安全漏洞。

---

# 1. 重入

*这种攻击利用被众多代码审计者错过很多很多次：代码审计者倾向于一次一个函数的审阅，并假定对安全子例程的调用将会如预期般的安全运行。
————Phil Daian*

重入攻击，也许是以太坊中最著名的漏洞了，当它第一次被发现时使所有人都大吃一惊。它第一次亮相就导致了数百万美元的资金被盗，并直接导致以太坊的硬分叉。重入攻击在两种场景下会发现：1、合约层面，外部合约被允许调用初始化未完成的合约；2、函数而言，其发生在合约状态因为执行过程中调用不可信合约或外部地址具有使用低层次函数而发生变化时。

__损失__：  
预计3.5M ETH（约合50M美元）  

__漏洞发现的时间线：__  
* [2016/06/05： Christian Reitwiessner在Solidity中发现了一个违反模式的问题](https://blog.ethereum.org/2016/06/10/smart-contract-security/)
* [2016/06/09： 更多的以太坊攻击：Race-To-Empty是真实的案例（vessenes.com）](http://vessenes.com/more-ethereum-attacks-race-to-empty-is-the-real-deal/)
* [2016/06/12： 在以太坊智能合约“递归调用”漏洞发现之后没有DAO资金有风险](https://blog.slock.it/no-dao-funds-at-risk-following-the-ethereum-smart-contract-recursive-call-bug-discovery-29f482d348b)
* [2016/06/17： 我认为TheDAO正被吸干（reddit.com）](https://www.reddit.com/r/ethereum/comments/4oi2ta/i_think_thedao_is_getting_drained_right_now/)
* [2016/06/24： DAO的历史和经验教训（blog.slock.it）](https://blog.slock.it/the-history-of-the-dao-and-lessons-learned-d06740f8cfa5)

__现实案例影响：__  
* [The DAO](https://en.wikipedia.org/wiki/The_DAO_(organization))

__示例：__  
1. 一个智能合约追踪一系列外部地址在该合约的余额，并允许用户使用 `withdraw()` 函数来提取余额。  
2. 一个攻击合约调用 `withdraw()` 来提取其所有的余额。  
3. 受害合约在更新攻击合约地址余额之前，使用低级别函数 `call.value(amount)()` 来发送ether到攻击合约。   
4. 攻击合约有一个使用payable修饰的 `fallback()` 函数：其被用来接收资金，并会回调受害合约中的 `withdraw()` 函数。  
5. 受害合约第二次执行代码并再次触发资金转移：还记得不，攻击合约地址的余额在第一次提取后还没有更新。结果是，攻击合约可以在几秒之内将受害合约中所有的ether取光。  

__代码示例：__  
如下函数包含一个重入攻击漏洞。当底层函数 `call()` 发送ether到 `msg.sender` 时，它就可能被攻击：如果转账地址是一个智能合约，该支付可以使用当前交易剩余的gas触发合约的回调函数。
```
function withdraw(uint _amount) {
	require(balances[msg.sender] >= _amount);
	msg.sender.call.value(_amount)();
	balances[msg.sender] -= _amount;
}
```

__其他资源：__
* [The DAO smart contract](https://etherscan.io/address/0xbb9bc244d798123fde783fcc1c72d3bb8c189413#code)
* [Analysis of the DAO exploit](http://hackingdistributed.com/2016/06/18/analysis-of-the-dao-exploit/)
* [Simple DAO code example](http://blockchain.unica.it/projects/ethereum-survey/attacks.html#simpledao)
* [Reentrancy code example](https://github.com/trailofbits/not-so-smart-contracts/tree/master/reentrancy)
* [How Someone Tried to Exploit a Flaw in Our Smart Contract and Steal All of Its Ether](https://blog.citymayor.co/posts/how-someone-tried-to-exploit-a-flaw-in-our-smart-contract-and-steal-all-of-its-ether/)

# 2. 访问控制
# 3. 算术计算
# 4. 未经检查的底层调用
# 5. 拒绝服务
# 6. 伪随机性
# 7. 前端运行
# 8. 时间操控
# 9. 短地址问题
# 10. 未知引起的新问题
