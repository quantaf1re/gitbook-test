# Overview

## What is Autonomy Network? Tl;dr

Autonomy Network is a decentralized automation protocol - it allows you to have transactions automatically execute in the future under arbitrary conditions.

It's an infrastructure service, and a tool, that allows anyone to automate any on-chain action with any on-chain condition. For example, limit orders/stop losses for traders of decentralized exchanges like Uniswap, and exit liquidity positions once impermanent loss becomes too great. A blog post giving a simple overview of Autonomy and each component can be found [here](https://autonomynetwork.medium.com/how-autonomy-works-simple-6c0059a2ee89). Autonomy is full composable, so it can even be used by on-chain entities like DAOs to automate things such as recurring salary payments. Because it's infrastructure that's meant to be used by other projects directly to add features, similar to ChainLink, users don't even need to know it exists - they just use a dapp that automates something, such as a limit order, and the experience is the same as it would be with a centralized exchange. Users don't need to learn anything or deploy a contract, it 'just works'!

Autonomy is ultimately a B2B tool. However, the team has been creating [user-facing dapps](https://app.gitbook.com/o/-MeA5RrsH6Q0j4yLUM7C/s/-MeA4rd8vgaMQcZpHyck/\~/changes/UhpBbn5ORQOyQ2bqWvF8/autonomy-network/products) to prove the feasibility and demand for new use cases, using them as a stepping stone for the automation features being integrated into other projects natively or having others fork our dapps and improve them while still using Autonomy under the hood.

## AUTO Token

Autonomy Network will have a token, AUTO, which can be used to:

1. Stake as an executor - in order to be able to execute `Request` s and earn the small fee from doing so
2. Pay a discounted fee as a user - users need to pay executors for the gas they spend executing their `Request`s, plus a fee as incentive for doing so

Currently the token is not live (coming soon ™), so anyone can execute `Request`s, and execution fees are paid in ETH.

## Autonomy as a protocol

The flow of events is:

1. A user makes a single `Request` transaction (usually via a dapp that implements Autonomy), which essentially says "Here are all the details for the thing I want to be executed, and conditions under which it should be executed"
2. The condition becomes true at some point in time
3. The executor at the time the condition becomes true executes the `Request` and receives a small fee (the execution would revert if the condition is false)

Autonomy itself is essentially an on-chain registry that:

1. Allows people to register `Requests` for things they want to happen in the future ([more](autonomy-network/system-components/the-registry.md))
2. Is monitored by a decentralized network of bots waiting to execute `Requests` when their corresponding conditions become true ([more](autonomy-network/system-components/executors.md))
3. Organizes the network of bots with a Proof of Stake algorithm that determines which bot is able to execute `Requests` at a given time ([more](autonomy-network/system-components/staking.md))

Bots locally simulate executing every `Request` after every block. Since the condition for execution is codified in a require statement in the target system's smart contract, such as a limit order:

`require(price >= limitPrice);`

Each `Request` will therefore always revert if the condition is not met, and won't revert if the condition is true. This way, the bots don't need to know about prices/tokens or use oracles in order to know whether a limit order is executable - they are fully generalized to automate anything.

There are 3 main components to Autonomy:

1. The `Registry` contract - this holds the details for all outstanding `Requests` ([more](autonomy-network/system-components/the-registry.md))
2. Executors - these are people (running bots almost certainly) that execute people's `Request`s in exchange for a small fee ([more](autonomy-network/system-components/executors.md))
3. The `StakeManager` contract - this decides which executor has the exclusive right to execute `Request`s for a 100 block period ([more](autonomy-network/system-components/staking.md))



