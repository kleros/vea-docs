# ❔ FAQ

## Why the name?

Vea is latin for a way, a road, a passage. :bridge\_at\_night: We think it's fitting for a cross-chain interoperability solution.

## Why another bridge?

We are building a lean solution specialized for message bridging with minimum sacrifice on security. This approach differs from most solutions designed for token bridges and relying on complex routing middleware.

## Wait, aren't other bridges more battle-tested?

If existing bridges are battle tested, they may as have lost the battle: Nomad, Wormhole, Ronin, BNB etc. That’s 5 out of the top 6 biggest hacks in crypto (excluding SBF). The design of Vea removes a lot of attack surface which exist with other bridges: no routing middleware, no governance, no multisig, no upgradability.

![](https://i.imgur.com/XxVOziF.png)

## What type of bridge is this?

A trust-minimized optimistically-verified bridge, open to any participant bonded to fulfill the roles of Oracle, Challenger or Relayer. The trust model requires only 1 live honest verifier, similarly to optimistic rollups.

## How is this secure?

As an optimistic bridge, it is cheap and fast to use in the happy case, where an Oracle makes a claim and no Challenger challenges.

While in the unhappy case, it is no different than using the canonical bridges operated by a particular rollup or side-chain.

There is no need for any additional trust assumption on say a Chainlink oracle or some slow governance mechanism or trusted DAO multisig to ensure that the message is relayed correctly.

As long as there is one honest participant running a working implementation of the light client specifications at any time, and anybody can take on this role.

## Is there any limitation? What's the trick?

Vea's design decisions do come with some limitations and may not be suitable for every use case. Firstly the focus is on message passing primarily (as opposed to tokens). And secondly the use case must tolerate the worst case latency when a slow bridge is involved (egress messages from optimistic rollups typically) which is 7 days, plus potentially 1 extra day for forced-inclusion in the most extreme case.

## Is it really safe to skip the 1 week delay of an optimistic rollup?

Once a L2 transaction is written (and heavily compressed) on Ethereum and given enough time for L1 finality ([60 to 95 beacon chain slots](https://notes.ethereum.org/@vbuterin/single\_slot\_finality#Paths-toward-single-slot-finality)), the L2 transaction can be safely considered as final. Therefore it is Vea's oracle role to verify that a) the L2 transaction is indeed included on L1, and b) that sufficient time has passed before submitting a Merkle proof on Vea's L1 contract (L1 finality). That is part of Vea's client specifications.

## Then why the week-long optimistic rollup mechanism?&#x20;

It is intended for the production of the L2 blocks to be written on L1 without relying on a centralized source of truth. This delay gives honest parties enough time to participate in a fraud-proving scheme by staking on the correct state (Arbitrum) or by challenging a state transition (Optimism). Even if this mechanism was to follow an unhappy path, it must eventually converge to a state where the L2 transaction is included, because it has already been written to L1.

The Arbitrum documentation puts it [this way](https://developer.offchainlabs.com/arbos/l2-to-l1-messaging#protocol-design-details):

> The moment a transaction is published on-chain, any observer can anticipate its result; however, for Ethereum itself to accept its result, the protocol must give time for Arbitrum validators to detect and prove fault if need-be.

This is [a good explainer](https://developer.offchainlabs.com/inside-arbitrum-nitro/#sequencing-followed-by-deterministic-execution) of what happens during the 1-week delay for the Arbitrum rollup.

![](https://i.imgur.com/YPF5VLt.png)

We believe that the 1 week delay is extremely conservative and is designed around a threat model where an attacker might be trying to steal everything of value on a rollup by issuing a single malicious state update and then censuring the honest parties. With different Vea deployments, we can calibrate the challenge period to better reflect the security needed by individual applications that are using it - for most applications the challenge periods in Vea will be adequately secure.

## Is there any other risk involved in bridging from an optimistic rollup?

Yes, in the extreme cases where a malicious sequencer would attempt to censor a L2 transaction by not including it on L1, or a L1 block producer would attempt not to include a transaction from the mempool.

### L2 censorship of the happy path

This is when the L2 batch message is censored on L2. As explained in the previous questions, a Vea oracle _must_ wait for this transaction to be written on L1 before submitting a proof, otherwise the proof must be challenged.

### L2 censorship of the unhappy path

This is when the fallback to the canonical bridge is censored. Arbitrum provides a mechanism allowing anyone to forcibly include a valid L2 transaction on L1 by submitting it to its delayed inbox contract. A malicious sequencer can [only delay a transaction by 24 hours but not censor it](https://developer.offchainlabs.com/inside-arbitrum-nitro/#inboxes-fast-and-slow). Eventually a Vea oracle will detect that the L2 transaction has indeed been written to L1 and will submit the corresponding proof.

### L1 censorship of the Oracle or the Challenger

Censorship resistance is guaranteed by the L1 blockchain protocol directly, which is Ethereum for Vea in all cases so far. For Ethereum, it is generally accepted that a transaction cannot be practically censored but only delayed by a reasonably small number of blocks. If this assumption breaks, the Ethereum community can resort to [social slashings](https://ethereum.org/en/developers/docs/consensus-mechanisms/pos/faqs/#what-is-social-slashing) against the attacker.

## What about ZK bridges?

ZK bridges like [Succinct](https://succinct.xyz/) certainly look promising. The underlying technologies are more bleeding-edge compared to optimistic mechanisms. ZK bridges make a different trade-off on the security spectrum, they are typically more trust-minimized but more complex, more susceptible to any implementation bugs being present (yes even more than smart contracts!).

In addition Vea may coexist alongside ZK bridges by relying on them in the unhappy path of the optimistic protocol, as a fallback bridge route.

## Do you support token bridging?

No, not in the near future. It would add complexity around balancing and storing liquidity of each token on each network which would be detrimental to the stated goal of building a lean interoperability solution. Another project may be able to build this on top of Vea.

## What are the next milestones?

* Publishing the client specification.
* Releasing the first client implementation in Typescript packaged as standalone, docker image and [DappNode](https://dappnode.com/) package.
* Deploying a stable instances between Arbitrum to Ethereum (and possibly Optimism to Ethereum).
* Deploying instances between Gnosis, Ethereum, Polygon, BSC.
* Developing Vea v2 with more sustainable incentives, more rewarding for protocol participants (those acting as oracle, challenger, relayer).

## Who is building this and why?

The [Kleros Cooperative](https://kleros.io/coop), steward of the [Kleros arbitration protocol](https://kleros.io/), is funding the development of Vea as a strategic solution towards [a cross-chain chain arbitration protocol](https://blog.kleros.io/towards-kleros-v2/) (as opposed to a multi-chain one). After surveying the existing bridging solutions, none of them satisfied the team's security appetite.

The original idea for Vea has been conceived in early 2021 as an internal solution for Kleros and mentioned in the [Kleros Yellow Paper (page 6, section 4.2)](https://kleros.io/yellowpaper.pdf) by Clément Lesaege, William George, and Federico Ast.

During development it emerged that the bridge could easily be application-agnostic and used by other projects without much extra effort. The absence of both privileged functions and reliance on centralized party has definitely made this easier.

## How does Vea make money?

Neither Vea as a protocol or an organization makes money in its current form. Vea is deployed as a public good for anyone to use, and for no one to own. Meanwhile we can gauge Vea's performance and utility when the rubber hits the ground.

A future version of the Vea protocol may bake in more sustainable incentives, more rewarding for protocol participants (those acting as oracle, challenger, relayer).

## Who are the protocol participants?

The oracle role is fulfilled by anyone who observes that a message batch is ready for bridging and submits the corresponding cryptographic proof across domain on the receiver side.

The challenger role is fulfilled by anyone who does not agree with the proof submitted by an oracle. The challenger does not need to provide an alternative proof, only to signal its disagreement.

The relayer role is fulfilled by anyone who relays the messages which have undergone verification through either the happy or unhappy path.

## Why would anybody bother acting a protocol participant?

In the unhappy path of the optimistic mechanism, there is a financial incentive for anyone to submit a challenge and be rewarded with the malicious oracle's deposit.

In the happy path, there is no direct financial incentive for anyone. The participants even spend their own funds for gas. So the primary incentive is the utility derived from getting the message across the bridge. In practice, any user, protocol, or entity whose message is part of an epoch batch has an incentive to participate in the bridging process (whether honestly or not).

## I looked at your code, ever heard of DRY?

It is Kleros smart contract development philosophy of _generally_ not using inheritance. DRY makes the most sense in traditional programming in an unconstrained environment.

Smart contracts are under stronger constraints of security and gas usage, making **the KISS principle more important than the DRY principle**.

![KISS](https://i.imgur.com/YpCuxSq.png)

DRY with smart contracts encourages layers of abstraction (to reuse code) which leads to obfuscation (harder to audit) and higher gas cost (more function calls).

To illustrate this point, Vyper, a newer language made specifically for smart contracts, [does not allow inheritance at all](https://vyper.readthedocs.io/en/stable/index.html?highlight=inheritance#principles-and-goals):

> "Class inheritance requires people to jump between multiple files to understand what a program is doing, and requires people to understand the rules of precedence in case of conflicts (“Which class’s function X is the one that’s actually used?”). Hence, it makes code too complicated to understand which negatively impacts auditability."

## Has the code been audited?

Vea is still under development, an audit may come later.

## Has the protocol been formally verified?

Touché, it has not. If you would like to contribute on this front, please reach out to us!

