# OT-RFC-23 Trustless NeuroWeb — Ethereum TRAC bridge

**Authors: OriginTrail Core Developers, team Snowfork**

**Date:** Mar 21, 2025

## **Introduction**

The NeuroWeb blockchain has emerged as the leading OriginTrail DKG chain, demonstrating substantial adoption through the publication of hundreds of millions of Knowledge Assets and the utilization of over 115 million TRAC tokens bridged from Ethereum, representing \~23% of all TRAC in existence. This high demand necessitates minimizing friction for TRAC token holders on NeuroWeb. Therefore, this RFC proposes implementing a trustless bridge to replace the current teleport system, aiming to streamline the users‘ use of TRAC tokens.

## **Bridge technology proposal**

After extensive discussions with multiple bridge-solution teams and a thorough review of bridging approaches, a comprehensive evaluation was conducted to determine the most robust and compatible solution for bridging TRAC. Key considerations included solution reliability, security, trust model, and especially the compatibility with the currently deployed TRAC token on NeuroWeb. As TRAC is the utility token of the DKG used in production, the bridge deployment has to take this into account and not disrupt the existing system in any way. 

Based on these factors, the decision was made to proceed with implementing Snowbridge.

[**Snowbridge**](https://app.snowbridge.network/) is a trustless, decentralized, and general-purpose bridge designed to connect Polkadot and Ethereum. As part of the Polkadot SDK, it operates as a common-good bridge on Bridge Hub, ensuring seamless integration with the broader Polkadot ecosystem. Unlike other bridging solutions, Snowbridge is fully owned by the Polkadot community and does not rely on problematic multisigs; instead, it leverages Polkadot validators for security. It also integrates smoothly with Polkadot’s Asset Hub and utilizes XCM as its message format, ensuring efficient interoperability. More details on Snowbridge can be found in their documentation: [Snowbridge Docs](https://docs.snowbridge.network/).

## **Technical approach**

To establish a compatible bridge with the existing TRAC token implementation on NeuroWeb, the proposal is to leverage the standard Snowbridge infrastructure and build a wrapper pallet on the NeuroWeb side. As Snowbridge operates with the Polkadot Asset Hub chain for bridging assets from Ethereum, an additional wrapper pallet is introduced on NeuroWeb to convert the Asset Hub registered bridging asset to a 1-to-1 parity of the TRAC XC20 asset on NeuroWeb. 

From a user perspective, this wrapping operation will be abstracted (it will happen “under the hood”), so regular users will be unaware of the complex bridging mechanics under the hood. The following diagram illustrates the TRAC bridging process in the direction from Ethereum to NeuroWeb (the process is equivalent in the other direction).

![OT-RFC-23 Figure 1](https://github.com/OriginTrail/OT-RFC-repository/blob/ot-rfc-23/RFCs/OT-RFC-23-Trustless_Neuroweb_-_Ethereum_TRAC_bridge/ot-rfc-23-figure-1.png?raw=true)

Figure 1: Bridging TRAC from Ethereum to NeuroWeb, with the wrapping transaction happening “under the hood”

Bridging from Ethereum requires a small amount of ETH for Ethereum gas fees, while bridging from NeuroWeb will require a small amount of DOT for the bridging fee, due to that being an XCM bridge transfer requirement. Capabilities of covering the bridging fee with a different token (such as NEURO or TRAC) will be explored as well as part of the continuous bridge UX improvement efforts.

As a consequence of this approach, the currently used XC20 TRAC token on NeuroWeb would be the primary token used within the Polkadot ecosystem (e.g., bridged through XCM to other Polkadot chains) to minimize friction for all users and enable the utilization of existing infrastructure of both Snowbridge and XC20 TRAC. From a security perspective, these two technical components require no modifications and, therefore, do not introduce any new security assumptions or require additional auditing. A similar approach has been applied and verified by Mythos (though only in one direction at the moment).

The only new component within the system is the TRAC wrapper pallet, which is trivial to implement as it just transforms the Snowbridge output token into an equal number of TRAC on NeuroWeb and vice versa. Introducing the TRAC wrapper pallet will further enhance the security of the entire system as the TRAC wrapper pallet, which is governed by the robust NeuroWeb Governance and secured by Polkadot, will be transferred minting and burning privileges over the XC20 TRAC asset (currently in control by OriginTrail Core Developers for Teleport purposes).

\* This RFC completes the initial bridging RFC, [OT-RFC-16](https://github.com/OriginTrail/OT-RFC-repository/tree/main/RFCs/OT-RFC-16-Parachain-Bridges-Implementation).

## **Rollout plan**

To facilitate adoption and to lower the time to deployment, a 3-phase rollout plan is proposed.

1. **Phase 1** will see the Teleport system concluding its operations. During this phase, everyone will have a chance to teleport in any direction. Should anyone be unwilling to use TRAC with Snowbridge as a successor to the Teleport system, they will have the option to “opt out” by teleporting back to Ethereum prior to Phase 2\.  
2. **Phase 2** will deploy a one-directional TRAC bridge from Ethereum to NeuroWeb. Transfers will not be possible from NeuroWeb to Ethereum until Phase 3  
3. **Phase 3** will enable the opposite direction (NeuroWeb to Ethereum), completing the rollout of the trustless TRAC bridge between NeuroWeb and Ethereum.

OriginTrail Core Developers and the Snowbridge team will collaborate closely on the rollout, initially verifying the integration on the NeuroWeb (Paseo) testnet. 

## **Conclusion**

Implementing a trustless Snowbridge-based TRAC bridge marks a significant step forward in improving the user experience, efficiency, and interoperability of TRAC token transfers between Ethereum and NeuroWeb. This proposal ensures long-term sustainability and seamless asset mobility within the OriginTrail ecosystem by replacing the current Teleport system with a decentralized, validator-secured bridge. The phased rollout plan allows for a smooth transition, providing TRAC holders ample opportunity to adapt. We invite the OriginTrail community to review this proposal and share their feedback in the official RFC GitHub issue.
