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

## 排序节点中的 PBFT

Fabric V1.x 中共识

# 参考文档

https://medium.com/coinmonks/how-a-miner-adds-transactions-to-the-blockchain-in-seven-steps-856053271476

https://medium.com/coinmonks/what-is-a-51-attack-or-double-spend-attack-aa108db63474

《mastering blockchain》
