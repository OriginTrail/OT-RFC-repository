# OT-RFC-23 Trustless NeuroWeb — Ethereum TRAC bridge

**Authors: OriginTrail Core Developers, team Snowfork**

**Publishing Date:** Mar 21, 2025
**Last updated date:** Jun 13, 2025

## **Introduction**

The NeuroWeb blockchain has emerged as the leading OriginTrail DKG chain, demonstrating substantial adoption through the publication of hundreds of millions of Knowledge Assets and the utilization of over 115 million TRAC tokens bridged from Ethereum, representing \~23% of all TRAC in existence. This high demand necessitates minimizing friction for TRAC token holders on NeuroWeb. Therefore, this RFC proposes implementing a trustless bridge to replace the current teleport system, aiming to streamline the users‘ use of TRAC tokens.

## **Bridge technology proposal**

After extensive discussions with multiple bridge-solution teams and a thorough review of bridging approaches, a comprehensive evaluation was conducted to determine the most robust and compatible solution for bridging TRAC. Key considerations included solution reliability, security, trust model, and especially the compatibility with the currently deployed TRAC token on NeuroWeb. As TRAC is the utility token of the DKG used in production, the bridge deployment has to take this into account and not disrupt the existing system in any way. 

Based on these factors, the decision was made to proceed with implementing Snowbridge.

[**Snowbridge**](https://app.snowbridge.network/) is a trustless, decentralized, and general-purpose bridge designed to connect Polkadot and Ethereum. As part of the Polkadot SDK, it operates as a common-good bridge on Bridge Hub, ensuring seamless integration with the broader Polkadot ecosystem. Unlike other bridging solutions, Snowbridge is fully owned by the Polkadot community and does not rely on problematic multisigs; instead, it leverages Polkadot validators for security. It also integrates smoothly with Polkadot’s Asset Hub and utilizes XCM as its message format, ensuring efficient interoperability. More details on Snowbridge can be found in their documentation: [Snowbridge Docs](https://docs.snowbridge.network/).

## **Technical approach**

To establish a compatible bridge with the existing TRAC token implementation on NeuroWeb, the proposal is to leverage the standard Snowbridge infrastructure and build a wrapper pallet on the NeuroWeb side. As Snowbridge operates with the Polkadot Asset Hub chain for bridging assets from Ethereum, an additional wrapper pallet is introduced on NeuroWeb to convert the Asset Hub registered bridging asset to a 1-to-1 parity of the TRAC XC20 asset on NeuroWeb. 

From a user perspective, this wrapping operation will be abstracted (it will happen “under the hood”), so regular users will be unaware of the complex bridging mechanics under the hood. The following diagram illustrates the TRAC bridging process in the direction from Ethereum to NeuroWeb (the process is equivalent in the other direction).

Bridging from Ethereum requires a small amount of ETH for Ethereum gas fees, while bridging from NeuroWeb will require a small amount of DOT for the bridging fee, due to that being an XCM bridge transfer requirement. Capabilities of covering the bridging fee with a different token (such as NEURO or TRAC) will be explored as well as part of the continuous bridge UX improvement efforts.


**Update Jun 13 2025:** After thorough RFC review together with Polkadot & OriginTrail community, the following updated proposal is introduced.

There are two broad scenarios for bridging TRAC from Ethereum to Polkadot:

1. Bridging TRAC for purpose of DKG utility (knowledge publishing, staking), which requires bridging TRAC to Neuroweb parachain
2. Bridging TRAC for other purposes within the Polkadot ecosystem, such as trading and providing liquidity on Polkadot DEXes for example

The optimal technical approach for supporting the two scenarios above, while maintaining the highest level of security, as well as optimal level of user and developer experience is presented in the figure below.

// NEW IMAGE

![OT-RFC-23 Figure 1](https://github.com/OriginTrail/OT-RFC-repository/blob/ot-rfc-23/RFCs/OT-RFC-23-Trustless_Neuroweb_-_Ethereum_TRAC_bridge/ot-rfc-23-figure-1.png?raw=true)

Figure 1: Bridging TRAC between Ethereum and Polkadot

The key components of the system are Snowbridge, the Polkadot Asset Hub (PAH) and Neuroweb TRAC wrapper pallet. 

**In case of Scenario 1**, the approach will entail a path through Snowbridge and Polkadot Asset Hub (PAH), finally using the Neuroweb TRAC wrapper pallet to reconcile the existing XC20 TRAC used for utility and the cannonical PAH TRAC at 1 to 1 parity. The wrapper pallet will effectively lock/unlock PAH TRAC on one side and mint/burn XC20 TRAC on the Neuroweb side, to be used for DKG utility (it just transforms the Snowbridge output token into an equal number of TRAC on NeuroWeb and vice versa). Introducing the TRAC wrapper pallet will further enhance the security of the entire system as the TRAC wrapper pallet, which is governed by the robust NeuroWeb Governance and secured by Polkadot, willhave minting and burning privileges over the XC20 TRAC asset (currently in control by OriginTrail Core Developers for Teleport purposes).

**For Scenario 2**, the approach only involves Snowbridge and Polkadot Asset Hub, removing any dependency on Neuroweb and the TRAC wrapper pallet. This will allow for the use of TRAC on other Polkadot parachains, such as Hydration, without the need to bridge TRAC to Neuroweb and without adding any additional technical complexity introduced by the TRAC wrapper pallet. This means that the **TRAC token on Polkadot Asset Hub will become the cannonical TRAC token for the Polkadot ecosystem**, making integration wtih any other Polkadot parachain "standardized", without any customizations required and dependency on Neuroweb.



\* This RFC completes the initial bridging RFC, [OT-RFC-16](https://github.com/OriginTrail/OT-RFC-repository/tree/main/RFCs/OT-RFC-16-Parachain-Bridges-Implementation).

## **Rollout plan (updated)**

Based on the updated approach, a 2-phase rollout plan is proposed

1. **Phase 1** will bring the initialization of cannonical TRAC for the Polkadot ecosystem, directly enabling Scenario 2 and trustless bridging of TRAC between Ethereum and Polkadot parachains (except Neuroweb). Phase 1 is expected to be completed by the end of June 2025.

2. **Phase 2** will bring the TRAC wrapper pallet to the Neuroweb parachain, enabling Scenario 1 and the use of TRAC for DKG utility. During this phase the Teleport system will be deprecated. Before teleport deprecation, everyone will have a chance to teleport in any direction one more time through teleport. Should anyone be unwilling to use TRAC with Snowbridge as a successor to the Teleport system, they will have the option to “opt out” by teleporting back to Ethereum prior to completion of Phase 2. Phase 2 is expected to be completed by the end of July 2025.

OriginTrail Core Developers and the Snowbridge team will collaborate closely on the rollout, initially verifying the integration on the NeuroWeb (Paseo) testnet. Exact timeline may be subject to change, depending on the dynamics of releasing the DKG V8.1 on the OriginTrail mainnet, and other ongoing activities. 

## **Conclusion**

Implementing a trustless Snowbridge-based TRAC bridge marks a significant advancement in the OriginTrail ecosystem by establishing a secure, decentralized bridge between Ethereum and Polkadot. This proposal introduces TRAC as a canonical token on Polkadot Asset Hub, enabling seamless integration with the broader Polkadot ecosystem while maintaining dedicated support for DKG utility on NeuroWeb through the TRAC wrapper pallet. The two-phase rollout plan ensures a smooth transition from the current Teleport system, first enabling trustless bridging for general Polkadot ecosystem usage, followed by DKG utility support on NeuroWeb. This approach provides optimal security through Polkadot validator consensus while maximizing both user and developer experience. We invite the OriginTrail community to review this proposal and share their feedback in the official RFC GitHub issue.
