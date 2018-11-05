智能合约安全
===================

Solidity项目在2014年开始，当时以太坊网络和虚拟机都还没有在真实世界得到测试，gas消耗方案也与现在有很大的区别。另外，早期的一些设计决定是从Serpent继承过来的。在过去的几个月里，一些最初被认为是最佳实践的示例和模式，在现在的以太坊中却是违反规则的。正因为此，我们最近更新了[Solidiy文档](https://solidity.readthedocs.org/)，但是很多人很可能并没有关注这个文档库在github的更新流。所以，我在这里对其中一些改变做一个重点强调。

我并不会讲那些低级的小问题，请你自行阅读[Solidity文档](http://solidity.readthedocs.io/en/latest/miscellaneous.html#pitfalls)。

---

# 发送Ether
发送ether在Solidity里边是最简单的事情之一，但是很多人可能并没有意识到其中一些细微的注意点。

最好的方案是以太坊的接收者发起支付。如下是一个拍卖合约中糟糕的示例：
```
// 这是一个错误的示例，请不要使用！
contract auction {
  address highestBidder;
  uint highestBid;
  function bid() {
    if (msg.value < highestBid) throw;
    if (highestBidder != 0)
      highestBidder.send(highestBid); // 退款给前一个投标者
    highestBidder = msg.sender;
    highestBid = msg.value;
  }
}
```

因为EVM栈最大深度为1024，所以新的投标者可以将栈垫高到1023再调用 `bid()`，这将导致其中调用`send(highestBid)`会失败（例如，前一个投标者收不到退款），但是新的投标者却会始终是出价最高的投标者。当然，可以通过检查`send`函数的返回值来判断是否发送ether成功（成功，返回true；否则，返回false）。

```
/// 这依然是一个错误的示例，请不要使用！
if (highestBidder != 0)
  if (!highestBidder.send(highestBid))
    throw;

throw语句会导致当前的调用被回滚恢复。这是一个不好的主意！例如，退款接收者可以在接收合约实现fallback函数成这样：function(){throw;}，它会一直使得ether转账失败，这样的结果是没有人可以竞标过他（其他所有人的投标都会失败）。
```

唯一可以防止这两种情况发生的方法是将`主动发送模式`转变成`授权退款接收者自行提取退款的模式`。
```
///  这依然是一个错误的示例，请不要使用！
contract auction {
  address highestBidder;
  uint highestBid;
  mapping(address => uint) refunds;
  function bid() {
    if (msg.value < highestBid) throw;
    if (highestBidder != 0)
      refunds[highestBidder] += highestBid;
    highestBidder = msg.sender;
    highestBid = msg.value;
  }
  function withdrawRefund() {
    if (msg.sender.send(refunds[msg.sender]))
      refunds[msg.sender] = 0;
  }
}
```
为什么说这个合约依然是一个“错误示例”？由于gas机制，这个合约实际上很好，但依然不是一个好的示例。原因是其无法阻止接收者在`send`函数调用时进行代码执行。这意味着`send`函数执行过程中，退款接收者可以再次调用`withdrawRefund`函数。而在`send`这个执行点，退款额度并没有改变，其可以再次获得相应额度的退款，并可以不停的迭代提取。在这个特殊的例子里，攻击不会生效，因为接收者仅仅获得有效的gas支助（21,000 gas），这样它就无法发起另外一个`send`操作。下面的代码，会使得这个攻击有效： `msg.sender.call.value(refunds[msg.sender])();`

综合上述考量，下面合约代码是好的（当然其并不是一个完整的拍卖合约示例）。
```
contract auction {
  address highestBidder;
  uint highestBid;
  mapping(address => uint) refunds;
  function bid() {
    if (msg.value < highestBid) throw;
    if (highestBidder != 0)
      refunds[highestBidder] += highestBid;
    highestBidder = msg.sender;
    highestBid = msg.value;
  }
  function withdrawRefund() {
    uint refund = refunds[msg.sender];
    refunds[msg.sender] = 0;
    if (!msg.sender.send(refund))
     refunds[msg.sender] = refund;
  }
}
```

我们并没有在发送ether失败时使用`throw`语句，因为其会手动的恢复所有的状态操作。不使用`throw`可以省去很多其带来的副作用。

# 使用throw
`throw`语句可以很方便的恢复当前调用对状态进行的一些改变。同时，你也应该意识到它将可能消耗所有的gas，也可能导致其他的调用不能进入这个函数。所以，建议仅仅在以下情况使用`throw`：

**在当前函数中恢复ether的转账。**
如果一个函数不是用来接收ether，或者当前状态不符合，或者参数不符合预期，使用`throw`来拒绝接收ether是一个好主意。

**恢复被调用函数产生的改变。**
在一个函数调用其他合约中的函数，你不会知道它是怎么实现的，这意味着你不知道他做了哪些修改。所以，这种情况下恢复状态唯一的方法是使用`throw`语句。

---

原文链接： https://blog.ethereum.org/2016/06/10/smart-contract-security/
