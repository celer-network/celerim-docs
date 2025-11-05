# Contract Framework

We provide a [dApp contract framework](https://github.com/celer-network/sgn-v2-contracts/blob/main/contracts/message/framework), which implements the common process of Celer IM.  By inheriting the app framework [MessageApp.sol](https://github.com/celer-network/sgn-v2-contracts/blob/main/contracts/message/framework/MessageApp.sol), **the cross-chain smart contract developers only need to focus on the app-specific logic.**

### Send Message

To send cross-chain messages, the dApp contract needs to interact with the Celer IM [MessageBus  (MessageBusSender) contract](https://github.com/celer-network/sgn-v2-contracts/blob/main/contracts/message/messagebus/MessageBusSender.sol) on the src chain.

#### Send a message without an associated cross-chain token transfer

The dApp contract should use the [sendMessage function](https://github.com/celer-network/sgn-v2-contracts/blob/563638cdd0f056d01f391f412dcec2cdca8914e3/contracts/message/framework/MessageSenderApp.sol#L18-L35) of the inherited MessageSenderApp contract to send a cross-chain message.

```solidity
function sendMessage(
    address _receiver,
    uint64 _dstChainId,
    bytes memory _message,
    uint256 _fee
) internal
```

The message will be typed as `MsgType.MessageOnly`, and will be identified in Celer SGN through its messageId, which is computed as a hash of `<msgType, sender, receiver, srcChainId, srcTxHash, dstChainId, message>`. The caller needs to make sure of the uniqueness of the messageId. If multiple messages with the same Id are sent, only one of them will succeed at the destination chain.

#### Send a message with an associated cross-chain token transfer&#x20;

The dApp contract should use the [sendMessageWithTransfer function](https://github.com/celer-network/sgn-v2-contracts/blob/563638cdd0f056d01f391f412dcec2cdca8914e3/contracts/message/framework/MessageSenderApp.sol#L48-L89) of the inherited MessageSenderApp contract to send messages with associated cross-chain token transfers.

```solidity
function sendMessageWithTransfer(
    address _receiver,
    address _token,
    uint256 _amount,
    uint64 _dstChainId,
    uint64 _nonce,
    uint32 _maxSlippage,
    bytes memory _message,
    MsgDataTypes.BridgeSendType _bridgeSendType,
    uint256 _fee
) internal returns (bytes32) // return transferId
```

The [`BridgeSendType`](https://github.com/celer-network/sgn-v2-contracts/blob/563638cdd0/contracts/message/libraries/MsgDataTypes.sol#L6-L15) could be `Liquidity, PegDeposit, PegBurn, PegV2Deposit, PegV2Burn, PegV2BurnFrom`.

The message will be typed as `MsgType.MessageWithTransfer`, and will be identified in Celer SGN through the returned `transferId`

#### Message Fee

For both function calls above, message fees are charged in the native gas token. Here is how to [calculate and query the fee](https://github.com/celer-network/sgn-v2-contracts/blob/563638cdd0f056d01f391f412dcec2cdca8914e3/contracts/message/messagebus/MessageBusSender.sol#L128-L135).

### Receive Message

To receive cross-chain messages, the dApp contract needs to implement (some of) the `message execution` functions defined in the inherited [MessageReceiverApp](https://github.com/celer-network/sgn-v2-contracts/blob/main/contracts/message/framework/MessageReceiverApp.sol).&#x20;

These functions return `ExecutionStatus`, which could be `Success`, `Fail`, or `Retry`. The [MessageBus (MessageBusReceiver) contract](https://github.com/celer-network/sgn-v2-contracts/blob/main/contracts/message/messagebus/MessageBusReceiver.sol) which calls the dApp's message execution functions will ensure that _each message will be executed exactly once_ if the function returns `Success` or `Fail` or is reverted. If the function returns `Retry`, the message can be executed again later.

#### Receive a message without an associated cross-chain token transfer

The dApp contract should implement the [executeMessage interface](https://github.com/celer-network/sgn-v2-contracts/blob/6e19c559a8b8c644060af53286aa57cb1fc9a46c/contracts/message/framework/MessageReceiverApp.sol#L14-L26) to receive a message without associated token transfer.

```solidity
function executeMessage(
    address _sender,
    uint64 _srcChainId,
    bytes calldata _message,
    address _executor
) external payable virtual override onlyMessageBus returns (ExecutionStatus) {}
```

Do not forget to use the `onlyMessageBus` modifier, as message execution functions should only be called by the `MessageBus` contract.

#### Receive a message with an associated cross-chain token transfer

The dApp contract should implement the [executeMessageWithTransfer interfaces](https://github.com/celer-network/sgn-v2-contracts/blob/6e19c559a8b8c644060af53286aa57cb1fc9a46c/contracts/message/framework/MessageReceiverApp.sol#L37-L90) to receive a message with an associated cross-chain token transfer.

The [MessageBus contract](https://github.com/celer-network/sgn-v2-contracts/blob/main/contracts/message/messagebus/MessageBusReceiver.sol) will guarantee that the correct amount of tokens have already been received by the dApp contract before calling the dApp's message execution functions.

```solidity
// receive and execute messages at dst chain
function executeMessageWithTransfer(
    address _sender,
    address _token,
    uint256 _amount,
    uint64 _srcChainId,
    bytes calldata _message,
    address _executor
) external payable virtual override onlyMessageBus returns (ExecutionStatus) {}
```

If the function above got reverted for any reason. The dApp contract can optionally implement a fallback function to decide what to do with the received tokens.&#x20;

```solidity
// optional fallback function at dst chain
function executeMessageWithTransferFallback(
    address _sender,
    address _token,
    uint256 _amount,
    uint64 _srcChainId,
    bytes calldata _message,
    address _executor
) external payable virtual override onlyMessageBus returns (ExecutionStatus) {}
```

A cross-chain token transfer could fail due to bad slippage or other reasons. In this case, the Celer SGN will refund the token to the dApp contract on the source chain, which should implement a message execution function to handle the possible refund.

```solidity
// handle messages with refunded token transfer at src chain
function executeMessageWithTransferRefund(
    address _token,
    uint256 _amount,
    bytes calldata _message,
    address _executor
) external payable virtual override onlyMessageBus returns (ExecutionStatus) {}ol
```

