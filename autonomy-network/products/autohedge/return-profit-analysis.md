# Return/Profit Analysis

## Net APY

The overall APY of the position is the net sum of all revenue and cost APYs in the system.

The revenues are:

* trading fees from LPing on the DEX (A)
* lending interest from lending out the DEX LP token (B)
* lending interest from lending out the stablecoins that were traded into after borrowing the volatile token to short (in point 4. in the text list above) (C)

The ongoing costs are:

* The interest rate paid on borrowing the volatile asset (in point 3. in the text list above) (D)
* The gas cost of rebalancing (E)

The net APY is therefore:

`Net APY = A + B + C - D - E`

However, A, B, C, & D all grow proportionally as the TVL of AutoHedge grows, whereas E is independent as it's the same no matter what the TVL is. Therefore with large TVL, E becomes insignificant to the point where we can ignore it, especially on chains where the gas cost is cheap anyway. When TVL is small, E is significant, and Autonomy will seed new pairs with native chain tokens to subsidize this cost. The owned/debt threshold for rebalancing can be changed from the default 1% to reduce or increase E as needed depending on the situation.

(A), (B), and (C) are always a positive number by definition, so profitability of the system is just a question of whether:

(A) + (B) + (C) > (D) + (E)

In practice in the crypto markets, rates on stablecoins are typically much higher (by a few % on average) than rates for volatile tokens - this means (C) > (D) and so that eliminates the main cost, and really the only cost as the system scales. This rate difference is because there's a much higher supply of volatiles owned and being lent out compared to stables (thank fuck for the degens).

## Other Costs

There are some one-time costs to using AutoHedge when depositing and withdrawing, each of which occur in a single tx.

For example when depositing 8k DAI, 4k of that is swapped to a volatile token like WETH, which has a 0.3% cost ($12, leaving $3988 of WETH). After LPIng with those assets and lending out the DEX LP token as collateral and borrowing $3988 of WETH, that's then swapped into DAI with another 0.3% cost ($11.964, leaving $3976.036 of DAI). At this point the total cost of the deposit is therefore $23.964, or 0.29955%, and the total position value is $7976.036.

At the end of the deposit, 0.3% of all the AH LP tokens that are minted are sent to the Autonomy team, or to the referrer if the depositer used a referral link. This effectively costs the user $23.928108, leaving the overall position worth $7952.107892. The total cost is therefore 0.5986%, plus the gas cost of the tx.

Withdrawing is the same process in reverse, just in the reverse order, and without the 0.3% protocol/referrer fee. Withdrawing therefore costs \~0.29955%.

## Drifting

Since the 'short' (the volatile asset debt) amount syncs with the 'long' (the volatile asset held in the DEX LP) amount only after a difference of 1% (which is changeable), the hedge isn't perfect, and so the value of the overall position can 'drift' away from the initial position value over time. However, it's to such a small degree that in practice it isn't an issue.

Simulations of a 20% change in price of ETH in a DAI-ETH pair resulted in a 0.07% change in position value in the direction of the price movement - so when the ETH price increased by 20%, the overall position value went up by 0.07% and vice versa. This makes the hedge 99.65% effective, i.e. for every \~1% change in price of the volatile asset, the value of the position changes by 0.0035%.

Since the net APY of the overall position is typically so much larger (in the range of 10%-50%), any loss from drift is completely outweighed by the return and can be ignored in practice.
