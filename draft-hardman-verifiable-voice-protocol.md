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
  RFC3261:
  RFC5280:
  ISO.17442-1:
  ISO.17442-3:
  TOIP.CESR:
  TOIP.ACDC:
  TOIP.KERI:

informative:
  I-D.ietf-stir-passport-rcd:
  I-D.ietf-oauth-selective-disclosure-jwt:
  I-D.ietf-sipcore-callinfo-spam:
  ATIS.1000074:
  W3C.REC-did-core-20220719:
  W3C.REC-vc-data-model-20220303:  


--- abstract

Verifiable Voice Protocol (VVP) proves the identity and authorization of parties who are accountable for telephone calls, eliminating trust gaps that scammers exploit. VVP aims at roughly the same target as STIR ({{I-D.ietf-stir-passport-rcd}}), SHAKEN ({{ATIS.1000074}}), RCD ({{I-D.ietf-stir-passport-rcd}}), and related technologies. Like these approaches, it uses binds strong cryptographic evidence to headers in the SIP {{RFC3261}} INVITE that initiates a phone call, and allows this evidence to be verified downstream. However, VVP builds from different technical and governance assumptions. This foundation makes VVP simpler, more decentralized and cross-jurisdictional, cheaper to deploy and maintain, more private, more scalable, and higher assurance than alternatives. It can be used by itself, or as an inexpensive supplement to bridge gaps in existing approaches.

--- middle

# Introduction

When we get phone calls, we want to know who's calling, and why. If we could answer such questions with confidence, we would make reasonable judgments about whether to answer, based on the relationship and reputation context. 

Unfortunately, providing quality answers is much harder than it seems. Scammers love to impersonate, and they succeed more often than we'd like.

Regulators are mandating increased transparency to address this risk. This is wise, and it can be effective. However, it introduces some heavy burdens. Reliably vetting the identity of every possible caller is an ambitious task. Maintaining a comprehensive, up-to-date graph of certificates and cryptographic keys to attest vetted identities is harder. Operating a central registry of such identities and evidence, especially as phone calls increasingly cross jurisdictional boundaries, is harder still. Regulatory constraints on data movement, shifting technical standards, variety in PSTNs and system interconnects, and many other factors further complexify.

However, the inherent limit in today's transparency efforts is more fundamental: there exists a pernicious gap between the party accountable for a phone call, and the party who answers the "who's calling?" question. A decade ago, the TSP told us who's calling in a probabilistic way, based on unreliable metadata about a call's source. More recently, approaches like STIR/SHAKEN move the answer closer to the origin, and add tamper protections. However, there is still a gap: the signer of a SHAKEN passport is the OSP, not the caller, and the OSP may not know what's happening in this gap.

In theory, the remaining gap could be eliminated by requiring a perfect authentication and authorization between callers and OSPs, for every phone call. However, this is impractical for many legitimate business reasons. To cite one obvious example, Organization X might hire Call Center Y to call on their behalf, using X's reputation/brand and X's phone number. In such a situation, X is the relevant answer to the "who's calling" question, but Y is the originator of the call -- and there's no good way to guarantee that everyone knows the existence and status of that relationship. In addition, the market manifests a demand for wholesalers/aggregators of phone numbers, and is characterized by complex system layers and interconnects, that further challenge transparency.

VVP solves these problems by applying two crucial innovations:

* It uses an evidence format called Authentic Chained Data Container (ACDC) -- {{TOIP.ACDC}}. The chaining feature in ACDCs is safer, more powerful, and easier to maintain than the one in X509 certificates ({{RFC5280}}). It is also capable of modeling delegated relationships such as the one between Organization X and Call Center Y. This eliminates the gap between accountable party and caller, while maintaining perfect transparency. Y can sign a call on behalf of X, using a number allocated to X, and using X's brand, without impersonating X, and they can prove to any OSP or any other party, in any jurisdiction, that they have the right to do so. The evidence that Y cites can be easily built and maintained by X and Y and does not need to be published in a central registry. Further, when evidence about X and Y is filtered through suballocations or crosses jurisdictional boundaries, it can be reused, or linked and transformed, without altering its robustness or efficiency. ACDCs are lossless with respect to identity relationships, whereas formats like W3C Verifiable Credentials and SD-JWTs are lossy; this means ACDCs can be used to generate derivative forms of evidence for subsets of an ecosystem that prefer to consume evidence differnetly. The result is that no third party has to guess who's accountable; the accountable party is transparently and provably accountable, period. Notwithstanding this transparency, ACDCs support a form of pseudonymity and graduated disclosure that satisfies vital privacy and data processing constraints.

* Although VVP can work inside governance frameworks such as SHAKEN {{ATIS.1000074}}, it dramatically upgrades at least one key ingredient: the foundational vetting mechanism. The ACDC format used by VVP is also the format used by the Verifiable Legal Entity Identifier (vLEI) standardized in {{ISO.17442-3}}. vLEIs use a KYC approach established by the Regulatory Oversight Committee of the G20, based on the LEI ({{ISO.17442-1}}) that's globally required in cross-border banking. This means the basis of vLEI trust is already adopted, and it is not limited to any particular jurisdiction or to telecom. VVP offers two-way, easy bridges between identity in phone calls and identity in financial, legal, technical, logistic, regulatory, web, email, and social media contexts.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Overview

Verifiable Voice Protocol depends on three interrelated activities:

* Curating evidence
* Citing evidence
* Checking evidence

Citing and checking evidence occur in realtime during actual phone calls, and are therefore the heart of VVP. However, curated evidence must exist before it can be cited; curating is an ongoing, parallel activity as phone calls occur; and citing and checking must react properly to changes in evidence. Further, existing approaches have trust gaps precisely because they impose too few requirements on the evidence that backs their ecosystems. Therefore, the protocol specification includes normative statements about the nature of evidence and the processes that create and maintain it.

Before we explain the evidence, however, we must define some roles.

## Roles

For a given phone call, the Terminating Party (TP) is the party that receives the call. The direct service provider of the TP is the Terminating Service Provider (TSP). 

The Originating Party (OP) is the caller. The direct service provider of the OP is the Originating Service Provider (OSP). For a given phone call, there may be many layers, boundaries, and transitions between OSP and TSP.

Accountable Parties (AP) are organizations or individuals who hold the right to use a telephone number (RTU) in the eyes of a regulator. APs MAY be OPs, but this relationship does not always hold. A business can hire a call center, and delegate to the call center the right to use its phone number. In such a case, the business is the AP, but the call center is the OP.

Verifiers (V) are parties that want to know who's calling, and why, and that evaluate the answers to these questions by examining formal evidence. TPs, TSPs, OSPs, government regulators, law enforcement doing lawful intercept, auditors, and even APs or OPs can be verifiers. Each may need to see different views of the evidence about a particular phone call, and it may be impossible to comply with various regulations unless these views are kept distinct -- yet each wants similar and compatible assurance. The verifier role in VVP does not just check the validity of cryptographic evidence; it also considers how that evidence matches business rules. For example, a verfier may do more than decide whether Call Center Y has the right to make calls on behalf of Organization X; it may decide whether Y has this right at a particular time of day, when calling into a particular jurisdiction, for a particular business purpose.

## Evidence

The term "credential" has a fuzzy meaning in casual conversation. However, understanding how evidence is built from credentials in VVP requires considerably more precision. Let's start from lower-level concepts.

A digital signature over arbitrary data D constitutes evidence that the signer processed D with a signing function that took D and the signer's private key as inputs. The evidence can be verified by checking that the signature is associated with the public key of the signer. Assuming that the signer has not lost unique control of the private key, and that cryptography is appropriately strong, we are justified in the belief that the signer must have taken deliberate action and seen an unmodified D in its entirety.

VVP uses digital signatures as one form of evidence.

Most other evidence in VVP uses the ACDC format. This is a normalized, serialized form of data with an associated digital signature. Signatures over ACDCs are not contained inside the ACDC; rather, they are anchored elsewhere and referenced by the ACDC. ACDCs can be freely converted between text and binary representations, and either type of representation can also be compacted or expanded to support nuanced disclosure goals. None of these transformations invalidate the associated digital signatures, which means that any variant of a given ACDC is equivalently verifiable. We adopt many terms and concepts from the ACDC spec. 

Some formal definitions will help to characterize ACDC-based evidence.

Cryptographically verifiable data (CVD) is data that's associated with a digital signature and a claim about who signed it.

When CVD is an assertion, we make the additional assumption that the signer intends whatever the data asserts, since they took an affirmative action to create non-repudiable evidence that they processed it.

A Credential is a special kind of CVD that asserts the entitlement of its bearer -- *and only its bearer* -- to a privilege. CVD that says Organization X exists with a particular ID number in government registers, and with a particular legal name, is not necessarily a credential. In order to be a credential, it would have to include an assertion that its bearer -- *and only its bearer* -- is entitled to use the identity of Organization X. If signed data merely enumerates properties without conferring privileges on a specific party, it is just CVD.

A Bearer Token is a Credential that satisfies the binding requirements by trivial possession -- like a movie ticket, the first party that presents the artifact to a verifier gets the privilege. Since Bearer Credentials can be stolen, this is risky. Although they can be protected by secure channels and expiration dates, JWTs, session cookies, and other artifacts in mainstream identity technology are typically bearer tokens.

A Targeted Credential is CVD that identifies an Issuee as the bearer, and that requires the Issuee to prove their identity cryptographically (e.g., to produce a proper digital signature) in order to claim the associated privilege.

A Justifying Link (JL) is a reference, inside of one CVD, to another CVD that justifies what the first CVD is asserting.

With this background, we can now say that VVP depends on the following types of evidence:

* A Vetting Credential. This is a targeted credential that enumerates the formal and legal attributes of a unique legal entity. It MUST include a legal identifier that makes the entity unique in its home jurisdiction, and it MUST include a cryptographic identifier that is unique globally. It is called a vetting credential because it MUST be issued according to a documented vetting process that offers formal assurance that is is only issued with accurate information, and only to the AP it describes. A vetting credential confers the privilege of acting with the associated legal identity if and only if the bearer can prove their identity as issuee via a digital signature. A vetting credential MUST include a JL to a credential that qualifies the issuer as a party trusted to do vetting. This linked credential that qualifies the issuer of the vetting credential MAY contain a JL that qualifies its own issuer, and such JLs MAY be repeated through as many layers as desired. In VVP, the reference type of a Vetting Credential is a vLEI. This implies both a schema and a governance framework. Other Vetting Credential types are possible, but they MUST be true credentials that meet the normative requirements here. They MUST NOT be bearer tokens.

* A Brand Credential. This is a targeted credential that enumerates brand properties such as a brand name and a logo. It MUST be issued to an AP as a legal entity, but it does not enumerate the formal and legal attributes of the AP; rather, it enumerates properties that would be meaningful to a TP who's deciding whether to take a phone call. It confers on its issue the right to use the described brand. This credential MUST be issued according to a documented brand research process that offers formal assurance that it is only issued with accurate information, and only to an AP that has the right to use the described brand. A single AP MAY have multiple brand credentials (e.g., Coca Cola Holdings Germany, Gmbh holds a brand credential for Coke and for Sprite), and rights to use the same brand MAY be conferred to on multiple APs (Coca Cola Holdings Germany, Gmbh and Coca Cola Canada, Ltd may both possess brand credentials for Sprite). A brand credential MUST contain a JL to a vetting credential, that shows that the right to use the brand was evaluated only after using a vetting credential to prove the identity of the issuee.

* A TNAlloc Credential. This is a targeted credential that confers on its issuee the right to use one or more telephone numbers. If the issuer is not a regulator, it MUST contain a JL to an TNAlloc credential that justifies the issuer's right to pass a right to use downstream.

Some Not all CVD The signer of a Credential is called an Issuer.

A Bearer Credential confers entitlement through simple possession.
 

## Curation


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.

## References

### Normative References

## References

### Normative References

- **[RFC3261]**  
  Rosenberg, J., Schulzrinne, H., Camarillo, G., Johnston, A., Peterson, J., Sparks, R., Handley, M., and E. Schooler, "SIP: Session Initiation Protocol", RFC 3261, DOI 10.17487/RFC3261, June 2002, <https://www.rfc-editor.org/info/rfc3261>.

- **[RFC5280]**  
  Cooper, D., Santesson, S., Farrell, S., Boeyen, S., Housley, R., and W. Polk, "Internet X.509 Public Key Infrastructure Certificate and Certificate Revocation List (CRL) Profile", RFC 5280, DOI 10.17487/RFC5280, May 2008, <https://www.rfc-editor.org/info/rfc5280>.

- **[I-D.ietf-stir-passport-rcd]**  
  Wendt, C., Peterson, J., and A. Rescorla, "PASSporT Extension for Rich Call Data", Work in Progress, Internet-Draft, draft-ietf-stir-passport-rcd-21, July 2023, <https://datatracker.ietf.org/doc/draft-ietf-stir-passport-rcd/>.

- **[ISO.17442-1]**  
  International Organization for Standardization, "Financial services – Legal entity identifier (LEI) – Part 1: Assignment", ISO 17442-1:2020, 2020.

- **[ISO.17442-3]**  
  International Organization for Standardization, "Financial services – Legal entity identifier (LEI) – Part 3: Verifiable LEIs (vLEIs)", ISO 17442-3:2024, 2024.

- **[TOIP.CESR]**  
  Trust Over IP Foundation, "Composable Event Streaming Representation (CESR)", 7 Nov 2023, <https://trustoverip.github.io/tswg-cesr-specification/>

- **[TOIP.ACDC]**  
  Trust Over IP Foundation, "Authentic Chained Data Containers (ACDC)", 6 Nov 2023, <https://trustoverip.github.io/tswg-acdc-specification/>

- **[TOIP.KERI]**  
  Trust Over IP Foundation, "Key Event Receipt Infrastructure (KERI)", 5 Jan 2024, <https://trustoverip.github.io/tswg-keri-specification/>


### Informative References

- **[W3C.REC-did-core-20220719]**  
  World Wide Web Consortium, "Decentralized Identifiers (DIDs) v1.0", W3C Recommendation, 19 July 2022, <https://www.w3.org/TR/2022/REC-did-core-20220719/>.

- **[W3C.REC-vc-data-model-20220303]**  
  World Wide Web Consortium, "Verifiable Credentials Data Model v1.1", W3C Recommendation, 3 March 2022, <https://www.w3.org/TR/2022/REC-vc-data-model-20220303/>.

- **[ATIS.1000074]**  
  Alliance for Telecommunications Industry Solutions, "Signature-Based Handling of Asserted Information Using toKENs (SHAKEN)", ATIS-1000074, 2020.

- **[I-D.ietf-oauth-selective-disclosure-jwt]**  
  Jones, M., Bradley, J., and N. Sakimura, "Selective Disclosure of Claims in JWTs", Work in Progress, Internet-Draft, draft-ietf-oauth-selective-disclosure-jwt-05, October 2024, <https://datatracker.ietf.org/doc/draft-ietf-oauth-selective-disclosure-jwt/>.

- **[I-D.ietf-sipcore-callinfo-spam]**  
  Johnston, A., "A Mechanism for Indicating Spam in the SIP Call-Info Header Field", Work in Progress, Internet-Draft, draft-ietf-sipcore-callinfo-spam-02, October 2024, <https://datatracker.ietf.org/doc/draft-ietf-sipcore-callinfo-spam/>.
