# dApp Safeguard

## App Guardian

[Celer IM app guardian](https://github.com/celer-network/im-guardian) is run by the dApp community to ensure their application security.

The app guardian monitors the [message `Executed` events](https://github.com/celer-network/sgn-v2-contracts/blob/main/contracts/message/messagebus/MessageBusReceiver.sol#L28-L35) emitted from the `MessageBus` contracts on the destination chains, and uses the `srcChainId` and `srcTxHash` fields in the event to look for the matched [`Message` events](https://github.com/celer-network/sgn-v2-contracts/blob/main/contracts/message/messagebus/MessageBusSender.sol#L15) from the `MessageBus` contracts on the source chains. If it fails to find a matched event on the source chain, it will try to [pause the message receiver (dApp) contracts](https://github.com/celer-network/im-guardian/blob/main/guardian/message.go#L35-L39) or execute any dApp-specific logic if added.

Note that Celer IM is already secured by the Celer State Guardian Network (SGN), which is a proven secure decentralized platform that has processed a [large volume](https://cbridge-analytics.celer.network/) of cross-chain asset transfers and tons of cross-chain messages without any security incident. This app guardian is for dApp communities who do not fully trust Celer SGN and want further safety guarantees even if Celer IM is compromised.

## Delayed Message Execution

Message dApps that require extra safeguards can choose to integrate the app contract with the [MessageReceiverAdapter](https://github.com/celer-network/sgn-v2-contracts/blob/main/contracts/message/apps/adapter/MessageReceiverAdapter.sol), which allows configured delayed message execution.

Integration with the adapter is simple: 1) deploy a separate adapter contract for your dApp; 2) set the [allowed senders](https://github.com/celer-network/sgn-v2-contracts/blob/main/contracts/message/apps/adapter/MessageReceiverAdapter.sol#L64) to restrict who can send messages to the dApp; and 3) enforce your dApp message receiver function to only accept [external calls](https://github.com/celer-network/sgn-v2-contracts/blob/main/contracts/message/apps/adapter/MessageReceiverAdapter.sol#L57-L60) from this adapter.

For each delayed message, the [message executor](https://github.com/celer-network/im-executor) (detailed in the next section) will wait for the delay period to pass and then automatically [execute the delayed messages](https://github.com/celer-network/sgn-v2-contracts/blob/main/contracts/message/apps/adapter/MessageReceiverAdapter.sol#L40) which trigger calls to the receiver dApp contract.

The [app guardian](https://github.com/celer-network/im-guardian) described above keeps monitoring and verifying the messages as soon as they enter the delayed queue, and will pause the adapter contract immediately if any invalid message is detected during the delay period, so that no invalid message will be executed in the receiver dApp contract.&#x20;
