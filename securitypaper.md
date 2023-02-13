**Bitcoin Layer 2 Executive Summary**

**Lightning Network Security Overview**

_Ronald Bynoe_

> Table of Contents
* [Revisions](#revisions)
* [Disclosures](#disclosures)
* [Introductions](#introductions)
  * [Bitcoin](#bitcoin)
  * [Layer 2](#layer-2)
  * [Lightning](#lightning)
* [Security Considerations](#security-considerations) 
  * [Infrastructure](#infrastructure)
  * [Node Operation](#node-operation)
  * [Wallet Operation](#wallet-operation)
  * [Social Engineering](#social-engineering)
  * [Examples](#examples)
* [Conclusion](#conclusion)
* [Additional Reading](#additional-reading)
* [References](#references)

# Revisions

2023-02-12 – Initial Draft

# Disclosures

The author of this summary is neither a programmer nor a professional security researcher. I have been using the Lightning Network for payments since Mach of 2019, and have been operating a mid-sized Lightning Routing Node[<![if !supportFootnotes]>[i]<![endif]>](#_edn1) since block 686061 in June of 2021.

This is a best effort high level assessment of the potential risk exposure at running a Lightning Node and conducting business on the Lightning Network. Maintaining a Bitcoin Node with a balance of Bitcoin is a potentially high risk venture. Operating a Lightning Node is by definition high risk and requires a varying level of technical sophistication, and a high degree of technical and security know-how to execute well enough to avoid common pitfalls for loss of funds. This document is not to be taken as comprehensive, and this environment changes quickly which will result in the contents being outdated relatively rapidly. Never risk more capital than you are willing to lose on any experimental financial tool, and ensure that your network & backup strategies are sound and in place prior to beginning to explore the Lightning Network.

# Introductions

## Bitcoin

Bitcoin is a decentralized currency secured by proof-of-work providing essentially a protocol for economic movement that is disassociated from any single government’s control. Bitcoin blocks are secured by the electrical grids of miners around the globe, which results in a block emission fixed to average 1 block of transactions every 10 minutes (this is an average over time, so some blocks confirm mere seconds after the previous block, and one took 122 minutes![<![if !supportFootnotes]>[ii]<![endif]>](#_edn2)). The mining difficulty retargets as the global hashrate changes roughly every 2 weeks to maintain that 10 minute average confirmation time. Each block can contain a maximum of 4MB of data, which on average is about 2,280[<![if !supportFootnotes]>[iii]<![endif]>](#_edn3) transactions per block, or 3.8 transactions per second. Bitcoin has a small language of only 186 Opcodes[<![if !supportFootnotes]>[iv]<![endif]>](#_edn4) and changes presented as either hard-forks or soft-forks come through extensive validation & conversation via the formal BIP[<![if !supportFootnotes]>[v]<![endif]>](#_edn5) Process.

## Layer 2

Layer 2 (L2) refers to any technology built upon an existing blockchain (not Bitcoin specific) designed typically to remediate perceived deficiencies or to bypass specific safeguards in the underlying protocol by building atop it, thus leveraging its base value proposition.

## Lightning

The Lightning Network is a specific L2 implementation built atop the Bitcoin Protocol to reduce the friction present with sending Bitcoin between parties in on-chain transactions. Specifically, Lightning seeks to address the speed with which a transaction may be sent and receipt confirmed (from highly variable and often over 60 minutes, to instantaneous), as well as the expense of sending said transaction (measured in vBytes per Satoshi) from 8,900 sats per transaction on-chain to under 10 sats per transaction off-chain. Final settlement between two nodes always results in an on-chain transaction, which serves to secure the Lightning Network from fraud. At the very outset, the concept of the Lightning Network has been advertised as being a work-in-progress where activity moves rapidly, often with insufficient validation for new features. Lightning development occurs with a modus operandi of “#Reckless” where the risky nature of these enhancements upon Bitcoin Core is acknowledged at every step.

# Security Considerations

Any technology such as Bitcoin introduces multiple new attack vectors on the storage and transmission of currency, many of which are still being discovered and exploits against existing fixes may arise in the future. Keeping the overall approach as simple as possible and leveraging existing security models as well as is possible is key to keeping Bitcoin secure from a financial markets perspective. Thus, by necessity, building a second layer of complexity atop Bitcoin introduces additional potential attack surfaces. We will attempt to introduce and explore some of those here.

There are two significant attack vectors that are common between Lightning and Bitcoin: Operating secure infrastructure for the safe storage of a wallet and funds, and safely and securely exchanging Satoshis between two wallets. For on-chain Bitcoin numerous software and hardware wallets exist to simplify the task of holding Bitcoin securely, additionally multi-signature wallets and encrypted seed phrases can improve the security of an account with moderate increases to the complexity of doing so. Likewise, an on-chain Bitcoin transaction is secured through a well understood and uncompromisable signing of a transaction, and record of the transaction being recorded immutably on the Bitcoin Blockchain.

## Infrastructure

For Lightning, the infrastructure complexity begins with the Bitcoin Core software running on a host platform where every additional installed piece of software is a potential attack vector. Resting encryption of the private key for the node, and the requirement to keep it decrypted while the system is running makes Lightning Nodes susceptible to network intrusions causing the exfiltration of funds if not properly locked down[<![if !supportFootnotes]>[vi]<![endif]>](#_edn6). Due to the dynamic nature of operating a Lightning Node, the underlying support software changes more rapidly and the software dependencies list grows regularly. These changes also result in system demand characteristics changing more rapidly than with Bitcoin Core. Whereas Bitcoin grows at a rate of roughly 50GB per year, the introduction of Lightning transactions can dramatically increase the storage requirements of the hardware as well as the CPU requirements, as more tasks are taken on by the node operator.

## Node Operation

As an explorative technology, Lightning also has a proposal system like Bitcoin’s BIPS, called BOLTs[<![if !supportFootnotes]>[vii]<![endif]>](#_edn7) but unlike Bitcoin, lacks the rigorous demands for backward or cross-compatibility, and is considered (as a more experimental platform) acceptable to make big changes more rapidly. There are currently two main node servers: LND by Lightning Labs and CLN by Core Lightning (there are additional smaller implementations such as Éclair and Electrum). These two Lightning implementations have minor inconsistencies in which BOLTs they implement, and the manner by which their supported BOLTs are implemented, causing interoperability issues at times. As an example, both LND and CLN support opening balanced channels, however, a balanced Lightning Channel cannot be opened between an LND-based server and a CLN-based sever. And under the current implementation, LND allows multiple channels between two nodes (often used early on for creating balanced connections between nodes) however CLN does not support multiple channels between nodes. Additionally, in a commercial node there are considerations for routing nodes (designed to provide liquidity to the network) and for custodial nodes (providing wallets for individual users). These considerations involve the installation of additional 3rd party software such as LNDg, balanceofsatoshis, LNBits, electrs plugins, etc. This additional software depends on Macaroons[<![if !supportFootnotes]>[viii]<![endif]>](#_edn8) for authorizing varying levels of security access to wallet operations, a process which is proven secure. However, an inexperienced node operator may create a macaroon for a tool with excessive permissions granted. Likewise inexperienced node operators often opt to save the password to decrypt the node’s private key on disk so it unlocks on boot, and often fail to secure the filesystem sufficiently to protect against targeted attacks.

The additional software installed to extend the functionality of the Lightning Node has its own dependencies and is often updated rapidly. Like all modern system administration, many depend on nodejs, npm, or docker hub, and pull in a considerable number of unvetted dependencies. Often, security update notices come through non-traditional avenues such as Twitter, Telegram, or Substack articles. When bugs are discovered, they are often 0-day exploited by the researcher, causing immediate issues with noderunners across the domain. To their credit, software developers stay on top of the technology, and push out updates within hours of issues being reported. Remediation however then becomes incumbent upon the node operator and failure to rapidly address the issue can result in node downtime, the force closure of channels, or the theoretical loss of funds by a sophisticated attack mechanism (the author is not aware of any such successful exploits as of the writing of this document). Operating a custodial wallet brings additional concerns, well highlighted by the recent issues with the lntxbot Telegram Lightning Bot that offered the ability to send/receive payments from a custodial wallet within the Telegram ecosystem. When the bot host experienced a hardware issue, and then subsequent software issues, users experienced challenges & delays in recovering their funds while the author (who maintains the tool in his free time) worked on providing a mechanism for recovery. Issues with running complex software offerings on top of the Lightning server itself benefit from a threat mitigation and security assessment prior to making public, and a proactive set of strategies for software, protocol, or social engineering eventualities.

## Wallet Operation

As a Lighting user, the threat landscape is far more straight-forward. The primary threat vector is from a compromised counterparty or loss of security of the custodial application itself. In the event above with lntxbot, although not a malicious act, the wallet users in this case were subject to delayed access to funds due to a bug with the wallet provider, invoking their anger and some even threatening legal action if access to their funds was not immediately restored. The common phrase amongst the community in this case is, “Not your keys, not your coins”. That is to say, if you rely on a custodial service and do not self-host your own Bitcoin and/or Lightning wallets, then you do not truly own those Satoshis.

## Social Engineering

For the custodial application vector, this is the same as with any other application you use for storage of funds. If the device the software is installed on, the credentials to access the software, or the software itself has a security vulnerability either through social engineering or technical, then funds can be stolen. One of the most common threat vectors is a social engineering attack where the attacker convinces the mark to provide credentials or to pay funds in an apparent effort to verify a balance or by way of a scam to receive additional Bitcoin. The efficacy of such attacks is poorly documented, and presumed rare in the Lightning community due to the current high degree of technical savvy necessary to use the network successfully.

## Examples

There have been numerous technical challenges as the Lightning Network has grown since 2018. These have had varying levels of success, but are extremely well documented across multiple sources[<![if !supportFootnotes]>[ix]<![endif]>](#_edn9). A couple good examples that highlight the types of issues the network faces are presented here.

On October 9th, 2022 in Bitcoin block 757922[<![if !supportFootnotes]>[x]<![endif]>](#_edn10) a security researcher known on Twitter as Buraq[<![if !supportFootnotes]>[xi]<![endif]>](#_edn11) announced that he had successfully crafted a 998-of-999 tapscript multisig transation. This transaction triggered a bug in the popular LND Lightning server that caused it to fail to synchronize its on-chain wallet at startup, so all nodes rebooted after that transaction would fail to start. The issue was reported[<![if !supportFootnotes]>[xii]<![endif]>](#_edn12) to the LND team within minutes, and was fixed and pushed within 5 hours, active nodes had already begun updating from the pending pull request[<![if !supportFootnotes]>[xiii]<![endif]>](#_edn13), and although unattended nodes eventually went dark until their noderunners updated them or shut them down, the event itself was well handled and did not result in significant downtime for LND based nodes. As a research project the news it generated[<![if !supportFootnotes]>[xiv]<![endif]>](#_edn14) highlighted the need for improved backup channel options for node operators such as through the employment of watchtower services.

On November 1st, 2022 the same security researcher created a transaction that was included in block 761249[<![if !supportFootnotes]>[xv]<![endif]>](#_edn15) which utilized a common Bitcoin feature to record witness items into the transaction, but in this case, 500,003 entries were encoded. The researcher announced his activity this time on Twitter.[<![if !supportFootnotes]>[xvi]<![endif]>](#_edn16) This transaction’s unrealistically large message size exceeded the maximum hard-coded limit in btcd, a Bitcoin client that LND uses (CLN used a different client and thus was unimpacted). Again, LND nodes were unable to sync, a bug was filed[<![if !supportFootnotes]>[xvii]<![endif]>](#_edn17), and a patch was merged just an hour later[<![if !supportFootnotes]>[xviii]<![endif]>](#_edn18). Upon triage no known loss of funds took place through this event. The security researcher published a good overview of the process he employed[<![if !supportFootnotes]>[xix]<![endif]>](#_edn19), and this encouraged further scrutiny of not just LND, but also its core dependencies.

# Conclusion

The Lightning Network represents a ground-breaking and novel approach to disruption of the legacy financial systems. Concepts like SWIFT and SPFS running international settlement layers, with financial instruments such as the Visa & Master Card networks, and moderately innovative payment gateways like PayPal, Zelle, CashApp,Strike, etc, are all constructed within high trust environments with low amounts of transparency. These more rigorous formal institutions are change adverse and resistant to technologies which may disrupt their extremely lucrative revenue streams through remittance processing and bank settlements.

As a result of being uniquely situated at the inflection point in global finance Bitcoin and by extension the Lightning Network combine many of the challenges facing a moderately sized financial institution performing payment processing of around $8.2 trillion in 2022[<![if !supportFootnotes]>[xx]<![endif]>](#_edn20). This exposes the Lightning network to the same degree of scrutiny by criminal activity as a banking network. Currently the Lightning Network has over 5,500 Bitcoin locked in channels (over $121m at the time of writing)[<![if !supportFootnotes]>[xxi]<![endif]>](#_edn21) and as that quantity grows, so does the value proposition that a successful technical exploit against the network represents. Although there is a considerable “bounty” on a successful hack, executing such a hack remains extremely challenging due to the underlying security the Bitcoin network provides, the deliberate avoidance of unnecessary complexity within Lightning itself, and the Open Source approach to software maintenance and encouragement of white hat adversarial activities to improve the security of the network.

In addition to facing the challenges the legacy financial system brings, Lightning also brings the complexity of building within an emergent ecosystem of cryptocurrency, some of which as detailed above with the complexity in developing this software, in successfully running it with real capital behind it, and in navigating the potential regulatory and legislative morass that is rapidly growing around these new agile payment tools.

Even with these daunting challenges considered, the Lightning Network represents such a significant “warp speed” improvement upon existing financial systems across key value propositions in velocity, user cost, micropayment capacity, and resistance to debasement through inflation. These benefits have led multiple financial institutions such as River[<![if !supportFootnotes]>[xxii]<![endif]>](#_edn22) Financial, Block (formerly Square)[<![if !supportFootnotes]>[xxiii]<![endif]>](#_edn23), Strike[<![if !supportFootnotes]>[xxiv]<![endif]>](#_edn24), and even MicroStrategy[<![if !supportFootnotes]>[xxv]<![endif]>](#_edn25) to make heavy investments in this nascent environment, all of which with great success. These efforts seek to challenge the status quo and disrupt it at such a rapid pace, that the only avenue the existing institutions have at their disposal is to seek legal action to prevent its adoption & growth. However, just as unsuccessful as hacking Lightning so far has been, legislative action has been equally challenging due to the fact that its underpinning technology, Bitcoin, is not a security per the SEC, and distinct from all other cryptocurrencies in its truly decentralized nature and Lightning avoids the foibles that banking is currently suffering from through its overt reliance on fractional reserve banking and under-collateralized loan instruments. By creating a sound alternative to the legacy financial system, and improving upon it not merely in confidence in the currency but also in the other areas mentioned above, it is uniquely positioned to grow extremely rapidly, consuming market share from slower moving incumbents adapting poorly to the changing financial environment. Although there will be additional security issues, complexity and cost of running a routing node will increase, and the market will not be without its own challenges, it remains in the author’s opinion, a very bright new avenue for financial technological innovation.

# Additional Reading

Some papers relevant to the security complexities facing the Lightning network have been compiled here for reference.

* [https://arxiv.org/abs/2208.01908](https://arxiv.org/abs/2208.01908)
* [https://www.semanticscholar.org/paper/An-Empirical-Analysis-of-Privacy-in-the-Lightning-Kappos-Yousaf/886d22608c6704a000c59e6d52d14bd06b0a3565](https://www.semanticscholar.org/paper/An-Empirical-Analysis-of-Privacy-in-the-Lightning-Kappos-Yousaf/886d22608c6704a000c59e6d52d14bd06b0a3565)
* [https://arxiv.org/abs/2103.08576](https://arxiv.org/abs/2103.08576)
* [https://www.computer.org/csdl/proceedings-article/csf/2020/09155145/1m1jOxuJKF2](https://www.computer.org/csdl/proceedings-article/csf/2020/09155145/1m1jOxuJKF2)
* [https://docs.lightning.engineering/lightning-network-tools/lnd/secure-your-lightning-network-node](https://docs.lightning.engineering/lightning-network-tools/lnd/secure-your-lightning-network-node)
* [https://river.com/learn/what-is-the-lightning-network/](https://river.com/learn/what-is-the-lightning-network/)
* [https://dci.mit.edu/lightning-network](https://dci.mit.edu/lightning-network)
* [https://www.coinhouse.com/insights/news/lightning-network-technical-introduction/](https://www.coinhouse.com/insights/news/lightning-network-technical-introduction/)
* [https://kryptografen.no/lightning-101/lightning-101-why-is-the-lightning-network-secure/](https://kryptografen.no/lightning-101/lightning-101-why-is-the-lightning-network-secure/)
* [https://www.coindesk.com/tech/2020/10/27/4-bitcoin-lightning-network-vulnerabilities-that-havent-been-exploited-yet/](https://www.coindesk.com/tech/2020/10/27/4-bitcoin-lightning-network-vulnerabilities-that-havent-been-exploited-yet/)
* [https://coingeek.com/the-unsecure-lightning-network-as-btc-layer-2-scaling-protocol/](https://coingeek.com/the-unsecure-lightning-network-as-btc-layer-2-scaling-protocol/)

# References

<![if !supportEndnotes]>  

----------

<![endif]>

[<![if !supportFootnotes]>[i]<![endif]>](#_ednref1) [https://amboss.space/c/reckless](https://amboss.space/c/reckless)

[<![if !supportFootnotes]>[ii]<![endif]>](#_ednref2) [https://u.today/bitcoin-just-recorded-longest-time-between-two-blocks-in-almost-10-years](https://u.today/bitcoin-just-recorded-longest-time-between-two-blocks-in-almost-10-years)

[<![if !supportFootnotes]>[iii]<![endif]>](#_ednref3) [https://ycharts.com/indicators/bitcoin_average_transactions_per_block](https://ycharts.com/indicators/bitcoin_average_transactions_per_block)

[<![if !supportFootnotes]>[iv]<![endif]>](#_ednref4) [https://wiki.bitcoinsv.io/index.php/Opcodes_used_in_Bitcoin_Script](https://wiki.bitcoinsv.io/index.php/Opcodes_used_in_Bitcoin_Script)

[<![if !supportFootnotes]>[v]<![endif]>](#_ednref5) [https://github.com/bitcoin/bips](https://github.com/bitcoin/bips)

[<![if !supportFootnotes]>[vi]<![endif]>](#_ednref6) [https://www.computer.org/csdl/proceedings-article/csf/2020/09155145/1m1jOxuJKF2](https://www.computer.org/csdl/proceedings-article/csf/2020/09155145/1m1jOxuJKF2)

[<![if !supportFootnotes]>[vii]<![endif]>](#_ednref7) [https://github.com/lightning/bolts](https://github.com/lightning/bolts)

[<![if !supportFootnotes]>[viii]<![endif]>](#_ednref8) [https://docs.lightning.engineering/the-lightning-network/lsat/macaroons](https://docs.lightning.engineering/the-lightning-network/lsat/macaroons)

[<![if !supportFootnotes]>[ix]<![endif]>](#_ednref9) [https://github.com/davidshares/Lightning-Network](https://github.com/davidshares/Lightning-Network)

[<![if !supportFootnotes]>[x]<![endif]>](#_ednref10) [https://mempool.space/tx/7393096d97bfee8660f4100ffd61874d62f9a65de9fb6acf740c4c386990ef73](https://mempool.space/tx/7393096d97bfee8660f4100ffd61874d62f9a65de9fb6acf740c4c386990ef73)

[<![if !supportFootnotes]>[xi]<![endif]>](#_ednref11) [https://twitter.com/brqgoo/status/1579216353780957185](https://twitter.com/brqgoo/status/1579216353780957185)

[<![if !supportFootnotes]>[xii]<![endif]>](#_ednref12) [https://github.com/lightningnetwork/lnd/issues/7002](https://github.com/lightningnetwork/lnd/issues/7002)

[<![if !supportFootnotes]>[xiii]<![endif]>](#_ednref13) [https://github.com/lightningnetwork/lnd/pull/7004](https://github.com/lightningnetwork/lnd/pull/7004)

[<![if !supportFootnotes]>[xiv]<![endif]>](#_ednref14) [https://protos.com/taproot-bug-freezes-bitcoin-inside-lightning-network-for-hours/](https://protos.com/taproot-bug-freezes-bitcoin-inside-lightning-network-for-hours/)

[<![if !supportFootnotes]>[xv]<![endif]>](#_ednref15) [https://mempool.space/tx/73be398c4bdc43709db7398106609eea2a7841aaf3a4fa2000dc18184faa2a7e](https://mempool.space/tx/73be398c4bdc43709db7398106609eea2a7841aaf3a4fa2000dc18184faa2a7e)

[<![if !supportFootnotes]>[xvi]<![endif]>](#_ednref16) [https://twitter.com/brqgoo/status/1587397646125260802](https://twitter.com/brqgoo/status/1587397646125260802)

[<![if !supportFootnotes]>[xvii]<![endif]>](#_ednref17) [https://github.com/lightningnetwork/lnd/issues/7096](https://github.com/lightningnetwork/lnd/issues/7096)

[<![if !supportFootnotes]>[xviii]<![endif]>](#_ednref18) [https://github.com/lightningnetwork/lnd/pull/7098](https://github.com/lightningnetwork/lnd/pull/7098)

[<![if !supportFootnotes]>[xix]<![endif]>](#_ednref19) [https://burakkeceli.medium.com/channel-addresses-bd85e9ab8fe1](https://burakkeceli.medium.com/channel-addresses-bd85e9ab8fe1)

[<![if !supportFootnotes]>[xx]<![endif]>](#_ednref20) [https://coinunited.io/news/en/2023-01-04/crypto/btc-blockchain-processed-over-8-trillion-in-transactions-last-year-as-bitcoin-soars](https://coinunited.io/news/en/2023-01-04/crypto/btc-blockchain-processed-over-8-trillion-in-transactions-last-year-as-bitcoin-soars)

[<![if !supportFootnotes]>[xxi]<![endif]>](#_ednref21) [https://www.theblock.co/post/208817/lightning-network-reaches-all-time-high-in-bitcoin-capacity](https://www.theblock.co/post/208817/lightning-network-reaches-all-time-high-in-bitcoin-capacity)

[<![if !supportFootnotes]>[xxii]<![endif]>](#_ednref22) [https://www.youtube.com/watch?v=Itkcurc0Bms](https://www.youtube.com/watch?v=Itkcurc0Bms)

[<![if !supportFootnotes]>[xxiii]<![endif]>](#_ednref23) [https://www.youtube.com/watch?v=rSSnyJpFNZU](https://www.youtube.com/watch?v=rSSnyJpFNZU)

[<![if !supportFootnotes]>[xxiv]<![endif]>](#_ednref24) [https://www.youtube.com/watch?v=dD2-T7TX2rk](https://www.youtube.com/watch?v=dD2-T7TX2rk)

[<![if !supportFootnotes]>[xxv]<![endif]>](#_ednref25) [https://www.youtube.com/watch?v=Cgdwme_GhIw](https://www.youtube.com/watch?v=Cgdwme_GhIw)
