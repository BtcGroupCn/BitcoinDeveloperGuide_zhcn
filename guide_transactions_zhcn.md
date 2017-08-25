This file is licensed under the [MIT License (MIT)](http://opensource.org/licenses/MIT) available on http://opensource.org/licenses/MIT.

[对应英文文档](https://github.com/bitcoin-dot-org/bitcoin.org/blob/master/_includes/devdoc/guide_transactions.md)


# 交易
交易使用户可以消费他们的比特币。每个交易有多个部分组成，既包括简单直接的付款，也包括复杂交易。这一部分将会讨论交易的每个部分，并演示如果将它们组合成一个完整的交易。

简单起见，这部分将忽略生成交易。生成交易只能被比特币矿工创建，并且它们对下面的规则存在许多例外情况。这里建议读者在区块链一章了解生成交易的一些细节，本章将不在交易规则后面单独注明对生成交易的例外情况。

![image](http://note.youdao.com/yws/public/resource/580887bc0d4654b83202bc1e47db9af9/xmlnote/919B881287B741A19BAA6E01740C1773/115892)

上图展示了比特币交易的主要部分。每笔交易至少有一个输入和一个输出，每个输入会花费上个输出产生的比特币，每个输出都作为UTXO直到被作为输入花费掉。当你的比特币钱包告诉你你还有10000聪的比特币，它实际上说的是你拥有的一个或多个UTXO输出中包含10000聪。

每个交易的前缀为4字节的交易版本号，告诉比特币终端和矿工，采取什么样的策略验证它。这种设计可以让开发者在不影响旧区块的前提下为未来交易创建新的一致性规则。

![image](http://note.youdao.com/yws/public/resource/580887bc0d4654b83202bc1e47db9af9/xmlnote/EDED19CA4D414A399D552FE83952E137/115894)

一个输出有一个基于其在交易中位置的隐含索引号码，第一个输出的索引是0。同时，输出还有一定数量的比特币，这些比特币会支付给可以满足公钥脚本指定条件的人。任何可以满足该公钥脚本条件的人可以消费这些比特币。

一个输出使用交易标识和输出索引号（常被称为vout，代指输出向量output vector）来区分一个将要使用的输出。同时，它还有一个签名脚本，可以用来提供满足输出公钥脚本的条件的参数。（序列号和锁定时间是相关的，将在后续章节介绍）

下面的图片通过展示Alice给Bob一些币然后Bob把币花掉两个交易过程，反映出这些特性具体是如何被使用的。Alice和Bob都将使用最常见的P2PKH(Pay-To-Public-Key-Hash)的交易方式。P2PKH使Alice将比特币支付给一个地址，然后Bob再使用简单的密钥对继续将这些比特币花掉。

![link](http://note.youdao.com/yws/public/resource/580887bc0d4654b83202bc1e47db9af9/xmlnote/B62CE1A7DDDE4452AA3998034055A169/115896)

Bob必须首先创建一个公钥/私钥对，然后Alice才能创建第一各交易。比特币使用ECDSA签名算法结合secp256k1，secp256k1私钥是一个256bit的随机数。这个随机数被确定性地转换为一个secp256k1的公钥。由于这个转换过程是确定可重复的，因此不需要在本地保存公钥。

对公钥(pbkey)使用加密hash得到hash值，公钥hash过程也是可重复的，因此也没有必要保存该hash结果。Hash可以是公钥变短并模糊，方便手动誊写，并且提供一定的安全性保证防止未来不可预知的通过公钥重建私钥的问题。

Bob把公钥hash提供给Alice。公钥hash通常像比特币地址一样进行编码后再进行传输，编码方式为使用base68对地址版本号、hash值、错误校验和编码为一个字符串。地址可以通过任何的媒介进行阐述，包括防止花费者联系接受者的单项没接，同时还可以进一步编码为另外的格式，比如包含`bitcoin:`URI的二维码。

Alice收到该地址后会把它转换为标准hash后，就可以进行第一次交易了。她将创建一个包含指令的标准的P2PKH交易输出，允许任何可以证明自己拥有Bob提供的公钥对应私钥的人花费该笔输出。输出中的指令被称为公钥脚本。

Alice将交易广播出去后该交易会被添加到区块链。网络将他分类为UTXO，此时Bob的钱包软件将它展示为可花费的现金。

当后面Bob决定花费这份UTXO时，他必须创建一个指向Alice通过他提供的hash创建的交易的输入，称为交易标志（Transcation Identifier, txid），然后通过输出所以指定她使用的输出。Bob要创建一份签名，改签名包含满足Alice在签一份输出中保存的公钥脚本的条件。签名脚本又被称为ScriptSigs。

公钥脚本、签名脚本、secp256k1公钥、签名加上条件逻辑，可以创建一个可编程授权机制。

![image](http://note.youdao.com/yws/public/resource/580887bc0d4654b83202bc1e47db9af9/xmlnote/75D786EB038F4D9DA3DBD79CA5F5AEF9/115898)

对于一个P2PKH类型的输出，Bob的签名脚本将要包含如下两条数据：
- 他的完整的公钥（非hash），因此公钥脚本可以检查它可以hash得到和Alice提供的值一样的结果。
- 一个通过ECDSA加密算法利用指定的交易数据（下面讲解）结合Bob的私钥计算得到的secp256k1签名。这使得公钥脚本可以校验Bob持有对应公钥的私钥。

Bob的secp256k1签名不仅仅证明Bob控制着自己的私钥，同时还可以防止他交易中的非签名脚本部分被攻击，以保证Bob可以在p2p网络中安全地广播自己的交易。

![image](http://note.youdao.com/yws/public/resource/580887bc0d4654b83202bc1e47db9af9/xmlnote/8CC7360A8D0A48229C526ADB87452479/115900)

如上图所示，Bob签名的数据包括上次交易的txid和输出所以、上次交易的输出公钥脚本、Bob创建用于后续接受者花费这次交易输出的公钥脚本、本次转支付给下一个接收者的比特币数量。本质上来讲，除了包含公钥和secp256k1签名的签名脚本外整条交易的内容都被做了签名。

当把他的签名和公钥放到签名脚本后，Bob把这次交易通过p2p网络广播道所有的比特币矿工。每个节点和矿工在进一步广播或尝试把它添加到新区块前独立的校验交易信息。

## P2PKH 脚本验证
验证过程需要执行签名脚本和公钥脚本。在一个P2PKH的输出，公钥脚本为：
```
OP_DUP OP_HASH160 <PubkeyHash> OP_EQUALVERIFY OP_CHECKSIG
```
花费者的签名脚本被执行后添加到脚本的前面。在一个P2PKH交易中，签名脚本包含secp256k1（sig）签名和完整的公钥（pubkey），创建如下的串行过程：
```
<Sig> <PubKey> OP_DUP OP_HASH160 <PubkeyHash> OP_EQUALVERIFY OP_CHECKSIG
```
脚本语言是一种类Forth基于堆栈的语言，专门设计为无状态和非图灵完备的。无状态可以保证一旦一个交易被添加到区块链，没有条件能够表明它一直未被消费。非图灵完备（主要指没有循环和goto语句）可以使脚本不过于灵活便于预测，可以大大简化安全模型。

为了测试交易是否有效，签名脚本和公钥脚本中的操作一次执行一条，从Bob的签名脚本开始知道Alice的公钥脚本结束。下面的图片展示了标准P2PKH 公钥脚本的执行，图片下面是对执行过程的描述。

![link](http://note.youdao.com/yws/public/resource/580887bc0d4654b83202bc1e47db9af9/xmlnote/962C38FFDD254AD69C52FAEAA74F8296/116173)

1. 签名（来自Bob的签名脚本）被添加到一个空栈中。由于它只是数据而已，所以不需要做其他的工作。公钥Key（也是来自签名脚本）添加在签名的上方。
2. 对于Alice的公钥脚本，首先执行最开始的`OP_DUP`操作。`OP_DUP`把当前堆栈顶部的数据复制一份再压入堆栈，这里指的是复制一份Bob提供的公钥。
3. 接下来执行`OP_HASH160`操作，得到堆栈顶部数据的Hash值并放在堆栈中。这里是创建的Bob的公钥的hash值。
4. 接下来Alice的公钥脚本将之前Bob在第一次交易中提供的公钥hash值入栈。到这里，堆栈顶部包含了2份Bob公钥的hash值。
5. 从这里开始变的有趣：Alice的公钥脚本执行`OP_EQUALVERIFY`操作。`OP_EQUALVERIFY`等价于顺序执行`OP_EQUAL`和`OP_VERIFY`（未显示）。    
    `OP_EQUAL`（未显示）价差堆栈顶部的两个元素。这里是检查由Bob提供的公钥的hash值和Alice在第一交易中获得的公钥Hash值是否相等。`OP_EQUAL`将栈顶参与比较的两个值出栈，并把它们的比较结果（0表示false，1表示true）压入堆栈。     
    `OP_VERIFY`（未显示）检查栈顶元素。如果这个值是false，它立即结束校验过程，当前交易校验结果失败。否则它将栈顶的true元素弹出。
6. 最终，Alice的公钥脚本执行`OP_CHECKSIG`，通过刚刚检验过的Bob提供的公钥来检查Bob提供的用私钥产生的签名。如果签名跟公钥匹配，并且其中的内容时使用都是需要签名的数据，`OP_CHECKSIG`把true添加到堆栈顶部。

如果执行完公钥脚本后最终栈顶元素不是false，则判定交易有效（假定这里交易没有其他的问题）。

## P2SH脚本

公钥脚本由支出者创建，支出者对脚本内容的兴趣其实不大。实际收款人才是真正关心脚本条件的人，并且如果他们愿意，他们可以让支出者是哟个指定的公钥脚本。不幸的是，配置的公钥脚本没有较短的比特币地址简单，并且没有标准的方式能够把他们和BIP70（后面进行讨论）广泛应用前的程序结合起来。

为了解决这些问题，P2SH（pay-to-script-hash，支付到脚本hash）交易在2012年产生，让支出者创建一个包含兑换脚本hash值的公钥脚本。

基本的P2SH工作流程示例如下，它看起来和P2PKH工作流几乎相同。Bob使用自己喜欢的任意的脚本作为兑换脚本，计算它的hash值，然后将兑换脚本的hash值提供给Alice。Alice创建一个P2SH版本的输出，其中包含了Bob的兑换脚本的hash值。
![image](http://note.youdao.com/yws/public/resource/580887bc0d4654b83202bc1e47db9af9/xmlnote/5250794517F146339A62842CC01ED364/116175)

当BObo想花掉这笔输出，他在签名脚本中将完整的兑换脚本和签名一起提供。p2p网络保证完整的兑换脚本hash值和Alice在她的输出中存入的hash值相同，然后就像执行公钥脚本一样执行该兑换脚本，即如果兑换脚本返回非false则允许Bob花费这笔输出。

![image](http://note.youdao.com/yws/public/resource/580887bc0d4654b83202bc1e47db9af9/xmlnote/EADBCDB1F6B24918A96053F519E693EB/116177)

兑换脚本的hash值和公钥的hash值具有同样的特点，因此它也可以一种比特币地址格式，但是为了区分它和标准地址格式需要对它做一些小的改变。存储P2SH格式的地址和存储P2PKH格式的地址一样简单。如果P2PKH公钥hash一样，它的hash值同样会隐藏公钥值，所以二者同样安全。

## 标准交易
比特币在发现了几个早期版本的危险Bug之后，增加了一个测试。该测试仅仅来自指信任网络的交易，信任网络需满足1)公钥脚本和签名脚本匹配一个信任模板小集合，2)该网络的其他交易没有违反另外一个保证网络行为正常的集合的内容。这就是`IsStandard()`测试，传递给该测试的交易称为**标准交易**。

非标准交易是指那些未通过测试的交易，它们仍然可能被没有使用默认比特币核心设置的节点接受。如果他们被包含到区块当中，同样需要避免`IsStandard()`测试和被处理。

除了使通过广播有害交易方式攻击比特币变得更加困难外，标准交易测试还可以帮助防止“用户今天创建区块使未来增加新的交易特性更加困难”的情况。举例来说，如上面所述，每个交易都包含一个版本号，如果用户任意修改版本号的数值，那么未来使用版本号作为后向兼容特性工具的措施就会失效。

对于比特币核心0.9，标准的公钥脚本类型如下：

### P2PKH（支付给公钥hash，Pay To Public Key Hash）
P2PKH是用来发送交易到一个或多个比特币地址最常用的公钥脚本格式。
```
Pubkey script: OP_DUP OP_HASH160 <PubKeyHash> OP_EQUALVERIFY OP_CHECKSIG
Signature script: <sig> <pubkey>
```

### P2SH（支付给脚本Hash，Pay To Script Hash）
P2SH用于将一笔交易发送到一个脚本hash地址。每个标准的公钥脚本都可以作为一个P2SH兑换脚本，但是实际应用中只有多签名公约脚本有效，其他的暂时都不是标准交易格式。

```
Pubkey script: OP_HASH160 <Hash160(redeemScript)> OP_EQUAL
Signature script: <sig> [sig] [sig...] <redeemScript>
```

### 多签名（Multisig）
尽管P2SH多签名现在通常用于多签名交易，这种基础脚本也可以在UTXO可被消费前用于请求多重签名。

在多签名公共脚本中，被称为m-ofn，m代表满足公钥的最少签名个数；n代表提供的公钥的个数。m和n应为`OP_1`到`OP_16`操作码对应的数字之一。

由于必须对原始比特币实现中的off-by-one错误保持兼容性，`OP_CHECKMULTISIG`比对应的m多消耗一个参数，所以签名脚本中对应的secp256k1 签名需要以一个额外的`OP_0`操作码开始，该值被消耗弹出但是实际不使用。

签名脚本提供签名中对应公钥需要和公钥脚本及兑换脚本以同样的顺序，`OP_CHECKMULTISIG`细节参见下面的描述：
```
Pubkey script: <m> <A pubkey> [B pubkey] [C pubkey...] <n> OP_CHECKMULTISIG
Signature script: OP_0 <A sig> [B sig] [C sig...]
```
下面是一个2-of-3的P2SH多重签名的示例：
```
Pubkey script: OP_HASH160 <Hash160(redeemScript)> OP_EQUAL
Redeem script: <OP_2> <A pubkey> <B pubkey> <C pubkey> <OP_3> OP_CHECKMULTISIG
Signature script: OP_0 <A sig> <C sig> <redeemScript>
```

### 公钥
公钥输出是P2PKH公钥脚本的一种简单形式，但是不如P2PKH安全，现在已基本不再使用。
```
Pubkey script: <pubkey> OP_CHECKSIG
Signature script: <sig>
```

### 空数据(待修正)
从比特币核心0.9.0版本开始空数据交易默认进行转发和挖掘，后来它增加了使用任意数据来确认为未花费公钥脚本，全节点不需要把这些数据保存在它们的UTXO数据库中。空数据交易不能被自动裁剪，因此鼓励在交易中使用空数据交易来扩充UTXO数据库。但是，如果可能的话把数据保存在交易的外是更好的。

在没有违反其他一致性规则的前提下，允许的空数据输出最大为公钥脚本大小10,000字节，因此任何推送数据都不能超过520字节。

默认情况下，比特币核心0.9.x到0.10.x单个数据推送可转发和挖掘的空数据交易可以达到40字节，并且只有一个空数据输出支付金额为0。
```
Pubkey Script: OP_RETURN <0 to 40 bytes of data>
(Null data scripts cannot be spent, so there's no signature script.)
```
比特币核心0.11.x把这个默认值增加到80字节，同时其他规则保持不变。

比特币核心0.12.0版本，在没有超出总字节限制的前提下，默认的转发和挖掘的空数据输出可达83字节及任意长度的数据推送。这里还是需要保证只能有1个空数据输出并且它的支付金额为0。

比特币核心配置项`-datacarriersize`允许设定你愿意转发和挖掘的最大空数据输出大小。

## 非标准交易
如果你不遵循输出中使用标准公钥脚本，使用默认比特币核心配置的节点和矿工将不会接受、广播，也不会打包你的交易。当并试图向你使用默认比特币核心配置伙伴节点广播你的交易时将会收到一个错误。

如果你创建一个兑换脚本，对它做hash，然后在P2SH输出中使用这个hash值，配比特币网络只能看到这个hash值，此时无论兑换脚本内容如何他们都会认为这个交易有效并接受它。这样就可以允许非标准脚本支付，从比特币核心0.11开始，几乎所有的有效兑换脚本都可以成功支付。但是使用了未标记的NOP操作符的脚本除外，这些操作符被保留待未来软分叉使用，它们只能被不遵循标准矿池协议的节点处理。

> 注意：标准交易的涉及是为了保护比特币网络，不是阻止用户犯错。创造一个比特币不可消费的标准交易是很容易的。

从比特币核心0.9.3开始，标准交易必须遵循如下条件：
- 交易必须是可终止的。要么它的锁定时间在过去（或小于等于当前区块高度），要么它所有序列号都是0xffffffff。
- 交易大小必须小于100,000字节。这大约是典型单输入单输出P2PKH交易的200倍。
- 交易中的每个签名脚本都必须小于1650字节。这各大小已经足够采用压缩公钥的15-of-15的P2SH多签名交易使用。
- 需要超过3个公钥的裸多签名交易（非P2SH）现在是非标准的。
- 交易签名脚本只能向脚本执行栈中压入数据和只有压入数据功能的执行码，不能压入其他类型的执行码。
- 当接收少于典型交易花费的1/3时，交易当中不能包含任何输入。在比特币核心采用默认中继费用时，当前对于P2PKH或P2SH输出該值为546聪。这里有各例外，标准的空数据交易只能接收0聪。

## 签名Hash类型
`OP_CHECKSIG`从每个它要验证的签名中抽取一个非栈参数，允许签名者决定对交易的哪个部分签名。由于签名可以保证这些被签名的部分不被更改，这可以让签名者选择是否允许其他人修改他们的交易。

指定签名元素的选项称作hash类型，当前共有3中基本的SIGHASH类型可用：
- `SIGHASH_ALL`，默认的，对所有的输入和输出签名，保护除了签名脚本外其他所有内容不被更改。
- `SIGHASH_NONE`，所有输入签名，所有输出不签名，除非使用其他的签名类型进行签名保护输出，否则即为允许任何人可以更改比特币的趋向。
- `SIGHASH_SINGLE`，仅对本输入对应的输出进行签名（拥有和输入索引相同的数字的输出索引），确保任何人都不能更改交易中属于你的部分，但是允许其他人改变该交易中属于他们自己的部分。对应的输出必须存在或者破坏安全方案将值“1”签名。所有的输入都被包含在签名中，但是其他输入对应的输入都没有被包含在签名中，都是可被修改的。

基本的hash类型可以和`SIGHASH_ANYONECANPAY`(any one can pay)组合，产生三种新的组合类型：
- `SIGHASH_ALL | SIGHASH_ANYONECANPAY`，签名所有的输入和当前输入，允许其他任何人添加或移除其他输入，所以任何人都能贡献额外的比特币，但是他们不能控制发送的数量和目的地。
- `SIGHASH_NONE | SIGHASH_ANYONECANPAY`，签名当前输入，允许其他任何人添加或移除其他的输入和输出，所以任何人获取当前输入的副本后都可以任意花费。
- `SIGHASH_SINGLE | SIGHASH_ANYONECANPAY`，签名当前输入和对应的输出。允许任何人添加和移除他们自己的输入。

由于每个输入都被签名，一个多输入交易可以有多种签名类型来签名交易的不同部分。比如，一个单输入交易使用None签名可以允许它的输出被把该交易放到区块链上的矿工修改。另一方面，一个辆输入的交易可以有一个签名为None，另一个签名为All，签名为all的签名者可以单独决定如何花费交易中的金额，但是没人能修改交易中的内容。

## 锁定时间和序列号
所有的签名hash类型都进行签名的一个字段是locktime。（在比特币核心代码中称为nLocakTime。）locktime代指交易能被添加到区块链的最早时间。

locktime允许签名者创建一个时间锁交易，该交易仅在未来有效，给交易双方可以更改决定的机会。

如果任何一个签名者改变了想法，他们可以创建一个新的非锁定交易。新的交易将使用原来locktime输入使用的相同的输出。如果新的交易在锁定时间前加入到区块链，这样将会使之前提交的locktime交易失效。

之前版本的比特币核心提供了一个特性，防止交易签名者使用上面的方法取消时间锁定交易，但是该特性的一个必须方面为了防止否认服务攻击很快被停止了。由该特性引入的为每个输入指定一个4字节序列号被保留下来。序列号意味着允许多重签名者统一更新这个交易，当他们完成对交易的更新以后，他们可以统一把所有的输入序列号设置为4字节最大值0xffffffff，一次允许该交易打破时间锁限制添加到区块链。

即使现在，设置序列号为0xffffffff（比特币核心默认）仍然可以打破时间锁；因此，如果你想使用时间锁，则至少一个输入的序列号需要设置为小于0xffffffff的值。由于序列号现在已不应用在其他地方，可以在设置时间锁的时候把序列号设置为0。

所动时间是一个无符号4字节证书，可以以两种方式解释：
- 如果小于500,000,000，锁定时间会被认为是区块高度。交易可以被添加到高度大于等于该值的区块中。
- 如果大于等于500,000,000，锁定时间被认为是unix时间戳（从UTC 1970年1月1日0时0分0秒到现在的秒数，现在已经超过1395000）格式的表示。交易可以被添加到块时间大于该锁定时间的区块中。

## 交易费用和找零
交易费用的支付基于签名交易总的字节大小。每字节费用是通过当前区块空间需求程度决定的，需求程度越大费用越高。交易费用被支付给比特币矿工，前面`区块链`一节已经提到，最终交易费用是由比特币矿工选择他们能够接受的最小交易费用决定。

这里还有一个概念称为“高优先级交易”，通过花费更多的交易费用来避免等待较长时间。

在之前 ，这种优先交易不跟正常交易费用需求一起处理。在比特币核心0.12以前，每个区块预留50KB给这些高优先级交易，但是现在已经默认把预留空间设置为0。不使用专门的优先级区域后，所有的交易按它们的每字节交易费用排列优先级，按照单字节交易费用由高到低的顺序添加到区块中直到填满区块所有空间。

从比特币核心0.9开始，为在比特币网络中广播交易指定了最小费用（当前是1000聪）。任何只支付最小交易费用的交易需要做需要等待较长时间才能被打包到稀缺的区块空间当中的准备。关于这种策略的重要意义，请查看`验证交易支付`一节。

由于每个交易都要花费UTXOs，而每个UTXO都只能被花费一次，因此UTXO当中的比特币必须一次花费完或作为交易费用支付给比特币矿工。很少有人持有的UTXOs中比特币的个数就是他们刚好要支付的个数，因此绝大多数交易都有一个找零输出。

找零输出是消耗一个UTXO支出后的剩余返还给支出者的常规输出。它们可以复用与UTXO相同的P2PKH公钥hash或P2SH脚本hash，但是考虑到下节将要讨论的原因，强烈建议找零输出使用一个新的hash地址。

## 避免Key重用
在交易当中，支出者和接收者都拥有彼此在交易中使用所有的公钥或地址。他们中的任何一个人都可以利用这些信息在公共区块链上追溯对方过去和未来使用相同的公钥的交易。

如果经常重用同一个公钥，这在人们使用一个地址（hash的公钥）作为比特币固定接收地址时容易出现，其他人可以很容易的追溯该地址的比特币来源和去向，以及当前该地址控制多少比特币。

上面的问题很容易解决。如果一个公钥只使用两次，一次接收，一次消费，那么用户就可以拥有很多的金融安全性。

如果更进一步，如果每次接收付款或找零时都使用一个新的公钥或不重复的比特币地址，可以跟其他技术结合（如下面将要讨论的交易联合或融合避免），使仅通过区块链本身可信地追溯用户收支过程十分困难。

避免蜜月重用还可以防止从公钥或签名对比（在特定情况下，现在更可能出现假定）来重建私钥的安全攻击。

- 非重用不重复的P2PKH和P2SH地址可以防止第一种类型的攻击。直到第一次比特币付款时才把ECDSA公钥暴露出来，除非攻击者能在一两个小时以内即区块未受妥善保护前重建私钥，否则就是无用的。
- 非重用不重复的私钥可以防止第二种类型的攻击。每个私钥只产生一个签名，因此攻击者无法获得试用一个私钥生成的一些列签名来进行比对攻击。当前的比对攻击只有当随机性不够或随机状态被破解时才会有效，比如边带攻击。

因此，无论从隐私角度，还是从安全角度，我们都建议你在创建自己的程序时避免重用公钥；如果可能，让用户优先选择使用新地址。如果你的程序要提供一个固定的接受付款的URI，可以参照下面URI一节的`bitcoin:`。

## 交易重塑
所有比特币签名hash类型都不保护签名脚本，这样就留下了一个受限的否定服务的后门，成为交易重塑。签名脚本包含secp256K1，它不能签名自己，这就给攻击者在不改变交易校验结果的情况下伪造一个非功能性修改。比如，攻击者可能在签名脚本中添加一些数据，这些数据被前一个交易的公约脚本处理掉。

尽管修改是非功能性的，不会改变交易使用的输入和输出结果，但是它们改变了交易的hash值。由于每个交易和前一个交易都是通过或hash得到的交易标识相互关联，修改以后得到的交易标识是交易创建者预料之外的。

这对大多数比特币立即添加到区块链的交易来说，这不是问题。但是对于使用该笔交易的后续交易先添加到区块链的情况就有问题了。

比特币开发者一直致力于减少标准交易中的重塑交易，其中的一个成果是BIP 141：隔离见证，该技术已经被比特币核心支持但是尚未启用。目前，新的交易不能依赖未添加到区块链的历史交易，尤其是至关重要的大宗比特币交易。

交易重塑同样影响支付追踪。比特币核心RPC接口允许用户通过txid追踪交易，但是如果由于交易内容更改导致txid改变，就可能导致交易在区块链上消失。

当前交易追踪最佳实践规定，新交易应该可以通过作为其输入的历史交易的输出追踪到，因为只要历史交易不失效该交易就不能更改。

最佳实践进一步规定，如果一个交易可能真的从网络中消失了，那么它需要重新发布，发布的方式应使之前的旧交易失效。一种可行的方式为保证重新发布的交易把丢失交易使用的交易输出全部花完。


