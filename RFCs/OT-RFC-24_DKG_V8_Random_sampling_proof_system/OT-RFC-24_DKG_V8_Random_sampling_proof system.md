**OT-RFC-24 DKG V8 Random sampling proof system** 

## Authors: OriginTrail Core Developers

**Date:** Apr 2, 2025

Table of contents

[Introduction](#introduction)

[Problem definition](#problem-definition)

[Functional Requirements (FRs)](#functional-requirements-\(frs\))

[Non-Functional Requirements (NFRs)](#non-functional-requirements-\(nfrs\))

[Solution: Random sampling, a scalable proof-of-knowledge-based system](#solution:-random-sampling,-a-scalable-proof-of-knowledge-based-system)

[Proof system sequence](#proof-system-sequence)

[System performance](#system-performance)

[Pre-V8.1  random sampling reward distribution](#pre-v8.1-random-sampling-reward-distribution)

[Rollout plan](#rollout-plan)

[Conclusion](#conclusion)

# Introduction {#introduction}

This RFC aims to introduce a scalable knowledge availability proof system in the OriginTrail Decentralized Knowledge Graph (DKG) V8. As knowledge publishers populate and grow the DKG by minting Knowledge Assets on the public DKG network, the **DKG Core Nodes are responsible** **for the continuous availability of the public DKG content, metadata, and provenance proofs of the DKG.** For providing a high-quality service, DKG Core Nodes compete based on multiple factors—high uptime, economic stake, publishing factor, and service ask requirements—and are incentivised through TRAC token network fees, funded by the knowledge publishers at publishing time.

As a decentralized system, the DKG network incentive system is completely handled by the DKG-smart-contract-based proof system, which requires periodic proof submissions from DKG Core Nodes to ensure the service availability and quality.

This RFC will detail the most advanced DKG proof system yet, called **random sampling**, which aims to ensure scalability and low cost for the entire system while maintaining strong availability guarantees.

# Problem definition {#problem-definition}

With the previous implementation of DKG V6, the network encountered practical scalability limitations due to its intensive block space utilization and high demands placed on DKG node resources. These constraints represented a bottleneck to achieving Internet-scale adoption, limiting the network's growth and the widespread usability.

DKG V8 aims to address these scalability challenges. DKG V8.0, released in December 2024, introduced the first scalability feature known as **batch minting**, which significantly increased the scalability by aggregating multiple minting operations into a single blockchain transaction, dramatically reducing the consumption of block space for knowledge publishing. As a result, batch minting immediately unlocked greater scalability and operational efficiency (at the time of writing this RFC, the DKG state has crossed a total of 530M knowledge assets hosted, jumping from \~10M in only 3 months),  enabling broader adoption without compromising decentralization or security.

Building on these advancements, the upcoming DKG V8.1 releases aim to further enhance the scalability and sustainability of the proof system. The core ambition of DKG V8.1 is to substantially **lower both block-space requirements** and **resource demands** **placed upon DKG Core Nodes**. By optimizing the underlying proof mechanisms through a random-sampling-proof-challenge scheme, V8.1 will maintain strong security guarantees while significantly increasing scalability. This new scheme, detailed further in this document, will facilitate higher throughput and broader node participation, ensuring that the OriginTrail DKG remains robust, accessible, and prepared to meet the evolving demands of global-scale decentralized knowledge sharing.

To properly address the above ambition for lower block space consumption and scalable node resource allocation, the following requirements have been identified:

## **Functional Requirements (FRs)** {#functional-requirements-(frs)}

* **FR1 Core Node scalability requirement:** DKG Core Nodes do not need to host the entire DKG in the long run, but rather store sectors of the DKG. A DKG Core Node may store 1 or more sectors.  
* **FR2 Trusted proof validation:** Enough data needed for challenge generation and proof verification should be available on chain, so that smart contracts can independently verify them based on DKG Core Node proof submissions.  
* **FR3 Safety requirement:** The DKG proof system should be sufficiently random, so that DKG Core Nodes cannot practically predict future challenges.  
* **FR4 TRAC Incentives favor positive network activity:** High uptime, high economic stake, high publishing ratio, and lower service ask are incentivized.  
* **FR5 Proof system agnostic to network topology:** The system is not dependent on Core Node hash ring position (as pre-V8).  
* **FR6 Real-time proof system:** The system should operate on the basis of the current DKG node state—current uptime, active stakes, publishing factors, service ask values (e.g., Core Nodes should not be able to “claim uptime” in the past).

## **Non-Functional Requirements (NFRs)** {#non-functional-requirements-(nfrs)}

1. **NFR1 The DKG proof system scalability requirement:** The proof system should scale up to at least 100B knowledge assets.  
2. **NFR2 Blockchain gas cost:** Total gas consumption per node, per day for all random sampling transactions should be under 100M gas .   
3. **NFR3 Core Node usability:** The random sampling system implementation should operate without necessary involvement from the node operator (the node operators are incentivized to focus on uptime, stake accrual, publishing, and managing node service ask, for the benefit of the entire network).  
4. **NFR4 Reward collection before V8.1:** Nodes should be able to collect accumulated rewards before the V8.1 release.

# Solution: Random sampling, a scalable proof-of-knowledge-based system {#solution:-random-sampling,-a-scalable-proof-of-knowledge-based-system}

Random sampling is one implementation of a “Proof of Knowledge” (PoK), used as a key feature of the DKG to implement incentives for nodes to store Knowledge Assets and have high uptime. 

To understand the proof system concept in more detail, we introduce a few important terms:

* **Knowledge sector:** In DKG V8, knowledge collections are organized into “sectors”—large “virtual repositories” of knowledge maintained by a set of DKG nodes and registered on 1 blockchain. Sectors were introduced to enable easier horizontal scaling and will be beneficial once DKG reaches several billions of Knowledge Assets in size (at the current time, there is only 1 active DKG sector per blockchain, so a total of 3 sectors). Each sector is hosted by a group of DKG Core Nodes, which constantly submit knowledge availability proofs to DKG smart contracts—1 proof per sector in a proof period. Once more sectors are introduced, it will be possible for DKG Core Nodes to host multiple sectors at the same time (ability to vertically scale one node, dependent, of course, on hardware resource availability).   
* **Knowledge collection:** A set of Knowledge Assets, materialized as an NFT collection on the blockchain, and a set of triples and named graph in the DKG node graph databases. All Knowledge Assets in one collection have the same life-time, measured in epochs.  
* **Knowledge chunk:** Each Knowledge Collection (KC) is split into 32-byte-sized “chunks”, from which a Merkle root hash is computed on publishing. DKG Core Nodes verify this hash every time they receive newly published data by recomputing it and confirming a match with the Knowledge Collection on-chain Merkle root.   
* **Epoch:** Used to measure Knowledge Collection lifetime. For example, 1 knowledge collection can live for 12 epochs, roughly equivalent to 1 year. One epoch is set at 30 days.  
* **Proof period**: A short period of time (measured in minutes) in which a node is expected to submit a proof of knowledge via the random sampling system.

## **Proof system sequence** {#proof-system-sequence}

1. A DKG Core Node periodically interacts with the proof system. Every ***proofPeriod***, a challenge is generated by the DKG smart contract, which randomly selects a KC chunk for the node to prove from 1 sector. Challenges are unique for each node. The challenge is presented as two valid random numbers—***KCID*** (a knowledge collection ID) and ***CN*** (chunk number). A valid random number is considered a number in the possible range (existing and non-expired KC, valid chunk number).

challenge(nodeId, t) \= ( random(KnowledgeCollectionIDactive), random(chunkNumberKCID) 

2. Once the challenge has been generated on chain, DKG Core Nodes compute a Proof of Knowledge (the correct Merkle proof for the challenged **KCID** and **CN**)  and submit it to the proof system smart contracts. The proof consists of the requested chunk data and corresponding Merkle proof for that Knowledge Collection.  
3. If the sent proof of knowledge can be used to reproduce the correct Merkle root in the smart contracts (submitted and validated on publishing), the proof is considered valid and a **proofScore** is computed as a function of total node stake, ask, and publishing factor at time of proof submission (as in the tokenomics formula). The indicative formula is  
   *score= f(uptime, nodeStake, nodePublishingFactor,nodeAsk)*

   If, for whatever reason, a node “misses” to submit a valid proof during the *proofPeriod*, *proofScore* \= 0 for that *proofPeriod.*  
4. As proofs are submitted during the epoch duration, a ***totalNodescore(node\_id**)* is computed as the sum of all individual *proofScores totalNodeScorenodeId \= ∑* **proofScore, for all proof scores**   
5. Once an epoch is completed, the appropriate amount of rewards is unlocked for all the delegators (proportional to their stake delegation)and node runers based on the node’s individual *totalNodeScore* for that epoch. 

The individual nodeScore formula is:![][image1]

All individual scores (per proof) are then accumulated

totalNodeScorenodeId,epoch= ∑ nodeScore(t), over the time of one epoch.

The total amount of TRAC fees (accumulated from publishing) in one epoch is then divided across all nodes by the following formula:

*![][image2]*

Each delegator (and node runner) is then eligible for their share of node rewards, according to their stake contribution, over a period of time during their stake availability in that epoch (*stakingPeriod*). The *nodeScore* (and therefore *delegatorScore*) is calculated on an ongoing basis per proof submission, while *totaTracFees* is accumulated over the epoch duration. This prevents the possibility that a delegator could stake at the end of an epoch, and claim rewards for the entire epoch (“free riding”).

Given the DKG hosts over 500M+ Knowledge Assets at the moment (est. 5B+ chunks), the probability of a chunk selection is 2\-10 , which will further diminish with the growth of the DKG, making these challenges “harder” over time as nodes need to maintain larger amounts of knowledge in a sector.

With this approach, all of the above functional requirements are satisfied:

* **FR1 The DKG proof system scalability requirement: satisfied (no impediments)**  
* **FR2 Core Node scalability requirement: satisfied (no impediments)**  
* **FR3 Simplified system-wide time management: satisfied**  
* **FR4 Trusted proof validation: satisfied**  
* **FR5 Safety requirement: satisfied**  
* **FR6 TRAC incentives favor positive network activity: satisfied**  
* **FR7 Proof system agnostic to network topology: satisfied**  
* **FR8 Real-time proof system: satisfied**

## **System performance** {#system-performance}

The above system is designed to be a lightweight addition to the existing DKG Core Node operations, ensuring that nodes are not burdened with significant additional computational and operational overhead. Proof generation and submission are streamlined, with most of the heavy lifting handled off-chain, yet not overloading the nodes too frequently, allowing them to focus on uptime, stake accrual, publishing, and service optimization. 

The intention is also to maintain low gas costs for proof submission per node, keeping on-chain interactions economically feasible for all participants. With the proof system optimized for minimal blockchain resource usage, the focus of future scalability efforts will shift toward scaling the graph database layer itself. This includes ongoing integrations with projects such as Amazon Neptune, as well as continued improvements to the performance and capacity of Blazegraph, ensuring the DKG infrastructure can support massive growth in Knowledge Assets and maintain high throughput for querying and publishing. Several intermediate releases are already underway, which will prepare the DKG for the random sampling system rollout. 

The initial proof period will start from 1 proof every 30 minutes, with the intention to lower the time to strengthen the availability requirements over the course of the DKG lifetime.  The nodes will maintain a robust proof submission system, attempting to submit proofs multiple times. The overall implementation is expected to be tuned with [OT-RFC-21](https://github.com/OriginTrail/OT-RFC-repository/blob/main/RFCs/OT-RFC-21_Collective_Neuro-Symbolic_AI/OT-RFC-21%20Collective%20Neuro-Symbolic%20AI.pdf) in mind, prioritizing high uptime, higher node stake, higher publishing factor, and lower service ask, in that order of priority.

With that in mind, the non-functional requirements indicated above are expected to be satisfied through iterative development and system tuning over its lifetime, particularly in the rollout period. 

## **Pre-V8.1  random sampling reward distribution** {#pre-v8.1-random-sampling-reward-distribution}

The random sampling system is to be deployed with 2 compatibility modules covering the period before the V8.1 release. Specifically, the system will include:

* **A V6 compatibility module** (covering the period before V8.0 release), which will enable nodes that were active during the V6 period to compete for token rewards for V6 Knowledge Assets, during a period of 12 months since V8.0 launch. The compatibility module will apply the same Random Sampling system (as indicated above), applied to V6 rewards.  
* **A Tuning Period compatibility module**, which will enable nodes active since the V8.0.0  release to collect rewards for the period between the V8.0.0 and V8.1.0  releases. This compatibility module will apply the same principles as indicated in this RFC, with the assumption of 100% uptime for all nodes in the Tuning Period. A prerequisite for the Tuning Period compatibility module deployment is the initial random sampling release, further explained below in the “Rollout plan” section.

To ensure the system is as transparent as possible, the DKG Staking UI will be updated to improve the visibility of system operations, including its compatibility modules, by adding additional features for tracking and monitoring staking performance and events in the DKG Staking UI.

# Rollout plan {#rollout-plan}

The rollout of the fully feature-complete random sampling system will follow the release sequence outlined below:

* **RFC confirmation** on Github,  
* **V8.1.0 release** with the feature implementation—from the moment of this release, DKG Core Nodes will be able to compete for newly incoming rewards (post-release). V8.1.0 is also a necessary prerequisite for the V8.1.2 release,  
* **V8.1.1 release** with the V6 compatibility module, from which nodes active during the V6 era will be able to compete for V6 rewards,  
* **V8.1.2 release** with the Tuning Period compatibility module.

The V8.1.X releases are intended to be rolled out in a rapid sequence (in a matter of weeks) as soon as development and testing are completed for each of the components (V8.1.0 and V8.1.1 are already in development as of the writing of this document).

# Conclusion {#conclusion}

The random sampling proof system introduced in this RFC represents a major step forward in the scalability, efficiency, and robustness of the OriginTrail Decentralized Knowledge Graph. By significantly reducing on-chain costs, minimizing operational burden on node operators, and enabling targeted scalability through sector-based knowledge storage, this system lays the foundation for sustainable, global-scale adoption of the DKG. With continued improvements to the graph database backend and a clear rollout path that includes backward compatibility, the DKG ecosystem is well-positioned to support the next wave of decentralized knowledge publishing. 

We invite all community members, node operators, and stakeholders to review this proposal in detail and provide feedback via the OriginTrail GitHub RFC discussion. Your insights are valuable in shaping a resilient, efficient, and inclusive knowledge infrastructure.

[image1]: <data:image/png;base64,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>

[image2]: <data:image/png;base64,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>