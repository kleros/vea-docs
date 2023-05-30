# ❔ FAQ

## Why the name?

Vea is latin for a way, a road, a passage. :bridge\_at\_night: We think it's fitting for a cross-chain interoperability solution.

## Why another bridge?

Vea is a lean solution specialized for authenticated message bridging with minimum sacrifice on security. This approach differs from most solutions designed for token bridges and relying on complex routing middleware.

## Wait, aren't other bridges more battle-tested?

If existing bridges are battle-tested, they may as have lost the battle: Nomad, Wormhole, Ronin, BNB, etc. That’s 5 out of the top 6 biggest hacks in crypto (excluding SBF). The design of Vea removes a lot of attack surface which exist with other bridges: no routing middleware, no governance, no multisig, and no upgradability.

![](https://i.imgur.com/XxVOziF.png)

## What type of bridge is this?

A trust-minimized optimistically-verified bridge, open to any participant to fulfill the roles of Oracle, Challenger, or Relayer. The trust model requires only 1 live honest verifier, similar to optimistic rollups.

## What are the advantages of Vea?

<figure><img src=".gitbook/assets/image (4).png" alt=""><figcaption><p>$ - Vea is cheaper 🏃 - Vea is faster      than native bridges.</p></figcaption></figure>

Vea is faster and cheaper when bridging from Optimistic rollups to Ethereum consensus L1s like Ethereum mainnet and Gnosis Chain. When bridging to L2s from side-chains, Vea is not faster but much cheaper than native bridges. You can bridge from Gnosis to Arbitrum without directly interacting with Ethereum mainnet, meaning Vea provides a lower cost but slower bridging speed.

## How is this secure?

As an optimistic bridge, it is cheap and fast to use in the happy case, where an Oracle makes an unchallenged claim.

While in the unhappy case, it is no different than using the canonical bridges operated by a particular rollup or side-chain.

There is no need for any additional trust assumption on say a 3rd-party oracle or some slow governance mechanism or trusted DAO multisig to ensure that the message is relayed correctly.

Vea works as long as there is one honest participant, and since Vea is permissionless, you can be that honest participant.

## Is there any limitation? What's the trick?

Vea's design choices do come with some limitations and may not be suitable for every use case. Firstly the focus is on message passing primarily (as opposed to tokens). And secondly, the use case must tolerate the worst-case latency when a slow bridge is involved (egress messages from optimistic rollups typically) which is 7 days, plus potentially 1 extra day for forced inclusion in the most extreme case.

## Is it safe to use a shorter challenge period than Optimistic Rollups (1 week)?

Many optimistic rollups were designed and deployed at a time when Ethereum had a proof-of-work consensus model, as a result, many decisions were made considering the uncertain finality guarantees of a proof-of-work consensus model. Under proof-of-stake, the settlement layer has strong finality guarantees. As a result, chain re-organizations are much more rare. The relevance for optimistic systems involves the risk that transactions including honest challenges to fraudulent claims could be censored by malicious actors attacking the settlement layer causing block reorganizations.

The 1-week delay gives honest parties enough time to participate in an interactive fraud-proving scheme by staking on the correct state or by challenging a state transition in a multistep interactive game. Even if this mechanism was to follow an unhappy path, it must eventually converge to a state where the L2 transaction is included, because it has already been written to L1.

The Arbitrum documentation puts it [this way](https://developer.offchainlabs.com/arbos/l2-to-l1-messaging#protocol-design-details):

> The moment a transaction is published on-chain, any observer can anticipate its result; however, for Ethereum itself to accept its result, the protocol must give time for Arbitrum validators to detect and prove fault if need be.

Kevin Fitcher [summarized](https://kelvinfichter.com/pages/thoughts/challenge-periods/) some of the key decisions in designing the week-long challenge period. The 1-week delay is extremely conservative and is designed around a threat model where an attacker might be trying to steal everything of value on a rollup by issuing a single malicious state update and then censoring the honest parties on a proof-of-work settlement layer.

<figure><img src=".gitbook/assets/image (32).png" alt=""><figcaption><p>L2 Rollup native bridge threat model</p></figcaption></figure>

In short, Vea is not a token bridge. Vea is designed specifically for arbitrary messages and authenticated data. Unlike the native bridges, there is no $1 billion honey pot to 'win' by attacking Vea. Vea is designed for low-value high-volume dapp use cases for querying cross-chain information (oracle outputs, governance votes, nft ownership, etc).

Vea's launch partner, Kleros, a decentralized subjective oracle, demonstrates the different threat models using Vea.&#x20;

Consider an example where smart contracts on a chain like Gnosis escrow tokens between Gnosis chain participants. When there is a dispute between the escrow parties, they query an oracle, Kleros, that lives on a different chain. This type of oracle query is an unauthenticated call and any cross-chain relayer network like Gelato could fulfill the cross-chain query.

<figure><img src=".gitbook/assets/image (54).png" alt=""><figcaption><p>Smart contracts on Gnosis chain query an oracle living on an Optimistic Rollup</p></figcaption></figure>

When the oracle returns its result, the authenticated data can be sent with Vea. Note that Vea is the only 3rd party bridge designed for sending authenticated data. Other 3rd party bridges are specialized in token transfer and unauthenticated calls (relayer networks) and are not capable of sending authenticated data.

<figure><img src=".gitbook/assets/image (8).png" alt=""><figcaption><p>Cross-chain oracle returns the result of a query through Vea.</p></figcaption></figure>

Dapps integrating Vea escrow tokens typically of value much less than a billion USD and usually between predetermined parties with fixed outcomes. The end users of these protocols tend to be uncorrelated, and the escrowed token amounts are ephemeral. The tokens are only escrowed for the duration of the oracle query, upon receiving the result, the escrowed tokens are released, unlike native L2 bridges which indefinitely escrow billions of assets where a single fraudulent state update can drain all funds to an arbitrary attacker's address.

<figure><img src=".gitbook/assets/image (59).png" alt=""><figcaption><p>Vea threat model: A relatively 'small' application specific honeypot (much less than $10 billion).</p></figcaption></figure>

In Vea, the value an attacker can gain is specific to the contracts in which the attacker is personally escrowing tokens. There's no common honeypot pool of tokens growing indefinitely. This is the problem native rollup bridges face: the honeypot grows until it's profitable for an attacker to launch a network-level attack on the settlement layer. With different Vea deployments, we can calibrate the challenge period to better reflect the consensus model of the settlement layer, and the security needed by individual applications that are using it - for most applications the challenge periods in Vea will be adequately secure.

## Do you support token bridging?

No. Vea is an arbitrary message bridge. Token bridges could be built on top, but Vea is first and foremost a message bridge.

## What are the next milestones?

<figure><img src=".gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

Currently, we have a permissioned devnet providing developers a good integration experience testing Vea without the latency which will be experienced in the full testnet or mainnet deployments.

The next milestone is the full permissionless testnet, for which we invite as many people as possible to run watcher nodes.

## Who is building this and why?

The [Kleros Cooperative](https://kleros.io/coop), steward of the [Kleros arbitration protocol](https://kleros.io/), is funding the development of Vea as a strategic solution towards [a cross-chain chain arbitration protocol](https://blog.kleros.io/towards-kleros-v2/) (as opposed to a multi-chain one). After surveying the existing bridging solutions, none of them satisfied the team's security appetite.

The original idea for Vea has been conceived in early 2021 as an internal solution for Kleros and mentioned in the [Kleros Yellow Paper (page 6, section 4.2)](https://kleros.io/yellowpaper.pdf) by Clément Lesaege, William George, and Federico Ast.

During development, it emerged that the bridge could easily be application-agnostic and used by other projects without much extra effort. The absence of both privileged functions and reliance on a centralized party has made this easier.

## How does Vea make money?

Neither Vea as a protocol nor as an organization makes money. Vea is deployed as a public good for anyone to use, and for no one to own.

## Who are the protocol participants?

The oracle role is fulfilled by anyone who observes that a snapshot of the state of messages in the bridge is ready for bridging on the sending chain and submits a claim about that state on the receiver chain.

The challenger role is fulfilled by anyone who does not agree with the claim. The challenger does not need to provide an alternative claim, only to signal its disagreement.

The relayer role is fulfilled by anyone who relays the messages encoded in the verified claims on the receiving chain.

## Why would anybody bother acting a protocol participant?

In the unhappy path of the optimistic mechanism, there is a financial incentive for anyone to submit a challenge and be rewarded with the malicious oracle's deposit.

In the happy path, there is no direct financial incentive for anyone. The participants even spend their own funds for gas. So the primary incentive is the utility derived from getting the message across the bridge. In practice, any user, protocol, or entity whose message is part of an epoch batch has an incentive to participate in the bridging process (whether honestly or not).

## I looked at your code, ever heard of DRY?

It is Kleros' smart contract development philosophy of _generally_ not using inheritance. DRY makes the most sense in traditional programming in an unconstrained environment.

Smart contracts are under stronger constraints of security and gas usage, making **the KISS principle more important than the DRY principle**.

![KISS](https://i.imgur.com/YpCuxSq.png)

DRY with smart contracts encourages layers of abstraction (to reuse code) which leads to obfuscation (harder to audit) and higher gas cost (more function calls).

To illustrate this point, Vyper, a newer language made specifically for smart contracts, [does not allow inheritance at all](https://vyper.readthedocs.io/en/stable/index.html?highlight=inheritance#principles-and-goals):

> "Class inheritance requires people to jump between multiple files to understand what a program is doing, and requires people to understand the rules of precedence in case of conflicts (“Which class’s function X is the one that’s actually used?”). Hence, it makes code too complicated to understand which negatively impacts auditability."

## Has the code been audited?

Vea is still under development, an audit may come later.

## Has the protocol been formally verified?

Touché, it has not. If you would like to contribute on this front, please reach out to us!
