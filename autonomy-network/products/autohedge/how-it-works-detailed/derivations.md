# Derivations

## Deposit

### Key Formulas

`flFeeVol = flAmountVol * flRate`

`flFeeStable = getAmountsIn(flFeeVol)`

`flAmountVol = (depositAmountStable - flFeeStable) / volPrice`

`protocolFeeStable = (depositAmountStable - flFeeStable) * protocolFeeRate`

`flAmountVol = (w - (Maths.sqrt((w - (4 * t) * (z / w))) * Maths.sqrt(w))) / (2 * t)` (definition of w, t, & z below)

We can figure out exactly how much of a given deposit amount will be used for fees and how much left over will be used to actually LP with. Or how much we need to deposit to result in a specific LP amount. Using linear equations, these define the system (fl is short for flashloan):

Whatever amount of volatile token we borrow in the flashloan, the fee has to be paid in the same token on top\
(a) `flFeeVol = flAmountVol * flRate`

The fee needs to be taken from what the user has (stables) and converted into volatile tokens, which incurs a swapping fee and slippage. We need to use the DEX's swapping logic specifically to account for slippage, so for UniswapV2 we use `getAmountIn()` (we can use `getAmountIn` instead of `getAmountsIn` since AutoHedge only uses the pair it's LPing to swap, and therefore only uses a single hop):\
(b) `flFeeStable = getAmountsIn(flFeeVol)`

The amount of vol tokens that's flashloaned is the same value of the stables that are left over\
(c) `flAmountVol = (depositAmountStable - flFeeStable) / volPrice`

The protocol fee is taken as AHLP tokens when they're minted, which are ultimately valued in terms of stables\
(d) `protocolFeeStable = (depositAmountStable - flFeeStable) * protocolFeeRate`



Sub (b) into (c):

(e) `flAmountVol = (depositAmountStable - getAmountsIn(flFeeVol)) / volPrice`

Sub (a) into (e):

(f) `flAmountVol = (depositAmountStable - getAmountsIn(flAmountVol * flRate)) / volPrice`&#x20;

We need to isolate `flAmountVol`, but it's impossible in its current form, we need to expand `getAmountIn`. In UniswapV2:

`amountIn = getAmountIn(amountOut) = ((reserveIn * amountOut * 1000) / ((reserveOut - amountOut) * 997)) + 1`

In our context:

`amountIn = flFeeStable`&#x20;

`amountOut = flFeeVol`

Therefore we can transform (f):

(g) `flFeeStable = getAmountIn(flFeeVol) = ((reserveIn * flFeeVol * 1000) / ((reserveOut - flFeeVol) * 997)) + 1`

Rearranging this gets pretty gnarly - for brevity, you can check it with Wolfram etc, the result is:

(h) `flAmountVol = (w - (Maths.sqrt((w - (4 * t) * (z / w))) * Maths.sqrt(w))) / (2 * t)`

Where (it's split up like this because as you can see, some terms are repeated, and it would be very unreadable written out in full):

`t = ((reserveStable * FLASH_LOAN_FEE * 997)) / FLASH_LOAN_FEE_PRECISION`

`w = (amountStableInit * reserveVol * FLASH_LOAN_FEE * 997) / FLASH_LOAN_FEE_PRECISION + reserveStable * reserveVol * 997 + (reserveVol * reserveStable * 1000 * FLASH_LOAN_FEE) / FLASH_LOAN_FEE_PRECISION`

`z = amountStableInit * reserveVol * reserveVol * 997`

I know it's not nice and neat, but hey, that's how linear equations are in real life, and it works without any rounding errors ;)

## Withdraw



## Rebalance
