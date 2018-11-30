# Bitcoin

## Bitcoin 交易处理的7个步骤

1、用户通过自己的钱包APP（轻客户端） 发起并签名一笔交易；

2、钱包APP 会发送交易到就近的矿工节点，矿工节点验证交易之后会放到本地“未被确认的交易池”中，并广播给其他节点（其他节点收到后也会放到本地的“未被确认的交易池”中）；

3、网络中的矿工会从交易池中选择交易并打包到本地区块内；

4、矿工需要本地挖矿，通过不断调整 nonce 值来找到满足 difficulty 的区块头 Hash 值；

5、算出有效的区块头 Hash 值之后，矿工会广播该区块给网络中的其他节点；

6、其他节点收到区块后会首先验证区块是否有效，如果有效就加到本地的区块链上；

7、如果多数矿工都对这个块达成共识（会加到本地的区块链上）可以认为该区块内的交易上链了，但最终的确认还需要等待一定的区块高度（和下面讲的双花攻击相关）。

## 双花攻击

由于基于 PoW 的 bitcoin 是全网中所有的节点来各自延长本地认为最长的链来达成共识的，这就会导致全网视角的链在某些时刻有可能分叉（由于网络延迟等原因），但这也同样会被拥有高算力的攻击者利用来进行“双花”攻击。

所谓“双花”攻击就是：一笔 bitcoin 做了两笔不同的交易（其中一笔的交易方会损失本该得到的 bitcoin）。

核心思路是：先在链的某个块高度 H 花掉 bitcoin，然后再从块高度 H 之前的块开始延长链的另外一个分叉，就最终让另外一个分叉成为最长的链，从而使之前在高度 H 完成交易的链无效。具体见下图。

1、攻击者从 Block 39 开始本地挖矿和出块来延长自己的链，但并不广播后续的区块到网络；

![image](https://github.com/nil-zhang/consensus/blob/master/images/double-spend-1.png)

2、攻击者在当前网络的 Block 40 花掉 100 BTC，但在本地链 Block 40 里面并未包含自己的交易；

![image](https://github.com/nil-zhang/consensus/blob/master/images/double-spend-2.png)

3、然后攻击者开始广播本地链的区块，使网络中的链在 Block 38 之后分叉，并持续延长自己的分叉来使网络中的多数节点认为攻击者的分叉是最长的链，从而使之前花掉 100 BTC 交易所在的分叉无效。

![image](https://github.com/nil-zhang/consensus/blob/master/images/double-spend-5.png)

# HyperLedger Fabric

前提：由于是联盟链，成员都需要 CA中心 等机制来完成身份认证。

## Fabric 处理交易的9个步骤

1、客户端提交交易并发送给背书节点（endorsing peers）；

2、背书节点模拟执行交易并生成交易相关的读写集，这里背书节点执行了交易的合约代码（也是合约唯一执行的地方）但并没有更新账本；

3、经过背书的交易（包含读写集）会再次返回给客户端；

4、客户端提交背书交易和读写集到排序节点（ordering peers）；

5、排序节点对交易进行分 channel 的排序并打包进区块；

6、排序节点广播区块给所有的记账节点（committing peers）；

7、记账节点验证交易；

8、记账节点更新账本；

9、记账节点给客户端返回交易的执行结果。

![image](https://github.com/nil-zhang/consensus/blob/master/images/The%20transaction%20flow%20of%20fabric.jpg)

## Fabric 中的 PBFT
Fabric V0.6 中是支持 PBFT 共识协议的，但后来由于性能问题被暂停支持；

Fabric V1.x 中共识机制是通过排序节点（ordering）来实现的，主要是负责区块内交易的排序，目前支持的是 SOLO（test） 和 Kafka（product），后续会支持 SBFT。

为什么交易的排序在共识中如此重要，其实 PBFT 共识是 state machine replication 模型，该模型的两个基本条件是：

1、所有的节点基于一个给定的状态执行一个操作（交易）的结果是确定的而且是相同的；

2、所有的节点从同一个状态开始执行。

有了上面的条件，***共识算法只要保证所有的节点在执行交易的时候对交易的编号是达成一致的，就能保证所有节点最终的state也是一致的***；这也就是交易排序对共识算法的重要性所在。

同时 Fabric 还引入了新的交易执行模型：***execute-order-validate***。

Bitcoin 和 Ethereum 区块的交易模型都是：order-execute，就是先排序再执行，这就要求智能合约代码执行的结果必须是确定的，所以只能使用 solidity 等受限制的合约语言。

但 Fabric 把交易的执行和账本的更新分开处理，就是先执行，再排序，最后验证和更新账本；

这样的好处是：

1、可以用任意 x86 语言来编写合约；

2、同时并行执行交易来提高系统吞吐；

3、同时可以通过不同的背书策略来做细粒度的隐私保护。

BTW：由于 BFT 算法具有及时中止性，不存在分叉，所有不会有双花攻击。

# 参考文档

https://medium.com/coinmonks/how-a-miner-adds-transactions-to-the-blockchain-in-seven-steps-856053271476

https://medium.com/coinmonks/what-is-a-51-attack-or-double-spend-attack-aa108db63474

mastering blockchain

Practical Byzantine Fault Tolerance
