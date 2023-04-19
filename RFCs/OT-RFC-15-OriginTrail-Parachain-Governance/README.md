# OT-RFC-15 OriginTrail Parachain Governance


### Contributors: OriginTrail Core Developers, BRX, Milian, Dmitry, LuKu, Simon Pucko
Version: 1 \
Date: 19.04.2023.

The implementation of the OriginTrail Parachain Governance system is an important step in the parachain roadmap, aimed towards increasing the capacity of the OriginTrail community to inclusively guide the OriginTrail Parachain development and evolution. Governance mechanisms play a crucial role in ensuring the community alignment, security, and evolution of decentralized systems. With the introduction of OTP Governance, all OriginTrail Parachain ecosystem stakeholders will be able to participate in important decisions in a decentralized manner, which will improve the overall efficiency of the network.

The Polkadot ecosystem has a well-established track record when it comes to governance implementations. Polkadot is known for its innovative approach to governance, so the proposed OTP Governance implementation builds on this foundation, starting with OTP Governance v1.0, which is based on Polkadot Gov 1.0. This will provide a solid foundation for OTP Governance, which will then be further enhanced in future versions.

One of the immediate key benefits of implementing OTP Governance is that it will enable the removal of the "sudo" account. In Substrate, the "sudo" account is a powerful account that can perform any action on the network, and is typically used at the initial phases of the parachain development for faster iterations. By implementing governance features and removing the "sudo" account, the new Governance mechanisms will ensure that decisions are made in a more inclusive, decentralized and transparent manner.

This RFC proposes an iterative approach to OTP Governance that follows the developments of the Polkadot ecosystem governance mechanisms, formulating the general direction for OTP Governance 2.0 (designed to resemble Polkadot Gov 2.0 - OpenGov). 


# OTP Gov v1.0

In the OriginTrail Parachain Governance v1.0, **active OTP token holders** and **the council **together administrate network upgrade decisions. Whether the proposal is proposed by the public (token holders) or the council, it will eventually have to go through a **referendum** to let all OTP holders, weighted by stake and conviction, collectively make a decision on the proposal. To ensure timely governance, a General Council will be elected to propose, cancel and fast track referendums. Once implemented, a transfer of the Root origin account will happen from the sudo account to the democracy pallet, making it the new **Root origin** account. Root origin is a system level origin that has the highest privilege level and can be thought of as the superuser of the runtime origin.

**Note:** This document focuses on OriginTrail Parachain Governance - the tailored Layer 1 blockchain spawned by the OriginTrail ecosystem, with its OTP native token. It does not involve TRAC tokens or governance decisions on Layer 2 (OriginTrail DKG). For more information on the OriginTrail two-layered architecture, please refer to [this link](https://docs.origintrail.io/general/dkgintro).


## General council

The General council is an on-chain entity comprising several actors, each represented as an on-chain account. On OriginTrail Parachain, the initial council consists of 5 members, however the number of council members can be increased over time via governance proposals. The council is called upon primarily for three tasks of governance: 



1. proposing sensible referenda,
2. canceling uncontroversially dangerous or malicious referenda,
3. fast track referenda proposal.

Initially, members will be included in the council by using sudo pallet, and later, a Referenda will be started to move governance to the Elected Council phase where the candidacy of councilors is open, and councilors are elected by public voting.

The eligibility for candidacy for a council member position will include validation of at least 2 on-chain visible activity on the OriginTrail Parachain. The group of eligible activities include:



* Teleporting TRAC,
* Publishing Knowledge Assets,
* Running a DKG node on OriginTrail Parachain,
* 50.000 OTP stake amount for your bid,  
* OriginTrail Parachain auction slot participation.


### Prime member

The council implements what's called a prime member whose vote acts as the default for other members that fail to vote before the timeout.



* The purpose of having a prime member of the council is to ensure a quorum, even when several members abstain from a vote. Council members might be tempted to vote a "soft rejection" or a "soft approval" by not voting and letting the others vote. With the existence of a prime member, it forces councilors to be explicit in their votes or have their vote counted for whatever is voted on by the prime.


### Proposing



* For a referendum to be proposed by the council, a strict majority of members must be in favor, with no member exercising a veto. Vetoes may be exercised only once by a member for any single proposal; if, after a cool-down period, the proposal is resubmitted, they may not veto it a second time.
* Council motions which pass with a 3/5 (60%) super-majority - but without reaching unanimous support - will move to a public referendum under a neutral, majority-carrying voting scheme. In the case that all members of the council vote in favor of a motion, the vote is considered unanimous and becomes a referendum with negative adaptive quorum biasing.


### Canceling



* A proposal can be canceled if the council unanimously agrees to do so, or if Root origin (e.g. sudo) triggers this functionality (Democracy pallet). A canceled proposal's deposit is burned.
* Additionally, a two-thirds majority of the council can cancel a referendum. This may function as a last-resort if there is an issue found late in a referendum's proposal such as a bug in the code of the runtime that the proposal would institute.
* If the cancellation is controversial enough that the council cannot get a two-thirds majority, then it will be left to the stakeholders en masse to determine the fate of the proposal.


### Blacklisting



* A council blacklisted proposal and its related referendum (if any) are immediately canceled. Additionally, a blacklisted proposal's hash cannot re-appear in the proposal queue. Blacklisting is useful when removing erroneous proposals that could be submitted with the same hash.


### Fast tracking



* The general council has the power to fast-track proposals by using the Democracy pallet, and is the only origin that is able to trigger the fast-tracking functionality. 
* Fast-tracked referenda are the only type of referenda that can be active alongside another active referendum. Thus, with fast-tracked referenda it is possible to have two active referendums at the same time. Voting on one does not prevent a user from voting on the other.


## Democracy

Democracy module handles the administration of general stakeholder voting and has the ability to perform root calls so it can be used as a replacement for sudo pallet.

There are two different queues that a proposal can be added to before it becomes a referendum:



1. the proposal queue consisting of all public proposals,
2. the external queue consisting of a single proposal that originates from one of the external origins (such as the council).

Every launch period, the [Democracy pallet](https://crates.parity.io/pallet_democracy/index.html) launches a referendum from a proposal that it takes from either the proposal queue or the external queue in turn. Any token holder in the system can vote on referenda. The voting system uses **time-lock token based voting** by allowing the token holder to set their **conviction** behind a vote. The conviction will dictate **the length of time the tokens will be locked**, as well as the multiplier that scales the vote power. For more details on the implementation of the Democracy module in Substrate, please refer to this [link](https://crates.parity.io/pallet_democracy/index.html), and for more details on how to participate in democracy, please refer to this [link](https://wiki.polkadot.network/docs/maintain-guides-democracy).


### Referenda

Referenda are simple, inclusive, stake-based voting schemes. Each referendum has a specific proposal associated with it that takes the form of a privileged function call in the runtime (that includes the most powerful call: set_code = runtime upgrade).

Referenda are discrete events, have a fixed period during which the voting happens, and then are tallied and the function call is made if the vote is approved. Referenda are always binary; your only options in voting are "**aye**", "**nay**", or abstaining entirely.

Referenda can be launched in one of 3 ways:



1. Publicly submitted proposals;
2. Proposals submitted by the general council, either through a majority or unanimously;
3. Emergency proposals submitted by the general council (fast-tracked).

In case of options 1 and 2, the voting lasts for a fixed period of time (usually 28 days), and for option 3, the period can be set differently every time.


### Voting

To vote in a referendum, a voter generally must lock their tokens up for at least the enactment delay period beyond the end of the referendum. This is in order to ensure that some minimal economic buy-in to the result is needed and to dissuade vote selling.

Depending on which entity proposed the proposal and whether all council members voted yes, there are three different scenarios:



1. Public - Positive Turnout Bias (Super-Majority Approve);
2. The general council (Complete agreement)  - Negative Turnout Bias (Super Majority Against);
3. The general council (Majority agreement) - Simple majority.


#### **Super-Majority Approve**

A positive turnout bias, whereby a heavy super-majority of aye votes is required to carry at low turnouts, but as turnout increases towards 100%, it becomes a simple majority-carries as below.


![alt_text](images/image1.png "image_tooltip")



#### **Super-Majority Against**

A negative turnout bias, whereby a heavy super-majority of nay votes is required to reject at low turnouts, but as turnout increases towards 100%, it becomes a simple majority-carrying as below.


![alt_text](images/image2.png "image_tooltip")



#### **Simple-Majority**

Majority-carries, a simple comparison of votes; if there are more aye votes than nay, then the proposal is carried, no matter how much stake votes on the proposal.

![alt_text](images/image3.png "image_tooltip")



#### **Example**:

For the demonstration below, let’s assume there are 1500 OTP tokens in the network and the proposal is issued by the public (not by the general council).



* Peter: Votes No with 150 OTP for a 16 week lock period => 150 x 3* = 450 Votes
* Logan: Votes Yes with 500 OTP for a 4 week lock period => 500 x 1 = 500 Votes
* Kevin: Votes Yes with 100 OTP for a 4 week lock period => 100 x 1 = 100 Votes

\*The conviction multiplier increases the vote multiplier by one every time the number of lock periods doubles.

To calculate the voting result, we need 4 variables and the formulas listed below:



* Approve (number of aye votes) = 600
* Against (number of nay votes) = 450
* Turnout (total number of voting tokens, does not include conviction) = 750
* Electorate (total number of tokens issued in the network) = 1500

Since the referendum was proposed by the public (not the general council),  “Super Majority Approve” formula would be used to calculate the result. 


![alt_text](images/image4.png "image_tooltip")
 \
which yields \


![alt_text](images/image5.png "image_tooltip")


Super Majority Approve requires more aye votes to pass the referendum when turnout is low, therefore, based on the above result, the referendum will be rejected. In addition, only the winning voter's tokens are locked. If the voters on the losing side of the referendum believe that the outcome will have negative effects, their tokens are transferable so they will not be locked into the decision.

More details on voting and referendums are available [here](https://wiki.polkadot.network/docs/learn-governance#council).


# OTP Gov v2.0

The second iteration of the OTP Governance implementation (2.0) would entail further improvements in the Governance mechanics for the OriginTrail Parachain. As with implementation of v1 of Governance, we propose striving to learn and reuse as much of useful discoveries and outcomes derived from [Polkadot Governance improvements](https://wiki.polkadot.network/docs/learn-opengov), as well as OTP Governance v1 activities. OTP Governance v2 will be subject of a detailed RFC at a future time.


# Conclusion

The purpose of this document is to propose the implementation of the OriginTrail Parachain governance system, specifying the process mechanics and indicating its evolutionary direction. The OriginTrail community is invited to participate and provide feedback through the RFC process via Github. 
