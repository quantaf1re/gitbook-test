# Part 4: bonus - how to call manually

So if you didn't want to use the AutoStation UI, and instead wanted to use AutoStation manually for whatever reason, you'd need to make a request on Autonomy that asks it to call the `FundsRouter`, and in turn ask that to call whatever condition/action contracts you want to use. In this example, we'll increment a variable, `x`, in a contract called `MockTarget` (a contract that's just for the purposes of demoing/testing AutoStation, where the relevant function here is `incrementX`) every 5 minutes. We'll use Brownie for this, partly because I used it for testing these contracts in the first place and I procrastinated making this tutorial until the day before the ETHBogota hackathon, and partly because it's way better than Truffle (Python gang let's goo!) and all the cool kids use it now, trust me.

![](../../.gitbook/assets/image.png)

Since we need to ask the Registry to ask the Router to call some functions in `TimeConditions` and `MockTarget`, it means we need to work backwards from that flow - we need to get the calldata for the calls to `everyTimePeriod` and `incrementX`, then get the calldata for `forwardCalls` in the `FundsRouter` which includes the calldata for `everyTimePeriod` and `incrementX`, then use that as an input to `newReq` in the \`Registry\` which actually makes the request - kind of like layers of an onion wrapping eachother.

### 1.

First, for some context, this is the interface for `forwardCalls` in `FundsRouter`.

<figure><img src="../../.gitbook/assets/Screenshot from 2022-10-06 18-20-29.png" alt=""><figcaption></figcaption></figure>

Notice that `fcnData` is in the format of a `FcnData` struct (cause we're great at naming things). This struct is:

<figure><img src="../../.gitbook/assets/Screenshot from 2022-10-06 18-22-40.png" alt=""><figcaption></figcaption></figure>

So we need to use a tuple/array of this info when gathering the info about each call and putting it into the `fcnData` array. We can put either the condition or the action first in `fcnData` and it'll operate in the exact same way regardless of the ordering here, let's do the condition first:

<figure><img src="../../.gitbook/assets/Screenshot from 2022-10-06 18-36-16.png" alt=""><figcaption></figcaption></figure>

Here, `tc` is a contract object in Brownie/Web3.py that can be called like `tc.fcn` to call a function called `fcn`, or `tc.address` to get the address of the specific contract. I omitted some variable instantiation/setup for brevity since it's unrelated to the point of this tutorial - you're welcome to read the tests for the full picture here. The 1st element in the tuple is the contract address being called (`tc.address`). The 2nd element is the calldata for the `everyTimePeriod` function - we need to put the first argument as the user's address (here, `user_a` is the address making the request), and we'll use a `callId` of 1. We'll use a start time of right now, and a period of 5 minutes. So this request should be executed immediately by Autonomy and then every 5 minutes after. Since we don't want to send ETH along with the call, we put the 3rd element in the tuple as 0. The 4th element is `true` because we want to first argument of the calldata to be verified by `FundsRouter` so that `everyTimePeriod` knows it can trust it and use that info to distinguish between different requests.

### 2.

Now we need to do the same for `incrementX` in `MockTarget`.

<figure><img src="../../.gitbook/assets/Screenshot from 2022-10-06 18-38-35.png" alt=""><figcaption></figcaption></figure>

Similarly to `tc`, `mock_target` is a contract object for the `MockTarget` contract. The 1st element is the address of `MockTarget`. The 2nd element is the calldata for `incrementX`, which doesn't have any arguments. The 3rd element is 0 again because we don't want to send any ETH when calling `incrementX`. The 4th element is now `False` because we don't want the first argument of `incrementX` to be forced to be the requester's address (and it wouldn't make sense to want to here anyway since there aren't any arguments so it would be an invalid request).

### 3.

Now we have everything we need to call `forwardCalls` in `FundsRouter` - we need to get the calldata for calling `forwardCalls` which includes `tc_fcn_data` and `mock_target_fcn_data` we generated earlier:

<figure><img src="../../.gitbook/assets/Screenshot from 2022-10-06 18-43-49.png" alt=""><figcaption></figcaption></figure>

Again the 1st argument of `forwardCalls` is the user's address, since `forwardCalls` needs to know it so it knows which user's deposit to deduct the execution fee from. The 2nd argument is the amount of execution fees (in ETH) the user needs to pay to the Autonomy Registry. Since this input is replaced in the Registry before making the call to `forwardCalls`, it doesn't matter what we put here, so we can just put `0`. The 3rd argument is ofcourse all the info about what contracts/functions needs to be called, and is the data we generated earlier put into an array.

### 4.

Finally, we can make the request to Autonomy!

<figure><img src="../../.gitbook/assets/Screenshot from 2022-10-06 18-58-20.png" alt=""><figcaption></figcaption></figure>

`newReq` is the main function in the Autonomy `Registry` for creating a new request. In the 1st argument, `fr` is the `FundsRouter` contract object and so this is the address of it, since we're asking the Registry to call it with the fcn calldata we specify in the 3rd argument. In the 2nd argument, this is the `referrer` - it's unrelated and can just be left as `0x00...00`. The 3rd argument is the calldata for the `forwardCalls` function we generated earlier. The 4rd argument is 0 since we don't want to send any ETH along with the request. The 5th argument, `verifyUser` is `True` since we want to enforce that the 1st argument of the function we're automating (`forwardCalls`) is the user's address. The 6th argument, `insertFeeAmount`, is `True` since we want the Registry to replace the 2nd argument to `forwardCalls` with the amount of fees that it (and therefore the user using `FundsRouter`) needs to pay for the execution each time. The 7th argument, `isAlive`, is `True` because we want this automation to be recurring, not just execute once.

And that's it! Clear as mud. Now you can manually automate anything your little heart dreams - not sure why you'd want to do it without the UI, but hey, you do you, ser!
