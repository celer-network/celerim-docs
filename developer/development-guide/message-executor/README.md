# Message Executor

[The executor](https://github.com/celer-network/im-executor) monitors the Celer SGN for messages ready to be submitted (with enough validator signatures) and submits the [message execution transactions](https://github.com/celer-network/sgn-v2-contracts/blob/1c65d5538ff8509c7e2626bb1a857683db775231/contracts/message/interfaces/IMessageBus.sol#L50-L98) to the [MessageBus ](https://github.com/celer-network/sgn-v2-contracts/blob/main/contracts/message/messagebus/MessageBusReceiver.sol)contract.

**In most cases, we recommend dApp developers use the shared executor services provided by the Celer Network team** so that the developers do not need to worry about the executor server configuration and operation.&#x20;

> IMPORTANT: Fill out this form for Celer to set up a hosted executor service for you: [https://form.typeform.com/to/RsiUR9Xz](https://form.typeform.com/to/RsiUR9Xz)

**If you choose to run your own executors**, please refer to the integration guides in the following sections You can also find the config handbook of the message executor in the [GitHub Readme](https://github.com/celer-network/sgn-v2-networks/tree/main/executor).
