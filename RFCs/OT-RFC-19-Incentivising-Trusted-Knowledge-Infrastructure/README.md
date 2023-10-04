# OT-RFC-19 - Incentivising Trusted Knowledge Infrastructure: OriginTrail Parachain Collators Framework

**Authors:** OriginTrail Core Developers

**Date:** October 3rd 2023


## Introduction

The OriginTrail Trusted Knowledge Foundation is based on a two layer decentralized system - the multi-chain consensus layer (or the blockchain layer), and the Decentralized Knowledge Graph (DKG - the knowledge layer). Each of the layers plays a key role in the ecosystem, designed with “separation of concerns” in mind, yet enabling a multitude of different decentralized systems to synergize within the architecture, such as blockchains like Ethereum, Polkadot, Gnosis, Polygon and others.

OriginTrail Parachain is one of the blockchains in the multichain consensus layer, launched successfully in 2022 as an innovation hub for OriginTrail expansion, specifically designed to support the growth of OriginTrail DKG and implement incentive strategies for the OriginTrail ecosystem (some of which have been introduced in OT-RFC-18). The topic of this OT-RFC-19 is to specifically focus on incentivising OriginTrail Parachain as a trusted knowledge infrastructure component through incentivising the parachain nodes, also called collators. More information on OriginTrail and OriginTrail Parachain as its innovation hub can be found in the [Whitepaper 2.0](https://origintrail.io/ecosystem/whitepaper).


## OT Parachain Consensus Overview

Polkadot’s architecture has designated a particular function that needs to be performed by any active parachain as it creates its blocks - collating transactions. Collators maintain parachains by collecting parachain transactions from users and producing state transition proofs for Relay Chain validators (which provide security). Collating is performed by collator nodes of the OriginTrail Parachain network that need to adhere to certain technical requirements (e.g. running full nodes) and stake requirements (e.g. bonding a particular amount of OTP), all of which are covered by this document.


### OriginTrail Parachain Collators

The framework proposal introduces a decentralized way for selecting those collator candidates that are selected in the active set of collators eligible to produce state transition proof for relay chain validators. All collators that will not be in the active set still remain in the candidate pool, eligible to be added to the active set in the next round.``

OriginTrail Parachain would use a Delegated Proof of Stake (DPoS) system that determines which collators are eligible to produce blocks based on their total stake in the network. The DPoS system is powered by the parachain staking pallet, allowing token holders (delegators) to express exactly which collator candidates they would like to support and with what quantity of tokens. The design of the parachain staking pallet is such that it enforces shared risk/reward on chain between delegators and candidates. The parachain staking pallet is developed by the Moonbeam team as part of a grant received from Web3 foundation and is widely used across the Polkadot ecosystem.


## Key Framework Parameters

The framework guiding the collator system for OriginTrail Parachain consists of several key parameters that determine how nodes can get included in the pool of collator candidates, how the active set of collators is selected out of the collator candidates pool and how collator rewards get distributed among the active set participants. The one parameter spanning through all of the above is the duration of the round, which is set at 1800 blocks (or 6 hours) for OriginTrail Parachain.


### OriginTrail Parachain Collator Rounds

The timing of a round is an important measurement as it defines the period in which a selected active set of collators is producing blocks and collecting rewards. After the round of 1800 blocks ends, a new active set of collators is selected for block production, based on parameters defined below.  


### Collator Rewards

Rewards for collators and their delegators are calculated at the start of every round for their work prior to the reward payout delay. As the reward payout delay is 2 rounds (12 hours),  this means that the compensation reflects the contributions made by collators and their delegators during the two rounds preceding the reward distribution.The calculated rewards are then paid out on a block-by-block basis starting at the second block of the round. For every block, one collator will be chosen to receive their entire reward payout from the prior round, along with their delegators, until all of the rewards have been paid for that round. For example, if there are 64 collators who produced blocks in the prior round, all of the collators and their delegators would be paid by block 64 of the new round.

The total amount of OTP that gets distributed equally throughout all 1800 blocks in round consists of: 



* Inflation amount - 30% of the annual OTP inflation rate gets distributed to collators (read more about inflation rate [here](https://origintrail.io/documents/OriginTrail_Ecosystem_White_Paper_2.0.pdf)). In the first year the total amount will be 7,500,000 OTP (roughly 1041 OTP per round or 0.58 OTP per block).
* Kickstart pool amount - within the genesis allocation, 45% of OTP gets allocated to the kickstart pool. Out of that amount, 14% is intended to be used for collators rewards (70,000,000 OTP in total). In the first year it is expected that 28,500,000 OTP out of the total amount will be distributed (roughly 3958 OTP per round). Kickstart pool amount will decrease every year until fully depleted after 10 years. 
* Accumulated fees - 30% of all collected transaction fees are intended to be used for collators awards. They get sent to the collator pool at the finish of every round and are allocated for distribution in the following round (the amount per round and per block depends on the collected fees).

All three pools combined is what the active set of collators in a particular round will be able to claim in each block they are selected to produce by the randomisation mechanism.


### Collator candidates

In order to be eligible for the above rewards, a parachain node needs to first become a candidate collator. It achieves that status by performing OTP bonding (staking). The collator bond is crucial to determine the pecking order in the candidate collator pool out of which the active set is determined. As mentioned above, the active set of collators for a particular round is then responsible for producing blocks in that round and receives associated block rewards intended for performing this function.

By providing both bonds (above the thresholds), a node is eligible to become a member of the collator candidates pool.

The initial key parameters pertaining to a candidate collator are as follows:



* Minimum collator bond (self-bond) - 50.000 OTP
* Eligible collator entering candidates pool - Immediate (eligible for next round)
* Collator leaving candidates pool - 28 rounds (7 days) wait time 
* Increase self-bond amount -: Immediate
* Reduce self-bond - 28 rounds (7 days)

A delay in certain activities (leaving the candidates pool and reducing self-bond) is required to ensure stability of the network and lower the spamming potential in the network. The specific parameters may be modified through OriginTrail Parachain Governance proposals in the future.


### Active set of collators

A group of collator candidates selected to produce blocks in a particular round are called an active set. For that round, an active set of collators is also eligible to receive their block rewards and distribute them among the delegators and themselves. 

In order for a candidate collator to be eligible to get included in the active set, their collator bond (sum of self-bond and delegated bond) needs to be in the high enough range. Each round, all collator candidates are ranked based on the collator bond amount from highest to lowest and the group with highest bond amounts are then selected in the active set for the next round. The exact number of selected collators depends on the current setting of the active set size parameter.

Key parameters pertaining to an active collator set are as follows:



* Active set size - 12 collators
* Rewards payouts (after current round) - 2 rounds (12 hours)

The adjustment of the active set size will be governed by the OriginTrail Parachain governance process. The active set size parameter is designed to incrementally expand, with the goal of accommodating up to 64 collators. These increments will be made gradually, typically ranging from 4 to 8 collators at a time.


### Delegating OTP to a collator

In order to increase a chance for a collator to move from candidate pool to an active set, the collator needs not to solely rely on a self-bond amount of OTP. In addition to the initial amount, collators can also accept OTP from the delegators.

An OTP delegator will receive a portion of the rewards based on their relative stake in the total bond amount. As an example, if a delegator contributes 1000 OTP in a collators total bond amount of 50.000 OTP, it will give this delegator a claim to 2% of the earned rewards in that particular round.  

The possibility to delegate OTP is limited only by the minimum delegated bond amount currently set at 100 OTP and by the maximum number of delegators allowed to delegate for each collator, which is set at 300. If more than 300 delegators bond their OTP with a collator, only the top 300 will be distributed the rewards.

Other key parameters pertaining to delegators are:
* Revoke delegation - 28 rounds (7 days)
* Rewards payouts (after current round) - 2 rounds (12 hours)

You can choose to auto-compound your delegation rewards so you no longer have to manually delegate rewards. If you choose to set up auto-compounding, you can specify the percentage of your rewards to be auto-compounded and they'll automatically be added to the delegation in which you received the rewards from.

The specific parameters mentioned above can be adjusted through OriginTrail Parachain Governance proposals in the future. 


## Other Collator Requirements


### Hardware requirements

The recommended hardware requirements for running a collator are expected to be above those running a regular parachain node in order to be able to handle higher throughputs required by OriginTrail Parachain. Initial tests have shown that following set-up has been performing well (reserving the use of the server solely for purposes of the OriginTrail Parachain): 



* Recommended CPUs - Intel Xeon E-2386/2388 or Ryzen 9 5950x/5900x
* Recommended NVMe - 1 TB NVMe
* Recommended RAM - 32 GB RAM

Detailed hardware recommendations, such as CPU specifics and tuning strategies will be provided in the official documentation.


### Account Requirements

As with (almost) any activity in the Polkadot ecosystem, running a collator equally requires an account. It should be a H256 account where the private keys are in your custody. That being said, it is worth repeating that your keys are your responsibility and mishandling them might lead to damages and your funds being lost. All industry best practices should be followed. 


### Community Guidelines

Lastly, running an OriginTrail Parachain collator is an activity with and for the OriginTrail community. It is therefore highly recommended that collators are active in the community (Discord) and contribute in as many ways as they best see fit (education, open-source contributions, voting on proposals…).


## Conclusion

The OriginTrail Parachain Collators Framework, as outlined in this RFC, represents an important step towards strengthening trusted knowledge infrastructure within the OriginTrail ecosystem. It lays the foundation for a decentralized, inclusive, and robust collator network for the OriginTrail Parachain, ensuring that all participants, from collators to delegators, share in both the risks and rewards.

As we move forward, we invite the OriginTrail community and all interested parties to provide valuable feedback and insights. Your perspectives are instrumental in refining this framework, making it even more effective in incentivizing and supporting OriginTrail Parachain's growth as a trusted knowledge infrastructure component.
