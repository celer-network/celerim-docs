# End-to-End Workflow

This section shows the more detailed end-to-end cross-chain message implementation diagrams.&#x20;

### Cross-chain message without token transfer

SrcApp at the source chain wants to send an arbitrary message to DstApp at the destination chain without associated token transfer. Figure below describes the end-to-end workflow.

The SrcApp sends a message to the MessageBus contract on the source chain, which emits the message event. SGN catches the event and collects signatures from all validators. The executor then submits the SGN-signed message to the MessageBus contract on the destination chain, which will verify the message info and then call the message execution function of the DstApp.

![](../../.gitbook/assets/msg-only-flow.png)

### Cross-chain message with token transfer

SrcApp at the source chain wants to send some tokens to DstApp at the destination chain, along with an arbitrary message associated with the transfer. Figure below describes the end-to-end flow of such transfers. <mark style="color:blue;background-color:blue;">Blue</mark> represents token transfer flow, while <mark style="color:green;background-color:green;">green</mark> represents message passing flow. Numbers show time sequence, which means steps with the same number can happen concurrently.

The SrcApp sends both cross-chain token transfer and message passing requests in a single transaction. SGN catches and correlates both events, then completes the token transfer at the destination chain. The executor then submits the SGN-signed message and token transfer info to the MessageBus at the destination chain, which will verify the submitted info and call DstApp to execute the message.

![](../../.gitbook/assets/image.png)

