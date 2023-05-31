# 🤩 Getting Started

### Integrating Vea

For each sending and receiving chain pair supported by Vea, there is a separate set of Vea contract [deployments](../introduction/deployment-addresses.md). For each chain, there is exactly one deployed contract.

* Sending Chain: [VeaInbox](https://github.com/kleros/vea/blob/2410617e6e6c243bc3108059c39703350031ead2/contracts/src/arbitrumToEth/VeaInboxArbToEth.sol) - Manages state of all messages sent through Vea.
* Receiving Chain: [VeaOutbox](https://github.com/kleros/vea/blob/2410617e6e6c243bc3108059c39703350031ead2/contracts/src/arbitrumToEth/VeaOutboxArbToEth.sol) - Manages optimistic game over inbox state

To integrate Vea, sender and receiver gateway contracts need to be deployed to interface with the Vea inbox and outbox.

<figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption><p>Vea Integration with Gateways</p></figcaption></figure>

### 1. Sender Gateway

For each sending, receiving chain pair supported by Vea, there is a 'Vea Inbox' contract [deployed](../introduction/deployment-addresses.md) on the sending chain. Contracts send messages through Vea by calling the 'sendMessage(...)' function in the Vea Inbox.

```solidity
function sendMessage(address _to, bytes4 _fnSelector, bytes memory _data) 
```

* `_to` The address on the receiving chain to call
* `_fnSelector` The function selector of the receiving contract to call
* `_data` The abi encoded calldata to pass with the function call.
  * e.g. abi.encode(arg1, arg2, arg3. . .)

The sender gateway should implement the ISenderGateway [interface](https://github.com/kleros/vea/blob/add723da87c885a4d939da396279daa5fd688677/contracts/src/interfaces/gateways/ISenderGateway.sol).

```solidity
interface ISenderGateway {
    function veaInbox() external view returns (IVeaInbox);

    function receiverGateway() external view returns (address);
}
```

where the IVeaInbox [interface](https://github.com/kleros/vea/blob/add723da87c885a4d939da396279daa5fd688677/contracts/src/interfaces/inboxes/IVeaInbox.sol) includes the function stub to send messages through Vea.

```solidity
interface IVeaInbox {
    /// @dev Sends an arbitrary message to receiving chain.
    /// Note: Calls authenticated by receiving gateway checking the sender argument.
    /// @param _to The cross-domain contract address which receives the calldata.
    /// @param _fnSelection The function selector of the receiving contract.
    /// @param _data The message calldata, abi.encode(...)
    /// @return msgId The index of the message in the inbox, as a message Id, needed to relay the message.
    function sendMessage(
        address _to,
        bytes4 _fnSelection,
        bytes memory _data
    ) external returns (uint64 msgId);
}
```

To send a message through Vea, the sending gateway should implement a function to call sendMessage(...) in the Vea Inbox. Here is an example [mock implementation](https://github.com/kleros/vea/blob/add723da87c885a4d939da396279daa5fd688677/contracts/src/test/gateways/SenderGatewayMock.sol#L25).

```solidity

pragma solidity 0.8.18;

import "IReceiverGatewayMock.sol";
import "ISenderGateway.sol";

/// Sender Gateway
/// Counterpart of `ReceiverGatewayMock`
contract SenderGatewayMock is ISenderGateway {
    IVeaInbox public immutable override veaInbox;
    address public immutable override receiverGateway;

    event veaMessageSent(uint64 msgId);

    constructor(IVeaInbox _veaInbox, address _receiverGateway) {
        veaInbox = _veaInbox;
        receiverGateway = _receiverGateway;
    }

    function sendMessage(uint256 _data) external {
        bytes4 methodSelector = IReceiverGatewayMock.receiveMessage.selector;
        bytes memory data = abi.encode(_data);
        uint64 msgId = veaInbox.sendMessage(receiverGateway, methodSelector, data);
        emit veaMessageSent(msgId);
    }
}
```

Notice that the sendMessage(...) call in the Vea Inbox returns a uint64 message id. This id is used to relay the message on the receiving chain. Your dapp will probably want to index these messages with an event like below to later relay the message.

```solidity
emit veaMessageSent(msgId);
```

In this example, the sender gateway is sending some uint256 \_data. The Vea Inbox contract expects a bytes array encoding the calldata to be passed with the cross-chain call.

```solidity
bytes memory data = abi.encode(_data);
```

The data which you include in the cross-chain message depends on your specific application.&#x20;

### 2. Receiver Gateway

For each sending, receiving chain pair supported by Vea, there is a 'Vea Outbox' contract [deployed](../introduction/deployment-addresses.md) on the receiving chain. Receiver gateways receive messages from Vea by receiving calls from the '[sendMessage](https://github.com/kleros/vea/blob/add723da87c885a4d939da396279daa5fd688677/contracts/src/arbitrumToEth/VeaOutboxArbToEth.sol#L243)(...)' function in the Vea Outbox.

```solidity
function sendMessage(
    bytes32[] calldata _proof,
    uint64 _msgId,
    address _to,
    bytes calldata _message)
```

* `_proof` The merkle inclusion proof
* `_msgId` The message id to relay
* `_to` The address to call
* `_data` The message data to relay

The [Vea SDK](vea-sdk.md) provides utility functions to calculate proofs and fetch message data to relay.

#### IReceiverGateway Function Specification

In order to implement a cross-chain call, you need to specify the function selector in the receiver gateway to call. You should define the function stub to call in an interface for the receiver gateway. Here's an example where we define the function stub `receiveMessage`.

```solidity
interface IReceiverGatewayMock is IReceiverGateway {
    /// Receive the message from the sender gateway.
    function receiveMessage(address msgSender, uint256 _data) external;
}

interface IReceiverGateway {
    function veaOutbox() external view returns (address);

    function senderGateway() external view returns (address);
}
```

Note that Vea passes the msg.sender who called the sendMessage(...) function on the sending chain as the first argument of any cross-chain call. This means that the IReceiverGateway interface should always contain the msgSender as the first argument.

<pre class="language-solidity"><code class="lang-solidity">interface IReceiverGatewayMock is IReceiverGateway {
    /// Receive the message from the sender gateway.
<strong>    function receiveMessage(address msgSender, /*your types here*/) external;
</strong><strong>}
</strong></code></pre>

Here's a complete example of a ReceiverGateway.

```solidity
pragma solidity 0.8.18;

import "./IReceiverGatewayMock.sol";

/// Receiver Gateway Mock
/// Counterpart of `SenderGatewayMock`
contract ReceiverGatewayMock is IReceiverGatewayMock {
    address public immutable override veaOutbox;
    address public immutable override senderGateway;

    uint256 public data;

    constructor(address _veaOutbox, address _senderGateway) {
        veaOutbox = _veaOutbox;
        senderGateway = _senderGateway;
    }

    modifier onlyFromAuthenticatedVeaSender(address messageSender) {
        require(veaOutbox == msg.sender, "Vea Bridge only.");
        require(messageSender == senderGateway, "Only the sender gateway is allowed.");
        _;
    }

    /// Receive the message from the sender gateway.
    function receiveMessage(
        address messageSender,
        uint256 _data
    ) external onlyFromAuthenticatedVeaSender(messageSender) {
        data = _data;
    }
}

```

#### Message Sender Authentication

The message sender of a cross-chain call is always the first argument of calldata passed to any receiver gateways.&#x20;

```solidity
modifier onlyFromAuthenticatedVeaSender(address messageSender) {
    require(veaOutbox == msg.sender, "Vea Bridge only.");
    require(messageSender == senderGateway, "Only the sender gateway is allowed.");
    _;
}

function receiveMessage(
    address messageSender,
    uint256 _data
) external onlyFromAuthenticatedVeaSender(messageSender) {...}
```

Cross-chain call authentication requires checking that the msg.sender on the sending chain is the Vea Outbox contract, and checking the first argument of the call is equal to the sender gateway like shown in the modifier `onlyFromAuthenticatedVeaSender`.&#x20;

#### Customizing the data types sent between gateways

To support transfer of data types other than uint256 as shown in this example, simply define those custom arguments in the function stub of the IReceiverGateway and encode those data types into a bytes array to pass in the SenderGateway when calling `sendMessage(...)` in the Vea Inbox.

For example, to support a call passing data of a string, uint64, and a bytes array, the ISenderGateway would instead implement&#x20;

```solidity
function sendMessage(string memory _data1, uint64 _data2, bytes memory _data3) external {
    bytes4 methodSelector = IReceiverGatewayMock.receiveMessage.selector;
    bytes memory data = abi.encode(_data1, _data2, _data3);
    uint64 msgId = veaInbox.sendMessage(receiverGateway, methodSelector, data);
    emit veaMessageSent(msgId);
}
```

and the IReceiverGateway interface would appropriately include a function selector expecting the data types string, uint64, and bytes, with an additional argument in the first position which will always include the message sender (Vea ensures the message sender will be passed in this argument).

```solidity
interface IReceiverGatewayMock is IReceiverGateway {
    /// Receive the message from the sender gateway.
    function receiveMessage(
        address msgSender,
        string memory _data1,
        uint64 _data2,
        bytes memory _data3
    ) external;
}
```

### 3. Relaying your message

Once messages are bridged by Vea, which can take hours to days depending on the sending and receiving chain pairs, the messages can be relayed by providing a merkle proof of message inclusion in the Vea Outbox contract.

The [Vea SDK](vea-sdk.md) provides utility functions to calculate merkle inclusion proofs given the msgId. Here is an example of relaying a message given the msgId.

```typescript
import { Wallet } from "@ethersproject/wallet";
import VeaSdk from "../src/index";

// Create the Vea client
const vea = VeaSdk.ClientFactory.arbitrumGoerliToChiadoDevnet(
  "https://rpc.goerli.eth.gateway.fm",
  "https://rpc.chiado.gnosis.gateway.fm"
);

// Get the message info
const messageId = 42;
const messageInfo = await vea.getMessageInfo(messageId);

// Relay the message
const privateKey = process.env["PRIVATE_KEY"] ?? "";
const wallet = new Wallet(privateKey, vea.outboxProvider);
await vea.relay(messageInfo, wallet);
```

