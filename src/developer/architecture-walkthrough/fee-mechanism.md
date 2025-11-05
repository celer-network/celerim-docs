# Fee Mechanism

### SGN Fee

SGN charges fees to sync, store, and sign messages. Whoever calls `sendMessageWithTransfer` or `sendMessage` in [MessageBusSender](https://github.com/celer-network/sgn-v2-contracts/blob/main/contracts/message/messagebus/MessageBusSender.sol) should put some fee as `msg.value` in the transaction, which will later be distributed to SGN validators and delegators. The fee amount is calculated as [`feeBase + _message.length * feePerByte`](https://github.com/celer-network/sgn-v2-contracts/blob/1c65d5538f/contracts/message/messagebus/MessageBusSender.sol#L133-L135).

### Executor Fee

Executor charges fees to submit execute message transactions. How to charge and distribute executor fees is entirely decided at the application level. Celer IM framework does not enforce any executor fee mechanism.
