# What is it?

AutoStation is a no-code interface for automating anything on-chain! It allows you to create an automation that chains together an arbitrary number of actions and conditions. In addition to being an easy way to automate things, it's also a platform that allows people to upload actions and conditions for other people to use - like an app store for automation. AutoStation being a platform means that users who don't want to code get access to a huge library of community-built conditions and actions that would be impossible for a single team to build on their own.

# How It Works

AutoStation is made up of the \`FundsRouter\` contract and any condition\/action contracts you want to connect to. The \`FundsRouter\` lets you do 3 things:

 - deposit ETH to pay for recurring automations

 - withdraw ETH

 - make calls to all the contracts\/functions you specify

For example if you want to increment a variable \`x\` every day, then after you've deposited some ETH to the \`FundsRouter\`, you would make a request on Autonomy that asks it to call the \`forwardCalls\` function in the \`FundsRouter\`. In the function inputs to \`forwardCalls\`, you would specify that you want to then call \`everyTimePeriod\` in the \`TimeConditions\` contract and also the \`incrementX\` function in the \`MockTarget\` contract. Since the \`everyTimePeriod\` function will revert the whole tx if too little time has passed since the last call, then \`incrementX\` will only ever be called when the time condition is satisfied. The flow of logic for the tx that executes the request successfully is:

Registry -&gt; FundsRouter -&gt; TimeConditions

 -&gt; MockTarget

This architecture allows you to mix and match any amount of conditions with any amount of actions in a completely modular way. As a developer you can create condition and action contracts and upload them to AutoStation so that anyone can use them with the UI without having to code, kind of like an app in an app store!

# Fees

Executions in Autonomy have a fee, which is the gas cost of the execution + 30%, so if it costs the bot $0.10 to execute the request, the user pays $0.13 in ETH out of the funds deposited into \`FundsRouter\`. Autonomy itself is very flexible in the way the fees can be charged \(pre-paying when making the request or paying during execution of a request\), but AutoStation requires the requester to deposit ETH \(or whatever the base token of the blockchain is\) to the \`FundsRouter\` contract, where every time an execution is done, part of the deposit is used to pay for the execution.

