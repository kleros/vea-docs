---
cover: .gitbook/assets/vea-gitbook-cover.svg
coverY: 46
---

# 🌉 Vea

Vea is a cross-chain message bridge which enables fast and secure interoperability specifically designed with optimistic rollups in mind. Vea specifically solves an unmet need of bridging authenticated data, which no other 3rd party bridge supports.

## Without Vea

Currently the only available options for cross-chain interoperability with optimistic rollups fall into two categories:

* Slow and secure canonical bridge which can take at least 7 days in the case of Optimistic Rollups, or
* Fast and insecure third party bridges taking just between 10 minutes to 1 hour but adding many trust assumptions.

Fast 3rd party bridges today work well for unauthenticated communication. If Alice (chain A) wants to send Bob (chain B) ETH, she can use a liquidity network where a liquidity provider, Charlie, immediately sends Bob (chain B) ETH to speed up the process and is later compensated with ETH locked up by Alice.

With authenticated communication though, no 3rd party can "supply liquidity" fast, for example, NFT transfers. If Alice wants to send Bob an NFT, no liquidity provider can send Bob the same NFT on Alice's behalf, because the NFT is non-fungible, only one exists. More generally, oracle queries are another example of authenticated communication where the result of an oracle cannot be provided by a 'liquidity provider' because the data is unique and the data provider needs to attest to the authenticity of the data.

## With Vea

Vea is a fast and secure bridging protocol which uses a hybrid strategy where a message is optimistically verified.

Vea enables cross-chain authenticated data transfer with faster latency than native bridges. A smart contract on chain A can query an oracle on chain B and receive the oracle response cheaper and sometimes faster than using native bridges. This opens a new possibility of high volume, low-value cross-chain interoperability (cross-chain oracle queries, NFT transfer, governance).

### The Happy Path

The messages are optimistically verified. The bridging speed can take anywhere from hours to a couple of days, depending on the source and destination chain.&#x20;

### The Unhappy Path

The message is sent over the canonical bridge, which can take 7-8 days when sent from optimistic rollups.

### Protocol Design

There is a fundamental trade-off between latency and security: a protocol can be fast & insecure or slow & secure. However, a protocol can be fast & secure as long as the fast mechanism falls back on a slow secure system. Vea is a cross-chain optimistic game that passes messages fast, but always falls back on secure, slow native bridges in case of disputes. This means that Vea is usually fast, but could sometimes slow down.

### Security

Vea is a 1-of-N security model, meaning only 1 honest participant can force the correct bridge execution. More importantly, Vea is a permissionless protocol, meaning you can make sure you are the one honest participant in the protocol.

The contracts are immutable, with no governance nor multisig upgradability risk.

### Interoperability

Vea can connect any L1, L2, L3... etc which settle to Ethereum, including ZK rollups. However, when interoperating with optimistic rollups, there are no added trust assumptions since optimistic rollups use the same trust model as Vea. For this reason, Vea is particularly well suited for optimistic rollup interoperability. Vea can even pass messages between ZK rollups and optimistic rollups without any extra trust assumptions, because to interoperate with an optimistic rollup already accepts a 1 of N security assumption for the optimistic rollup state verification.

## Learning more about Vea

Read on to further explore the technical deep-dive of the Vea protocol and how to get started building cross-chain dapps with Vea.

## Video Explainers

### Vea Overview with Shotaro - Eth Taipei 2023

{% embed url="https://www.youtube.com/watch?v=Pz8MitQoZAs" %}
Vea bridge: A permissionless, immutable optimistic bridging primitive | Kleros | ETHTaipei 2023
{% endembed %}
