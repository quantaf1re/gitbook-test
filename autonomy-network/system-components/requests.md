# Requests

Everything that's automated in Autonomy is in the form of a `Request` - users register `Request`s in the `Registry`, and executors monitor the `Registry` for `Request`s that can be executed, then execute them when the conditions for execution are true. Having a single `Registry` that is **fully generalised** to be able to automate **any** on-chain action makes Autonomy the only automation solution that can be used off-the-shelf and doesn't need manual support to be added for novel use cases.

&#x20;A `Request` details everything needed for some action to be taken on behalf of the user. It's made up of the following components:

* `user` - the user (i.e. `msg.sender` ) who made the `Request`&#x20;
* `target` - the contract address that needs to be called when executed
* `referer` - the dapp that integrated Autonomy (to be rewarded in tokens for every successfully executed `Request` )
* `callData` - the details of the function that the user wants to be called, along with the function inputs
* `initEthSent` - the amount of ETH sent when the `Request` was made (for both paying for the eventual execution of the `Request` , and also for any ETH that needs to be sent along with the function call that is being requested, such as an eth-to-token Uniswap trade)
* `ethForCall` - the amount of ETH that should be sent in the requested function call
* `verifyUser` - whether or not the 1st input to the function call being requested is the address of the user registering the current `Request` (if true, the call will be routed through a separate `Forwarder` contract with a different address, meaning that if a contract receives a call with `msg.sender` that is that `Forwarder`,  it's guaranteed that the 1st input to the function is the original `user` who made the `Request`. This enables authentication, allowing them to safely call `transferFrom` etc to move user funds)
* `insertFeeAmount` - whether the fee being charged for execution of the `Request` should be forwarded along with the call, so that the receiving contract can pay it, for example in the case of a limit order, the fee can be taken out of the output token. If `verifyUser` is `false`, then the fee will be inserted into the 2nd input parameter of the call. If `verifyUser` is `false`, then the fee will be inserted into the 1st input parameter of the call
* `payWithAUTO` - whether the gas cost for execution + fee should be paid in AUTO (if `true`, the fee is only an extra 10% of the gas cost of execution, whereas if `false`, the fee in ETH is an extra 30% of the gas cost of execution). Paying with AUTO also allows you to keep funds in your wallet and only pay when the execution takes place (because of `transferFrom`), rather than paying upfront
* `isAlive` - whether the `Request` should be deleted after executing, i.e. whether it's recurring or not

A `Request` is executed by an executor who calls `executeHashedReq`. This essentially:

1. Calls the desired function with `target.call{value: ethForCall}(callData)`&#x20;
2. Measures the gas spent doing so
3. Charges the user for that gas + fee and sends it to the executor

Since Autonomy is a staked system for executing, any execution attempt also checks that the executor is the current executor for that epoch. If the executor of this epoch hasn't been 'set' yet, then before any execution begins, the `StakeManager` will calculate which executor can execute for this epoch, set that for the whole epoch, then check whether the current executing executor is the selected executor for this epoch.



