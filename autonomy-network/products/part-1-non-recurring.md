# Part 1: non-recurring

In general, conditions are enforced on-chain by \`require\` statements, so a condition can be as simple as a function with a single line of code inside, which is the require statement. Some conditions can just read some state of the blockchain and can therefore be \`view\` functions. Others may need to store data or change some storage - either is fine.

### 1.

We need to create a contract - this is going to be a simple contract that doesn't import any other code, so you can use any code editer, such as Remix. We can call the contract \`TimeConditions\`, because we're imaginative like that, and add some comments.

<figure><img src="../../.gitbook/assets/Screenshot from 2022-10-06 17-05-12.png" alt=""><figcaption></figcaption></figure>

### 2.

We need to add a function that can be called by `FundsRouter`. This function needs to:

* have logic for triggering after a specific time (`afterTime`). Because there will always be some delay and uncertainty in when an execution will actually happen because we don't know when the execution tx will be mined, it's best to include a maximum time that the tx will fail after (`beforeTIme`) - this effectively gives a window where the trigger will be valid
* force the above condition by making the whole tx revert if it's not true

We can do this with 2 simple `require` statements

<figure><img src="../../.gitbook/assets/Screenshot from 2022-10-06 17-07-04.png" alt=""><figcaption></figcaption></figure>

That's literally it - super simple! Any tx that calls `betweenTimes` outside of the specified times will now revert the whole tx, enforcing the condition.

Go to part 2 for how to create the recurring condition.
