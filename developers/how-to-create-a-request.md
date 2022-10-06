# How to create a Request

How you create a `Request` usually depends on whether there is some kind of authentication involved (such as moving user funds), or whether you need to add a condition to an external function call.

To recap, the interface for `newReq` is:

```
    /**
     * @notice  Creates a new request, logs the request info in an event, then saves
     *          a hash of it on-chain in `_hashedReqs`. Uses the default for whether
     *          to pay in ETH or AUTO
     * @param target    The contract address that needs to be called
     * @param referer       The referer to get rewarded for referring the sender
     *                      to using Autonomy. Usally the address of a dapp owner
     * @param callData  The calldata of the call that the request is to make, i.e.
     *                  the fcn identifier + inputs, encoded
     * @param ethForCall    The ETH to send with the call
     * @param verifyUser  Whether the 1st input of the calldata equals the sender.
     *                      Needed for dapps to know who the sender is whilst
     *                      ensuring that the sender intended
     *                      that fcn and contract to be called - dapps will
     *                      require that msg.sender is the Verified Forwarder,
     *                      and only requests that have `verifyUser` = true will
     *                      be forwarded via the Verified Forwarder, so any calls
     *                      coming from it are guaranteed to have the 1st argument
     *                      be the sender
     * @param insertFeeAmount     Whether the gas estimate of the executor should be inserted
     *                      into the callData
     * @param isAlive       Whether or not the request should be deleted after it's executed
     *                      for the first time. If `true`, the request will exist permanently
     *                      (tho it can be cancelled any time), therefore executing the same
     *                      request repeatedly aslong as the request is executable,
     *                      and can be used to create fully autonomous contracts - the
     *                      first single-celled cyber life. We are the gods now
     * @return id   The id of the request, equal to the index in `_hashedReqs`
     */
    function newReq(
        address target,
        address payable referer,
        bytes calldata callData,
        uint112 ethForCall,
        bool verifyUser,
        bool insertFeeAmount,
        bool isAlive
    ) external payable returns (uint id);
```

It's also worth noting that there is also `newReqPaySpecific` which has an addition parameter, `payWithAUTO`. This parameter is left out in `newReq` as there's a default selection for that. Currently, the default is `false`, since the `AUTO` token isn't live yet, but will be turned to `true` at some point after the token goes live. If you need to use a specific token to pay, best to use `newReqPaySpecific`.

#### Simplest - no condition or authentication

Say you wanted to automate 'poking' a contract, where the function being called already has some conditions built in. For example, calling `me` in the following contract:

```
contract Touch {
    function me() external {
        require(block.timestamp >= 1626356451, "Too soon");
        // Do something
    }
}
```

The `callData` for `me` can be obtained in `web3.py` by (docs):

```
callData = touch.encodeABI(fn_name="me", args=[])
```

The `callData` for `me` can be obtained in `web3.js` by ([docs](https://docs.ethers.io/v5/api/utils/abi/interface/#Interface--encoding)):

```
callData = touch.encodeABI("me", [])
```

Let's go through how `newReq` would be called:

* `target` - the address of the contract which has the function we want to call, so the deployed `Touch` contract address
* `referer` - to keep it simple, it can be set to `0x00...00`
* `callData` - the calldata for calling `me` , as generated above
* `ethForCall` - we don't want to send any ETH to `me` , and even if we did, it would revert because the function is not `payable`, so this should be `0`
* `verifyUser` - since this requires that the first argument to `me` is the user's address for authentication, which isn't required here (and would also revert anyway since `me` doesn't accept any inputs), this needs to be `false`
* `value` - this obviously isn't a direct input the the `newReq` function, but it's the amount of ETH to be sent with the `newReq` transaction that is actually sent by the user immediately when registering their `Request`. Here, since we don't need to send any ETH to `touch.me` directly, and because we want to pay for the execution in ETH, we only need to send enough ETH for the execution. Doing this properly requires estimating how much gas the execution will cost, then using the worst-acceptable-gasPrice, but here we can just use 0.01 ETH for the sake of simplicity, which should be more than enough on any testnet. Any excess ETH will get sent back to the user. For example if the actual execution costs 0.002 ETH, then since the fee is `0.002 * 0.3 =` 0.0006 ETH, the total that the user will receive back is `0.01 - 0.002 - 0.0006 =` 0.0074 ETH, with the rest going to the executor
