# How It Works

## Deposit

All of the following happens with a single `deposit` tx using AutoHedge. Assuming you want to participate in an DAI-ETH liquidity pool on Uniswap with $8k in DAI and the price of ETH is $2k:

1. AutoHedge takes the 8k DAI and 4k DAI is swapped to ETH at a price of $2k for a total of 2 ETH and 4,000 DAI.
2. AutoHedge takes those 4k DAI and 2 ETH and deposits them into Uniswap’s liquidity pool.
3. All of the LP tokens it receives back, worth $8k, would be used as collateral (which are being lent out, so you receive interest on them) to borrow an amount of ETH equal to the ETH that’s deposited in the liquidity pool (2 ETH).
4. That amount of 2 ETH is sold for 4k DAI, essentially shorting ETH an equal amount that you're long. This means you're now fully hedged against ETH’s price volatility and can even get an additional revenue stream from lending out that DAI again. However, the issue at this point is that the amount of ETH that you can claim back by redeeming the LP token changes as the price of ETH changes and people add or remove ETH from the pool by trading in it.\*
5. This is where AutoHedge’s automation capabilities come in: we then need to regularly update the debt position to reflect the amount of ETH we have represented by the DEX’s LP token. Thanks to Autonomy Network, AutoHedge is able to automatically keep the debt in sync with the amount of ETH in the pool. It borrows or pays back some ETH depending on how the balance of the Uniswap pool is changing. It does this every time the debt and the ETH amounts have a difference of 1%, though this parameter can be set.
6. AutoHedge then mints a 2nd LP token, known as AH LP (AutoHedge LP), that wraps the whole position (the Uniswap position, the Uniswap LP tokens being lent out, the ETH debt, and the DAI being lent out). This token represents a delta-neutral position that also receives LP yields.

A diagram showing the logic flow of a single `deposit` transaction:

![](https://lh6.googleusercontent.com/UwzF1pc\_m2Oq9973sT5B3bd3oHJKlXJdUcM9JajwRyormr3OXNegAlNcUdZ0CoCKReXvVRtXMG1bcHmYSZBomc5xPmMSgg3aFqWKcor86avVTUb5tKbLRhSRadF8VOlf\_fQ1rQnhwPNfmtuEMw)

\*A caveat with 4. above is that we need to lend out a _proportional_ amount of DAI such that the % we increase the pool's lending position by equals the % we increased the pool's DEX position by. This is because, as described below, when withdrawing, you're withdrawing and repaying an equal % of all positions and debt.

For example if the pool already had 100 DEX LP tokens and was already lending out 200 DAI, then if someone deposited such that their deposit returned 10 DEX LP tokens, they would've increased the pool's DEX position by 10% - this means that we want to increase the lending position by 10%, i.e. 20 DAI.

If using those 10 DEX LP tokens as collateral to borrow some volatile tokens, then selling those volatile tokens for DAI, ended up as 30 DAI, we'd have too much, and so 20 DAI would be added to the lending position and 10 DAI would be sent back to the user.

If, however, using those 10 DEX LP tokens as collateral to borrow some volatile tokens, then selling those volatile tokens for DAI, ended up as 10 DAI, we'd have too little, and so 10 extra DAI would be transferred from the user to make up the deficit, and the total 20 DAI would be added to the lending position.

## Withdraw

`withdraw` essentially reverses every step in `deposit`. You burn some AH LP tokens and receive a proportional amount of every position and repay the debt proportionally. For example if the price of ETH is $2k and the 3 positions on a DAI-WETH AH pair for the whole pool are:

* 100 DEX LP tokens lent (redeemable for 2 WETH and 4k DAI on the DEX)
* 2 WETH borrowed
* 4.2k DAI lent out

Then if there are 50 AH LP tokens in existence, and the user wants to burn 5 to withdraw, then they'll get 10% of everything:

* receive 10 DEX LP tokens which are redeemed for 0.2 WETH and 400 DAI
* repay 0.2 WETH
* receive 420 DAI from the lending position

What actually happens during the 'unwinding' of the position is that the 400 DAI from the 420 DAI from the lending position are converted to 0.2 WETH to repay that debt, leaving 20 DAI (A) extra profit. The 0.2 WETH from the DEX position is also converted back into 400 DAI (B), and the 400 original DAI (C) from the DEX position are all sent to the user.

So the user ends up with A + B + C = 820 DAI at the end.

If, however, the pool's lending position in the above example was only 3.5k DAI for whatever reason, such that the user's portion is 350 DAI, then that wouldn't be enough to cover the 0.2 WETH debt, so funds from the DEX position would be used. This would mean that the 50 DAI deficit would come from assets in the user's portion of the DEX position, which means they'd end up with 750 DAI.

## Rebalance

If you initially LP with 1 WETH, then you'll borrow 1 WETH to hedge when depositing as described above. However, as the price of ETH changes and traders trade in/out of the DEX pair, the amount of WETH tokens that you own in the DEX will go up or down depending on if the price of ETH goes down or up respectively. If you have, say, 1.1 WETH, but still only have a debt of 1 WETH, that means you're exposed to the price risk of the difference (0.1 WETH).

Therefore in order to stay as close to fully hedged as possible, each pair needs to keep the dept of the volatile token in sync with the amount of that token owned in the DEX position - this rebalancing is impossible to do automatically in a decentralized way without Autonomy. By default the rebalancing occurs when the amounts go out of sync by 1% in either direction. For example if there was 100 WETH in the underlying DEX pair, and 100 WETH borrowed, and 200k DAI lent out, and the price of ETH is $2k:

* If the price of ETH increased such that the amount of WETH in the DEX pair dropped to 99 WETH, that would trigger a rebalance, and 1 WETH would need to be repaid. (slightly more than) 2k DAI would be withdrawn from the lending position and swapped for 1 WETH and used to repay the debt.
* If the price of ETH fell such that the amount of WETH in the DEX pair rose to 101 WETH, that would trigger a rebalance, and the debt would need to be increased by 1 WETH. Since the value of the collateral (the DEX LP token) roughly stayed the same value, but the value of our debt (100 WETH) fell, we can borrow 1 WETH extra without any collateral issue (and it's always collateralised by atleast 2x anyway from the DEX LP token alone). That 1 WETH would then be sold for (slightly less than) 2k DAI and that DAI would be added to the lending position.

## AutoHedge Pairs

Each AH pair corresponds to a particular DEX trading pair, and, in the initial version of AH, each pair has to have a stablecoin as 1 of the tokens in the pair (though the team is already developing an upgrade to AH that removes this requirement, coming soon ™). Also, technically the native asset of the chain cannot be used (for simplifying the code to only tokens and reducing attack surface etc), but it also isn't used under the hood in DEXes like UniswapV2 either anyway, so it doesn't affect the user (and the user only needs to provide stablecoins anyway). For example there could be USDC-WETH and DAI-UNI pairs, but not UNI-WETH or USDC-DAI. The future upgrade to AH will enable pairs like UNI-WETH. Stablecoin-stablecoin pairs are obviously already delta-neutral, so we're not concerned with them.

Pairs can be created in a similar way to UniswapV2 pairs - there is a factory contract which has a `createPair` function. Technically an AH pair is specific to a certain DEX, trading pair on that DEX, and lending platform. In practice at launch, AH will be hardcoded to use the most liquid DEX on each blockchain and hardcoded to a specific lending platform, too.

The lending platform that AH uses is called [Fuse](https://app.rari.capital/fuse), made by Rari Capital, now part of Tribe DAO. It's essentially a platform that allows you to create your own Compound fork and add arbitrary assets to the fork. We chose Fuse since we needed to be able to add many DEX LP tokens, 1 for each pair that we enable AH to use (anyone can create a pair, but the lending platform that's hardcoded needs to support each LP token otherwise it's impossible to create the corresponding AH pair). It also allows us to add each AH LP token to be lent out and used as collateral, which is necessary in order to leverage LP (more on that below). Fuse is only on Ethereum and a couple of L2s, but a friendly-fork project, [Midas Capital](https://www.midascapital.xyz/), is launching on BSC and many other EVM chains.



Note: there are many places in which the mechanisms for all parts of the system could be far more efficient. We wanted to get the simplest design out into market as fast as possible, and are actively working on improvements, which is why we made everything upgradable.
