# Part 3: uploading to AutoStation

We can upload the contract to AutoStation so that we and everyone else can mix and match with everyone else's actions/conditions contracts. Since we've actually made 2 conditions in `TimeConditions` (each function is a separate condition), we'll have to upload each one separately. Let's do `betweenTimes` first.

### 1.

Deploy the contract using Remix or any framework (our repo uses Brownie and has a deploy script here) and verify the code on Polygonscan (we'll use Polygon for this tutorial). Verifying the code is needed for the AutoStation UI to know the ABI so that it can display the inputs to users in the UI when someone uses this condition. Our contract is [0x171ee22328a936d7e9D67Af0978fE973403F09c7](https://polygonscan.com/address/0x171ee22328a936d7e9d67af0978fe973403f09c7#code). For the constructor we need the `routerUserVeriForwarder` variable. This can be found in [Deployed Contracts](../../developers/deployed-contracts.md).

<figure><img src="../../.gitbook/assets/Screenshot from 2022-10-07 16-28-34.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/Screenshot from 2022-10-07 16-28-47.png" alt=""><figcaption></figcaption></figure>

### 2.

Go to [https://autostation.io/app/upload](https://autostation.io/app/upload) and fill out the inputs. The name should describe what it does at a glance. The function from the contract should be `betweenTimes`.

<figure><img src="../../.gitbook/assets/Screenshot from 2022-10-07 20-07-25.png" alt=""><figcaption></figcaption></figure>

### 3.

Hit "Upload Condition" and make the transaction to add it to the UI. This transaction essentially calls an AutoStation contract that just emits an event specifying all the information about the condition so that the UI can watch it and add it to the UI. The transaction cost compared to using a centralized database acts as a spam preventor.

### 4.

Repeat steps 1-3 for `everyTimePeriod`, except selecting `everyTimePeriod` as the function this time.

Now we're done! You can find your condition in the UI, marked as unverified. Submit a request in the discord for someone from the review team to review the condition and add it to the verified list. This doesn't guarantee that the condition works or even that it's not malicious, as that can never be guaranteed, but you can be reasonably sure that they work as intended. The next part describes how to create a request on Autonomy that uses this condition - this isn't necessary to know since the AutoStation UI does this for you, but it's useful for understanding how it all works.
