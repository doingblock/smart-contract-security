DASP Top 10
---

这个项目由NCC集团发起。它是一个开发合作项目，力求集合安全社区的努力来发现智能合约中的安全漏洞。

---

## 1.重入

*这种攻击利用被众多代码审计者错过很多很多次：代码审计者倾向于一次一个函数的审阅，并假定对安全子例程的调用将会如预期般的安全运行。
————Phil Daian*

重入攻击，也许是以太坊中最著名的漏洞了，当它第一次被发现时使所有人都大吃一惊。它第一次亮相就导致了数百万美元的资金被盗，并直接导致以太坊的硬分叉。重入攻击在两种场景下会发现：1、合约层面，外部合约被允许调用初始化未完成的合约；2、函数而言，其发生在合约状态因为执行过程中调用不可信合约或外部地址具有使用低层次函数而发生变化时。

__损失__  
预计3.5M ETH（约合50M美元）  

__漏洞发现的时间线__  
* [2016/06/05： Christian Reitwiessner在Solidity中发现了一个违反模式的问题](https://blog.ethereum.org/2016/06/10/smart-contract-security/)
* [2016/06/09： 更多的以太坊攻击：Race-To-Empty是真实的案例（vessenes.com）](http://vessenes.com/more-ethereum-attacks-race-to-empty-is-the-real-deal/)
* [2016/06/12： 在以太坊智能合约“递归调用”漏洞发现之后没有DAO资金有风险](https://blog.slock.it/no-dao-funds-at-risk-following-the-ethereum-smart-contract-recursive-call-bug-discovery-29f482d348b)
* [2016/06/17： 我认为TheDAO正被吸干（reddit.com）](https://www.reddit.com/r/ethereum/comments/4oi2ta/i_think_thedao_is_getting_drained_right_now/)
* [2016/06/24： DAO的历史和经验教训（blog.slock.it）](https://blog.slock.it/the-history-of-the-dao-and-lessons-learned-d06740f8cfa5)

__现实案例影响__  
* [The DAO](https://en.wikipedia.org/wiki/The_DAO_(organization))

__示例：__  
1. 一个智能合约追踪一系列外部地址在该合约的余额，并允许用户使用 `withdraw()` 函数来提取余额。  
2. 一个攻击合约调用 `withdraw()` 来提取其所有的余额。  
3. 受害合约在更新攻击合约地址余额之前，使用低级别函数 `call.value(amount)()` 来发送ether到攻击合约。   
4. 攻击合约有一个使用payable修饰的 `fallback()` 函数：其被用来接收资金，并会回调受害合约中的 `withdraw()` 函数。  
5. 受害合约第二次执行代码并再次触发资金转移：还记得不，攻击合约地址的余额在第一次提取后还没有更新。结果是，攻击合约可以在几秒之内将受害合约中所有的ether取光。  

__代码示例__  
如下函数包含一个重入攻击漏洞。当底层函数 `call()` 发送ether到 `msg.sender` 时，它就可能被攻击：如果转账地址是一个智能合约，该支付可以使用当前交易剩余的gas触发合约的回调函数。
```
function withdraw(uint _amount) {
	require(balances[msg.sender] >= _amount);
	msg.sender.call.value(_amount)();
	balances[msg.sender] -= _amount;
}
```

__其他资源__
* [The DAO smart contract](https://etherscan.io/address/0xbb9bc244d798123fde783fcc1c72d3bb8c189413#code)
* [Analysis of the DAO exploit](http://hackingdistributed.com/2016/06/18/analysis-of-the-dao-exploit/)
* [Simple DAO code example](http://blockchain.unica.it/projects/ethereum-survey/attacks.html#simpledao)
* [Reentrancy code example](https://github.com/trailofbits/not-so-smart-contracts/tree/master/reentrancy)
* [How Someone Tried to Exploit a Flaw in Our Smart Contract and Steal All of Its Ether](https://blog.citymayor.co/posts/how-someone-tried-to-exploit-a-flaw-in-our-smart-contract-and-steal-all-of-its-ether/)

## 2. 访问控制
_通过调用`initWallet`函数，可以将Parity钱包库合约转变成一个常规的多重签名钱包，并使你成为他的所有者。——Parity_  

访问控制问题在所有的程序中都很常见，并不仅仅是智能合约中。事实上，访问控制问题在[OWASP Top 10中排名第5](https://www.owasp.org/images/7/72/OWASP_Top_10-2017_%28en%29.pdf.pdf)。人们通常通过智能合约中的public或external函数来使用合约功能。然而，不安全的**可见性**设置给了黑客直接访问合约私有值和代码逻辑的途径。另外，绕过访问控制有时候更微妙。这些漏洞在以下情况下可能发生：1、合约使用废弃的`tx.origin`来鉴别调用者；2、使用冗长的`require`来处理大型的认证逻辑；3、鲁莽的在代理库或代理合约中使用`delegatecall`。

**损失**   
接近150,000ETH（约30M美元）  

**真实案例影响**  
* [Parity钱包中Multi-sig缺陷1](http://paritytech.io/the-multi-sig-hack-a-postmortem/)
* [Parity钱包中Multi-sig缺陷2](http://paritytech.io/a-postmortem-on-the-parity-multi-sig-library-self-destruct/)
* [Rubixi](https://blog.ethereum.org/2016/06/19/thinking-smart-contract-security/)

**示例**  
1. 一个合约授权初始化它的地址作为合约的拥有者，这是一个授予特殊权限（例如，提取合约资金的权限）的通用做法。
2. 不幸的是，初始化函数设置为任何人可调用——即使在它已经被调用过。这样，允许任何人成为合约的拥有者，并有权去提现合约资金。

**代码示例**  
在下面的示例中，合约`初始化函数initContract`中设置了调用者作为它的拥有者。然而，这个函数与合约的构造函数是分离的，并且没有记录追踪该函数是否已经被调用过。
```
function initContract() public { 
    owner = msg.sender; 
}

```
在Parity多重签名（multi-sig）钱包，初始化函数定义在一个“库”合约中，与钱包本身是分离的。用户可以通过`delegateCall`调用库函数来初始化他自己的钱包。不幸的是，同我们的示例代码一样，该初始化函数并没有检查钱包是否已经初始化。更糟的是，库是一个智能合约，任何人都可以初始化这个库和调用它的析构函数（销毁合约）。  

**其他资源**  
* [Fix for Parity multi-sig wallet bug 1](https://github.com/paritytech/parity/pull/6103/files)
* [Parity security alert 2](http://paritytech.io/security-alert-2/)
* [On the Parity wallet multi-sig hack](https://blog.zeppelin.solutions/on-the-parity-wallet-multisig-hack-405a8c12e8f7)
* [Unprotected function](https://github.com/trailofbits/not-so-smart-contracts/tree/master/unprotected_function)
* [Rubixi's smart contract](https://etherscan.io/address/0xe82719202e5965Cf5D9B6673B7503a3b92DE20be#code)

## 3. 算术计算

也称为整数上溢和整数下溢  
_溢出情况得出的是错误的结果，特别是在未考虑这种可能性的时候会严重损害程序的可靠性和安全性。——Jules Dourlens_  

整数上溢和下溢并不是新的漏洞类型，但它们在智能合约中尤其的危险，因为无符号整数无处不在，但绝大多数的程序员习惯简单的使用`int`作为整型。如果溢出发生，很多原本不可达的代码路径变成了黑客的攻击向量。

**现实案例影响**  
* [The DAO](http://blockchain.unica.it/projects/ethereum-survey/attacks.html)
* [BatchOverflow (multiple tokens)](https://peckshield.com/2018/04/22/batchOverflow/)
* [ProxyOverflow (multiple tokens)](https://peckshield.com/2018/04/25/proxyOverflow/)

**示例**  
1. 一个智能合约的`withdraw()`函数允许人们提取其账户中的ether余额。
2. 攻击者尝试提取超过他/她账户的金额。
3. `withdraw()`函数检查结果一直显示是正数，则允许攻击者提取超过他账户余额的ether。由此产生余额下溢，并变成一个比应有的大一个数量级的值。

**代码示例**  
最直接的示例是函数没有对整数下溢做检查，允许你提取无穷大数量的令牌：
```
function withdraw(uint _amount) {
	require(balances[msg.sender] - _amount > 0);//漏洞点
	msg.sender.transfer(_amount);
	balances[msg.sender] -= _amount;
}
```

第二个示例是一个叫1偏移错误（`off-by-one error`），一个典型的例子就是数组的长度是使用无符号整数表示：
```
function popArrayOfThings() {
	require(arrayOfThings.length >= 0);
	arrayOfThings.length--; 
}
```

第三个示例是第一个例子的变种：两个无符号整数的算数运算结果依旧是无符号数。
```
function votes(uint postId, uint upvote, uint downvotes) {
	if (upvote - downvote < 0) {
		deletePost(postId)
	}
}
```

第四个示例的特征是使用很快就会被弃用的关键字`var`。因为`var`会将其表示的数值类型自适应到最小类型：比如赋值0时变量为`unit8`类型，如果这个时候一个循环会遍历超过255次，那么其永远也不会增长到那个数，其会在gas消耗完之后才停止。
```
for (var i = 0; i < somethingLarge; i ++) {
	// ...
}
```

**其他资源**  
* [SafeMath to protect from overflows](https://ethereumdev.io/safemath-protect-overflows/)
* [Integer overflow code example](https://github.com/trailofbits/not-so-smart-contracts/tree/master/integer_overflow)


## 4. 未经检查的底层调用
也被称为（或相关）**默认失败的send，未经检查的send**  
_任何时候都不要使用底层的"call"函数，否则在未恰当的处理其返回值的情况下它将导致不可预知的行为。——Remix_  

Solidity中一个深层次的特性是其底层函数`call()`,`callcode()`,`delegatecall()`和`send()`，它们在处理错误时的行为与Solidity中其他的函数完全不同：这些函数不会传播（向上冒泡）错误/异常，也不会回滚恢复当前执行的操作。取而代之，它们会在失败时返回`false`，随后代码继续执行。这样的行为令程序员们很惊讶，如果这种底层函数调用的返回值没有进行检查，将会毫无容错并导致非预期结果。记住，send可能会失败！  

**现实案例影响**  
* [King of the Ether](https://www.kingoftheether.com/postmortem.html)
* [Etherpot](http://www.dasp.co/)

**示例**  
如下代码是一个忘记检查`send()`返回值将会导致错误的示例。如果该调用是用来向一个不能接受ether的合约（比如合约没有一个使用payable修饰的fallback函数）发送ether，EVM会将返回值置为`false`。由于返回值并没有进行检查，这个函数对合约状态所做的修改不会得到恢复，结果就是`etherLeft`会跟踪记录一个错误的数值。  
```
function withdraw(uint256 _amount) public {
	require(balances[msg.sender] >= _amount);
	balances[msg.sender] -= _amount;
	etherLeft -= _amount;
	msg.sender.send(_amount);
}
```

**其他资源**  
* [Unchecked external call](https://github.com/trailofbits/not-so-smart-contracts/tree/master/unchecked_external_call)
* [Scanning Live Ethereum Contracts for the "Unchecked-Send" Bug](http://hackingdistributed.com/2016/06/16/scanning-live-ethereum-contracts-for-bugs/)

## 5. 拒绝服务
包括**达到gas上限**，**非预期的throw**，**非预期的kill**，**访问控制被破坏**。
_我只是偶然将它kill掉。——devops199_

拒绝服务在以太坊世界里是致命的：其他类型的应用程序最终都可以恢复，然而智能合约只要遭遇这样的攻击就会永久下线。许多的方式可以导致拒绝服务，包括交易的接收者发出恶意的行为，人为的提高计算一个函数所需的gas，滥用访问控制来访问智能合约的私有组件，利用混淆和疏忽点，等等。这种类型的攻击包括很多的变种，我们将会在接下来几年里看到很多的发展。

**损失**  
预估514,874ETH（约合300M美元）

**现实案例影响**  
* [GovernMental](https://www.reddit.com/r/ethereum/comments/4ghzhv/governmentals_1100_eth_jackpot_payout_is_stuck/)
* [Parity Multi-sig wallet](http://paritytech.io/a-postmortem-on-the-parity-multi-sig-library-self-destruct/)

**示例**  
1. 一个`拍卖合约`允许用户对各种资产进行投标。
2. 想要投标，用户必须携带其竞标的ether额度调用`bid(uint object)`函数。拍卖合约将竞标的ether保存在第三方托管直到商品的所有者接受标的或初始竞标者取消标的。这就意味着拍卖合约必须保存未完成的竞标的资金。
3. `拍卖合约`同样有一个`withdraw(uint amount)`函数允许admin来提取合约中的资金。由于该函数将`amount`数量的ether发送到硬编码的固定地址，开发者决定将该函数设置为public。
4. `攻击者`看到了潜在的攻击场景，并调用了withdraw函数，将合约中所有的资金发送到admin的地址。这个行为破坏了第三方托管的承诺和阻塞了所有未完成的竞标。
5. 当然admin可以将资金退回到合约，但是攻击者可以不断的提现来持续发起攻击。

**代码示例**  
如下的示例（受以太王国所启发）中，游戏合约中的一个函数可以让你成为主席，如果你公开的贿赂前一个主席。不幸的是，如果前一个主席是一个智能合约，并且其拒绝接受资金，这样的话权利的转移会失败，恶意的合约将会永远是游戏的主席。听起来像是一个独裁主义：
```
function becomePresident() payable {
    require(msg.value >= price); // 必须完成支付才能成为主席
    president.transfer(price);   // 向前一个主席支付金额
    president = msg.sender;      // 为新主席加冕
    price = price * 2;           // 为主席设置价位
}
```

在第二个例子中，调用者可以决定谁是下一个奖励的调用。因为在`for`循环中有高昂费用的指令，攻击者可以指定一个特别大的数来遍历，（由于以太坊中gas的限制）会阻塞该函数的正常功能。
```
function selectNextWinners(uint256 _largestWinner) {
	for(uint256 i = 0; i < largestWinner, i++) {
		// heavy code
	}
	largestWinner = _largestWinner;
}
```

**其他资源**  
* [Parity Multisig Hacked. Again](https://medium.com/chain-cloud-company-blog/parity-multisig-hack-again-b46771eaa838)
* [Statement on the Parity multi-sig wallet vulnerability and the Cappasity token crowdsale](https://blog.artoken.io/statement-on-the-parity-multi-sig-wallet-vulnerability-and-the-cappasity-artoken-crowdsale-b3a3fed2d567)

# 6. 伪随机性
# 7. 前端运行
# 8. 时间操控
# 9. 短地址问题
# 10. 未知引起的新问题
