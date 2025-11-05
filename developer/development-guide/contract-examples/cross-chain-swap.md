# Cross-Chain Swap

Here is an example app that allows swapping one token on chain1 to another token on chain2 through cBridge and DEXes on both chain1 and chain2.

[Source code on GitHub](https://github.com/celer-network/sgn-v2-contracts/blob/main/contracts/message/apps/examples/TransferSwap.sol).

For the simplicity of explanation, let's say we deploy this contract on chain1 and chain2, and we want to input tokenA on chain1 and gain tokenC on chain2.

Public functions `transferWithSwap` and `transferWithSwapNative` are called by a user to initiate the entire process. These functions takes in a `SwapInfo` struct that specifies the behavior or "route" of the execution, and execute the process in the following fashion:

1. Swap tokenA on the source chain to gain tokenB
2. Packages a `SwapRequest` as a "message", which indicates the swap behavior on chain2
3. `sendMessageWithTransfer` is then called internally to send the message along with the tokenB through the bridge to chain2
4. On chain2, `executeMessageWithTransfer` is automatically called when the bridge determines that the execution conditions are met.
5. This contract parses the message received to a `SwapRequest` struct, then executes the swap using the tokenB received to gain tokenC. (Note: when `executeMessageWithTransfer` is called, it is guaranteed that tokenB already arrives at the TransferSwap contract address on chain2. You can check out this part of verification logic in MessageBusReceiver.sol's `executeMessageWithTransfer`).
6. If the execution of `executeMessageWithTransfer` of TransferSwap contract on chain2 reverts, or if the `executeMessageWithTransfer` call returns `false`, then MessageBus would call `executeMessageWithTransferFallback`. This is the place where you implement logic to decide what to do with the received tokenB.

The following is a more graphical explanation of all the supported flows of this demo app:

```
1. swap bridge swap

|--------chain1--------|-----SGN-----|---------chain2--------|
tokenA -> swap -> tokenB -> bridge -> tokenB -> swap -> tokenC -> out

2. swap bridge

|--------chain1--------|-----SGN-----|---------chain2--------|
tokenA -> swap -> tokenB -> bridge -> tokenB -> out

3. bridge swap

|--------chain1--------|-----SGN-----|---------chain2--------|
                  tokenA -> bridge -> tokenA -> swap -> tokenB -> out

4. just swap

|--------chain1--------|
tokenA -> swap -> tokenB -> out
```

####
