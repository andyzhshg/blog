title: 比特币白皮书中文版
date: 2017-05-25 23:58:00
tags: [区块链, 比特币, BitCoin, 技术]
categories: 技术

-------


比特币的白皮书是比特币诞生的理论基础，网络上也流传着会多的翻译版本。之所以在翻译一遍，不是认为别的翻译有什么问题，而是接下来有计划写一篇关于白皮书内容的详解文章，所以，先自己翻译一遍，也能够进一步加深对比特币的了解。

比特币白皮书的原始文件可以参考：[https://bitcoin.org/bitcoin.pdf](https://bitcoin.org/bitcoin.pdf)

下边的行文，将采取英文原文和中文对照的方式，所有有疑问的地方，以英文原文为准。

## Bitcoin: A Peer-to-Peer Electronic Cash System  比特币：一种点对点的电子现金系统

> **Abstract.**   A  purely   peer-to-peer   version   of   electronic   cash   would   allow   onlinepayments   to   be   sent   directly   from   one   party   to   another   without   going   through   afinancial institution.   Digital signatures provide part of the solution, but the mainbenefits are lost if a trusted third party is still required to prevent double-spending.We propose a solution to the double-spending problem using a peer-to-peer network.The   network   timestamps   transactions   by   hashing   them   into   an   ongoing   chain   ofhash-based proof-of-work, forming a record that cannot be changed without redoingthe proof-of-work.   The longest chain not only serves as proof of the sequence ofevents witnessed, but proof that it came from the largest pool of CPU power.   Aslong as a majority of CPU power is controlled by nodes that are not cooperating toattack the network,  they'll  generate the  longest  chain  and  outpace attackers.   Thenetwork itself requires minimal structure.   Messages are broadcast on a best effortbasis,   and   nodes   can   leave   and   rejoin   the   network   at   will,   accepting   the   longestproof-of-work chain as proof of what happened while they were gone.

> **摘要** 

