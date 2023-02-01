# Return/Profit Analysis

## Net APY

The overall APY of the position is the net sum of all revenue and cost APYs in the system.

The revenues are:

* trading fees from LPing on the DEX (on the whole position) (A)
* lending interest from lending out the DEX LP token (on the whole position) (B)

The ongoing costs are:

* The interest rate paid on borrowing the volatile asset (on HALF the whole position) (C)
* The gas cost of rebalancing (on the whole position) (D)

The net APY is therefore roughly:

`Net APY = A + B - C/2 - D`

However, A, B, & C all grow proportionally as the TVL of AutoHedge grows, whereas D is independent as it's the same no matter what the TVL is. Therefore with large TVL, D becomes insignificant to the point where we can ignore it, especially on chains where the gas cost is cheap anyway. When TVL is small, D is significant, and Autonomy will seed new pairs with native chain tokens to subsidize this cost. The owned/debt threshold for rebalancing can be changed from the default 1% to reduce or increase D as needed depending on the situation - this will have to be optimized for each chain AH is deployed onto.

(A) and (B) are always a positive number by definition, so profitability of the system is just a question of whether:

`A + B > C/2 + D`

What if the above isn't true? It'd be great if, under that condition, you could automatically withdraw your position into stables. Fortunately, Autonomy's automation can be used for exactly that! We'll be adding that as a new feature soon.

## Other Costs

There are some one-time costs to using AutoHedge when depositing and withdrawing. For example when depositing and withdrawing 2k DAI as described in [How It Works (detailed)](how-it-works-detailed/) you pay 0.100475% of the deposited amount.

## Drifting

Since the 'short' (the volatile asset debt) amount syncs with the 'long' (the volatile asset held in the DEX LP) amount only after a difference of 1% (which is changeable), the hedge isn't perfect, and so the value of the overall position can 'drift' away from the initial position value over time. However, it's to such a small degree that in practice it isn't an issue.

Simulations of a 20% change in price of ETH in a DAI-ETH pair resulted in a 0.07% change in position value in the direction of the price movement - so when the ETH price increased by 20%, the overall position value went up by 0.07% and vice versa. This makes the hedge 99.65% effective, i.e. for every \~1% change in price of the volatile asset, the value of the position changes by 0.0035%.

Since the net APY of the overall position is typically so much larger (in the range of 10%-50%), any loss from drift is completely outweighed by the return and can be ignored in practice.
