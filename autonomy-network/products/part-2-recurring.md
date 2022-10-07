# Part 2: recurring

We're now going to add another condition, this time one that executes more than once - specifically on a start time and every X seconds after. Another condition means another function!

### 1.

First we create `everyTimePeriod` with inputs for the start time and period length in seconds. Using a \`require\` statement makes it impossible to execute before `startTIme`.

<figure><img src="../../.gitbook/assets/Screenshot from 2022-10-06 17-19-58.png" alt=""><figcaption></figcaption></figure>

But in order to be able to use `periodLength`, we have a problem. We need to know which execution cycle we're on - this requires saving state to the contract for the last time it executed.

### 2.

We can add `lastExecTime` as a state variable and use it in a `require` statement, so that the request can't be executed before `lastExecTime + periodLength` and then update `lastExecTime` every time the request is executed.

<figure><img src="../../.gitbook/assets/Screenshot from 2022-10-06 17-24-38.png" alt=""><figcaption></figcaption></figure>

This now ensures that the request executed from the start time and every period length after! Now the issue is that there's only a single `lastExecTime` variable in the contract, so if multiple requests tried to use it, they would mess up their times for eachother.

### 3.

To solve this, we need to be able to have a separate `lastExecTime` for each request. Autonomy is able to pass information about the request to the receiving function if certain flags are used when making the request. Ideally it'd pass the request id, which it's not able to do currently (that's coming up in a new release soon), but the next best thing is to get the original user who made the request, which can be done by adding an address as the first input (\`user\`) and using the `verifyUser` flag when calling `newReq` to make the request (this is done automatically by AutoStation when using the UI). This feature of Autonomy guarantees that the first argument in the function is the correct user who made the request - this is done by sending requests that have the `verifyUser` flag through a special forwarder, so we can trust that any call coming from this particular address has the first argument as the original user. We therefore need to enforce this in the contract by requiring that all calls coming to `everyTimePeriod` are coming from this particular forwarder (`routerUserVeriForwarder`) so that we can trust it every time. Since `routerUserVeriForwarder` is permanent and needs to be known all the time, we can add a constructor and store it on initialization.

Then, we can add `callId` which is user-defined. This is so that the contract can distinguish between different requests made by the same user. A user would use a different number for this every time they make a new recurring request - if they used the same one for multiple requests then those requests would clash, but rationally no user would do that and it's impossible to be malicious to other users because we have a different mapping of `callIds` for every user.

<figure><img src="../../.gitbook/assets/Screenshot from 2022-10-06 17-37-38.png" alt=""><figcaption></figcaption></figure>

That's it! Now we're done with the whole contract (the whole contract code can be found [here](https://github.com/Autonomy-Network/autostation-contracts/blob/develop/contracts/conditions/TimeConditions.sol). We can upload it to [AutoStation](https://autostation.io/) in the next part.
