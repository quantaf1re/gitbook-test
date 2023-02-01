# AutoHedge

## What Is [AutoHedge](https://hedge.autoswap.trade/)

[AutoHedge](https://hedge.autoswap.trade/) is a tool that allows you to use stablecoins to LP (liquidity provide) on DEXes and get the corresponding trading fees in a 'delta-neutral' way, i.e. the price movement of the tokens in the pair you LP on doesn't affect the value of your position, but you still accumulate fees on top.

For example, if you LP with 100 DAI onto an DAI-ETH pair, and the price of ETH halves, your position will still be worth 100 USDC, which you can withdraw any time. If the APY of the pair is 10%, and you hold it for a year, your position will be worth 110 USDC at that point.

In addition, you can leverage LP to up-to-10x the APY you receive. If the APY of the DAI-ETH pair is 10% and you want to 10x leverage LP, then you'll get 100% APY.

All this happens in a single click, with the same UX as providing liquidity on a DEX or using a stablecoin yield farm.

AutoHedge would be impossible without Autonomy's automation, which is needed to keep the amount you're trying to hedge in sync with the amount of the hedge itself - this is done by [rebalancing](https://autonomy-network.gitbook.io/autonomy-docs/autonomy-network/products/autohedge/how-it-works#rebalance).

## Problem

### Current LPers

For existing LPers, LPing in a bull market is fine because, even with impermanent loss (IL), their position still grows in value even without the trading fees. But in a bear market, the loss in value of the position from price decreases of the tokens could be more than the fees, and therefore they lose money overall. IL in addition also kills returns on DEXes.

### Institutions

Institutions sit on large pools of idle funds that could be put to work in the DeFi markets and provide game-changing amounts of liquidity, but regulations and internal policies don’t allow them to be exposed to the price volatility risk that's inherent to holding volatile assets like ETH. Since most DEX volume/yield is not in stablecoin-stablecoin pairs, that means they’re missing out on most of the profit and better yield opportunities that exist. Likewise, the markets are missing out on more liquidity.

## Solution

AutoHedge allows you to provide liquidity while remaining fully _hedged_. This means that no matter what the price movements of the underlying tokens are, your position value stays the same, but you still accumulate trading fees on top. This effectively turns a regular LP token into something that behaves the exact same as a yield-bearing stablecoin.

This allows existing LPers to remove by far the biggest source of risk to them, leading to them being able to LP with more funds, meaning better prices for traders. It also allows institutions to treat LPing into DEX pairs with volatile assets as holding a yield-bearing stablecoin and enter markets they previously couldn't.

## Leverage

Taking an LP token and using it to borrow more assets to LP with is not a new idea, however, because the AH LP tokens essentially behave like yield-bearing stablecoins, the collateral requirement can be the same as for regular stablecoins. On top of that, because stablecoins are the inputs to AH, then AH LP tokens would be used as collateral to borrow stablecoins, which has a very low risk of liquidation - it would require something to go seriously wrong in the mechanism of AH itself, otherwise theoretically it should be impossible to be liquidated.

If the LTV limit of AH LP tokens is 90% (i.e. a collateral requirement of 111%), then with $10 of initial assets, you could borrow $90 more, deposit them to AH and get $100 of AH LP tokens, then use that as collateral to back the $90 loan. The naive way of achieving this is to deposit $10, the lend out the AH LP tokens, borrow $9, deposit the $9, then lend out the $9 of AH LP tokens you receive from that, and repeat over and over, tho there are more elegant methods.

Leveraging increases all revenues and costs proportionally, so leveraging by a factor of X increases the net APY by a factor of X.

## Future Use Cases

Any time a yield-bearing stablecoin is desired, Ah is a solid use case. For example, a new stablecoin could be backed 1:1 by AH LP tokens, since they never go down in price by design, and only get safer over time as they accumulate yield. Currently, since each AH pair requires 1 of the assets to be a stablecoin, one could argue that this makes the new stablecoin a derivative of other stablecoins. This is true for the current version, but we're already developing the ability to have a hedged LP on volatile-volatile pairs, such as ETH-MKR. This would allow for a new stablecoin to be backed only by truly decentralized assets.



