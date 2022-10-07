# Tutorial: Create A Condition And Upload It

Typically every action requires a condition under which you want the action to execute, such as rebalancing a portfolio under the condition of time (e.g. every day), or making a trade under the condition of price (such as a limit order or stop loss). Some conditions are very common, such as time-based conditions, and the beauty of AutoStation is that, because you can mix and match any number of conditions with any number of actions, the **same** time conditions contract can be used with any other action or condition in a completely modular way!

This tutorial shows how to create a contract that is usable in AutoStation to add time conditions to any other action/condition. Since it's already created and uploaded to AutoStation, it's a bit pointless for anyone else to do it, but it shows every step required in the process so you can easily come up with your own condition and upload it to AutoStation. We're going to create a time-based condition contract that allows you to use 2 kinds of triggers/conditions:

1. At a specific time, e.g. "trigger on June 1st at 10am" (more suitable for non-recurring automations) - \`betweenTimes\`
2. Recurring with a constant time period "trigger starting on June 1st at 10am and every day after that" (obviously more suitable for recurring automations) - `everyTimePeriod`

Part 1 covers the specific time condition, and part 2 covers the recurring condition. The whole contract code can be found [here](https://github.com/Autonomy-Network/autostation-contracts/blob/develop/contracts/conditions/TimeConditions.sol).
