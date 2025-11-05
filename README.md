# Celer IM Overview

![](<.gitbook/assets/Celer IM.png>)

Celer IM fundamentally changes how multi-blockchain dApps are **built** and **used**. Instead of deploying multiple isolated copies of smart contracts on different blockchains, **developers** can now build **inter-chain-native** dApps with efficient liquidity utilization, coherent application logic, and shared states. **Users** of Celer IM-enabled dApps will enjoy the benefits of a diverse multi-blockchain ecosystem with the simplicity of a **single-transaction** UX, without complicated manual interactions across multiple blockchains.&#x20;

The Celer IM framework is very **easy-to-use** and allows a **“plug’n’play” upgrade** that often requires **no modifications** with already deployed code. As an example, **Uniswap** and **Sushiswap** can be transformed into a cross-chain DEX with just [a simple plug-in](https://github.com/celer-network/sgn-v2-contracts/blob/main/contracts/message/apps/examples/TransferSwap.sol). There have been some projects that implemented this functionality in production such as [ChainHop](https://app.chainhop.exchange/) and [Rango Exchange](https://app.rango.exchange/).

In the above cross-chain DEX example, a user of Sushiswap can swap their ETH on Arbitrum to BNB on Binance Smart Chain (BSC) **with a single, simple transaction**. Behind the scenes, the following has happened:

* ETH -> USDT on the Arbitrum-side via Sushiswap&#x20;
* USDT on Arbitrum is bridged to BSC via cBridge as part of the Celer IM framework
* An inter-chain message to execute a USDT->BNB swap is sent to BSC via Celer IM
* USDT -> BNB swap on BSC-side Sushiswap is triggered by that remote call on Sushiswap.
