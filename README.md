---
cover: >-
  .gitbook/assets/4064913893_a_futuristic_intergalactic_highway_named_vea__sharp.png
coverY: 128
---

# ✔ Vea

Vea is a message bridging protocol which enables a new tradeoff between latency and security for cross-chain interoperability from Optimistic Rollups to Ethereum L1 at first, and then to other networks.&#x20;

## Without Vea

Currently the only available options to pass a message from Arbitrum/Optimism to Ethereum fall into two categories:&#x20;

* The slow but very secure canonical bridge which takes at least 7 days, or&#x20;
* Third party bridges taking just between 10 minutes to 1 hour but adding many trust assumptions along with it.&#x20;

To achieve Vitalik’s vision of a rollup-centric roadmap for Ethereum, it is critical to keep on building better bridges that offer more security (5 out of the top 6 biggest hacks on [Rekt’s leaderboard](https://rekt.news/leaderboard/) are bridges), better intermediate tradeoffs between latency and security, better UX and DX.&#x20;

## With Vea

Vea is a new bridging protocol relying on a hybrid strategy where a message batch is optimistically-verified on L1 assuming only a 1 of N honest participant (this is the same trust model as the rollup itself) with a fallback on native verification in the unhappy path (through the canonical L2 to L1 bridge).&#x20;

### The Happy Path

The messages are optimistically verified within a day or possibly less as we build confidence in the protocol.

### The Unhappy Path

The message is sent over the canonical bridge which takes 7 days (or 8 days in the extreme case where a malicious sequencer does not include the message).

### Protocol Design

Leading to this design is the key realisation that once a L2 transaction is written on L1 (and heavily compressed) on Ethereum and given enough time for L1 finality ([60 to 95 beacon chain slots](https://notes.ethereum.org/@vbuterin/single\_slot\_finality#Paths-toward-single-slot-finality)), the L2 transaction can be safely considered as final, and therefore it can bridged without waiting for 7 days.&#x20;

This delay will not call into question the L2 transaction, it is intended for the production of the L2 blocks with the state state to be written on L1 without relying on a centralized source of truth, thereby giving honest parties enough time to participate in the 7-days fraud-proving scheme by staking on the correct state (Arbitrum) or by challenging a state transition (Optimism).&#x20;

### Security

Vea as a specialized bridge differs from the existing alternatives by taking a more conservative stance on security. It involves no routing middleware, no Chainlink oracle, no contract administrator role, no governance multisig, no upgradability. The smart contracts are disowned and immutable.

## Learning more about Vea

Read on to further explore the properties of the Vea protocol (trading additional latency in the unhappy scenario in favour of security), the most suitable use-cases (cross-chain governance, more generally high-value messaging which tolerate the worst-case latency), how to get started building cross-chain apps or validating message batches.
