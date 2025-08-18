## **OT-RFC-25: DKG V8 network ask tokenomics tuning** 

### **Introduction**

The OriginTrail Decentralized Knowledge Graph (DKG) tokenomics model plays a central role in ensuring network growth, fairness, and sustainability. Since the launch of the V8 mainnet, real-world economic dynamics have revealed behaviors that, while aligned with the current incentives, have unintended side effects on network tokenomics. This RFC proposes targeted adjustments to the tokenomics formula to address these issues, creating a more balanced, predictable, and cooperative environment for node operators and publishers.

### **The problem**

The current tokenomics design rewards a combination of high stake, high publishing factor, and lower ask price, with the intent of promoting affordability and adoption of DKG services.

For context, the publishing price (expressed in TRAC per kb\*epoch) , also called **network ask,** is calculated as a stake-weighted average of all node ask votes within the two-sigma range from the current network ask (excluding outliers).

In practice, however, nodes have been lowering their service ask to gain more “power” in the reward calculation, with the expectation of capturing a larger share of TRAC rewards. This behavior unintentionally pushes the entire network price downward, reducing publishing fees and ultimately lowering TRAC rewards for all participants, including the nodes that lowered their ask with the intention to increase their rewards.

The node operators community has described this as a *“race to the bottom”* situation. Technically, the network price is determined as a node stake-weighted “ask vote” (each node contributes in the network price scaled by the amount of stake it has attracted), but an individual node’s ask does not directly influence its TRAC earnings, rather indirectly as a vote in the total network ask. This creates an **asymmetry**, prices tend to drift downwards without a counterbalance, as nodes believe lower asks benefit the network and their own earnings, but in reality they erode the reward pool.

### **The solution proposal**

Since the DKG V8 roadmap is currently in the Tuning phase, we propose the two key adjustments:

#### 1. **Modify the ask impact in tokenomics**

1.1 Keep the network ask determination mechanism \- compute the network ask as the stake-weighted average of node "ask votes" within two sigma (unchanged)

$$dkgNetworkPrice = \frac{\sum_{k=1}^{n} stake_k \cdot askVote_k}{\sum_{k=1}^{n} stake_k}$$

Where:  
- $stake_k$ = stake of node *k*  
- $ask\_vote_k$ = ask vote of node *k,* set by the node operator  
- $\sigma$ is the standard deviation of the current *dkgNetworkPrice*  
- $n$ is the number of nodes that satisfy the condition:

$$dkgNetworkPrice - \sigma < askVote_k < dkgNetworkPrice + \sigma$$  

1.2 Introduce a deviation penalty: any node ask that deviates from the network ask reduces that node’s power, lowering its reward potential

<br>

The formula for node power remains the same:

$$nodePower = nodeStakeFactor + nodeAskFactor + nodePublishingFactor$$

<br>

however with an updated *nodeAskFactor*:

<br>

$$nodeAskFactor = 1 - \left|\frac{askVote - dkgNetworkPrice}{dkgNetworkPrice}\right|$$   

Meaning node ask\_vote deviation from the network price (both being higher and lower) will lower that node's ask factor, ultimately lowering its power in the network. (additional factors in the equation may apply and will be determined during the tuning process).  

This removes the existing asymmetry, aligning incentives so that nodes are rewarded for converging toward a rational, sustainable network ask price.

#### 2. **Introduce epoch-level ask voting to promote coordination**

   * Determine the network ask once per epoch instead of in real time (based on stake weighted average of all ask votes, same as now)  
   * Nodes can set their ask vote for the next epoch during the current epoch’s “voting” period. The nodes can change their ask vote multiple times, as long as the voting period is open, allowing all node operators to converge to an optimal network ask in an open and transparent way.  
   * The voting period would start at the beginning of an epoch and end 5 days before the next epoch starts.  
   * Once voting closes, the network price is locked for the duration of the next epoch.  
   * This also has a positive side effect of creating predictability for publishers and encourages deliberate, coordinated price-setting by node operators.

### **Considerations**

* **Node ask now officially presents a vote for the network price,** removing confusion and clarifying the ask influence. The design encourages convergence towards an optimal network service price, where each node gets to vote for the price based on their conviction. 

* **Minimal disruption:** Most of the random sampling and reward logic remains unchanged.

* **Balance restored:** The system becomes balanced and symmetrical, preventing persistent downward pressure on price.

* **Coordination benefits:** Predictable pricing encourages collaboration rather than destructive competition.

### **Implementation timeline**

This tokenomics adjustment is targeted for completion in **Q3 2025**, in parallel with the remaining V8.1.X releases.

### **Conclusion**

These changes are designed to strengthen the DKG economy by restoring balance, encouraging healthy coordination, and sustaining network growth. We invite the OriginTrail community, especially node operators and publishers, to review this proposal and share feedback before finalization.

**Call to Action:** Please submit your comments and suggestions on the official OriginTrail governance forum or community channels. Your input will directly shape the next stage of DKG tokenomics.

