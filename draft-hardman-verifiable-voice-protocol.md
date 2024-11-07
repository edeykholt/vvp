---
title: "Verifiable Voice Protocol"
abbrev: "VVP"
category: std

docname: draft-hardman-verifiable-voice-protocol-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: AREA
# workgroup: WG Working Group
keyword:
 - voip
 - telecom
 - telephone number
 - vetting
 - KYC
venue:
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  github: "dhh1128/verifiable-voice-protocol"
  latest: "https://dhh1128.github.io/verifiable-voice-protocol/draft-hardman-verifiable-voice-protocol.html"

author:
 -
    fullname: "Daniel Hardman"
    organization: Provenant, Inc
    email: "daniel.hardman@gmail.com"

normative:

informative:

--- abstract

Verifiable Voice Protocol (VVP) proves the identity and authorization of parties who are accountable for telephone calls, eliminating trust gaps that scammers exploit. VVP aims at roughly the same target as STIR/SHAKEN+RCD and related technologies. Like these approaches, it binds strong cryptographic evidence to the message that initiates a phone call, and allows this evidence to be verified downstream. However, VVP builds from different technical and governance assumptions. This foundation makes VVP simpler, more decentralized and cross-jurisdictional, cheaper to deploy and maintain, more private, more scalable, and higher assurance than alternatives. It can be used by itself, or as an inexpensive supplement to bridge gaps in existing approaches.

--- middle

# Introduction

When we get phone calls, we want to know who's calling, and why. If we could answer such questions with confidence, we would make reasonable judgments about whether to answer, based on the relationship and reputation context. 

Unfortunately, providing quality answers is much harder than it seems. Scammers love to impersonate, and they succeed more often than we'd like.

Regulators are mandating increased transparency to address this risk. This is wise, and it can be effective. However, it introduces some heavy burdens. Reliably vetting the identity of every possible caller is an ambitious task. Maintaining a comprehensive, up-to-date graph of certificates and cryptophic keys to attest vetted identities is harder. Operating a central registry of such identities and evidence, especially as phone calls increasingly cross jurisdictional boundaries, is harder still. Regulatory constraints on data movement, shifting technical standards, variety in PSTNs and system interconnects, and many other factors further complexify.

However, the inherent limit in today's transparency efforts is more fundamental: there exists a pernicious a gap between the party accountable for a phone call, and the party who answers the "who's calling?" question. A decade ago, the TSP told us who's calling in a probabilistic way, based on unreliable metadata about a call's source. More recently, approaches like STIR/SHAKEN move the answer closer to the origin, and add tamper protections. However, there is still a gap: the signer of a SHAKEN passport is the OSP, not the caller, and the OSP may not know what's happening in this gap.

In theory, the remaining gap could be eliminated by requiring a perfect authentication and authorization between callers and OSPs, for every phone call. However, this is impractical for many legitimate business reasons. To cite one obvious example, Organization X might hire Call Center Y to call on their behalf, using X's reputation/brand and X's phone number. In such a situation, X is the relevant answer to the "who's calling" question, but Y is the originator of the call -- and there's no good way to guarantee that everyone knows the existence and status of that relationship. In addition, the market manifests a demand for wholesalers/aggregators of phone numbers, and is characterized by complex system layers and interconnects, that further challenge transparency.

VVP solves these problems by applying two crucial innovations:

* It uses an evidence format called an ACDC (Authentic Chained Data Containers). The chaining feature in ACDCs is far more powerful, far safer, and far easier to maintain than certificates. It is also capable of modeling delegated relationships such as the one between Organization X and Call Center Y. This eliminates the gap between accountable party and caller, while maintaining perfect transparency. Y can sign a call on behalf of X, using a number allocated to X, and using X's brand, without impersonating X, and they can prove to any OSP or any other party, in any jurisdiction, that they have the right to do so. The evidence that Y cites can be easily built and maintained by X and Y and does not need to be published in a central registry. Further, when evidence about X and Y is filtered through suballocations or crosses jurisdictional boundaries, it can be reused, or linked and transformed, without altering its robustness or efficiency. ACDCs are lossless with respect to identity relationships, whereas formats like W3C Verifiable Credentials and SD-JWTs are lossy; this means ACDCs can be used to generate derivative forms of evidence for subsets of an ecosystem that prefer to consume evidence differnetly. The result is that no third party has to guess who's accountable; the accountable party is transparently and provably accountable, period.

* Although VVP can work inside the governance frameworks such as SHAKEN, it dramatically upgrades at least one key ingredient: the foundational vetting mechanism. The ACDC format used by VVP is also the format used by the Verifiable Legal Entity Identifier (vLEI) standardized in ISO 17442. vLEIs use a KYC approach established by the Regulatory Oversight Committee of the G20, based on the LEI that's globally required in cross-border banking. This means the basis of vLEI trust is already adopted, and it is not limited to any particular jurisdiction or to telecom contexts. VVP offers two-way, easy bridges between identity in phone calls and identity in financial, legal, technical, logistics, regulatory, web, email, and social media contexts.  

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Overview


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
