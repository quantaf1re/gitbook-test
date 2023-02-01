# How It Works

This describes the general steps involved in depositing, withdrawing, and rebalancing, while omitting some things like fees for simplicity. The version that includes every little detail is [How It Works (detailed)](https://app.gitbook.com/o/-MeA5RrsH6Q0j4yLUM7C/s/-MeA4rd8vgaMQcZpHyck/\~/changes/qfyK1EUZ2nvHVqQLLQdb/autonomy-network/products/autohedgev2/how-it-works-detailed).

## Deposit

All of the following happens with a single `deposit` tx using AutoHedge. Assuming you want to participate in an DAI-ETH liquidity pool on Uniswap with 2k DAI and the price of ETH is $2k:

1. 2k DAI is taken from the user and transferred to the AutoHedge DAI-ETH pool.
2. We need to take out a flashloan of WETH for an equal value of the DAI we're LPing with, therefore we flashloan 1 WETH which is also worth $2k.
3. AutoHedge takes those 2k DAI and 1 WETH and deposits them into Uniswap’s liquidity pool.
4. All of the DEX's LP tokens that are received back, worth $4k, are used as collateral on a lending platform (which are being lent out, so you receive interest on them too) to borrow an amount of WETH equal to the WETH that’s deposited in the liquidity pool (1 WETH). This means the pool has a collateral ratio of 200%, and stays there due to rebalancing (described below).
5. Repay the flashloan of 1 WETH with the 1 WETH borrowed from the lending platform.
6. AutoHedge then mints a 2nd LP token, known as AHLP (AutoHedge LP) that's specific to the DEX pair - in this case AHLP-DAI-ETH. This token wraps the whole position (the Uniswap position, the Uniswap LP tokens being lent out, and the WETH debt). This token represents a delta-neutral position that also receives LP yields, and therefore essentially acts like a yield-bearing stablecoin.

That's it! Now you're fully hedged. If the price of ETH suddenly halves, then the value of the DEX position (2k DAI and 1 WETH) is worth $3k, but the debt (1 WETH) that you have to repay when withdrawing is only worth $1k, so the net value of the position is still $2k that you started with. However, in reality the price of WETH changing would mean that people trade in the DEX pair which subsequently changes the ratio of assets you have. If the price of ETH went up then you'd end up with more DAI and less WETH in the DEX position than just after you deposited. Since the amount of WETH in the LP changes (to be < 1 WETH) but the debt doesn't change (still 1 WETH), that means the hedge is no longer exact - that's where Autonomy and rebalancing comes in, described below.

A diagram showing the logic flow of a single `deposit` transaction:

<figure><img src="../../../.gitbook/assets/Screenshot from 2022-08-28 18-41-09.png" alt=""><figcaption></figcaption></figure>

## Withdraw

`withdraw` essentially reverses every step in `deposit`, but has to take account of any APY or fees. You burn some AHLP tokens and receive a proportional amount the DEX position and repay the debt proportionally. Assuming you wanted to unwind the whole position of 2k DAI and 1 ETH created in `deposit` above:

1. Take out a flashloan of 1 ETH.
2. Repay the 1 ETH debt and withdraw the DEX LP tokens that were used as collateral for the loan.
3. Redeem the DEX LP tokens for 2k DAI and 1 ETH.
4. Repay the flashloan of 1 ETH.
5. Send the remaining 2k DAI to the user.
6. Burn the user's AHLP tokens.

However in reality both the amount of assets in the DEX LP and the debt change with time due to APY/interest. For example if the price of ETH is $2k and the 2 positions on a DAI-ETH AH pair for the whole pool started out as:

* 100 DEX LP tokens lent (redeemable for 1 WETH and 2k DAI on the DEX)
* 1 WETH borrowed

But the APY of the DEX is 10%, the borrowing cost of WETH is 2%, and the interest rate on the DEX LP tokens lent out is 1%, then after 1 year, assuming the price of ETH is still $2k, the whole pool will look like:

* 101 DEX LP tokens lent (redeemable for 1.111 WETH and 2,222 DAI on the DEX, since both the value of the LP tokens and the amount of LP tokens we have went up)
* 1.02 WETH borrowed

Then if there are 50 AHLP tokens in existence, and the user wants to burn 5 to withdraw, then they'll get 10% of everything:

* receive 10.1 DEX LP tokens which are redeemed for 0.1111 WETH and 222.2 DAI
* repay 0.102 WETH

So the total they're left with is 222.2 DAI and 0.0091 WETH. After converting the WETH to DAI, the total the user receives is 240.4 DAI, up from initially being 200 DAI. However, this figure isn't exact - see [How It Works (detailed)](https://app.gitbook.com/o/-MeA5RrsH6Q0j4yLUM7C/s/-MeA4rd8vgaMQcZpHyck/\~/changes/qfyK1EUZ2nvHVqQLLQdb/autonomy-network/products/autohedgev2/how-it-works-detailed).

## Rebalance

As described above, the amount of the volatile token in the DEX LP, such as WETH, can go 'out of sync' from the amount of debt due to price changes of the asset or from accumulating trading fees. If you have, say, 1.1 WETH in the LP, but still only have a debt of 1 WETH, that means you're exposed to the price risk of the difference (0.1 WETH) and are therefore no longer delta-neutral.

Therefore in order to stay as close to fully hedged as possible, each pair needs to keep the debt of the volatile token in sync with the amount of that token owned in the DEX position - this rebalancing is impossible to do automatically in a decentralized way without Autonomy. By default the rebalancing occurs when the amounts go out of sync by 1% in either direction. For example if there was 2k DAI and 1 WETH in the underlying DEX pair, and 1 WETH borrowed, and the price of ETH is $2k:

*   If the price of ETH increased such that the amount of WETH in the DEX pair dropped to 0.99 WETH, that would trigger a rebalance, and 0.01 WETH would need to be repaid. Due to the `x*y=k` curve used by AMMs, the amount of DAI in the pair would be 2,020, meaning the price of ETH is `2,020 / 0.99 =` $2,040. This would happen with the following steps:

    1. DEX LP tokens are withdrawn from the lending platform that are redeemable for 20.202 DAI and 0.01 WETH. This is safe to do because the collateral ratio of the debt was originally 200% and this only removes 1% of the collateral, meaning it's still far from liquidation.
    2. The 20 DAI is swapped for 0.01 WETH. The AutoHedge pool now has 0.02 WETH 'liquid'.
    3. The 0.02 WETH is used to repay the loan, reducing it to 0.98 WETH. There is now 2,000 DAI and 0.98 WETH in the DEX.

    With the new price of ETH, the total value is still $4k.
*   If the price of ETH decreased such that the amount of WETH in the DEX pair rose to 1.01 WETH, that would trigger a rebalance, and 0.01 WETH would need to be repaid. Due to the `x*y=k` curve used by AMMs, the amount of DAI in the pair would be 1,980, meaning the price of ETH is `1,980 / 1.01 =` $1,960. This would happen with the following steps:

    1. An additional 0.02 WETH would be borrowed from the lending platform. This is safe to do because the collateral ratio of the debt was originally 200% and this only adds 1% to the debt, meaning it's still far from liquidation.
    2. Half of the 0.02 WETH would be swapped to DAI. The pool now has 20 DAI and 0.01 WETH 'liquid'.
    3. The 20 DAI and 0.01 WETH are used to LP in the DEX. There is now 2,000 DAI and 1.02 WETH in the DEX.

    With the new price of ETH, the total value is still $4k.

## AutoHedge Pairs

Each AH pair corresponds to a particular DEX trading pair, and, in the initial version of AH, each pair has to have a stablecoin as 1 of the tokens in the pair (though the team is already developing an upgrade to AH that removes this requirement, coming soon ™). Also, technically the native asset of the chain cannot be used (for simplifying the code to only tokens and reducing attack surface etc), but it also isn't used under the hood in DEXes like UniswapV2 either anyway, so it doesn't affect the user (and the user only needs to provide stablecoins anyway). For example there could be USDC-WETH and DAI-UNI pairs, but not UNI-WETH or USDC-DAI. The future upgrade to AH will enable pairs like UNI-WETH. Stablecoin-stablecoin pairs are obviously already delta-neutral, so we're not concerned with them.

Pairs can be created in a similar way to UniswapV2 pairs - there is a factory contract which has a `createPair` function. Technically an AH pair is specific to a certain DEX, trading pair on that DEX, and lending platform. In practice at launch, AH will be hardcoded to use the most liquid DEX on each blockchain and hardcoded to a specific lending platform, too.

The lending platform that AH uses is called [Midas Capital](https://www.midascapital.xyz/), a friendly fork of Fuse by Rari Capital. It's essentially a platform that allows you to create your own Compound fork and add arbitrary assets to the fork. We chose Fuse since we needed to be able to add many DEX LP tokens, 1 for each pair that we enable AH to use (anyone can create a pair, but the lending platform that's hardcoded needs to support each LP token otherwise it's impossible to create the corresponding AH pair). It also allows us to add each AHLP token to be lent out and used as collateral, which is necessary in order to leverage LP (more on that below).



Note: there are many places in which the mechanisms for all parts of the system could be far more efficient. We wanted to get the simplest design out into market as fast as possible, and are actively working on improvements, which is why we made everything upgradable.
