# 比特幣：一種點對點的電子現金系統

​								作者：中本聰
​								satoshin@gmx.com
​								www.bitcoin.org 
​								2008.10.31

​								中文翻譯：李笑來
​								lixiaolai@gmail.com
​								2018.10.31

​								[Checkout Github Repo for this translation](https://github.com/xiaolai/bitcoin-whitepaper-chinese-translation)



> **Abstract.** A purely peer-to-peer version of electronic cash would allow online payments to be sent directly from one party to another without going through a financial institution. Digital signatures provide part of the solution, but the main benefits are lost if a trusted third party is still required to prevent double-spending. We propose a solution to the double-spending problem using a peer-to-peer network. The network timestamps transactions by hashing them into an ongoing chain of hash-based proof-of-work, forming a record that cannot be changed without redoing the proof-of-work. The longest chain not only serves as proof of the sequence of events witnessed, but proof that it came from the largest pool of CPU power. As long as a majority of CPU power is controlled by nodes that are not cooperating to attack the network, they'll generate the longest chain and outpace attackers. The network itself requires minimal structure. Messages are broadcast on a best effort basis, and nodes can leave and rejoin the network at will, accepting the longest proof-of-work chain as proof of what happened while they were gone. 
>
> **概要**：一個純粹的點對點版本的電子現金系統，將允許在線支付直接從一方發送到另一方，而無需通過金融機構。數字簽名雖然提供了部分解決方案，但，若是仍然需要被信任的第三方來防止雙重支出的話，那麼電子支付的主要優勢就被抵消了。我們提出一個方案，使用點對點網絡去解決雙重支出問題。點對點網絡將爲每筆交易標記時間戳，方法是：把交易的散列數據錄入一個不斷延展的、以散列爲基礎的工作證明鏈上，形成一個如非完全重做就不可能改變的記錄。最長鏈，一方面用來證明已被見證的事件及其順序，與此同時，也用來證明它來自於最大的 CPU 算力池。只要絕大多數 CPU 算力被良性節點控制 —— 即，它們不與那些嘗試攻擊網絡的節點合作 —— 那麼，良性節點將會生成最長鏈，並且在速度上超過攻擊者。這個網絡本身需要最小化的結構。信息將以最大努力爲基本去傳播，節點來去自由；但，加入之時總是需要接受最長的工作證明鏈作爲它們未參與期間所發生之一切的證明。

-----

## 1. 簡介 (Introduction)


Commerce on the Internet has come to rely almost exclusively on financial institutions serving as trusted third parties to process electronic payments. While the system works well enough for most transactions, it still suffers from the inherent weaknesses of the trust based model. Completely non-reversible transactions are not really possible, since financial institutions cannot avoid mediating disputes. The cost of mediation increases transaction costs, limiting the minimum practical transaction size and cutting off the possibility for small casual transactions, and there is a broader cost in the loss of ability to make non-reversible payments for non-reversible services. With the possibility of reversal, the need for trust spreads. Merchants must be wary of their customers, hassling them for more information than they would otherwise need. A certain percentage of fraud is accepted as unavoidable. These costs and payment uncertainties can be avoided in person by using physical currency, but no mechanism exists to make payments over a communications channel without a trusted party.

互聯網商業幾乎完全依賴金融機構作爲可信第三方去處理電子支付。雖然針對大多數交易來說，這個系統還算不錯，但，它仍然被基於信任的模型所固有的缺陷所拖累。完全不可逆轉的交易實際上並不可能，因爲金融機構不能避免仲裁爭議。仲裁成本增加了交易成本，進而限制了最小可能交易的規模，且乾脆阻止了很多小額支付交易。除此之外，還有更大的成本：系統無法爲那些不可逆的服務提供不可逆的支付。逆轉的可能性，造成了對於信任的需求無所不在。商家必須提防著他們的顧客，麻煩顧客提供若非如此（如若信任）就並不必要的更多信息。一定比例的欺詐，被認爲是不可避免的。這些成本和支付不確定性，雖然在人與人之間直接使用物理貨幣支付的時候是可以避免的；但，沒有任何一個機制能在雙方在其中一方不被信任的情況下通過溝通渠道進行支付。

What is needed is an electronic payment system based on cryptographic proof instead of trust, allowing any two willing parties to transact directly with each other without the need for a trusted third party. Transactions that are computationally impractical to reverse would protect sellers from fraud, and routine escrow mechanisms could easily be implemented to protect buyers. In this paper, we propose a solution to the double-spending problem using a peer-to-peer distributed timestamp server to generate computational proof of the chronological order of transactions. The system is secure as long as honest nodes collectively control more CPU power than any cooperating group of attacker nodes.

我們真正需要的是一種基於加密證明而非基於信任的電子支付系統，允許任意雙方在不需要信任第三方的情況下直接交易。算力保障的不可逆轉交易能幫助賣家不被欺詐，而保護買家的日常擔保機制也很容易實現。在本論文中，我們將提出一種針對雙重支出的解決方案，使用點對點的、分佈式的時間戳服務器去生成基於算力的證明，按照時間順序記錄每條交易。此係統是安全的，只要誠實節點總體上相對於相互合作的攻擊者掌握更多的 CPU 算力。

## 2. 交易 (Transactions)

We define an electronic coin as a chain of digital signatures. Each owner transfers the coin to the next by digitally signing a hash of the previous transaction and the public key of the next owner and adding these to the end of the coin. A payee can verify the signatures to verify the chain of ownership.

我們將一枚電子硬幣定義爲一個數字簽名鏈。一位所有者將一枚硬幣交給另一個人的時候，要通過在這個數字簽名鏈的末尾附加上以下數字簽名：上一筆交易的哈希（hash，音譯，亦翻譯爲「散列值」），以及新所有者的公鑰。收款人可以通過驗證簽名去驗證數字簽名鏈的所屬權。

![](images/transactions.svg)

The problem of course is the payee can't verify that one of the owners did not double-spend the coin. A common solution is to introduce a trusted central authority, or mint, that checks every transaction for double spending. After each transaction, the coin must be returned to the mint to issue a new coin, and only coins issued directly from the mint are trusted not to be double-spent. The problem with this solution is that the fate of the entire money system depends on the company running the mint, with every transaction having to go through them, just like a bank.

這個路徑的問題在於收款人無法驗證曾經的所有者之中沒有人雙重支付過。常見的解決方案是引入一個可信的中心化權威方，或稱「鑄幣廠」，讓它去檢查每一筆交易是否存在雙重支付。每一次發生交易之後，硬幣必須返回到鑄幣廠，鑄幣廠再發行一枚新的硬幣。進而，只有鑄幣廠直接發行的硬幣纔是可信的、未被雙重支付過的。這個解決方案的問題在於，整個貨幣系統的命運被拴在運營鑄幣廠的那個公司（就好像銀行那樣）身上，每一筆交易必須通過它。

We need a way for the payee to know that the previous owners did not sign any earlier transactions. For our purposes, the earliest transaction is the one that counts, so we don't care about later attempts to double-spend. The only way to confirm the absence of a transaction is to be aware of all transactions. In the mint based model, the mint was aware of all transactions and decided which arrived first. To accomplish this without a trusted party, transactions must be publicly announced[^1], and we need a system for participants to agree on a single history of the order in which they were received. The payee needs proof that at the time of each transaction, the majority of nodes agreed it was the first received.

我們需要一種方式，可以讓收款人確認之前的所有者並沒有在任何之前的交易上簽名。就我們的目的而言，只有最早的交易是算數的，所以，我們並不關心其後的雙重支付企圖。確認一筆交易不存在的唯一方法是獲悉所有的交易。在鑄幣廠模型之中，鑄幣廠已然知悉所有的交易，並且能夠確認這些交易的順序。爲了能在沒有「被信任的一方」參與的情況下完成以上任務，交易記錄必須被公開宣佈[^1]，進而我們需要一個系統能讓參與者們認同它們所接收到的同一個唯一的交易歷史。收款人需要證明在每筆交易發生之時，大多數節點能夠認同它是第一個被接收的。

## 3. 時間戳服務器 (Timestamp Server)

The solution we propose begins with a timestamp server. A timestamp server works by taking a hash of a block of items to be timestamped and widely publishing the hash, such as in a newspaper or Usenet post[^2] [^3] [^4] [^5]. The timestamp proves that the data must have existed at the time, obviously, in order to get into the hash. Each timestamp includes the previous timestamp in its hash, forming a chain, with each additional timestamp reinforcing the ones before it.

本解決方案起步於一種時間戳服務器。時間戳服務器是這樣工作的：爲一組（block）記錄（items）的哈希打上時間戳，而後把哈希廣播出去，就好像一份報紙所做的那樣，或者像是在新聞組（Usenet）裏的一個帖子那樣[^2] [^3] [^4] [^5]。顯然，時間戳能夠證明那數據在那個時間點之前已然存在，否則那哈希也就無法生成。每個時間戳在其哈希中包含著之前的時間戳，因此構成了一個鏈；每一個新的時間戳被添加到之前的時間戳之後。

![](images/timestamp-server.svg)

## 4. 工作證明 (Proof-of-Work)

To implement a distributed timestamp server on a peer-to-peer basis, we will need to use a proof-of-work system similar to Adam Back's Hashcash[^6], rather than newspaper or Usenet posts. The proof-of-work involves scanning for a value that when hashed, such as with SHA-256, the hash begins with a number of zero bits. The average work required is exponential in the number of zero bits required and can be verified by executing a single hash.

爲了實現一個基於點對點的分佈式時間戳服務器，我們需要使用類似亞當·伯克的哈希現金[^6]那樣的一個工作證明系統，而不是報紙或者新聞組帖子那樣的東西。所謂的工作證明，就是去尋找一個數值；這個數值要滿足以下條件：爲它提取散列數值之後 —— 例如使用 SHA-256 計算散列數值 —— 這個散列數值必須以一定數量的 0 開頭。每增加一個 0 的要求，將使得工作量指數級增加，並且，這個工作量的驗證卻只需通過計算一個哈希。

For our timestamp network, we implement the proof-of-work by incrementing a nonce in the block until a value is found that gives the block's hash the required zero bits. Once the CPU effort has been expended to make it satisfy the proof-of-work, the block cannot be changed without redoing the work. As later blocks are chained after it, the work to change the block would include redoing all the blocks after it.

在我們的時間戳網絡中，我們是這樣實現工作證明的：不斷在區塊之中增加一個隨機數（Nonce），直到一個滿足條件的數值被找到；這個條件就是，這個區塊的哈希以指定數量的 0 開頭。一旦 CPU 的耗費算力所獲的的結果滿足工作證明，那麼這個區塊將不再能被更改，除非重新完成之前的所有工作量。隨著新的區塊不斷被添加進來，改變當前區塊即意味著說要重新完成所有其後區塊的工作。

![](images/proof-of-work.svg)

The proof-of-work also solves the problem of determining representation in majority decision making. If the majority were based on one-IP-address-one-vote, it could be subverted by anyone able to allocate many IPs. Proof-of-work is essentially one-CPU-one-vote. The majority decision is represented by the longest chain, which has the greatest proof-of-work effort invested in it. If a majority of CPU power is controlled by honest nodes, the honest chain will grow the fastest and outpace any competing chains. To modify a past block, an attacker would have to redo the proof-of-work of the block and all blocks after it and then catch up with and surpass the work of the honest nodes. We will show later that the probability of a slower attacker catching up diminishes exponentially as subsequent blocks are added.

工作證明同時解決了如何決定誰能代表大多數做決定的問題。如果所謂的「大多數」是基於「一個IP地址一票」的方式決定的話，那麼任何一個可以搞定很多 IP 地址的人就可以被認爲是「大多數」。工作證明本質上來看，是「一個CPU一票」。所謂的「大多數決定」是由最長鏈所代表的，因爲被投入最多工作的鏈就是它。如果大多數 CPU 算力被誠實的節點所控制，那麼誠實鏈成長最爲迅速，其速度會遠超其他競爭鏈。爲了更改一個已經產生的區塊，攻擊者將不得不重新完成那個區塊以及所有其後區塊的的工作證明，而後還要追上並超過誠實節點的工作。後文展示爲什麼一個被拖延了的攻擊者能夠追上的可能性將隨著區塊的不斷增加而指數級降低。

To compensate for increasing hardware speed and varying interest in running nodes over time, the proof-of-work difficulty is determined by a moving average targeting an average number of blocks per hour. If they're generated too fast, the difficulty increases.

爲了應對硬件算力綜合的不斷增加，以及隨著時間推進可能產生的節點參與數量變化，工作證明難度由此決定：基於平均每小時產生的區塊數量的一個移動平均值。如果區塊生成得過快，那麼難度將會增加。

## 5. 網絡 (Network)

The steps to run the network are as follows:

> 1. New transactions are broadcast to all nodes.
> 2. Each node collects new transactions into a block.
> 3. Each node works on finding a difficult proof-of-work for its block.
> 4. When a node finds a proof-of-work, it broadcasts the block to all nodes.
> 5. Nodes accept the block only if all transactions in it are valid and not already spent.
> 6. Nodes express their acceptance of the block by working on creating the next block in the chain, using the hash of the accepted block as the previous hash.

運行網絡的步驟如下：

> 1. 所有新的交易向所有節點廣播；
> 2. 每個節點將新交易打包到一個區塊；
> 3. 每個節點開始爲此區塊找一個具備難度的工作證明；
> 4. 當某個區塊找到其工作證明，它就要將此區塊廣播給所有節點；
> 5. 衆多其他節點當且只當以下條件滿足纔會接受這個區塊：其中所有的交易都是有效的，且未被雙重支付；
> 6. 衆多節點向網絡表示自己接受這個區塊的方法是，在創建下一個區塊的時候，把被接受區塊的哈希當作新區塊之前的哈希。

Nodes always consider the longest chain to be the correct one and will keep working on extending it. If two nodes broadcast different versions of the next block simultaneously, some nodes may receive one or the other first. In that case, they work on the first one they received, but save the other branch in case it becomes longer. The tie will be broken when the next proof-of-work is found and one branch becomes longer; the nodes that were working on the other branch will then switch to the longer one.

節點始終認爲最長鏈是正確的那個，且會不斷向其添加新數據。若是有兩個節點同時向網絡廣播了兩個不同版本的「下一個區塊」，有些節點會先接收到其中一個，而另外一些節點會先接收到另外一個。這種情況下，節點將在它們先接收到的那個區塊上繼續工作，但也會把另外一個分支保存下來，以防後者成爲最長鏈。當下一個工作證明被找到，而其中的一個分支成爲更長的鏈之後，這個暫時的分歧會被打消，在另外一個分支上工作的節點們會切換到更長的鏈上。

New transaction broadcasts do not necessarily need to reach all nodes. As long as they reach many nodes, they will get into a block before long. Block broadcasts are also tolerant of dropped messages. If a node does not receive a block, it will request it when it receives the next block and realizes it missed one.

新的交易不見得一定要廣播到達所有的節點。只要到達足夠多的節點，那麼沒多久這些交易就會被打包進一個區塊。區塊廣播也容許一些消息被丟棄。如果一個節點並未接收到某個區塊，那麼這個節點會在它接收到下一個區塊的時候意識到自己錯失了之前的區塊，因此會發出補充那個遺失區塊的請求。

## 6. 獎勵 (Incentive)

By convention, the first transaction in a block is a special transaction that starts a new coin owned by the creator of the block. This adds an incentive for nodes to support the network, and provides a way to initially distribute coins into circulation, since there is no central authority to issue them. The steady addition of a constant of amount of new coins is analogous to gold miners expending resources to add gold to circulation. In our case, it is CPU time and electricity that is expended.

按照約定，每個區塊的第一筆交易是一個特殊的交易，它會生成一枚新的硬幣，所屬權是這個區塊的生成者。這麼做，使得節點支持網絡有所獎勵，也提供了一種將硬幣發行到流通之中的方式 —— 在這個系統中，反正也沒有一箇中心化的權威方去發行那些硬幣。如此這般穩定地增加一定數量的新硬幣進入流通，就好像是黃金開採者不斷耗用他們的資源往流通之中增加黃金一樣。在我們的系統中，被耗用的資源是 CPU 工作時間和它們所用的電力。

The incentive can also be funded with transaction fees. If the output value of a transaction is less than its input value, the difference is a transaction fee that is added to the incentive value of the block containing the transaction. Once a predetermined number of coins have entered circulation, the incentive can transition entirely to transaction fees and be completely inflation free.

獎勵還可以來自交易費用。如果一筆交易的輸出值小於它的輸入值，那麼其中的差額就是交易費；而該交易費就是用來獎勵節點把該交易打包進此區塊的。一旦既定數量的硬幣已經進入流通，那麼獎勵將全面交由交易手續費來完成，且絕對不會有通貨膨脹。

The incentive may help encourage nodes to stay honest. If a greedy attacker is able to assemble more CPU power than all the honest nodes, he would have to choose between using it to defraud people by stealing back his payments, or using it to generate new coins. He ought to find it more profitable to play by the rules, such rules that favour him with more new coins than everyone else combined, than to undermine the system and the validity of his own wealth.

獎勵機制也可能會鼓勵節點保持誠實。如果一個貪婪的攻擊者能夠網羅比所有誠實節點都更多的 CPU 算力，他必須做出一個選擇：是用這些算力通過把自己花出去的錢偷回來去欺騙別人呢？還是用這些算力去生成新的硬幣？他應該能夠發現按照規則行事是更划算的，當前規則使得他能夠獲得比所有其他人加起來都更多的硬幣，這顯然比暗中摧毀系統並使自己的財富化爲虛無更划算。

## 7. 回收硬盤空間 (Reclaiming Disk Space)

Once the latest transaction in a coin is buried under enough blocks, the spent transactions before it can be discarded to save disk space. To facilitate this without breaking the block's hash, transactions are hashed in a Merkle Tree[^2][^5][^7], with only the root included in the block's hash. Old blocks can then be compacted by stubbing off branches of the tree. The interior hashes do not need to be stored.

如果一枚硬幣最近發生的交易發生在足夠多的區塊之前，那麼，這筆交易之前該硬幣的花銷交易記錄可以被丟棄 —— 目的是爲了節省磁盤空間。爲了在不破壞該區塊的哈希的前提下實現此功能，交易記錄的哈希將被納入一個 Merkle 樹[^2][^5][^7]之中，而只有樹根被納入該區塊的哈希之中。通過砍掉樹枝方法，老區塊即可被壓縮。內部的哈希並不需要被保存。

![](images/reclaiming-disk-space.svg)

A block header with no transactions would be about 80 bytes. If we suppose blocks are generated every 10 minutes, 80 bytes * 6 * 24 * 365 = 4.2MB per year. With computer systems typically selling with 2GB of RAM as of 2008, and Moore's Law predicting current growth of 1.2GB per year, storage should not be a problem even if the block headers must be kept in memory.

一個沒有任何交易記錄的區塊頭大約是 80 個字節。假設每十分鐘產生一個區塊，80 字節乘以 6 乘以 24 乘以 365，等於每年 4.2M。截止 2008 年，大多數在售的計算機配有 2GB 內存，而按照摩爾定律的預測，每年會增加 1.2 GB，即便是區塊頭必須存儲在內存之中也不會是什麼問題。

## 8. 簡化版支付確認 (Simplified Payment Verification)

It is possible to verify payments without running a full network node. A user only needs to keep a copy of the block headers of the longest proof-of-work chain, which he can get by querying network nodes until he's convinced he has the longest chain, and obtain the Merkle branch linking the transaction to the block it's timestamped in. He can't check the transaction for himself, but by linking it to a place in the chain, he can see that a network node has accepted it, and blocks added after it further confirm the network has accepted it.

即便不用運行一個完整網絡節點也有可能確認支付。用戶只需要有一份擁有工作證明的最長鏈的區塊頭拷貝 —— 他可以通過查詢在線節點確認自己擁有的確實來自最長鏈 —— 而後獲取 Merkle 樹的樹枝節點，進而連接到這個區塊被打上時間戳時的交易。用戶並不能自己檢查交易，但，通過連接到鏈上的某個地方，他可以看到某個網絡節點已經接受了這個交易，而此後加進來的區塊進一步確認了網絡已經接受了此筆交易。

![](images/simplified-payment-verification.svg)

As such, the verification is reliable as long as honest nodes control the network, but is more vulnerable if the network is overpowered by an attacker. While network nodes can verify transactions for themselves, the simplified method can be fooled by an attacker's fabricated transactions for as long as the attacker can continue to overpower the network. One strategy to protect against this would be to accept alerts from network nodes when they detect an invalid block, prompting the user's software to download the full block and alerted transactions to confirm the inconsistency. Businesses that receive frequent payments will probably still want to run their own nodes for more independent security and quicker verification.

只要誠實節點依然在掌控網絡，如此這般，驗證即爲可靠的。然而，如果網絡被攻擊者所控制的時候，驗證就沒那麼可靠了。儘管網絡節點可以自己驗證交易記錄，但是，只要攻擊者能夠繼續控制網絡的話，那麼簡化版驗證方式可能會被攻擊者僞造的交易記錄所欺騙。應對策略之一是，客戶端軟件要接受來自網絡節點的警告。當網絡節點發現無效區塊的時候，即發出警報，在用戶的軟件上彈出通知，告知用戶下載完整區塊，警告用戶確認交易一致性。那些有高頻收付發生的商家應該仍然希望運行屬於自己的完整節點，以此保證更獨立的安全性和更快的交易確認。

## 9. 價值的組合與分割 (Combining and Splitting Value)

Although it would be possible to handle coins individually, it would be unwieldy to make a separate transaction for every cent in a transfer. To allow value to be split and combined, transactions contain multiple inputs and outputs. Normally there will be either a single input from a larger previous transaction or multiple inputs combining smaller amounts, and at most two outputs: one for the payment, and one returning the change, if any, back to the sender.

儘管逐個地處理硬幣是可能的，但爲每分錢設置一個單獨的記錄是很笨拙的。爲了允許價值的分割與合併，交易記錄包含多個輸入和輸出。一般情況下，要麼是一個單獨的來自於一個相對大的之前的交易的輸入，要麼是很多個輸入來自於更小金額的組合；與此同時，最多有兩個輸出：一個是支付（指向收款方），如果必要的話，另外一個是找零（指向發款方）。

![](images/combining-splitting-value.svg)

It should be noted that fan-out, where a transaction depends on several transactions, and those transactions depend on many more, is not a problem here. There is never the need to extract a complete standalone copy of a transaction's history.

值得注意的是，「扇出」在這裏並不是問題 —— 所謂「扇出」，就是指一筆交易依賴於數筆交易，且這些交易又依賴於更多筆交易。從來就沒有必要去提取任何一筆交易的完整獨立的歷史拷貝。

## 10. 隱私 (Privacy)

The traditional banking model achieves a level of privacy by limiting access to information to the parties involved and the trusted third party. The necessity to announce all transactions publicly precludes this method, but privacy can still be maintained by breaking the flow of information in another place: by keeping public keys anonymous. The public can see that someone is sending an amount to someone else, but without information linking the transaction to anyone. This is similar to the level of information released by stock exchanges, where the time and size of individual trades, the "tape", is made public, but without telling who the parties were.

傳統的銀行模型通過限制他人獲取交易者和可信第三方的信息而達成一定程度的隱私保護。出於對將所有交易記錄公開的需求否決了這種方法。但是，維持隱私可通過於另一處的切斷信息流來實現——公鑰匿名。公衆可以看到某某向某某轉賬了一定的金額，但是，沒有任何信息指向某個確定的人。這種水平的信息發佈有點像股市交易，只有時間和各個交易的金額被公佈，但是，沒有人知道交易雙方都是誰。

![](images/privacy.svg)

As an additional firewall, a new key pair should be used for each transaction to keep them from being linked to a common owner. Some linking is still unavoidable with multi-input transactions, which necessarily reveal that their inputs were owned by the same owner. The risk is that if the owner of a key is revealed, linking could reveal other transactions that belonged to the same owner.

還有另外一層防火牆。交易者應該針對每一筆交易啓用一對新的公私鑰，以便他人無法將這些交易追溯到同一個所有者身上。有些多輸入的交易依然難免被追溯，因爲那些輸入必然會被識別出來自於同一個所有者。危險在於，如果一個公鑰的所有者被曝光之後，與之相關的所有其他交易都會被曝光。

## 11. 計算 (Calculations)

We consider the scenario of an attacker trying to generate an alternate chain faster than the honest chain. Even if this is accomplished, it does not throw the system open to arbitrary changes, such as creating value out of thin air or taking money that never belonged to the attacker. Nodes are not going to accept an invalid transaction as payment, and honest nodes will never accept a block containing them. An attacker can only try to change one of his own transactions to take back money he recently spent.

假設一個場景，某個攻擊者正在試圖生成一個比誠實鏈更快的替代鏈。就算他成功了，也不會使當前系統置於模稜兩可的尷尬境地，即，他不可能憑空製造出價值，也無法獲取從未屬於他的錢。網絡節點不會把一筆無效交易當作支付，而誠實節點也永遠不會接受一個包含這種支付的區塊。攻擊者最多隻能修改屬於他自己的交易，進而試圖取回他已經花出去的錢。

The race between the honest chain and an attacker chain can be characterized as a Binomial Random Walk. The success event is the honest chain being extended by one block, increasing its lead by +1, and the failure event is the attacker's chain being extended by one block, reducing the gap by -1.

誠實鏈和攻擊者之間的競爭可以用二項式隨機漫步來描述。成功事件是誠實鏈剛剛被添加了一個新的區塊，使得它的優勢增加了 $1$；而失敗事件是攻擊者的鏈剛剛被增加了一個新的區塊，使得誠實鏈的優勢減少了 $1$。

The probability of an attacker catching up from a given deficit is analogous to a Gambler's Ruin problem. Suppose a gambler with unlimited credit starts at a deficit and plays potentially an infinite number of trials to try to reach breakeven. We can calculate the probability he ever reaches breakeven, or that an attacker ever catches up with the honest chain, as follows[^8]:

攻擊者能夠從落後局面追平的概率類似於賭徒破產問題。假設，一個拿著無限籌碼的賭徒，從虧空開始，允許他賭無限次，目標是填補上已有的虧空。我們能算出他最終能填補虧空的概率，也就是攻擊者能夠趕上誠實鏈的概率[^8]，如下：

$$
\begin{eqnarray*}
      \large p &=& \text{ 誠實節點找到下一個區塊的概率}\\
      \large q &=& \text{ 攻擊者找到下一個區塊的概率}\\
      \large q_z &=& \text{ 攻擊者落後 $z$ 個區塊卻依然能夠趕上的概率}
\end{eqnarray*}
$$

$$
\large q_z = \begin{Bmatrix}
				1 & \textit{if}\; p \leq q\\
				(q/p)^z & \textit{if}\; p > q
				\end{Bmatrix}
$$

Given our assumption that $p \gt q​$, the probability drops exponentially as the number of blocks the attacker has to catch up with increases. With the odds against him, if he doesn't make a lucky lunge forward early on, his chances become vanishingly small as he falls further behind.

既然我們已經假定 $p > q$, 既然攻擊者需要趕超的區塊數量越來越多，那麼其成功概率就會指數級下降。於贏面不利時，如果攻擊者沒有在起初就能幸運地做一個前移步刺，那麼他的勝率將在他進一步落後的同時消弭殆盡。

We now consider how long the recipient of a new transaction needs to wait before being sufficiently certain the sender can't change the transaction. We assume the sender is an attacker who wants to make the recipient believe he paid him for a while, then switch it to pay back to himself after some time has passed. The receiver will be alerted when that happens, but the sender hopes it will be too late.

現在考慮一下一筆新交易的收款人需要等多久才能充分確定發款人不能更改這筆交易。我們假定發款人是個攻擊者，妄圖讓收款人在一段時間裏相信他已經支付對付款項，隨後將這筆錢再轉回給自己。發生這種情況時，收款人當然會收到警告，但發款人希望那時木已成舟。

The receiver generates a new key pair and gives the public key to the sender shortly before signing. This prevents the sender from preparing a chain of blocks ahead of time by working on it continuously until he is lucky enough to get far enough ahead, then executing the transaction at that moment. Once the transaction is sent, the dishonest sender starts working in secret on a parallel chain containing an alternate version of his transaction.

收款人生成了一對新的公私鑰，而後在簽署之前不久將公鑰告知發款人。這樣可以防止一種情形：發款人提前通過連續運算去準備一條鏈上的區塊，並且只要有足夠的運氣就會足夠領先，直到那時再執行交易。一旦款項已被髮出，那個不誠實的發款人開始祕密地在另一條平行鏈上開工，試圖在其中加入一個反向版本的交易。

The recipient waits until the transaction has been added to a block and $z$ blocks have been linked after it. He doesn't know the exact amount of progress the attacker has made, but assuming the honest blocks took the average expected time per block, the attacker's potential progress will be a Poisson distribution with expected value:

收款人等到此筆交易被打包進區塊，並已經有 $z$ 個區塊隨後被加入。他並不知道攻擊者的工作進展究竟如何，但是可以假定誠實區塊在每個區塊生成過程中耗費的平均時間；攻擊者的潛在進展符合泊松分佈，其期望值爲：

$$
\large \lambda = z \frac qp
$$

To get the probability the attacker could still catch up now, we multiply the Poisson density for each amount of progress he could have made by the probability he could catch up from that point:

爲了算出攻擊者依然可以趕上的概率，我們要把每一個攻擊者已有的進展的帕鬆密度乘以他可以從那一點能夠追上來的概率：

$$
\large \sum_{k=0}^{\infty} \frac{\lambda^k e^{-\lambda}}{k!} \cdot
				\begin{Bmatrix}
				(q/p)^{(z-k)} & \textit{if}\;k\leq z\\
				1 & \textit{if} \; k > z
				\end{Bmatrix}
$$

Rearranging to avoid summing the infinite tail of the distribution...

爲了避免對密度分佈的無窮級數求和重新整理…

$$
\large 1 - \sum_{k=0}^{z} \frac{\lambda^k e^{-\lambda}}{k!}
				\left ( 1-(q/p)^{(z-k)} \right )
$$

Converting to C code...

轉換爲 C 語言程序……

```c
#include <math.h>
double AttackerSuccessProbability(double q, int z)
{
	double p = 1.0 - q;
	double lambda = z * (q / p);
	double sum = 1.0;
	int i, k;
	for (k = 0; k <= z; k++)
	{
		double poisson = exp(-lambda);
		for (i = 1; i <= k; i++)
			poisson *= lambda / i;
		sum -= poisson * (1 - pow(q / p, z - k));
	}
	return sum;
}
```

Running some results, we can see the probability drop off exponentially with $z$.

獲取部分結果，我們可以看到概率隨著 $z$ 的增加指數級下降：

```
   q=0.1
   z=0    P=1.0000000
   z=1    P=0.2045873
   z=2    P=0.0509779
   z=3    P=0.0131722
   z=4    P=0.0034552
   z=5    P=0.0009137
   z=6    P=0.0002428
   z=7    P=0.0000647
   z=8    P=0.0000173
   z=9    P=0.0000046
   z=10   P=0.0000012
   
   q=0.3
   z=0    P=1.0000000
   z=5    P=0.1773523
   z=10   P=0.0416605
   z=15   P=0.0101008
   z=20   P=0.0024804
   z=25   P=0.0006132
   z=30   P=0.0001522
   z=35   P=0.0000379
   z=40   P=0.0000095
   z=45   P=0.0000024
   z=50   P=0.0000006
```

Solving for P less than 0.1%...

若是 P 小於 0.1%……

```
   P < 0.001
   q=0.10   z=5
   q=0.15   z=8
   q=0.20   z=11
   q=0.25   z=15
   q=0.30   z=24
   q=0.35   z=41
   q=0.40   z=89
   q=0.45   z=340
```


## 12. 結論 (Conclusion)

We have proposed a system for electronic transactions without relying on trust. We started with the usual framework of coins made from digital signatures, which provides strong control of ownership, but is incomplete without a way to prevent double-spending. To solve this, we proposed a peer-to-peer network using proof-of-work to record a public history of transactions that quickly becomes computationally impractical for an attacker to change if honest nodes control a majority of CPU power. The network is robust in its unstructured simplicity. Nodes work all at once with little coordination. They do not need to be identified, since messages are not routed to any particular place and only need to be delivered on a best effort basis. Nodes can leave and rejoin the network at will, accepting the proof-of-work chain as proof of what happened while they were gone. They vote with their CPU power, expressing their acceptance of valid blocks by working on extending them and rejecting invalid blocks by refusing to work on them. Any needed rules and incentives can be enforced with this consensus mechanism.

我們提出了一個不必依賴信任的電子交易系統；起點是一個普通的使用數字簽名的硬幣框架開始，雖然它提供了健壯的所有權控制，卻無法避免雙重支付。爲瞭解決這個問題，我們提出一個使用工作證明機制的點對點網絡去記錄一個公開的交易記錄歷史，只要誠實節點能夠控制大多數 CPU 算力，那麼攻擊者就僅從算力方面就不可能成功篡改系統。這個網絡的健壯在於它的無結構的簡單。節點們可以在很少協同的情況下瞬間同時工作。它們甚至不需要被辨認，因爲消息的路徑並非取決於特定的終點；消息只需要被以最大努力爲基本去傳播即可。節點來去自由，重新加入時，只需要接受工作證明鏈，作爲它們離線之時所發生之一切的證明。它們通過它們的 CPU 算力投票，通過不斷爲鏈添加新的有效區塊、拒絕無效區塊，去表示它們對有效交易的接受與否。任何必要的規則和獎勵都可以通過這個共識機制來強制實施。

-----

## 參考文獻 (References)


[^1]: **b-money** Dai Wei (1998-11-01) <http://www.weidai.com/bmoney.txt>
[^2]: **Design of a secure timestamping service with minimal trust requirements** Henri Massias, Xavier Serret-Avila, Jean-Jacques Quisquater *20th Symposium on Information Theory in the Benelux* (1999-05) <http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.13.6228>
[^3]: **How to time-stamp a digital document** Stuart Haber, W.Scott Stornetta *Journal of Cryptology* (1991) <https://doi.org/cwwxd4> DOI: [10.1007/bf00196791](https://doi.org/10.1007/bf00196791)
[^4]: **Improving the Efficiency and Reliability of Digital Time-Stamping** Dave Bayer, Stuart Haber, W. Scott Stornetta *Sequences II* (1993) <https://doi.org/bn4rpx> DOI: [10.1007/978-1-4613-9323-8_24](https://doi.org/10.1007/978-1-4613-9323-8_24)
[^5]: **Secure names for bit-strings** Stuart Haber, W. Scott Stornetta *Proceedings of the 4th ACM conference on Computer and communications security - CCS ’97*(1997) <https://doi.org/dtnrf6> DOI: [10.1145/266420.266430](https://doi.org/10.1145/266420.266430)
[^6]: **Hashcash - A Denial of Service Counter-Measure** Adam Back (2002-08-01) <http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.15.8>
[^7]: **Protocols for Public Key Cryptosystems** Ralph C. Merkle *1980 IEEE Symposium on Security and Privacy* (1980-04) <https://doi.org/bmvbd6> DOI: [10.1109/sp.1980.10006](https://doi.org/10.1109/sp.1980.10006)
[^8]: **An Introduction to Probability Theory and its Applications** William Feller *John Wiley & Sons* (1957) <https://archive.org/details/AnIntroductionToProbabilityTheoryAndItsApplicationsVolume1>

