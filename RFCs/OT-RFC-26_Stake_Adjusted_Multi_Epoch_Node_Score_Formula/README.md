# RFC: Stake-Adjusted, Multi-Epoch Node Score Formula for DKG Rewards

**Status:** Draft (Request for Comments)

**Target activation:** End of next epoch (~February 10)

**Related parameter change:** Stake cap / delegation threshold increase **5M → 10M $TRAC** is imminent (separate but closely related)

This RFC proposes a new tokenomics scoring formula that: (1) prevents rewards from becoming overly dominated by stake as the stake cap increases (and as we plan to lift the cap entirely later), and (2) evaluates publishing contribution over a broader time window (multi-epoch), rather than only the current epoch.



## 1. Summary

This is a proposal to replace the curren DKG node scoring formula (used to determine node power for tokenomics / rewards) with a new formula that:

1. **Reduces stake dominance** by using **sublinear stake scaling** (square root) and normalizing stake against the current network maximum, helping us safely move toward **lifting the stake cap entirely** in the future.
2. **Incorporates publishing performance over multiple epochs** (current + previous epochs) to better reflect sustained contribution and avoid the "one-epoch-only" limitation.
3. **Preserves pricing discipline incentives** (ask alignment vs a network reference price) as a meaningful part of the score.

This is intentionally being introduced as an RFC **before implementation**, with the goal of collecting node operator and delegator feedback ahead of the planned epoch-boundary rollout (~Feb 10).


## 2. Motivation

### 2.1 Stake cap increase is imminent; the current formula will skew tokenomics toward stake

With the **stake cap / delegation threshold increasing to 10M**, the current scoring model will increasingly overweight stake and push the tokenomics distribution too strongly toward the highest-staked nodes.

### 2.2 We want a path to lifting the stake cap entirely

Longer term, the stake cap is expected to be **lifted entirely**. Under the current scoring design, removing the cap would further amplify stake dominance and reduce the relative importance of publishing/value production.

### 2.3 Current scoring only "sees" the current epoch

The current model's effective awareness of performance is limited to the current epoch, which can:

* over-reward short-term spikes,
* under-reward consistent publishers,
* increase volatility for both node operators and delegators.

### 2.4 Design intent: balance stake + value creation across time

Therefore, the proposal is to **adjust the parameters** so the contribution of node stake is compatible with lifting the stake cap, while **expanding publishing measurement across ~4 months** (implemented as a rolling multi-epoch window) to capture sustained knowledge produced value.


## 3. Current Formula (Legacy)

The current node scoring function is:
![Node Score Formula](https://github.com/OriginTrail/OT-RFC-repository/raw/main/RFCs/OT-RFC-24_DKG_V8_Random_sampling_proof_system/rfc-24-figure1.png)


**Observed issue (with 10M cap):** the squared stake term grows very fast as stake increases. As the cap/threshold rises (and especially if removed later), this shape risks turning the scoring mechanism into "mostly stake," reducing the intended influence of publishing/value production and other quality signals.


## 4. Proposed Formula (New)

The proposed scoring formula is:

$$
\text{nodeScore}(t) = 0.04 \cdot S(t) + 0.86 \cdot P(t) + 0.6 \cdot A(t) \cdot P(t)
$$

Where the factors are defined in the following way:

### 4.1 Stake factor S(t)

$$
S(t) = \sqrt{\frac{\text{nodeStake}(t)}{\text{STAKE\_CAP}}}
$$

* Uses **square root** scaling (sublinear): doubling stake no longer doubles score contribution.
* Normalized by STAKE_CAP (to be set at 10M TRAC).

### 4.2 Publishing / produced knowledge value factor P(t) over a multi-epoch window

Let:

- $n \in \mathcal{N}$ — A node in the network.
- $e \in \mathcal{E}_t$ — An epoch within the evaluation window ending at epoch $t$.
- $\mathcal{E}_t = \{t-3, t-2, t-1, t\}$ — The rolling window of the last **4 epochs**.
- $K_{n,e}$ — The TRAC-denominated knowledge value published by node $n$ in epoch $e$.

#### Node-Level Published Knowledge Value

The total knowledge value published by node $n$ over the evaluation window:

$$
K_n^{(t)} = \sum_{e \in \mathcal{E}_t} K_{n,e}
$$

#### Network-Level Published Knowledge Value

The total knowledge value published by all nodes over the same window:

$$
K_{\text{total}}^{(t)} = \sum_{e \in \mathcal{E}_t} \sum_{m \in \mathcal{N}} K_{m,e}
$$

#### Publishing Factor

The publishing factor for node $n$ at epoch $t$ is defined as:

$$
P_n(t) = \frac{K_n^{(t)}}{K_{\text{total}}^{(t)}} = \frac{\sum_{e \in \mathcal{E}_t} K_{n,e}}{\sum_{e \in \mathcal{E}_t} \sum_{m \in \mathcal{N}} K_{m,e}}
$$

* $P(t)$ is the node's **share** of produced knowledge value, normalized by the total produced knowledge value across all nodes.
* The proposal explicitly expands this to **current + 3 previous epochs** (rolling window) to capture sustained contribution rather than only the current epoch.

### 4.3 Network Ask alignment factor A(t)

$$
A(t) = 1 - \frac{|\text{askVote} - \text{dkgNetworkPrice}|}{\text{dkgNetworkPrice}}
$$

* Rewards nodes whose ask is close to the network reference price (pricing discipline).
* The absolute relative deviation ensures a symmetric penalty for being above or below the reference.

Details on the ask alignment factor and reasoning are explained in [OT-RFC-25](https://github.com/OriginTrail/OT-RFC-repository/tree/main/RFCs/OT-RFC-25_DKG_V8_Network_Ask_Tokenomics_Tuning).


## 5. Design Rationale

### 5.1 Why this reduces stake dominance


 **Old (quadratic)**:  $\left(\frac{\text{nodeStake}(t)}{STAKE\_CAP}\right)^2$ 

**New (sublinear)** : $S(t) = \sqrt{\frac{\text{nodeStake}(t)}{\text{STAKE\_CAP}}}$

This compresses the advantage of extremely high stake and makes the system more compatible with eventually removing the cap.

### 5.2 Why multi-epoch publishing improves incentives

A rolling window (current + previous epochs) reduces:

* "one-epoch gaming,"
* volatility,
* penalty for nodes that publish consistently but have a single weak epoch.

It also aligns rewards with **sustained knowledge/value production**, matching the intent described in the RFC suggestion.

### 5.3 Why A(t) multiplies P(t)

In the proposed formula, ask alignment amplifies publishing rewards via `A(t) · P(t)` rather than operating as a largely separate stake-weighted term.

This structure aims to ensure:

* Pricing discipline is valuable, but
* it is most valuable when paired with meaningful production.


## 6. Numerical Example (Old vs New)

### Scenario

A top-performing node grows from **5M → 10M TRAC** after the threshold increase. Publishing and pricing behavior remain unchanged.

**Assumptions**

* `nodePubFactor / maxPubFactor = 0.8`
* Ask competitiveness ratio (legacy) = `0.8`
* Publishing share over 4 epochs: `P = 0.04` (4% of network output)
* Ask alignment: `A = 0.98`
* `STAKE_CAP = 10M`

---

### Old Formula Results

| Stake | Node Score |
| ----- | ---------- |
| 5M    | **24.1**   |
| 10M   | **93.2**   |

**Increase:**
→ **3.87×** score increase from doubling stake

This demonstrates how the legacy quadratic stake term heavily skews rewards toward higher stake as the cap rises.

---

### New Formula Results

Using the formula: $\text{nodeScore}(t) = 0.04 \cdot S(t) + 0.86 \cdot P(t) + 0.6 \cdot A(t) \cdot P(t)$

| Stake | S(t)   | Node Score |
| ----- | ------ | ---------- |
| 5M    | 0.7071 | **0.0862** |
| 10M   | 1.0    | **0.0979** |

**Increase:**
→ **1.14×** total score increase (stake component increase acts sublinearly, but P and A terms remain constant)

**Stake still matters, but marginal gains are controlled and predictable.**

---

### Cross-Node Comparison

| Node              | Stake | S(t)   | Publishing Share (P) | Old Score  | New Score  |
| ----------------- | ----- | ------ | -------------------- | ---------- | ---------- |
| A (high publisher)| 5M    | 0.7071 | 4%                   | 24.1       | **0.0862** |
| B (low publisher) | 10M   | 1.0    | 1%                   | **65.2**   | 0.0545     |

**Outcome**

* **Old formula:** High-stake, low-publishing node wins (65.2 > 24.1)
* **New formula:** Sustained publisher wins despite lower stake (0.0862 > 0.0545)

This directly illustrates how the new model realigns incentives with long-term network value creation rather than capital concentration. 

---

## 7. Rollout Plan and Timeline

### 7.1 Related changes (already communicated)

* **Stake cap / delegation threshold:** increase from **5M to 10M $TRAC** is imminent (to accommodate more delegations on best performing DKG nodes).
* **Outstanding rewards:** following the current epoch end on **Jan 10**, a **snapshot** will be taken; outstanding rewards are planned to be released at end of next epoch (~**Feb 10**).

### 7.2 Proposed rollout for formula change

1. **RFC publication:** immediately (this document)
2. **Feedback window:** 1 week
3. **Implementation + testing:** in parallel (testnet)
4. **Activation:** **end of next epoch (~Feb 10)**, aligned with the previously communicated schedule for introducing the formula change.


## 8. Expected Impact

### 8.1 Delegators

* Delegations to top nodes remain possible with the increased threshold, but the **marginal rewards benefit of "more stake" is reduced**, helping keep publishing performance meaningful.
* Multi-epoch measurement should reduce sharp reward swings and better reflect sustained node performance.

### 8.2 Node operators

* Encourages sustained publishing/value creation over time.
* Encourages competitive asks close to reference network pricing.
* Reduces the risk that the 10M cap increase (and future cap removal) turns rewards into a near-pure stake competition.


## 9. Conclusion & Call for Feedback

This RFC proposes a significant improvement of the DKG node scoring formula,  designed to support the network's growth while preserving the core principle that **value creation should drive rewards**.

By introducing sublinear stake scaling and multi-epoch publishing evaluation, we aim to create a more balanced, sustainable tokenomics model that:

- Supports lifting the stake cap without destabilizing incentives
- Rewards consistent, long-term contributors over short-term optimizers
- Maintains pricing discipline as a meaningful factor

**We want your input.** This proposal directly affects node operators and delegators, and community feedback is essential to getting it right.

### How to contribute

- **GitHub:** Comment on the dedicated OT-RFC-26 issue here: https://github.com/OriginTrail/OT-RFC-repository/issues
- **Discord:** Join the discussion in the [OriginTrail Discord](https://discord.gg/origintrail) community channels

