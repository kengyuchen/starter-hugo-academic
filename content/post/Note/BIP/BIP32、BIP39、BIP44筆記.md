---
title: BIP32、BIP39、BIP44筆記
date: 2021-07-04 00:41:30
updated: 2021-07-04 00:41:30
categories: "Note"
tags: 
   - "Cryptography"
   - "Bitcoin"
---

BIP(Bitcoin Improvement Protocals)是Bitcoin社群為了增進Bitcoin的使用效率，使其更於方便使用而建立，其中BIP32、BIP39、BIP44定義了HD Wallet的使用規範與細節。以下是個人閱讀白皮書的筆記，希望能幫助自己or他人理解這些協定與背後原理。

另外，有關BIP的相關文件可以至[bitcoin/bips](https://github.com/bitcoin/bips)查看。

<!--more-->

## BIP32

[BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki)定義了什麼是HD Wallet(Hierarchical Deterministic Wallets)。

### Motivation

錢包(wallet)內含私鑰與公鑰(private key and public key)。一個private key能夠生成一個public key(by ECC)，而傳遞交易資訊的address是由public key經Sha-256生成。驗證所有交易的是個人持有的private key，要擁有private key，才能證明某筆交易與你有關。
重複使用同一個address理論上沒問題，但為隱私問題，Bitcoin仍建議每個address最好只使用一次。然而換address就意味著換private key，這使得一個錢包要裝一大堆的private key，而一把private key又是256bit的亂數，十分不好管理。

根據BIP32，過去的做法如下:

> In order to avoid the necessity for a backup after every transaction, (by default) 100 keys are cached in a pool of reserve keys. Still, these wallets are not intended to be shared and used on several systems simultaneously

因此BIP32就定義了一套用key生成key的方式(Child Key Derivation, a.k.a CKD)。一把key可以有很多個child key，而每個child key又能再生成很多個child key。如此一來自然就有非常多個address，而最重要的就是源頭的parent key，擁有它就能擁有所有的key，即能證明用這些address做的交易的合法性。
這做法的另一個優點是，你能夠發放不同的child key給不同人，而這些人使用這些key和其生成的address，都能代表擁有parent key的你，給予他們child key也不會洩漏你的資訊(child key cannot trace back to parent key)。這對公司、企業的商業管理提供了很大的便利性。

### Child Key Derivation(CKD)

#### Definition: ECC

此處公鑰的生成是藉由私鑰搭配橢圓曲線常數積(ECSM)生成。使用的橢圓曲線為[*secp256k1*](https://en.bitcoin.it/wiki/Secp256k1)，以下簡稱此橢圓曲線為E。令E的base point作P，設私鑰為k，則生成的公鑰為K=point(k)=k\*P。此安全性建立在橢圓曲線上的離散對數問題，即給定橢圓曲線上的一點Q=k\*P(公鑰)，很難還原出常數k。

#### Definition: Extended Key

在BIP32中一對extended key pair包含一把extended private key(寫作(k,c))與一把extended public key(寫作(K,c))。其中k為一256bit的數字，代表原本的私鑰；K為用E搭配常數k生成的點(K=point(k)=k\*P)，代表原本的公鑰。c稱做chain code，共256bit，為增加安全性設立。extended private key與extended public key的c是相同的。
因此，一組extended key pair的extended private key與extended public key都是512bit，且其後256bit(chain code)均相同。

#### Definition: Hardened

給定一個parent extended key與index，CKD能夠生成相應的child key。其中child key又有分hardened跟non-hardened，hardened的意思是**無法從parent extended public key生成驗證**，必須要有parent extended private key；而non-hardened則相反，**僅憑parent extended public key即可生成**。

#### Details

以下介紹CKD的實作細節。給定一把parent extended key(private:*(kpar,cpar)* or public:*(Kpar,cpar)*)與index i(i < $2^{32}$)，CKD定義了以下操作:

1. Private Parent Key -> Private Child Key: CKDpriv((kpar, cpar), i) → (ki, ci)
   * 確認Hardened or Non=hardened:
     * Hardened(if i >= $2^{31}$): 令I=HMAC-SHA512(Key=cpar,Data=0x00||kpar||i)
     * Non-hardened(if i < $2^{31}$): 令I=HMAC-SHA512(Key=cpar,Data=point(kpar)||i)
     * 注意兩者的差別。Hardened是直接用parent private key去生成，而non-hardened則是將private key轉public key後使用public key去生成。
   * 將I分成兩個均32byte長的序列，稱IL與IR。
   * Return child key ki為IL+kpar。
   * Return child chain code ci為IR。
   * 萬一發生IL大於E的order或ki=0，則return error，使用者須換一個index i。注意此發生的機率小於2^-127。

2. Public Parent Key -> Public Child Key: CKDpub((Kpar, cpar), i) → (Ki, ci)
   此方法與前面相似，注意根據定義這裡生成的child key只可能是non-hardened版本。
    * 確認Hardened or Non=hardened:
      * Hardened(if i >= $2^{31}$): 回傳error。
      * Non-hardened(if i < 2^31): 令I=HMAC-SHA512(Key=cpar,Data=Kpar||i)。
    * 將I分成兩個均32byte長的序列，稱IL與IR。
    * Return child key ki為point(IL)+Kpar。
    * Return child chain code ci為IR。
    * 萬一發生IL大於E的order或Ki為無窮遠點，則return error，使用者須換一個index i。

3. Private Parent Key -> Public Child Key:
   定義函數N((k,c))->(K,c)，作用為給定一把extended private key，生成其對應的extended public key。換句話說，K=point(k)，c相同不變。
   用private parent key生成public child key有兩種方法:
    * N(CKDpriv((kpar, cpar), i)) (hardened or non-hardened)。
    * CKDpub(N(kpar, cpar), i) (non-hardened only)。
    * 若1.與2.有看懂，應該會很清楚這兩個操作效果相同，且為甚麼第二個有non-hardened的限制。

4. Public Parent key → Private Child Key:
   基於橢圓曲線的安全性這件事應該要是做不到的(若做得到就代表secp256k1的ECC不再安全了。)

### Key Tree

有了CKD，我們可以為所有錢包中的key建構出一棵樹。令起點root為master extended private key m，N(m)=M。經一次CKD我們可以得到許許多多level-1的key，對level-1的key做CKD，我們可以再得到level-2的key，以此類推。
BIP32定義了相關的Notation。例如，CKDpriv(CKDpriv(CKDpriv(m,3H),2),5)將寫作m/3H/2/5。其中H代表使用hardened key，例如3H=3+2^31。
在這個notation下，我們有以下的關係:

* N(m/a/b/c) = N(m/a/b)/c = N(m/a)/b/c = N(m)/a/b/c = M/a/b/c。
* N(m/aH/b/c) = N(m/aH/b)/c = N(m/aH)/b/c。
* N(m/aH)不等於N(m)/aH，因為後者是不可行的操作。

### Master Key Generation

BIP32定義了一套生成master key的方法。

* 用亂數產生器(RNG)生成一個seed S，長度介於128bit~512bit。BIP32建議使用256bit
* 計算I=HMAC-SHA512(Key = "Bitcoin seed", Data = S)。
* 將I分成兩個均32byte長的序列，稱IL與IR。
* IL為master private key，IR為master chain code。
* 若IL=0或大於E的order，此master key無效，需重新生成S。

如圖：
![BIP32 Tree](https://github.com/bitcoin/bips/raw/master/bip-0032/derivation.png)

### Wallet Structure

有關使用BIP32的錢包架構，BIP44有更完整的定義。這邊僅介紹BIP32建議使用的模型。
將一個HDW組織成許許多多的account。每個account都有個編號，從0開始。
每個account都有兩個key pair chains，一個external一個internal。external用來生成新的public address，而internal用來做其他運算，包含更換、生成address等不需要公開的的事情。

* m/iH/0/k代表master key m底下的account i的第k組external keypair。
* m/iH/1/k代表master key m底下的account i的第k組internal keypair。

### Use Cases

BIP32提供了以下的幾種使用情境做參考。

1. Full Sharing: m
   當兩個公司需要使用相同的錢包時，藉由共同持有秘密m，可以讓兩者存取同一個錢包。
2. Audits: N(m/\*)
   公司的會計僅需知道所有的收入與支出，因此給予他master public key可讓其得知所有child key的交易資訊。
3. Per-office balances: m/iH
   企業底下的各個部門可以擁有不同的child private key。所有部門的交易均可歸於總部擁有的master key底下，且不同部門也能自己生成address做各自的交易，也允許跨部門間的交易。
4. Recurrent Business-to-Business Transaction: N(m/iH/0)
   當兩個or多個企業有密切頻繁的交易時，可以給予某個account的external chain的extended public key。這樣可以讓各公司輕易更換address，而不用每次交易均要請求一個新的address。
5. Unsecure Money Receiver: N(m/iH/0)
   當使用不安全的伺服器進行交易時，我們僅需給予public address就能接收付款。伺服器只需要知道某個external chain的extended public key，就能付款給錢包持有者。對伺服器進行攻擊的人最多僅能知道此public address對應的錢包獲得了多少錢，無法竊取更多資訊，也無法利用這個錢包進行交易。

### Security

BIP32的安全性建立在使用*secp256k1*的橢圓曲線公鑰密碼學上。這確保了以下兩點：

1. 給予一個child extended private key (ki,ci)和index i，攻擊者很難得到parent private key。(效率等同於暴力破解2^256的HMAC-SHA512)。

2. 給予一些tuples (i,(k,c))(index,extended private key)，攻擊者很難判斷這些是否來自同一個parent extended private key。(效率等同於暴力破解2^256的HMAC-SHA512)。

   
   以下這兩個安全性質並不存在：

   1. 給予一個parent extended public key (Kpar,cpar)和child public key (Ki)，要找到i很困難。
   2. 給予一個parent extended public key (Kpar,cpar)和non-hardened child private key (ki)，要找到parent private key kpar很困難。
      簡單來說就是private key很重要，不管是不是child key。

   

   BIP32說明保管好private key和public key都很重要。private key提供了使用錢包的安全性，而public key則提供了使用者的隱私(外人無法判斷你有多少錢)。
需注意的是，擁有parent extended public key (Kpar,cpar)和任一把non-hardened child private key (ki,ci)將可以推得parent private key (kpar,cpar)。作法如下：

> 根據CKD的定義，ki=IL+kpar，其中IL為chain code cpar、public key Kpar與一個小於2^31的index i經HMAC-SHA512所生成。cpar與Kpar我們已經知道了，而我們可以爆搜index i。藉由重複比對使用(Kpar,cpar)與某個i生成的child public key，與child private key生成的child public key，可以找出正確的i。而只要找出正確的i，即可生成正確的IL，而ki減去IL即可得kpar。


## BIP39

[BIP39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki)定義了如何透過所謂的mnemonic code生成整個HD wallet。

### Motivation

在BIP32定義了HD Wallet的建立方式，要對一個錢包進行存取等同於擁有某一個seed，只要有seed就能使用整個錢包。然而seed是一個128bit~512bit的數字，很不好記，BIP39便是提供or定義一套較人性化的方法去生成seed，稱做mnemonic code。透過mnecode code我們可以生成seed，因此記憶seed的問題就變成了記憶mnecode code。

### Generating the Mnemonic

BIP39定義一個mnecode須經由一個32bit倍數的entropy去生成。使用者可以用任何方式先生成一個initial entropy，令其長度為ENT。BIP39允許的ENT為128、160、192、224、256。
給定一個initial entropy，先生成一個checksum，這個checksum為entropy經SHA256後的前ENT/32 bit，令其長CS。舉例來說，若ENT為128，則checksum為initial entropy做SHA256後的前CS=4個bit。
將checksum接到initial entropy的後面，然後切割成每11個bit一組。每組可以換算成一個0~2047的數字，用這個數字至wordlist查表即可得到一個單字。所有單字組成一個mnemonic setence，其長度為MS。
下表給定了ENT、CS、MS的關係：

* CS = ENT / 32
* MS = (ENT + CS) / 11

| ENT  |  CS  | ENT+CS |  MS  |
| :--: | :--: | :----: | :--: |
| 128  |  4   |  132   |  12  |
| 160  |  5   |  165   |  15  |
| 192  |  6   |  198   |  18  |
| 224  |  7   |  231   |  21  |
| 256  |  8   |  264   |  24  |

### Wordlist

BIP39給予的wordlist滿足下列規範：

1. Smart Selection of Words:
   只要打前4個字母，即可唯一的找出wordlist中的某一個單字。這提供了使用wordlist的方便性。
2. Similar Words Avoided:
   使用的單字盡可能的不一樣。如build與built、woman與women、quick與quickly等，不應該重複出現在wordlist裡。這樣可以避免記憶上的困難與實務上出錯的機率。
3. Sorted Wordlist:
   wordlist的內容有經過排序。如此一來一方面可使使用者更有效率的查找單字，例如使用binary search；另一方面也可讓使用trie等資料結構更為方便。

另外BIP39規定了wordlist必須使用UTF-8 ecoding，並使用Normalization Form Compatibility Decomposition(NFKD)。

### From Mnemonic to Seed

首先，使用者可以自由定義一個密碼passphrase來更妥善的保護錢包，如未定義則passphrase設為空字串""。
BIP39使用了[PBKDF2](https://en.wikipedia.org/wiki/PBKDF2)函數去生成512bit的seed。PBKDF2的password參數代入mnemonic sentence(in UTF-8 NFKD)，字串"mnemonic"+passphrase做為salt。遞迴次數iteration count設為2048，且使用HMAC-SHA512做為pseudo-random function。生成長度定為512bit。


seed = U1 ^ U2 ^ ... ^ U2048
U1 = HMAC-SHA512(key=password, message=salt)
U2 = HMAC-SHA512(key=password, message=U1)
...
U2048 = HMAC-SHA512(key=password, message=U2047)

由於生成mnecode sentence與seed的過程可以完全獨立，使用者可以定義自己喜歡的wordlist，或整套wordlist生成規則。不過BIP39仍建議使用他們規範的mnemonic生成規則與他們的wordlist(見下文)。

### Wordlists

BIP39定義了一套各語言的wordlist，不過它仍建議使用英文的版本。

* [wordlist](https://github.com/bitcoin/bips/blob/master/bip-0039/bip-0039-wordlists.md)

### Reference Implementation

BIP39也推薦了一套程式來實作：

* [python-mnemonic](https://github.com/trezor/python-mnemonic)

另外網路上也有相關的網站提供BIP39的實作：

* [Mnemonic Code Converter](https://iancoleman.io/bip39/)

## BIP44

[BIP44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki)定義了一套基於BIP32的錢包的架構。

### Motivation

BIP44的目的是提供一套方便管理不同幣種、不同account、external/internal chain與不同address的錢包架構。

### Path Levels

定義以下的BIP32 path(參考&修改自[Wallet Structure](#Wallet-Structure))：

```
m / purpose' / coin_type' / account' / change / address_index
```

其中的`'`表示使用BIP32 hardened derivation。
底下定義每一層的意義。

#### Purpose

Purpose設成常數44'，代表使用BIP44。這一層使用了hardened derivation。

#### Coin Type

這層的目的是為了讓單一一個master node就能使用各式各樣的幣種。每一個加密貨幣都對應一個coin type，如Bitcoin為0，ETH為60等等。這一層使用了hardened derivation。
各貨幣的coin type可見[Registered coin types for BIP44](https://github.com/satoshilabs/slips/blob/master/slip-0044.md)。

#### Account

這層定義了每一個使用單位account，用來給予不同的使用者。使用者可以將此層想成銀行的帳戶，用來進行捐贈、付款、存錢等等。這一層同樣使用了hardened derivation。
BIP44規定了使用者需從index 0開始依序使用account，且必須記錄所有使用過的account，不可以跳號。
同理，當引入一個新的seed時，使用者也需要能判斷哪些account已經被使用了。

#### Change

這層只有兩個數字，0或1。0用作external chain，1用作internal chain。這邊定義的external/internal與BIP32講的稍有不同。此處的external是用作公開接收付款等交易的address，而internal是用來作為找零(change)的address。找零的這個internal chain address是不公開的。

根據[這篇文章](https://www.oreilly.com/library/view/mastering-bitcoin/9781491902639/ch05.html)的說明，每一筆在區塊鏈上的交易收入會形成unspent transaction output(UTXO)。UTXO是不可分割的，被記錄在鏈上。在區塊鏈上並沒有所謂的某個account或address的存款，只有某個使用者擁有的的分散的UTXO，而一個使用者的存款即是整個鏈上所有該歸屬於它的UTXO總和。
雖然每一筆交易(每一個UTXO)都可以是任意的數目，但一旦交易完成這個UTXO就不再能夠被分割。如果之後使用者欲支付的數目小於一個他擁有的UTXO，就會有所謂找零的動作。

#### Index

每個index即代表了一個address。

### Account Discovery

當我們引入一個新的錢包時(新的seed)，我們使用的software需要去找尋目前已有哪些account被使用過了：

1. 找出第一個account node(index = 0)
2. 找出這個account的external chain
3. 開始遍搜external chain上的所有address
   * 如果所有的address都不存在交易紀錄，則回傳當前的account
   * 如果有找到某個address存在交易紀錄，則account增加1，並重新回到2.

由於account必須要循序使用，不可以跳號，因此這個演算法可以確保真的找到第一個還未被使用的account。

#### Address Gap Limit

上述的第三步"遍搜所有的address"，最多搜索到address 20就停。意思就是說，我們不考慮20以後的address，也不預期有人會去使用。
此外我們只考慮external chain上的address的交易，因為每一筆internal chain上的交易應該都是來自其相應的external chain。

BIP44希望software去警告使用者使用的address超過20。

### Example

這邊是複製BIP44給予的example。

|      coin       | account |  chain   | address |           path            |
| :-------------: | :-----: | :------: | :-----: | :-----------------------: |
|     Bitcoin     |  first  | external |  first  | m / 44' / 0' / 0' / 0 / 0 |
|     Bitcoin     |  first  | external | second  | m / 44' / 0' / 0' / 0 / 1 |
|     Bitcoin     |  first  |  change  |  first  | m / 44' / 0' / 0' / 1 / 0 |
|     Bitcoin     |  first  |  change  | second  | m / 44' / 0' / 0' / 1 / 1 |
|     Bitcoin     | second  | external |  first  | m / 44' / 0' / 1' / 0 / 0 |
|     Bitcoin     | second  | external | second  | m / 44' / 0' / 1' / 0 / 1 |
|     Bitcoin     | second  |  change  |  first  | m / 44' / 0' / 1' / 1 / 0 |
|     Bitcoin     | second  |  change  | second  | m / 44' / 0' / 1' / 1 / 1 |
| Bitcoin Testnet |  first  | external |  first  | m / 44' / 1' / 0' / 0 / 0 |
| Bitcoin Testnet |  first  | external | second  | m / 44' / 1' / 0' / 0 / 1 |
| Bitcoin Testnet |  first  |  change  |  first  | m / 44' / 1' / 0' / 1 / 0 |
| Bitcoin Testnet |  first  |  change  | second  | m / 44' / 1' / 0' / 1 / 1 |
| Bitcoin Testnet | second  | external |  first  | m / 44' / 1' / 1' / 0 / 0 |
| Bitcoin Testnet | second  | external | second  | m / 44' / 1' / 1' / 0 / 1 |
| Bitcoin Testnet | second  |  change  |  first  | m / 44' / 1' / 1' / 1 / 0 |
| Bitcoin Testnet | second  |  change  | second  | m / 44' / 1' / 1' / 1 / 1 |

## 小結

這篇筆記提供了BIP32、BIP39與BIP44的實作細節說明，除了作為個人紀錄外，本意是希望能提供懶得看白皮書的人一些幫助。然而越寫愈多，要想看完好像也不比直接看白皮書輕鬆多少，所以若想真正了解這些協定，建議還是直接看白皮書比較實在。
