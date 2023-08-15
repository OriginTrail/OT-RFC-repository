# OT-RFC-17 - OriginTrail integration with EVM-compatible blockchains

**Authors:** OriginTrail Core Developers

**Date:** August 15th 2023


![visual](./images/OT-RFC-17-Visual.jpg)


## Introduction - bringing any message across blockchains

The OriginTrail technical architecture consists of 3 layers - the consensus layer, the Decentralized Knowledge Graph (DKG) layer and the application layer. Since the OriginTrail ecosystem inception and the first whitepaper, the consensus layer was intended to include multiple blockchains in order to ensure neutrality, robustness and inclusiveness.

We have seen the first implementation of the multi-chain approach in the V5 of the DKG which ended up being integrated with Ethereum, Gnosis and Polygon. The currently newest V6, initially deployed on the OriginTrail Parachain on Polkadot, has yet to become a multichain network and drive trusted knowledge infrastructure across blockchains. This document serves as an outline of the path towards reaching that.

Web3 is currently lacking an interoperable layer for maintaining a trusted knowledge base across a growing number of blockchain ecosystems which is limiting the possibility to create cross-blockchain applications or expand user experience and user base across multiple blockchain ecosystems. 

In addition to limiting the value that can be created for Web3 users, it also significantly discourages Web2 users from onboarding to Web3. [According to research](https://www.pewresearch.org/short-reads/2023/04/10/majority-of-americans-arent-confident-in-the-safety-and-reliability-of-cryptocurrency/),[ ](https://www.pewresearch.org/short-reads/2023/04/10/majority-of-americans-arent-confident-in-the-safety-and-reliability-of-cryptocurrency/)75% of Americans who have heard of cryptocurrencies are not confident in their safety and reliability. While reasons for this are multifaceted, it is clear, the growth of this sector is impeded by misinformation, poorly structured and siloed Web3. 

**Web3 requires a semantic layer to bring any message across blockchains in a verifiable manner.** Having discussed integrations with several leading EVM compatible blockchain ecosystem champions, the right time to enable such integrations is now.  


## OriginTrail integration with EVM - compatible blockchains

The ambition of the multichain OriginTrail project is to drive trusted knowledge exchange across Web3 communities by equipping builders from various ecosystems with tools and guidance to integrate DKG with their EVM-compatible chain. 

**The impact of OriginTrail integration with a blockchain ecosystem**: 


* Bringing Decentralized Knowledge Graph technology to any blockchain ecosystem enabling Trusted AI applications, such as AI-powered trusted semantic search ([ChatDKG](https://chatdkg.ai) initiative)
* Enabling Enterprise solutions leveraging DKG and AI technology on your chain (enterprises using OriginTrail [here](https://origintrail.io/solutions/overview))
* Cross-chain knowledge portability (e.g. bridging knowledge for multichain Dapps)
* Semantic data oracle functionality for smart contracts on your blockchain (e.g. RWA applications)
* Integrations with applications and protocols within your blockchain ecosystem, leveraging the interoperable design of Knowledge Assets (e.g. NFT marketplaces for trading Knowledge assets)
* Shared user experience between metaverses leveraging trusted Knowledge Assets across blockchains (e.g. [Tracverse](https://www.tracverse.com/))

Enablement of EVM- compatible blockchain ecosystems drives further exposure and awareness about OriginTrail DKG, resulting in synergies between crypto communities and ultimately the establishment of the **Multichain Trusted Knowledge Foundation.**


### EVM blockchains DKG integration steps

The process kick-starting the integration of Decentralized Knowledge Graph on an EVM compatible blockchains is straightforward. The first step is to **create a pull request in the [OT-RFC-repository](https://github.com/OriginTrail/OT-RFC-repository/)** using the template pull request with the following name: _RFC-17-CHAIN_NAME_integration_. In the PR you should create a JSON file with key details about the blockchain network (name, network ID, block explorer, website) and addresses of the smart contracts for TRAC bridge (both on Ethereum and destination chain). An example of the file is available [here](https://github.com/OriginTrail/OT-RFC-repository/blob/main/RFCs/OT-RFC-17-OriginTrail-integration-with-EVM-compatible-blockchains/chains/integration-requests/example.json).

Once the pull request is submitted, the core development team will validate the submission to **ensure that no inputs are missing and that all requirements are fully met**(see below). Depending on the findings, the integration might be approved for the next stage or a request for more information will be given.

Once approved, a dedicated channel on [OriginTrail Discord](https://discord.gg/cCRPzzmnNT) will be created for the blockchain network and integration will be scheduled for implementation (the exact date will be communicated in Discord). 


### EVM blockchain requirements for DKG integration

There are a few key criteria that need to be met in order for integration to proceed: 

* The blockchain network needs to be EVM-compatible.
* The blockchain network needs a functioning, preferably decentralized, bridge for the TRAC token.
* Engagement within the OriginTrail community - there should be a positive presence of the particular blockchain community present in the OriginTrail community and/or vice-versa. The signaling of desired DKG utility on a particular blockchain ensures that the deployed network will see reasonable activity. 
* 5MM+ TRAC bridged to the blockchain network showing support to initiate network infrastructure and initial DKG network activity.


## Conclusion

This RFC has presented the approach to integrating OriginTrail DKG with EVM-compatible blockchains, the value propositions and indicated requirements for such integrations. We invite the wider Web3 community to participate in the RFC process, provide their feedback and initiate integrations.
