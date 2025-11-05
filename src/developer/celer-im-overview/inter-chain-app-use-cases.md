# Inter-chain App Use Cases

The current common practice of making a “multi-blockchain” dApp is done by simply **replicating the same code on multiple chains** where the liquidity, application logic and states of the same dApp on different chains are **completely isolated from each other.** In fact, except for the UI and the protocol token, nothing is shared between these instances.&#x20;

This approach often suffers from low liquidity efficiency, disconnected states, and a degraded user experience. Celer IM allows dApps to tap into the true potential of the multi-blockchain world by enabling a **single-click user experience that benefits from much higher liquidity efficiency and coherent application logic**.&#x20;

Some high-level examples:

* **DEXes** that allow users to swap tokens across multiple chains from just one chain and with only a single transaction (e.g., [ChainHop](https://app.chainhop.exchange/) and [Rango Exchange](https://app.rango.exchange/)).
* **NFT Bridge** that allows users to send their NFTs across different chains (e.g., [cBridge NFT](https://cbridge.celer.network/nft)).
* **Yield aggregators** that allow users to manage multi-blockchain vaults from a single chain
* **Lending protocols** where collateral can be provided on one chain in order to borrow assets on a different chain
* **DAO governance protocols** that allow unified governance mechanisms without requiring governance tokens to be moved across different chains
* **NFT marketplaces** where a user from one chain can place bids on an auction taking place on a completely different chain
* **Metaverse games** where users can interact seamlessly in the game with virtual items from various chains
* **New kinds of cross-chain asset transfer bridges** with different liquidity models, validation models, or even privacy features that can be built and co-exist under the same framework. In fact, [cBridge](http://cbridge.celer.network/) can be seen as an asset bridge built on Celer IM.

Let’s walk through some examples in more detail and then we can dive into a more technical flow walkthrough.&#x20;

**Decentralized Exchanges**

Today a multi-blockchain DEX has to build liquidity pools for the same key asset pairs on every chain they are deployed on. As a result, a DEX has to spread out the farming incentives across all of these different chains for these pairs. Even though the total liquidity across all of the chains may be fairly high, the liquidity depth of each pool on each individual chain is actually spread thin. Unfortunately, this harms the overall trading experience by creating high slippage. In addition, for users who want to make a trade for a token where deep liquidity exists on a different chain, they have to manually swap on the originating chain, use a separate fund bridge app, and then switch to the other chain to make the final swap.&#x20;

DEXes built using Celer IM have a significantly improved trader experience by automatically routing their trade to the deep liquidity pool with just a single transaction. With this innovation, a DEX project will be able to concentrate the farming incentives for a pair of tokens on a single pool, creating deeper liquidity with low slippage.&#x20;

**Lending Protocols**

Today, if a user provides collateral in a lending protocol on one chain, they can only borrow assets on that same chain. In order to borrow assets from a different chain, they have to withdraw their liquidity, manually move it to a different chain, then provide liquidity in the new chain’s collateral pool.&#x20;

Celer IM enables a new kind of inter-chain lending where a user is able to seamlessly move their collateral from a liquidity pool on one chain to a pool on a different chain, all in a single transaction. Then, they can directly borrow assets on that new chain. With this functionality, users will have a simple and clean UX that lets them accomplish what they need to without having to leave the lending application!&#x20;

**NFT Marketplaces**

Today, if a user wants to participate in an NFT auction, they must have funds on the blockchain where the NFT exists. It excludes people who would normally partake in the auction but don’t have funds on that particular chain. When a marketplace like OpenSea is deployed on a chain like Ethereum, a good chunk of the audiences that are on other chains are excluded due to the complicated bridging operations and high gas costs.

Celer IM can help to expand NFT marketplaces to reach a wider audience. An auction will have the ability to take bids across chains other than where the NFT was initially minted. On top of this, there isn’t a need to make any individual cross-chain fund transfers before the auction results are finalized. This significantly reduces the costs of participating in an NFT auction, decreasing the barriers to entry and enlarging the trader pool for the marketplace as a whole.&#x20;
