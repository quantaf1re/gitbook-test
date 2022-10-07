# Tutorial 2: Example automation using AutoStation

In this tutorial we will use AutoStation to change a NFT image based on an arbitrary condition, essentially allowing NFT's to evolve or shape-shift automatically. The [**Mystique**](https://github.com/Autonomy-Network/autostation-tutorial-usecases/blob/remove-URImodifier/ShapeShifter/contracts/Mystique.sol) contract is a simple **ERC721** contract that has the ability to change **URI** when a certain condition is met, thereby ‘**shapeshifting**’. You can change the **baseURI** by calling the **setNewBaseURI** function. \
\
To automatically change the appearance of the NFT's from the **Mystique** contract follow the steps below:\
\
**1)** Enter [AutoStation](https://autostation.io/) connect your wallet and click the Manage tab. Deposit some native currency to fund your automations. You can withdraw at any time. \


<figure><img src="../../../.gitbook/assets/Screen Shot 2022-10-07 at 11.17.00 AM.png" alt=""><figcaption></figcaption></figure>

**2)** Head back to the Automate tab, and click on the 'Create Automation' button.\


<figure><img src="https://lh4.googleusercontent.com/nAk-uwNh_wPGTVcBBpnZqLkQi4N-nao4P1whWy6NhzA9u_Qku4J_pnaT-Lhb83eDTPwS44ktMqEBtsKe1JcYOUUTNK4Z84vB52nd5fuLWbFwHvpI2xTbXk7LyF1C96pG4KzME7rwQFsNBkX9w1ewqYr-KaBPGIicyFLFzLZPg8-dOr575a02NHR04A" alt=""><figcaption></figcaption></figure>

**3)** Input automation name, and pick an action card ( you can easily switch between action and condition by clicking the arrows). This view also shows many different cards, essentially there are three sets of cards, there is custom cards ( cards that let you specify any contract verified or non-verified ), autonomy preset cards (cards built by the Autonomy Team) and community cards (cards that are uploaded by anyone in the community, to view them unselect curated). For this example we will select the custom card, if your contract is not verified then there will be a field to manually input that.

<figure><img src="https://lh3.googleusercontent.com/QAwA7t0BfjHLLYVQOYWgxYic5ZLO8MDcEDU6M2h5H4VzTUTb1F3j51w64DsFQ7Hn540PA7HEWgJPQsJfLKrbcCTcGLYtJ6tPHXRJyulh6Q59D_f8wWcCBmHMSiCS36ENKgbPsumCRI_PwQBRty0h0IrgwtZxsnEdZZEJMjc7YTgwb0yEoe0C-LiFZw" alt=""><figcaption></figcaption></figure>

**4)** Once you have clicked on custom and added the address input you should be able to see a function selector. For this example you should select the **setNewBaseURI** function**.**

<figure><img src="https://lh5.googleusercontent.com/cgFNIyrldO70Cef9F3YA5aTJpeBenweJqrWPHgOt7kF6EcpXT-CEDvl4NddIKv_PA6-J2wQwnd-9p0WSDDV1fazXwbmasPnS7yLdHv7neYhQgwCqQoE2_o6-Ng803dVZcyny_DEpq-T0c7lwtR6-JC_Fc3TrNZJ2enBHhf67EYm7By4otO4PV6Waww" alt=""><figcaption></figcaption></figure>

**5)** Fill in the **\_newURI** parameter and click on the + button at the bottom of the page. (You can also collapse the card by clicking on the - button)&#x20;

<figure><img src="https://lh6.googleusercontent.com/qiscqpQm-ZGvaNvNv3kgzX5wwPaG3_cK0UqrHwwIxFTjw9pwx5JvZiIsC_tgqHFxhoC8s9_ep1ZiVtarRcUjnHF4n4QEaO-gkhlLb04XfUdhrCwGcpcSU_5FSYoQ-jL1o4BT7hW6dLhlwSw65HYQrZ-yFw_w8Op4QpLMVdoAU2VtXDFt7_JgzZVMDw" alt=""><figcaption></figcaption></figure>

**6)** Now we are going to set a specific time for when we want the URI to change, so we are going to create a condition card. This time we are going to pick an Autonomy Preset card ( they are different from custom because the Autonomy team designed a more friendly interface to interact with certain contracts). We will pick the Time Condition Card.

<figure><img src="../../../.gitbook/assets/Screen Shot 2022-10-07 at 11.56.16 AM (1).png" alt=""><figcaption></figcaption></figure>

**7)** We will select the one-time automation and fill in the inputs, to when you want your NFT to evolve or shape-shift. Then click on the **Automate** button.

<figure><img src="../../../.gitbook/assets/Screen Shot 2022-10-07 at 11.59.53 AM.png" alt=""><figcaption></figcaption></figure>

**8)** Once your transaction has been mined we are **DONE!** you can see your **newRequest** on the Autonomy Registry by looking up the address [`here`](https://autonomy-network.gitbook.io/autonomy-docs/developers/deployed-contracts) , you can now sit back and relax. Once the time condition is met, the Autonomy Bots will execute your request and the new URI will be set for the **Mystique** contract. You can setup more complex conditions, for example you could make it such that an NFT evolves based on the price of ETH or based on your USDC balance etc.\
\
\* **IMPORTANT**: Depending on what contracts you use or how you set up your contracts, you might need to define if you need to verify the user or send some native currency with the request, for example using a recurring automation typically requires  user verification. For more information please refer to the last 29 minutes of this [video](https://youtu.be/RRYhvAzyHDc) or read the [docs](https://autonomy-network.gitbook.io/autonomy-docs/autonomy-network/products/autostation) carefully. When setting up a custom automation you are able to set this variables by clicking on the **cog** button.&#x20;

<figure><img src="../../../.gitbook/assets/Screen Shot 2022-10-07 at 12.15.11 PM.png" alt=""><figcaption></figcaption></figure>
