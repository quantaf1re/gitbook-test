# How It Works (detailed)

This describes the **exact** steps involved in depositing, withdrawing, and rebalancing, with every little detail in the financial analysis. The version that describes these things more simply is [How It Works](https://app.gitbook.com/o/-MeA5RrsH6Q0j4yLUM7C/s/-MeA4rd8vgaMQcZpHyck/\~/changes/qfyK1EUZ2nvHVqQLLQdb/autonomy-network/products/autohedgev2/how-it-works). Derivations for all formulas are [here](https://app.gitbook.com/o/-MeA5RrsH6Q0j4yLUM7C/s/-MeA4rd8vgaMQcZpHyck/\~/changes/qfyK1EUZ2nvHVqQLLQdb/autonomy-network/products/autohedgev2/how-it-works-detailed/derivations).

## Deposit

All of the following happens with a single `deposit` tx using AutoHedge. Assuming you want to participate in a DAI-ETH liquidity pool on Uniswap with 2k DAI and the price of ETH is $2k, assuming the cost of a flashloan is 0.05%, the trading fee is 0.3%, and the trade size impacts the price of the pair so negligibly that we can ignore it:

1. 2k DAI is taken from the user and transferred to the AutoHedge DAI-ETH pool.
2. We need to take out a flashloan of WETH for an equal value of the DAI we're LPing with, but we need to be mindful of the flashloan fee that we need to set aside for paying when repaying the flashloan later. The relevant formulas are:\
   \
   `flAmountVol = (w - (Maths.sqrt((w - (4 * t) * (z / w))) * Maths.sqrt(w))) / (2 * t)` (definitions of w, t, & z in [Derivations](broken-reference))\
   \
   `flFeeVol = flAmountVol * flRate`\
   ``\
   ``Plugging in the known values:\
   `flAmountVol` = 0.999499 WETH\
   `flFeeVol` = 0.00049974937 WETH\
   \
   In addition, we need to pay the fee in WETH and we only have DAI, therefore there's an additional 0.3% fee on swapping the 0.00049974937 WETH flashloan fee from DAI, plus slippage. The formula is:\
   \
   `flFeeStable = getAmountsIn(flFeeVol)`\
   ``\
   ``flFeeStable = 1.002506 DAI (0.050125% of the initial deposit). This leaves 1998.99 DAI that we can LP with, which has the same value as the 0.999499 WETH we flashloaned.
3. AutoHedge takes those 1998.99 DAI and 0.999499 WETH and deposits them into the DEX's liquidity pool.
4. All of the DEX's LP tokens that are received back, worth $3997.98, are used as collateral on a lending platform (which are being lent out, so you receive interest on them too) to borrow an amount of WETH equal to the WETH that’s deposited in the liquidity pool (0.999499 WETH). This means the pool has a collateral ratio of 200%, and stays there due to rebalancing (described below).
5. Repay the flashloan of 0.999499 WETH with the 0.999499 WETH borrowed from the lending platform, in addition to the flashloan fee of 0.00049974937 WETH that we set enough stables aside for earlier.
6. AutoHedge then mints a 2nd LP token, known as AHLP (AutoHedge LP) that's specific to the DEX pair - in this case AHLP-DAI-ETH. This token wraps the whole position (the DEX position, the DEX LP tokens being lent out, and the WETH debt). This token represents a delta-neutral position that also receives LP yields, and therefore essentially acts like a yield-bearing stablecoin.

The actual value of the position is defined as whatever stables you're left with after withdrawing and paying all relevant fees. An estimate is however many stables are in the LP, since all the volatile tokens were borrowed - but, because a flashloan is required to unwind the position, it's slightly less than that. Since there's 1998.99 DAI in the LP at this point, an estimate of the value of the position is:\
\
`positionValue = stableAmount * (1 - flFeeRate)`\
\
positionValue (after withdrawing and all relevant fees) = 1997.9905 DAI

Therefore by simply depositing to and withdrawing from AH, you pay around `100 - 100*(1997.9905 / 2000) =` 0.100475% in total fees for flashloaning/swapping.

That's it! Now you're fully hedged. If the price of ETH suddenly halves, then the value of the DEX position (1998.99 DAI and 0.999499 WETH) is worth $2998.489, but the debt (0.999499 WETH) that you have to repay when withdrawing is only worth $999.499, so the net value of the position is still $1998.99 that you started with (accounting for the flashloan fee but not the protocol fee). However, in reality the price of WETH changing would mean that people trade in the DEX pair which subsequently changes the ratio of assets you have. If the price of ETH went up then you'd end up with more DAI and less WETH in the DEX position than just after you deposited. Since the amount of WETH in the LP changes (to be < 0.999499 WETH) but the debt doesn't change (still 0.999499 WETH), that means the hedge is no longer exact - that's where Autonomy and rebalancing comes in, described below.

A diagram showing the logic flow of a single `deposit` transaction:

<figure><img src="../../../../.gitbook/assets/Screenshot from 2022-08-28 18-41-09.png" alt=""><figcaption></figcaption></figure>

## Withdraw



## Rebalance

As described above, the amount of the volatile token in the DEX LP, such as WETH, can go 'out of sync' from the amount of debt due to price changes of the asset or from accumulating trading fees. If you have, say, 1.1 WETH in the LP, but still only have a debt of 1 WETH, that means you're exposed to the price risk of the difference (0.1 WETH) and are therefore no longer delta-neutral.

Therefore in order to stay as close to fully hedged as possible, each pair needs to keep the debt of the volatile token in sync with the amount of that token owned in the DEX position - this rebalancing is impossible to do automatically in a decentralized way without Autonomy. There are a couple of costs associated with rebalancing - the swap fee (see below) and the execution fee charged by Autonomy (gas cost + 30%). By default the rebalancing occurs when the amounts go out of sync by 1% in either direction. For example if there was 2k DAI and 1 WETH in our underlying DEX LP, and 1 WETH borrowed, the price of ETH is $2k, and the trading fee is 0.3%, and the Autonomy fee is $1.56 (around 800k gas for a rebalance, and assuming we're on BSC where AutoHedge is currently live, where the gas price is usually 5 gwei:\
800,000 \* 5\*10^9 \* 1.3 / 10^18 = 0.0052 BNB, at $300/BNB, is $1.56). We're using the example of ETH which has the same units of BNB for paying gas fees (and they have similar gas prices at the time of writing this anyway):&#x20;

### The price of ETH increases

If the price of ETH increased such that the amount of WETH in the DEX pair dropped to 0.99 WETH, that would trigger a rebalance, and 0.01 WETH would need to be repaid, and so we need 0.01523 ETH total to also pay for the execution fee. Due to the `x*y=k` curve used by AMMs, `k` was previously `1 * 2000` = 2000, the amount of DAI in the pair would be `2000 / 0.99` = 2,020.202, meaning the price of ETH is `2,020.202 / 0.99 =` $2,040.608 (ignoring any trading fees the pair received since we have no idea if the price went down in a straight line or it was more volatile and therefore generated more fees). The total amount that we need to take out of the LP to pay for things (the debt difference and the execution fee) is 0.01 + 0.0052 = 0.0152 WETH. This happens with the following steps:

1. We need to repay WETH, and so every time we take X WETH out of the LP, a corresponding value of DAI comes with it. Since taking out X WETH from the LP means we have to reduce the debt by X WETH also, the WETH that we actually repay the 0.0152 WETH debt with effectively comes from the DAI, so we need to account for the swapping fee included in that, too, therefore we need to take out:\
   \
   `0.0152 / 0.997 =` 0.0152457 WETH\
   Which, at the price of $2,040.608, is 31.11057 DAI\
   \
   Enough DEX LP tokens are withdrawn from the lending platform that are redeemable for 31.11057 DAI and 0.0152457 WETH. This is safe to do because the collateral ratio of the debt was originally 200% and this only removes 1% of the collateral, meaning it's still far from liquidation.
2. The 31.11057 DAI is swapped for 0.0152 WETH. The AutoHedge pool now has 0.0304457 WETH 'liquid'.
3. 0.0052 WETH is converted to ETH and used to pay the execution fee, leaving 0.0252457 'liquid'
4. The 0.0252457 WETH is used to repay the loan, reducing the debt to 0.9747543 WETH. Therefore the amounts left in the DEX LP are now:\
   \
   `2020.202 - 31.11057` = 1989.09143 DAI\
   `0.99 - 0.0152457` = 0.9747543 WETH in the LP, the exact same as the debt.

With the new price of ETH, the total value of the position is $1989.09143 (because the WETH position and debt cancel eachother out, we're just left with the DAI position). Note that the numbers here are just examples, it's likely the in real life the losses in rebalancing are smaller, mostly because the execution fee will be smaller. These fees are also spread out over all the capital in the pair, and therefore represents a tiny fraction of a % of all the capital in the pair, approaching 0% as the TVL grows. It is much smaller than the APY of the DEX pair itself, making it net profitable.&#x20;

This $1989.09143 is the same as the initial $2k deposit amount, minus costs involved in the rebalance:

* the execution fee (`0.0052 * 2040.608` = $10.611)
* the fee involved in the swap in stage 2. above (`31.11057 - (0.0152 * 2040.608)` = $0.0933)
* the "[drift](broken-reference)" (the exposure of a net 0.01 WETH of debt over the price increase from $2000 to $2040.608) which we can approximate by assuming the price change and debt increase is linear, and therefore use the midpoint of the debt difference of 0.005 WETH over the price change `0.005 * (2040.608 - 2000)` = $0.20304

The end balance plus all the costs are:\
`1989.09143 + 10.611 + 0.0933 + 0.20304` = $1999.99877 which is equal to the starting deposit of $2,000, taking into account the rounding of decimals I did in this example (for brevity/readability) and the slight inaccuracy of the approximation of the drift as linear.

### The price of ETH decreased

If the price of ETH decreased such that the amount of WETH in the DEX pair rose to 1.01 WETH, that would trigger a rebalance, and 0.01 WETH would need to be borrowed. Due to the `x*y=k` curve used by AMMs, the amount of DAI in the pair would be 1,980.198, meaning the price of ETH is `1,980.198 / 1.01 =` $1,960.592 (ignoring any trading fees the pair received since we have no idea if the price went down in a straight line or it was more volatile and therefore generated more fees). This happens with the following steps:

1. Without having to account for the 0.0052 execution fee, we would borrow roughly twice the debt difference (0.02 WETH total), because we know we have to swap roughly half of that into stables to LP with (because every time we LP with WETH, it doesn't count towards the _difference_ between the debt and WETH in the LP). However, the execution fee doesn't get half of it swapped, it just gets paid directly - it affects the debt position but not the LP. Therefore out of the 0.01 WETH we have to reduce the debt difference by, if we borrow the 0.0052 WETH to pay the execution fee, then we effectively only have to make up the remaining difference of 0.0048 WETH.\
   \
   Since there's a trading fee when swapping WETH to DAI, swapping 0.0048 WETH will result in:\
   `0.0048 * 0.997 * 1960.592 =` 9.382609 DAI\
   In order to match that equivalent value in WETH (because of value lost in the trading fee), we need  an extra `9.382609 / 1960.592 =` 0.0047856 WETH.\
   \
   So in total we borrow `0.0047856 + 0.0048 + 0.0052 =` 0.0147856 WETH. This is safe to do because the collateral ratio of the debt was originally 200% and this only temporarily decreases that by a small %.
2. 0.0052 WETH is converted to ETH and paid to Autonomy, leaving 0.0095856 WETH 'liquid' in the AutoHedge pool
3. 0.0048 WETH is converted into 9.382609 DAI, leaving that amount of DAI and 0.0047856 WETH 'liquid' in the AutoHedge pool
4. The 9.382609 DAI and 0.0047856 WETH are used to LP on the DEX, and the LP tokens added as collateral on the lending platform.\
   The debt is now `1 + 0.0147856 =` 1.0147856 WETH.\
   The amount of WETH in the LP is now `1.01 + 0.0047856 =` 1.0147856 WETH, exactly the same as the debt.\
   The amount of DAI in the LP is now `1980 + 9.382609 =` 1989.382609 DAI.

With the new price of ETH, the total value is 1989.382609 (because the WETH position and debt cancel eachother out, we're just left with the DAI position). This is the same as the initial $2k deposit amount, minus costs involved in the rebalance:

* the execution fee (`0.0052 * 1960.592` = $10.195)
* the fee involved in the swap in stage 1./2. above ((`0.0048 * 1960.592`) `- 9.382609` = $0.0282)
* the "[drift](broken-reference)" (the exposure of a net 0.01 WETH of debt over the price decrease from $2000 to $1,960.592) which we can approximate by assuming the price change and debt increase is linear, and therefore use the midpoint of the debt difference of 0.005 WETH over the price change `0.005 * (2000 - 1960.592)` = $0.19704

The end balance plus all the costs are:\
`1989.382609 + 10.195 + 0.0282 + 0.19704 =` $1999.802849 which is equal to the starting deposit of $2,000, taking into account the rounding of decimals I did in this example (for brevity/readability) and the slight inaccuracy of the approximation of the drift as linear.

## AutoHedge Pairs

Each AH pair corresponds to a particular DEX trading pair, and, in the initial version of AH, each pair has to have a stablecoin as 1 of the tokens in the pair (though the team is already developing an upgrade to AH that removes this requirement, coming soon ™). Also, technically the native asset of the chain cannot be used (for simplifying the code to only tokens and reducing attack surface etc), but it also isn't used under the hood in DEXes like UniswapV2 either anyway, so it doesn't affect the user (and the user only needs to provide stablecoins anyway). For example there could be USDC-WETH and DAI-UNI pairs, but not UNI-WETH or USDC-DAI. The future upgrade to AH will enable pairs like UNI-WETH. Stablecoin-stablecoin pairs are obviously already delta-neutral, so we're not concerned with them.

Pairs can be created in a similar way to UniswapV2 pairs - there is a factory contract which has a `createPair` function. Technically an AH pair is specific to a certain DEX, trading pair on that DEX, and lending platform. In practice at launch, AH will be hardcoded to use the most liquid DEX on each blockchain and hardcoded to a specific lending platform, too.

The lending platform that AH uses is called [Fuse](https://app.rari.capital/fuse), made by Rari Capital, now part of Tribe DAO. It's essentially a platform that allows you to create your own Compound fork and add arbitrary assets to the fork. We chose Fuse since we needed to be able to add many DEX LP tokens, 1 for each pair that we enable AH to use (anyone can create a pair, but the lending platform that's hardcoded needs to support each LP token otherwise it's impossible to create the corresponding AH pair). It also allows us to add each AHLP token to be lent out and used as collateral, which is necessary in order to leverage LP (more on that below). Fuse is only on Ethereum and a couple of L2s, but a friendly-fork project, [Midas Capital](https://www.midascapital.xyz/), is launching on BSC and many other EVM chains.



Note: there are many places in which the mechanisms for all parts of the system could be far more efficient. We wanted to get the simplest design out into market as fast as possible, and are actively working on improvements, which is why we made everything upgradable.
