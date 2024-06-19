---
title: 区块链入门
---

# 区块链入门

## 技术底层
北京大学肖臻老师《区块链技术与应用》公开课

<a-alert type="success" message="https://www.bilibili.com/video/BV1Vt411X7JF/?vd_source=46b87a83ff1ac8f06aa40d6801a00b21" description="" showIcon>
</a-alert>
<br/>

## 基本概念

### 公链 Public Blockchain

公有链(Public Blockchain)简称公链，是指全世界任何人都可随时进入读取、任何人都能发送交易且能获得有效确认的共识区块链。公链通常被认为是完全去中心化的，链上数据都是公开透明的，不可更改，任何人都可以通过交易或挖矿读取和写入数据。一般会通过代币机制(Token)来鼓励参与者竞争记账，来确保数据的安全性。

### 交易所 Exchang

与买卖股票的证券交易所类似，区块链交易所即数字货币买卖交易的平台。数字货币交易所又分为中心化交易所和去中心化交易所。
去中心化交易所：交易行为直接发生在区块链上，数字货币会直接发回使用者的钱包，或是保存在区块链上的智能合约。这样直接在链上交易的好处在于交易所不会持有用户大量的数字货币，所有的数字货币会储存在用户的钱包或平台的智能合约上。去中心化交易通过技术手段在信任层面去中心化，也可以说是无需信任，每笔交易都通过区块链进行公开透明，不负责保管用户的资产和私钥等信息，用户资金的所有权完全在自己手上，具有非常好的个人数据安全和隐私性。

### 中心化交易所

目前热门的交易所大多都是采用中心化技术的交易所，使用者通常是到平台上注册，并经过一连串的身份认证程序(KYC)后，就可以开始在上面交易数字货币。用户在使用中心化交易所时，其货币交换不见得会发生在区块链上，取而代之的可能仅是修改交易所数据库内的资产数字，用户看到的只是账面上数字的变化，交易所只要在用户提款时准备充足的数字货币可供汇出即可。当前的主流交易大部分是在中心化交易所内完成的，目前市面上的中心化交易所有币安，火币，OKEx 等。
### 节点 Node

在传统互联网领域，企业所有的数据运行都集中在一个中心化的服务器中，那么这个服务器就是一个节点。由于区块链是去中心化的分布式数据库，是由千千万万个“小服务器”组成。区块链网络中的每一个节点，就相当于存储所有区块数据的每一台电脑或者服务器。所有新区块的生产，以及交易的验证与记帐，并将其广播给全网同步，都由节点来完成。节点分为“全节点”和“轻节点”，全节点就是拥有全网所有的交易数据的节点，那么轻节点就是只拥有和自己相关的交易数据节点。由于每一个全节点都保留着全网数据，这意味着，其中一个节点出现问题，整个区块链网络世界也依旧能够安全运行，这也是去中心化的魅力所在。

### 共识 Consensus

共识算法主要是解决分布式系统中，多个节点之间对某个状态达成一致性结果的问题。分布式系统由多个服务节点共同完成对事务的处理，分布式系统中多个副本对外呈现的数据状态需要保持一致性。由于节点的不可靠性和节点间通讯的不稳定性，甚至节点作恶，伪造信息，使得节点之间出现数据状态不一致性的问题。通过共识算法，可以将多个不可靠的单独节点组建成一个可靠的分布式系统，实现数据状态的一致性，提高系统的可靠性。

区块链系统本身作为一个超大规模的分布式系统，但又与传统的分布式系统存在明显区别。由于它不依赖于任何一个中央权威，系统建立在去中心化的点对点网络基础之上，因此分散的节点需要就交易的有效与否达成一致，这就是共识算法发挥作用的地方，即确保所有节点都遵守协议规则并保证所有交易都以可靠的方式进行。由共识算法实现在分散的节点间对交易的处理顺序达成一致，这是共识算法在区块链系统中起到的最主要作用。

区块链系统中的共识算法还承担着区块链系统中激励模型和治理模型中的部分功能，为了解决在对等网络中(P2P),相互独立的节点如何达成一项决议问题的过程。简而言之，共识算法是在解决分布式系统中如何保持一致性的问题。

### 工作量证明 PoW(Proof of Work)

PoW(Proof of Work)是历史上第一个成功的去中心化区块链共识算法。工作量证明是大多数人所熟悉的，被比特币、以太坊，莱特币等主流公链广泛使用。
工作量证明要求节点参与者执行计算密集型的任务，但是对于其他网络参与者来说易于验证。在比特币的例子中，矿工竞相向由整个网络维护的区块链账本中添加所收集到的交易，即区块。为了做到这一点，矿工必须第一个准确计算出“nonce”，这是一个添加在字符串末尾的数字，用来创建一个满足开头特定个数为零的哈希值。不过存在采矿的大量电力消耗和低交易吞吐量等缺点。

### 权益证明 PoS(Proof of Stake)

PoS(Proof of Stake)——权益证明机制，一种主流的区块链共识算法，目的是为了让区块链里的分布式节点达成共识，它往往和工作量证明机制(Proof of Work)一起出现，两种都被认为是区块链共识算法里面的主流算法之一。作为一种算法，它通过持币人的同意来达成共识，目的是确定出新区块，这过程相对于 PoW，不需要硬件和电力，且效率更高。
PoS 共识中引入了 Stake 的概念，持币人将代币进行 Staking，要求所有的参与者抵押一部分他们所拥有的 Token 来验证交易，然后获得出块的机会，PoS 共识中会通过选举算法，按照持币量比例以及 Token 抵押时长，或者是一些其他的方式，选出打包区块的矿工。矿工在指定高度完成打包交易，生成新区块，并广播区块，广播的区块经过 PoS 共识中另外一道"门槛"，验证人验证交易，通过验证后，区块得到确认。这样一轮 PoS 的共识过程就进行完成了。权益证明通过长期绑定验证者的利益和整个网络的利益来阻止不良行为。锁定代币后，如果验证者存在欺诈性交易，那么他们所抵押的 Token 也会被削减。
PoS 的研究脚步还在不断前进，安全、性能和去中心化一直都是 PoS 所追求的方向，未来也将有更多 PoS 的项目落地。

### 委托权益证明 DPoS(Delegate Proof of Stake)

委托权益证明，其雏形诞生在 2013 年 12 月 8 日，Daniel Larimer 在 bitsharetalk 首次谈及用投票选择出块人的方式，代替 PoS 中可能出现的选举随机数被操纵的问题。在 DPoS 中，让每一个持币者都可以进行投票，由此产生一定数量的代表 ,或者理解为一定数量的节点或矿池，他们彼此之间的权利是完全相等的。持币者可以随时通过投票更换这些代表，以维系链上系统的“长久纯洁性”。在某种程度上，这很像是国家治理里面的代议制，或者说是人大代表制度。这种制度最大的好处就是解决了验证人过多导致的效率低下问题，当然，这种制度也有很明显的缺点，由于 “代表”制度，导致其一直饱受中心化诟病。

### 钱包Wallet
钱包(Wallet)是一个管理私钥的工具，数字货币钱包形式多样，但它通常包含一个软件客户端，允许使用者通过钱包检查、存储、交易其持有的数字货币。它是进入区块链世界的基础设施和重要入口。
冷钱包 Cold Wallet
冷钱包(Cold Wallet)是一种脱离网络连接的离线钱包，将数字货币进行离线储存的钱包。使用者在一台离线的钱包上面生成数字货币地址和私钥，再将其保存起来。冷钱包是在不需要任何网络的情况下进行数字货币的储存，因此黑客是很难进入钱包获得私钥的，但它也不是绝对安全的，随机数不安全也会导致这个冷钱包不安全，此外硬件损坏、丢失也有可能造成数字货币的损失，因此需要做好密钥的备份。
### 热钱包 Hot Wallet

热钱包(Hot Wallet)是一种需要网络连接的在线钱包，在使用上更加方便。但由于热钱包一般需要在线使用，个人的电子设备有可能因误点钓鱼网站被黑客盗取钱包文件、捕获钱包密码或是破解加密私钥，而部分中心化管理钱包也并非绝对安全。因此在使用中心化交易所或钱包时，最好在不同平台设置不同密码，且开启二次认证，以确保自己的资产安全。

### 公钥 Public Key

公钥(Public Key)是和私钥成对出现的，和私钥一起组成一个密钥对，保存在钱包中。公钥由私钥生成，但是无法通过公钥倒推得到私钥。公钥能够通过一系列算法运算得到钱包的地址，因此可以作为拥有这个钱包地址的凭证。

### 私钥 Private Key

私钥(Private Key)是一串由随机算法生成的数据，它可以通过非对称加密算法算出公钥，公钥可以再算出币的地址。私钥是非常重要的，作为密码，除了地址的所有者之外，都被隐藏。区块链资产实际在区块链上，所有者实际只拥有私钥，并通过私钥对区块链的资产拥有绝对控制权，因此，区块链资产安全的核心问题在于私钥的存储，拥有者需做好安全保管。
和传统的用户名、密码形式相比，使用公钥和私钥交易最大的优点在于提高了数据传递的安全性和完整性，因为两者——对应的关系，用户基本不用担心数据在传递过程中被黑客中途截取或修改的可能性。同时，也因为私钥加密必须由它生成的公钥解密，发送者也不用担心数据被他人伪造。

### 助记词 Mnemonic
由于私钥是一长串毫无意义的字符，比较难以记忆，因此出现了助记词(Mnemonic)。助记词是利用固定算法，将私钥转换成十多个常见的英文单词。可以简单认为助记词和私钥功能是类似的，它只是作为区块链数字钱包私钥的友好格式。所以在此强调：助记词同私钥一样重要！由于它的明文性，不建议它以电子方式保存，而是抄写在物理介质上保管好，它和 Keystore 作为双重备份互为补充。

### Keystore

Keystore 主要在以太坊钱包 App 中比较常见(比特币类似以太坊 Keystore 机制的是：BIP38)，是把私钥通过钱包密码再加密得来的，与助记词不同，一般可保存为文本或 JSON 格式存储。换句话说，Keystore 需要用钱包密码解密后才等同于私钥。因此，Keystore 需要配合钱包密码来使用，才能导入钱包。当黑客盗取 Keystore 后，在没有密码情况下, 有可能通过暴力破解 Keystore 密码解开 Keystore，所以建议使用者在设置密码时稍微复杂些，比如带上特殊字符，至少 8 位以上，并安全存储。