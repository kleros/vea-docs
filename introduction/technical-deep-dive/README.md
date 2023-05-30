# 🧠 Technical Deep Dive

## Contracts

For each sending and receiving chain pair, there is a separate set of Vea contract deployments. For each chain, there is exactly 1 deployed contract.

* Sending Chain: [VeaInbox](https://github.com/kleros/vea/blob/2410617e6e6c243bc3108059c39703350031ead2/contracts/src/arbitrumToEth/VeaInboxArbToEth.sol) - Manages the state of all messages sent through Vea.
* Intermediary Chain(s): [Router](https://github.com/kleros/vea/blob/2410617e6e6c243bc3108059c39703350031ead2/contracts/src/arbitrumToGnosis/RouterArbToGnosis.sol) - Routes native bridge messages
* Receiving Chain: [VeaOutbox](https://github.com/kleros/vea/blob/2410617e6e6c243bc3108059c39703350031ead2/contracts/src/arbitrumToEth/VeaOutboxArbToEth.sol) - Manages optimistic game over inbox state

## Epochs

<figure><img src="../../.gitbook/assets/image (55).png" alt=""><figcaption></figcaption></figure>

Time is partitioned into epochs defined by an epochPeriod. Epochs mark the period between potential bridging events. In other words, epochPeriod defines the highest frequency of bridge operation. On the devnet, we use 30 min. On the full, permissionless testnet and on mainnet we will likely use a period of several hours.

## Merkle Trees

### Motivation

Merkle trees are a common tool for data storage. All messages sent through Vea are inserted into an append-only merkle tree maintained in the vea inbox contract on the sending chain.

An optimistic mechanism transfers the root of the tree to vea outbox contract on the receiving chain, then messages can be relayed by proving inclusion in the merkle tree represented by the root (merkle proofs). These proofs are a logarithmic size of the number of messages in the tree.

### Merkle Mountain Range

<figure><img src="../../.gitbook/assets/image (44).png" alt=""><figcaption><p>Merkle Mountain range representing 7 messages</p></figcaption></figure>

The type of merkle tree implemented in the Vea contracts is sometimes referred to as a merkle mountain range ([MMR](https://github.com/mimblewimble/grin/blob/master/doc/mmr.md)). Without additional context, merkle trees are usually understood to be perfectly balanced binary trees with a fixed height. MMRs on the other hand grow in height as more leafs are inserted, and can be represented by a set of merkle subtrees, referred to as merkle "mountain ranges" due to their shape, see diagram above. For more, see the [implementation details](implementation-details.md).

## Optimistic Bridging

### 0. Send a Message (Sending Chain)

In the VeaInbox, messages are sent through Vea by calling&#x20;

```
    function sendMessage(address _to, bytes4 _fnSelector, bytes memory _data) 
```

This [function](https://github.com/kleros/vea/blob/2410617e6e6c243bc3108059c39703350031ead2/contracts/src/arbitrumToEth/VeaInboxArbToEth.sol#L78) encodes the message to send including the msg.sender of the call, and inserts a hash of the message encoding into the merkle tree. On average, this call has a constant cost overhead of \~20k gas on the sending chain.&#x20;

### 1. Saving the Inbox Merkle Tree State (Sending Chain)

On the sending chain, every epochPeriod, there is an opportunity to save a [snapshot](https://github.com/kleros/vea/blob/2410617e6e6c243bc3108059c39703350031ead2/contracts/src/arbitrumToEth/VeaInboxArbToEth.sol#L137) of the merkle root state in a mapping.

<figure><img src="../../.gitbook/assets/image (62).png" alt=""><figcaption></figcaption></figure>

### 2. Claiming The Inbox State (Receiving Chain)

On the receiving chain, claims can be made about the snapshot of the merkle root taken in the inbox contract on the sending chain. This state root represents all messages which have been sent through Vea. Claims require a deposit in eth. If the claim is honest, the claimer will be returned their deposit. If the claim is otherwise successfully challenged, then half the deposit is burned and the other half rewards the challenger. There is a mandatory burn to avoid the zero-cost delay-grief sybil problem where the claimer and challenger could be the same entity.&#x20;

<figure><img src="../../.gitbook/assets/image (60).png" alt=""><figcaption></figcaption></figure>

There are some stroboscopic effects due to the frequency of bridge operation --- snapshots can be taken at the beginning of an epoch or the end. To ensure sufficient time to claim and to ensure that challengers know the L2 state, we introduce a claim delay of roughly an epochPeriod. Only after this delay can claims be made about a past epoch, at which time the L2 state during that epoch is known.

### Challenge Period

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Once a claim is made a challenge period begins. We expect a testnet and mainnet challenge period of several hours. During this period, anyone can challenge a claim by leaving a deposit. We call native bridges to resolve the disputed claim, and the honest party recieves half the deposit as a reward.

#### Censorship Test

The idea is that ETH proof-of-stake consensus chains, eg Ethereum mainnet and Gnosis Chain, contain enough information in block headers to make statistical conclusions about censorship --- critical when determining safe challenge period in optimistic mechanisms such as Vea. The failure mode of optimistic mechanisms involves censorship of honest challengers. There are multiple types of censorship.

There is "weak" censorship where block producers refuse to include your transaction. This is the type of censorship experienced by Tornado Cash. For example, if 90% of block producers censor Tornado Cash transactions, then on average, transactions involving Tornado Cash will take 10 times as long as usual. Eventually a non-censoring block producer is chosen to propose a block.

<figure><img src="../../.gitbook/assets/image (64).png" alt=""><figcaption><p>Tornado Cash censoring blocks</p></figcaption></figure>

On the other hand, there is "strong" censorship where block producers themselves are censored and [reorged](https://ethereum.org/en/developers/docs/consensus-mechanisms/pos/attack-and-defense/#reorgs) out. This is the type of censorship Optimistic mechanism designers are worried about.

Since Ethereum transitioned to proof of stake, there are new methods to detect "strong" censorship. Ed Felten, the Chief Scientist at Offchain Labs, [proposed](https://ethresear.ch/t/reducing-challenge-times-in-rollups/14997) such a method introducing a 'censorship test'.

The main idea is that in proof of stake, blocks are produced like clockwork. There are slots for validators to propose new blocks. If the validator misses their slot for block proposal, or if the block is reorged out, there will be an empty slot without a block. Luckily, there is enough information in block headers on-chain to deduce the number of missing blocks. When a block is missing, the block number does not increment, but the timestamp will increment with the slot time.

<figure><img src="../../.gitbook/assets/image (67).png" alt=""><figcaption><p>Missing Block Example</p></figcaption></figure>

For example, on Ethereum, the slot time is 12 seconds. At t0 = 0, block 0 is proposed. At t1 = 12 seconds, Block 1 is proposed. In the next slot, the block is missing. At t3, Block 2 is proposed, but the block timestamp is 36, taking into account the missing block slot.

By comparing the difference in timestamp with the difference in blocknumber, we can detect missing blocks.

```
uint256 expectedBlocks = (block.timestamp - t0) / SLOT_TIME;
```

With this missing block information available on-chain, we can develop a censorship test based on statistical assumptions. If we assume some some p percent of validators do not censor transactions, then we can calculate given some risk tolerance the chances of censorship during a challenge period.

The research [post](https://ethresear.ch/t/reducing-challenge-times-in-rollups/14997) states the test best,

<figure><img src="../../.gitbook/assets/image (23).png" alt=""><figcaption><p><a href="https://ethresear.ch/t/reducing-challenge-times-in-rollups/14997">https://ethresear.ch/t/reducing-challenge-times-in-rollups/14997</a></p></figcaption></figure>

For the purposes of the censorship test, one assumes that every missing block is a malicious block reorganization. This means, during times of poor network performance, for example following hard forks, the censorship test is likely to fail, even if 'censorship' is not on-going and the missing blocks are due to validator client implementation instability. Notice in the eth research post calculation example, that 4 missing blocks our of 225 slots is a missing block rate of \~1.7%. Notice in the graph below that this threshold of 1.7% is exceeded twice in Ethereum's history. Once after the [shapella](https://arbitrum.notion.site/17-relayers-are-failing-with-the-prysm-client-post-Capella-Shanghai-2023-04-12-mainnet-0ef5ccd795e54ae4894fa695f1a3e70b) hardfork, and again [recently](https://offchain.medium.com/post-mortem-report-ethereum-mainnet-finality-05-11-2023-95e271dfd8b2).

For Vea, any claim which fails the censorship test during the challenge period cannot be optimistically verified. Instead, to resolve the claim (to refund the claimer's deposit), we must call the native bridges to relay the snapshot saved in the sending chain to compare with the claim. This means under normal network conditions, Vea operates optimistically, but during periods of poor network performance for example following hardforks, Vea slows down and uses native bridges.

<figure><img src="../../.gitbook/assets/image (33).png" alt=""><figcaption><p><a href="https://ethereumverse.vercel.app/">https://ethereumverse.vercel.app/</a>, first 'bump' shapella hardfork, second 'bump' recent finality issue</p></figcaption></figure>

Since [Ethereumverse](https://twitter.com/gnosischain/status/1633155083843039242) chains like Ethereum mainnet and Gnosis Chain share the same consensus mechanism, this censorship test applies equally well adjusting for the different slot time.&#x20;

On Etheruem mainnet we are considering a missingBlock threshold of 601 in 7200 slots (8%) assuming atleast 10% of validators are non-censoring and a risk tolerance of 1:1 million.&#x20;

On Gnosis Chain we are considering a missingBlock threshold of 3207 in 17280 slots (18.5%) assuming atleast 20% of validators are non-censoring and a risk tolerance of 1:1 million.&#x20;

You can test our censorship parameter calculator [here](https://colab.research.google.com/drive/1-hcsqzQZX2OZouVfJZJ7tCjSQJ9Dh5Ay?usp=sharing).

### Optimistic Verification

<figure><img src="../../.gitbook/assets/image (56).png" alt=""><figcaption><p>Vea Optimistic Verification Sequence Diagram</p></figcaption></figure>

### Happy Path

If the claim is unchallenged and the censorship test passes, the claim can be optimistically [verified](https://github.com/kleros/vea/blob/2410617e6e6c243bc3108059c39703350031ead2/contracts/src/arbitrumToEth/VeaOutboxArbToEth.sol#L178).

<figure><img src="../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

### Unhappy Path

Claims can be challenged by leaving a deposit. Disputed claims are resolved by calling native bridges. In the case of optimistic rollups as intermediary or source chains for the native bridge routing, the resolution period can be 7-8 days, waiting for the latency of the 1 week challenge period of optimistic rollups.

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

### Delay Griefs

<figure><img src="../../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

Vea is a permissionless protocol. Anyone can make claims or challenges. As a result, Vea can be delay griefed. Malicious actors can slow down the bridge by making false claims up to a maximum delay of the latency of the native bridges. Since the claimer and challenger could be the same entity, to ensure that delays incur real costs, half of the deposit from proven dishonest actors are burned. The griefing factor is the deposit / 2 per epochPeriod --- in other words, the cost of delaying Vea by 1 epoch period is half a deposit. The maximum delay possible is the latency of calling native bridges to resolve the disputes.

The potential reward for honest actors, half the deposit, must be sufficient to pay for gas fees to resolve the claim, calling native bridges, even in high gas environments. For example, assuming 100k mainnet gas to resolve claims from an L2, given 10,000 gwei gas, the deposit should be at least 2 eth.

**How do we set a reasonable deposit?**

When bridging from Optimistic rollups, the native bridge takes at least 7 days. 1 day extra in case of L2 censorship by the sequencer (transactions can be force included on L1 after 24 hours). Add some margin for censorship by block producers or potentially of block producers, and the time to resolve disputes over vea inbox stateroots can take \~10 days --- atleast 7 days for Optimistic Rollup challenge period, an extra day in case of L2 censorship, and some buffer time.

An honest actor needs enough liquidity to challenge fraudulent claims. Given an epochPeriod and liquidity budget we can find the deposit size by

deposit = liquidity budget / (10 days / epochPeriod)

eg. suppose epochPeriod = 6 hours and liquidity budget = 800 eth. Then 10 days / (6 hours/ epoch ) = 40 epochs. In other words, an honest actor needs enough liquidity to cover 40 epochs of deposits to prevent an attacker. So finally, we calculate a reasonable deposit given the budget as

deposit = 800 eth / 40 = 20 eth.

### What happens if the L2 sending chain experiences some failure?

All most all rollups today are in "[stage 0](https://ethereum-magicians.org/t/proposed-milestones-for-rollups-taking-off-training-wheels/11571)" with training wheels on. As a result of the permissioned operation of L2s, the L2s lack liveliness guarantees. There's no guarantee that new L2 blocks will be produced by the permissioned set of L2 block producers, meaning there is no guarantee that the native bridges are available --- eg. L2 users are stuck.

On Arbitrum, If the validators stop producing blocks for 7 days, the [permissioning](https://github.com/OffchainLabs/nitro-contracts/blob/08ac127e966fa87a4d5ba3d23cd3132b57701132/src/rollup/RollupUserLogic.sol#L55) of the validator set is dropped, however the arbitrum protocol today is delay griefable, so there is a possibility that Arbitrum grinds to a halt, or the Arbitrum contracts are arbitrarily upgraded by the security council multisig.

For these reasons, we can't assume that challenged claims in the vea outbox can ever be resolved. timeoutEpochs defines the number of epochs of bridge inactivity, after which the vea outbox is considered [shutdown](https://github.com/kleros/vea/blob/2410617e6e6c243bc3108059c39703350031ead2/contracts/src/arbitrumToEth/VeaOutboxArbToEth.sol#L80), and all pending claims and challenges can withdraw their deposits. For example, 3 weeks might be a healthy value when bridging from optimistic rollup L2s.

### Summary

<figure><img src="../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

* **Vea is inclusive** Vea is a **permissionless** system: anyone can participate as a watcher or a validator.
* **Anyone can force correct execution** Anyone can take part in Vea, and it requires only one honest participant in the system to force the bridge in properly executing.
* **Vea is permanent** By permanent, we mean **immutable** and **ungoverned**. Once the parameters are deemed **safe**, the contracts are deployed based on them. There are **no potential changes**, no governance parameters, and no governance tokens.
* **Vea is a public good** Vea is a primitive that Kleros needs and have chosen to share with everyone to use, participate or build on top of it. They are **no fees**, **no rent-seeking**, and **no tokens**.
* **Vea is griefable** As an inclusive system, Vea exposes itself to griefing attacks introducing delays. But the cost increase linearly and is bound to a maximum of **7 days**, or **8 days in extreme cases** for optimistic systems.
