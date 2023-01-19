# Integrating Limits/Stops Into Your DEX

Especially after the collapse of FTX, centralized exchange (CEX) users are moving to DEXes, compounding the trend of DEX volume eating up CEX volume. Currently DEXes typically only provide market orders, and these new users coming from CEXes are demanding the features they're used to using - namely, limit orders and stop losses. Fortunately it's easy to add these onto an existing DEX without any modification to the underlying DEX! [AutoSwap](https://autoswap.trade/) is an example of this.

Generally, as with anything on Autonomy, the condition and action of a `Request` needs to be defined in a smart contract so that the condition can be guaranteed - this is the smart contract that is the `target` of a `Request` . The same condition/action contract can be used for many DEXes that have the same interface, such as UniswapV2 forks. DEXes that are unique require a modified condition/action contract that uses the interface of the unique DEX.

## UniswapV2 Forks

The condition/action contract for UniswapV2 forks [already exists](https://github.com/Autonomy-Network/uniV2-limits-stops/blob/master/contracts/UniV2LimitsStops.sol) and is deployed on many EVM chains! Integrating limits/stops into your DEX therefore requires no new contracts to be deployed, it only requires modification to your UI.

Fortunately this modification is made even easier with our frontend SDKs:\
[https://www.npmjs.com/package/@autonomylabs/limit-stop-orders](https://www.npmjs.com/package/@autonomylabs/limit-stop-orders)\
[https://www.npmjs.com/package/@autonomylabs/limit-orders-react](https://www.npmjs.com/package/@autonomylabs/limit-orders-react)

## Other DEXes

If your DEX is not a UniswapV2 fork, then the [UniswapV2 condition/action contract](https://github.com/Autonomy-Network/uniV2-limits-stops/blob/master/contracts/UniV2LimitsStops.sol) needs to be modified slightly to use the interface of your DEX. This can be done by your team or Autonomy depending on the specific situation - hop into our Discord and create a ticket to chat with us about your situation! Either way it should require fairly minor changes and take only 1-2 weeks.
