---
description: Merkle Mountain Range implementation details for contract reviewers.
---

# 📝 Implementation Details

### Overview

The type of merkle tree implemented in the Vea [inbox](https://github.com/kleros/vea/blob/2410617e6e6c243bc3108059c39703350031ead2/contracts/src/arbitrumToEth/VeaInboxArbToEth.sol) is sometimes referred to as a merkle mountain range ([MMR](https://github.com/opentimestamps/opentimestamps-server/blob/master/doc/merkle-mountain-range.md)). Without additional context, merkle trees are usually understood to be perfectly balanced binary trees with a fixed height. MMRs on the other hand grow in height as more leafs are inserted, and can be represented by a set of merkle subtrees, referred to as merkle "mountain ranges" due to their shape, see diagram below.

<figure><img src="../../.gitbook/assets/image (44).png" alt=""><figcaption><p>Merkle Mountain Range representing 7 messages</p></figcaption></figure>

### Leaf Hashing

All leafs are double hashed to avoid [second preimage attacks](https://flawed.net.nz/2018/02/21/attacking-merkle-trees-with-a-second-preimage-attack/).

### Inbox Data Structure

Merkle mountain ranges efficiently represent the state of all messages with a limited amount of data made available in the vea inbox contract stored in the [inbox](https://github.com/kleros/vea/blob/2410617e6e6c243bc3108059c39703350031ead2/contracts/src/arbitrumToEth/VeaInboxArbToEth.sol#L34) variable.

```
bytes32[64] public inbox; // stores minimal set of complete subtree roots of the merkle tree to increment.
```

The [inbox](https://github.com/kleros/vea/blob/c78180985507611b3f6b69c2863a7a36e1daed47/contracts/src/arbitrumToEth/VeaInboxArbToEth.sol#L50) represents the 'mountain ranges' or subtrees of varying height. eg inbox\[4] = root of a merkle tree of size 2^4 = 16 leaves.

The inbox state for the above merkle tree example with 7 messages is given by the table below.

<table><thead><tr><th width="101">Count</th><th width="201" align="center">Inbox[2]</th><th width="180" align="center">Inbox[1]</th><th align="center">Inbox[0]</th></tr></thead><tbody><tr><td>0b101</td><td align="center">H(1,4)</td><td align="center">H(5,6)</td><td align="center">H(7)</td></tr></tbody></table>

where we use the notation H as a short-hand for:

* H(1,4) = root of merkle tree representing messages m1, m2, m3, m4
* H(5,6) = root of merkle tree representing messages m5, m6
* H(7) = root of merkle tree representing message m7

### Notation

$$
H(n):= keccak(keccak(m_n)))
$$

H(n) represents the n-th leaf which is the double hash of the n-th message m\_n.

H(n,m) represents an interior node of the tree which is the merkle root representing the leaves H(n), H(n+1), ..., H(m).

The parent of a pair of nodes is calculated by sorting & concatenating the nodes, and hashing the result.

for example

$$
H(1,2):= keccak(H(1) \mathbin{||}H(2))
$$

$$
H(3,4):= keccak(H(3) \mathbin{||}H(4)
$$

$$
H(1,4):= keccak(H(1,2) \mathbin{||}H(3,4))
$$



Note that we should sort hash pairs before hashing. eg. if H(2) < H(1), then we would concatenate in the opposite order.

$$
H(1,2) = keccak(H(2) \mathbin{||}H(1))
$$

Above we neglect the sorting notation for brevity.

### Inbox State Examples & Properties

Another example, showing the inbox state step by step for 7 insertions.

<table><thead><tr><th width="106">Count</th><th align="center">Inbox[2]</th><th align="center">Inbox[1]</th><th align="center">Inbox[0]</th></tr></thead><tbody><tr><td>0b001</td><td align="center">---</td><td align="center">---</td><td align="center">H(1)</td></tr><tr><td>0b010</td><td align="center">---</td><td align="center">H(1,2)</td><td align="center">"H(1)"</td></tr><tr><td>0b011</td><td align="center">---</td><td align="center">H(1,2)</td><td align="center">H(3)</td></tr><tr><td>0b100</td><td align="center">H(1,4)</td><td align="center">"H(1,2)"</td><td align="center">"H(3)"</td></tr><tr><td>0b101</td><td align="center">H(1,4)</td><td align="center">"H(1,2)"</td><td align="center">H(5)</td></tr><tr><td>0b110</td><td align="center">H(1,4)</td><td align="center">H(5,6)</td><td align="center">"H(5)"</td></tr><tr><td>0b111</td><td align="center">H(1,4)</td><td align="center">H(5,6)</td><td align="center">H(7)</td></tr></tbody></table>

Note some properties about the on bits ("1s") in count:

* The fist set bit of count corresponds to the modified inbox index.
* The on bits of count indicate the minimal data to represent the tree.

For example, when count = 0b010, inbox\[0] is set to H(1). However inbox\[1] = H(1,2) which implicitly includes H(1), so inbox\[0] is not needed to encode the tree. From the perspective of data availability, we can forget about H(1) in the slot represented by inbox\[0]. For that reason it is represented in quotation marks and can be overwritten in future steps, reusing the dirty inbox slot for efficiency.

### Calculating the root example

To calculate the root, we hash together the data in each inbox slot corresponding to an on bit in count, from the lowest index to the highest.

<table><thead><tr><th width="96">Count</th><th width="96" align="center">Inbox[2]</th><th width="93" align="center">Inbox[1]</th><th width="96" align="center">Inbox[0]</th><th align="center">root</th></tr></thead><tbody><tr><td>0b001</td><td align="center">---</td><td align="center">---</td><td align="center">H(1)</td><td align="center">H(1)</td></tr><tr><td>0b010</td><td align="center">---</td><td align="center">H(1,2)</td><td align="center">"H(1)"</td><td align="center">H(1,2)</td></tr><tr><td>0b011</td><td align="center">---</td><td align="center">H(1,2)</td><td align="center">H(3)</td><td align="center">H( H(3) , H(1,2) )</td></tr><tr><td>0b100</td><td align="center">H(1,4)</td><td align="center">"H(1,2)"</td><td align="center">"H(3)"</td><td align="center">H(1,4)</td></tr><tr><td>0b101</td><td align="center">H(1,4)</td><td align="center">"H(1,2)"</td><td align="center">H(5)</td><td align="center">H( H(1,4) , H(5) )</td></tr><tr><td>0b110</td><td align="center">H(1,4)</td><td align="center">H(5,6)</td><td align="center">"H(5)"</td><td align="center">H( H(1,4) , H(5,6) )</td></tr><tr><td>0b111</td><td align="center">H(1,4)</td><td align="center">H(5,6)</td><td align="center">H(7)</td><td align="center">H(H(1,4),H(H(5,6),H(7)))</td></tr></tbody></table>

## Other resources

Here are some useful resources to better understand merkle mountain ranges. The notation and indices used in the below resources differs from the notation used in this document. Resources are meant to be illustrative and supplemental.

* [opentimestamps/opentimestamps-server](https://github.com/opentimestamps/opentimestamps-server/blob/master/doc/merkle-mountain-range.md)
* [mimblewimble/grin](https://github.com/mimblewimble/grin/blob/master/doc/mmr.md)

