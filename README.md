---
cover: .gitbook/assets/vea-gitbook-cover.svg
coverY: 46
---

# 🌉 Vea

Vea is a cross-chain message bridge which enables fast and secure  interoperability specifically designed with optimistic rollups in mind.

## Without Vea

Currently the only available options for cross-chain interoperability with optimistic rollups fall into two categories:

* Slow and secure canonical bridge which takes at least 7 days, or
* Fast and insecure third party bridges taking just between 10 minutes to 1 hour but adding many trust assumptions along with it.

## With Vea

Vea is a fast and secure bridging protocol which uses a hybrid strategy where a message is optimistically verified.

### The Happy Path

The messages are optimistically verified. The bridging speed can take anywhere from hours to a couple days depending on the source and destination chain.&#x20;

### The Unhappy Path

The message is sent over the canonical bridge which can take 7-8 days when sending from optimistic rollups.

### Protocol Design

There is a fundamental trade off between latency and security --- a protocol can be fast & insecure or slow & secure. However a protocol can be fast & secure aslong as the fast mechanism fallsback on a slow secure system. Vea is a cross-chain optimistic game that passes messages fast, but always fallsback on secure, slow native bridges in case of disputes. This means that Vea is usually fast, but could sometimes slow down.

### Security

Vea is a 1 of N security model, meaning only 1 honest participant can force the correct bridge execution. More importantly, Vea is a permissionless protocol, meaning you can make sure your are the one honest participant in the protocol.

The contracts are immutable, with no governance nor multisig upgradability risk.

### Interoperability

Vea can connect any L1, L2, L3... etc which settle to Ethereum including zk rollups. However, when interoperating with optimistic rollups, there are no added security assumptions since optimistic rollups use the same security model as Vea. For this reason, Vea is particularly well suited for optimistic rollup interoperability. Vea can even pass messages between zk rollups and optimistic rollups without any extra security assumptions, because to interoperate with an optimistic rollup already accepts a 1 of N security assumption for the optimistic rollup state verifcation, even which an interoperable dapp lives natively on a zk-rollup

## Learning more about Vea

Read on to further explore the technical deep-dive of the Vea protocol,  use-cases (cross-chain governance, more generally high-value messaging which tolerate the worst-case latency), how to get started building cross-chain dapps with vea.
