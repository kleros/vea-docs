---
description: https://vea-lightbulb.vercel.app/
---

# 💡 Lightbulb Demo

The lightbulb [demo](https://vea-lightbulb.vercel.app/) demonstrates a simple dapp integrated with Vea. The switch lives on Arbitrum Goerli and the Lightbulb lives on a different chain, either Chiado or Goerli, and they communicate with Vea.

[Try it out](https://vea-lightbulb.vercel.app/)! Don't have Arbitrum Goerli eth? No problem, there's a [faucet](https://faucet.triangleplatform.com/arbitrum/goerli) for that.

## User flow

### 1. Hit The Switch (Arbitrum Goerli)

<figure><img src="../../.gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure>

### 2. Wait for Vea

<figure><img src="../../.gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

The status of the Vea bridging process and whether your messages have been bridged can be tracked in [VeaScan](https://veascan.io/). You can find the messageId associated with the switch transaction in the event log.

<figure><img src="../../.gitbook/assets/image (43).png" alt=""><figcaption><p>Switch Transaction event log</p></figcaption></figure>

When the Vea devnet has bridged your message, you can find your message in a recently bridged 'snapshot'.

<figure><img src="../../.gitbook/assets/image (24).png" alt=""><figcaption><p>VeaScan successfully bridged message.</p></figcaption></figure>

### 3. Wait For Relayers

Now that the message to turn on the lightbulb is ready, we can relay it manually, or we can wait for relayers. For the demo lightbulb deployment, we have relayers setup to run 5 minutes after bridging. For your own lightbulb deployment and other Vea integrations, you will need to relay the transactions yourself.

<figure><img src="../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

## The Final Result

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>
