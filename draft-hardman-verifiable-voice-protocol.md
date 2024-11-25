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
  RFC4648:
  RFC8032:
  RFC8225:
  FIPS186-4:
    target: https://doi.org/10.6028/NIST.FIPS.186-4
    title: Digital Signature Standard (DSS)
    author:
      org: National Institute of Standards and Technology (NIST)
    date: July 2013
  TOIP-CESR:
    target: https://trustoverip.github.io/tswg-cesr-specification/
    title: "Composable Event Streaming Representation (CESR)"
    author:
      org: Trust Over IP Foundation
    date: 7 Nov 2023
  TOIP-KERI:
    target: https://trustoverip.github.io/tswg-keri-specification/
    title: "Key Event Receipt Infrastructure (KERI)"
    author:
      org: Trust Over IP Foundation
    date: 5 Jan 2024
  TOIP-ACDC:
    target: https://trustoverip.github.io/tswg-acdc-specification/
    title: "Authentic Chained Data Containers (ACDC)"
    author:
      org: Trust Over IP Foundation
    date: 6 Nov 2023
  ISO-17442-1:
    target: https://www.iso.org/standard/78829.html
    title: "Financial services – Legal entity identifier (LEI) – Part 1: Assignment"
    author:
      org: International Organization for Standardization
    date: 2020
  ISO-17442-3:
    target: https://www.iso.org/standard/85628.html
    title: "inancial services – Legal entity identifier (LEI) – Part 3: Verifiable LEIs (vLEIs)"
    author:
      org: International Organization for Standardization
    date: 2024

informative:
  RFC3986:
  RFC7519:
  RFC8588:
  I-D.ietf-stir-passport-rcd:
  I-D.ietf-oauth-selective-disclosure-jwt:
  I-D.ietf-sipcore-callinfo-rcd:
  ATIS-1000074:
    target: https://atis.org/resources/signature-based-handling-of-asserted-information-using-tokens-shaken-atis-1000074-e/
    title: "Signature-Based Handling of Asserted Information Using toKENs (SHAKEN)"
    author:
      org: Alliance for Telecommunications Industry Solutions
    date: Feb 2019
  W3C.REC-did-core-20220719:
  W3C.REC-vc-data-model-20220303:

--- abstract

Verifiable Voice Protocol (VVP) proves the identity and authorization of parties who are accountable for telephone calls, eliminating trust gaps that scammers exploit. VVP aims at roughly the same target as STIR {{I-D.ietf-stir-passport-rcd}}, SHAKEN {{ATIS-1000074}}, RCD {{I-D.ietf-sipcore-callinfo-rcd}}, and related technologies. Like these approaches, it binds strong cryptographic evidence to headers in the SIP {{RFC3261}} INVITE that initiates a phone call, and allows this evidence to be verified downstream. However, VVP builds from different technical and governance assumptions, with different evidence. This foundation makes VVP simpler, more decentralized and cross-jurisdictional, cheaper to deploy and maintain, more private, more scalable, and higher assurance than alternatives. VVP can be used by itself, or as a lightweight supplement to bridge gaps in these alternatives.

--- middle

# Introduction

When we get phone calls, we want to know who's calling, and why. If we could answer such questions with confidence, we would make reasonable judgments about whether to answer, based on the relationship and reputation context.

Unfortunately, providing quality answers is much harder than it seems. Scammers love to impersonate, and they succeed more often than we'd like.

Regulators are mandating increased transparency to address this risk. This is wise, and it can be effective. However, it introduces some heavy burdens. Reliably vetting the identity of every possible caller is an ambitious task. Maintaining a comprehensive, up-to-date graph of certificates and cryptographic keys to attest vetted identities is harder. Operating a central registry of such identities and evidence, especially as phone calls increasingly cross jurisdictional boundaries, is harder still. Regulatory constraints on data movement, shifting technical standards, variety in PSTNs and system interconnects, and many other factors further complexify.

Yet the inherent limit in today's transparency efforts is even more fundamental: there exists a pernicious gap between the party accountable for a phone call, and the party that provides evidence about who's calling. A decade ago, the TSP told us who's calling in a probabilistic way, based on unreliable metadata about a call's source. More recently, approaches like STIR/SHAKEN move the evidence source closer to the origin, and add tamper protections. However, there is still a gap: the signer of a SHAKEN passport is the OSP, not the caller, and the OSP may not know what's happening in this gap. A high percentage of STIR/SHAKEN traffic does not receive an A attestation, and even the traffic that does receive it may receive it on imperfect evidence.

In theory, the remaining gap could be eliminated by requiring a perfect authentication and authorization between callers and OSPs, for every phone call -- simply refuse to connect calls that do not match an ideal profile for trust. However, this is impractical for many legitimate business reasons. To cite one obvious example, Organization X might hire Call Center Y to call on their behalf, using X's reputation/brand and X's phone number. In such a situation, X is the relevant answer to the "who's calling" question, but Y is the originator of the call -- and there's no good way to guarantee that everyone knows the existence and status of that relationship. In addition, the market manifests a demand for wholesalers/aggregators of phone numbers, and is characterized by complex system layers and interconnects, that further challenge transparency.

VVP solves these problems by applying two crucial innovations.

## Evidence format
VVP uses an evidence format called Authentic Chained Data Container (ACDC) -- {{TOIP-ACDC}}. The chaining feature in ACDCs is safer, more powerful, and easier to maintain than the one in X509 certificates {{RFC5280}}. It is also capable of modeling nuanced delegated relationships such as the one between Organization X and Call Center Y. This eliminates the gap between accountable party and caller, while maintaining perfect transparency. Given X's formal approval, Y can sign a call on behalf of X, using a number allocated to X, and using X's brand, without impersonating X, and they can prove to any OSP or any other party, in any jurisdiction, that they have the right to do so. The evidence that Y cites can be built and maintained by X and Y, and does not need to be published in a central registry. Further, when such evidence is filtered through suballocations or crosses jurisdictional boundaries, it can be reused, or linked and transformed, without altering its robustness or efficiency. ACDCs verify data back to a root through arbitrarily long and complex chains of issuers, whereas formats like W3C Verifiable Credentials {{W3C.REC-vc-data-model-20220303}} and SD-JWTs {{I-D.ietf-oauth-selective-disclosure-jwt}} require direct trust in the proximate issuer. This means ACDCs are lossless in delegation, not lossy, and flexible, not fragile. The result is that no third party has to guess who's accountable; the accountable party is transparently and provably accountable, period. Notwithstanding this transparency, ACDCs support a form of pseudonymity and graduated disclosure that satisfies vital privacy and data processing constraints.

## Vetting refinements
Although VVP can work inside governance frameworks such as SHAKEN {{ATIS-1000074}}, it dramatically upgrades at least one key ingredient: the foundational vetting mechanism. The ACDC format used by VVP is also the format used by the Verifiable Legal Entity Identifier (vLEI) standardized in {{ISO-17442-3}}. vLEIs use a KYC approach established by the Regulatory Oversight Committee of the G20, based on the LEI {{ISO-17442-1}} that's globally required in cross-border banking. This means the basis of vLEI trust is already adopted, and it is not limited to any particular jurisdiction or to telecom. VVP offers two-way, easy bridges between identity in phone calls and identity in financial, legal, technical, logistic, regulatory, web, email, and social media contexts.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Overview

Verifiable Voice Protocol depends on three interrelated activities with evidence:

* Curating
* Citing
* Verifying

The second and third items in this list, citing and verifying evidence. occur in realtime during phone calls. They are the heart of VVP. However, curated evidence must exist before it can be cited; curating is an ongoing, parallel activity as phone calls occur; and citing and verifying must react properly to changes in evidence. Further, some existing trust gaps occur precisely because too few requirements are imposed on evidence. Therefore, this specification also includes normative statements about the nature of evidence and the processes that create and maintain it.

Before we explain the evidence, however, we must define some roles.

## Roles

### Allocation Holder, TNU
An Allocation Holder (AH) is a party that controls how a telephone number is used, in the eyes of a regulator. Typically, range holders are AHs, and so are the direct telephone number users (TNUs). TNUs are customers of range holders -- enterprises and consumers. Range holders are service providers; although they are AHs, they may not be TNUs for all the phone numbers in their allocated ranges. It is possible for an ecosystem to include other parties as AHs (e.g., wholesalers, aggregators). However, some regulators dislike this outcome, and prefer that such parties broker allocations without actually holding the allocations as intermediaries.

### Terminating Party, Terminating Service Provider
For a given phone call, the Terminating Party (TP) is the party that receives the call. These can be individual consumers or organizations. The direct service provider of the TP is the Terminating Service Provider (TSP).

### Originating Party, Originating Service Provider
The Originating Party (OP) is the caller. The direct service provider of the OP is the Originating Service Provider (OSP). For a given phone call, there may be many layers, boundaries, and transitions between OSP and TSP.

### Accountable Party
Accountable Parties (AP) are TNUs -- organizations or individuals who hold the right to use a telephone number (RTU) in the eyes of a regulator. APs MAY be OPs, but this relationship does not always hold. For example, a business can hire a call center, and delegate to the call center the right to use its phone number. In such a case, the business is the AP, but the call center is the OP.

### Verifiers
Verifiers are parties that want to know who's calling, and why, and that evaluate the answers to these questions by examining formal evidence. TPs, TSPs, OSPs, government regulators, law enforcement doing lawful intercept, auditors, and even APs or OPs can be verifiers. Each may need to see different views of the evidence about a particular phone call, and it may be impossible to comply with various regulations unless these views are kept distinct -- yet each wants similar and compatible assurance.

The verifier role in VVP does not just check the validity of cryptographic evidence; it also considers how that evidence matches business rules and external conditions. For example, a verfier may do more than decide whether Call Center Y has the right to make calls on behalf of Organization X; it may decide whether Y has this right at the particular time of day when a call occurs, or in the jurisdiction of a particular TP, given the business purpose asserted in a particular call.

## Evidence

### General categories

The term "credential" has a fuzzy meaning in casual conversation. However, understanding how evidence is built from credentials in VVP requires considerably more precision. We will start from lower-level concepts.

#### SAID
A self-addressing identifier (SAID) is the hash of a JSON object, encoded in CESR format {{TOIP-CESR}}. The raw bytes from the digest function are left-padded to the nearest 24-bit boundary, transformed to base64url {{RFC4648}}, and prefixed with a character that tells which digest function was used.

An example of a SAID is `E81Wmjyz5nXMCYrQqWyRLAYeKNQvYLYqMLYv_qm-qP7a`, and a regex that matches all valid SAIDs is: `[EFGHI][-_\w]{43}|0[DEFG][-_\w]{86}`. The `E` prefix in this example indicates that it is a Blake3-256 hash.

SAIDs are evidence that hashed data has not changed. They can also function like a reference, hyperlink, or placeholder for the data that was hashed to produce them, though they are more similar to URNs than to URLs {{RFC3986}}, since they contain no location information.

#### Signature
A digital signature over arbitrary data D constitutes evidence that the signer processed D with a signing function that took D and the signer's private key as inputs: `signature = sign(D, privkey)`. The evidence can be verified by checking that the signature is bound to D and the public key of the signer: `valid = verify(signature, D, pubkey)`. Assuming that the signer has not lost unique control of the private key, and that cryptography is appropriately strong, we are justified in the belief that the signer must have taken deliberate action that required seeing an unmodified D in its entirety.

The assumption that a signer has control over their private keys may often be true at the time a signature is created. However, after key compromise, an attacker can create and sign evidence that purports to come from an earlier time period, unless signatures are anchored to a data source that detects anachronisms. Lack of attention to this detail undermines the security of many credential schemes. VVP explicitly addresses this concern. 

#### AID
An autonomic identifier (AID) is a short string that can be resolved to one or more cryptographic keys at a specific version of their key state. AIDs are like W3C DIDs {{W3C.REC-did-core-20220719}}}, and can be transformed into DIDs.

An example of an AID is `EMCYrQqWyRLAYqMLYv_qm-qP7eKN81Wmjyz5nXQvYLYa`. AIDs are created by calculating the hash of the identifier's inception event data structure; they therefore match the same regex as SAIDs in general.

AIDs possess several security properties that are not guaranteed by DIDs in general (self-certification, support for prerotation and multisig, support for witnesses, cooperative delegation, and so forth). Using cryptographic keys, a party can prove it is the controller of an AID by creating digital signatures.

#### Signatures over SAIDs
Since neither a SAID value nor the data it hashes can be changed without breaking the correspondence between them, signing over a SAID is equivalent, in how it commits the signer to content and provides tamper evidence, to signing over the data that the SAID hashes. Since SAIDs can function as placeholders for JSON objects, a SAID can function as a placeholder for a JSON object in a larger data structure. And since SAIDs can function as a reference without making a claim about location, it is possible to combine these properties to achieve some indirections in evidence that are crucial in privacy and regulatory compliance. This is explored in greater detail below.

VVP uses SAIDs and digital signatures as primitive forms of evidence.

#### X509 certificates
VVP does not depend on X509 certificates {{RFC5280}} for any of its evidence. (However, if deployed in a hybrid mode, it MAY be used beside alternative mechanisms that are certificate-based. In such cases, self-signed certificates that never expire might suffice to tick certificate boxes, while drastically simplifying the burden of maintaining accurate, unexpired, unrevoked views of authorizations and reflecting that knowledge in certificates. This is because deep authorization analysis flows through VVP's more rich and flexible evidence chain.)

#### Passport
VVP emits a STIR PASSporT {{RFC8225}}. Conceptually, it is similar to a SHAKEN passport {{RFC8588}}. It MUST be secured by an EdDSA digital signature {{RFC8032}}. Instead of including granular fields in the claims of its JWT, it cites a rich data graph of evidence by referencing the SAID of that data graph. This indirection and its implications are discussed below.

```                                                                    
     SHAKEN PASSporT                         VVP PASSporT              
┌───────────────────────┐             ┌───────────────────────┐        
│ protected             │             │ protected             │        
│   kid: pubkey of OSP ─┼───┐         │   kid: AID of caller ─┼───┐    
│ payload               │   │         │ payload               │   │    
│   iat                 │   │         │   iat                 │   ▼    
│   orig                │   │         │   orig                │ TEL+KEL
│   dest                │   │         │   dest                │        
│   attest              │   │         │   evd (SAID = ref to)─┼───┐    
│   ...more claims      │   │         │ signature             │   │    
│ signature             │   │         └───────────────────────┘   │    
└───────────────────────┘   │                                     │    
                            │                                     │    
    pubkey in cert ◄────────┘          data graph of evidence ◄───┘
```
Figure 1: SHAKEN PASSporT compared to VVP PASSporT

#### ACDCs
Besides digital signatures and SAIDs, and the ephemeral PASSporT, VVP's long-lasting evidence uses the ACDC format {{TOIP-ACDC}}. This is normalized, serialized data with an associated digital signature. Unlike X509 certificates, ACDCs are bound directly to AIDs, not to public keys. This has a radical effect on the lifecycle of evidence, because keys can be rotated without invalidating ACDCs. Unlike X509 certificates, JWTs {{RFC7519}}, and W3C Verifiable Credentials {{W3C.REC-vc-data-model-20220303}}, signatures over ACDC data are not *contained* inside the ACDC; rather, they are *referenced by* the ACDC and *anchored in* a verifiable data structure called a Key Event Log or KEL {{TOIP-KERI}}.

The KEL is somewhat like a blockchain. However, it specific to one AID only (the AID of the issuer of the ACDC) and thus is not centralized. The KELs for two different AIDs need not (and typically do not) share any common storage or governance, and do not require coordination or administration. KELs thus suffer none of the performance and governance problems of blockchains, and incur none of blockchain's difficulties with regulatory requirements like data locality or right to be forgotten.

ACDCs can be freely converted between text and binary representations, and either type of representation can also be compacted or expanded to support nuanced disclosure goals. An ACDC is also uniquely identified by its SAID, which means that a SAID can take the place of a full ACDC in certain data structures and processes. None of these transformations invalidate the associated digital signatures, which means that any variant of a given ACDC is equivalently verifiable.

The revocation of a given ACDC can be detected via the witnesses declared in its issuer's KEL. Discovering, detecting, and reacting to such events can be very efficient. Any number of aggregated views can be built on demand, for any subset of an ecosystem's evidence, by any party. This requires no special authority or access, and does not create a central registry as a source of truth, since such views can be served by untrusted parties. Further, different views of the evidence can contain or elide different fields of the evidence data, to address privacy, regulatory, and legal requirements.

ACDCs are capable of holding more generic categories of data than just credentials.

#### Verifiable data
Cryptographically verifiable data (CVD) is data that's associated with a digital signature and a claim about who signed it. When CVD is an assertion, we make the additional assumption that the signer intends whatever the data asserts, since they took an affirmative action to create non-repudiable evidence that they processed it. CVD can be embodied in many formats, but in VVP, all instances of CVD are ACDCs.

#### Credential
A Credential is a special kind of CVD that asserts entitlements for its legitimate bearer -- *and only its bearer*. CVD that says Organization X exists with a particular ID number in government registers, and with a particular legal name, is not necessarily a credential. In order to be a credential, it would have to be associated with an assertion that its legitimate bearer -- *and only its bearer* -- is entitled to use the identity of Organization X. If signed data merely enumerates properties without conferring privileges on a specific party, it is just CVD. Many security gaps in existing solutions arise from conflating CVD and credentials.

#### Bearer token
VVP never uses bearer tokens, but we define them here to provide a contrast. A bearer token is a credential that satisfies binding requirements by a trivial test of possession -- like a movie ticket, the first party that presents the artifact to a verifier gets the privilege. Since Bearer Credentials can be stolen, this is risky. JWTs, session cookies, and other artifacts in familiar identity technologies are often bearer tokens, even if they carry digital signatures. Although they can be protected to some degree by expiration dates and secure channels, these protections are imperfect. For example, TLS channels can be terminated and recreated at multiple places between call origination and delivery.

#### Targeted credential
A targeted credential is CVD that identifies an issuee as the bearer, and that requires the issuee to prove their identity cryptographically (e.g., to produce a proper digital signature) in order to claim the associated privilege. All credentials in VVP are targeted credentials.

#### Justifying link
A Justifying Link (JL) is a reference, inside of one CVD, to another CVD that justifies what the first CVD is asserting. JLs can be SAIDs that identify other ACDCs.

### Specific evidence

#### Message-specific signature
Each voice call begins with a SIP INVITE, and in VVP, each SIP INVITE MUST contain a message-specific signature (MSS) from the OP. This signature MUST be the result of running a signing function over input data that consists of the following metadata about a call: the source phone number, the destination phone number, an identifier for the AP, a timestamp, and a reference to Accountable Party Evidence (APE).

APE consists of several credentials, detailed below. It MUST include a vetting credential for the AP. If the source telephone number is allocated to the AP (which is true unless a proxy is the OP and uses their own telephone number), it MUST include a TNAlloc credential for the AP. If the AP intends to contextualize the call with a brand, it MUST include a brand credential for the AP. If no brand credential is present, verifiers MUST NOT impute a brand to the caller on the basis of any VVP guarantees. If there is a nuanced relationship between the AP as a legal entity and the AP in some limited manifestation, the APE MUST also include a delegation credential that nuances the relationship. For example, this could distinguish between Acme Corporation, and software operated by Acme's IT department for the express purpose of signing voice traffic. The former has a vetting credential and legal accountability, and can act as the company in all contexts; the latter can only sign voice calls on Acme's behalf.

If the OP and the AP are the same party, the digital signature and the relationship between AIDs in issuer and issuee roles in the evidence MUST bind the AP to the APE. Otherwise, the input data for the signature MUST also include an additional AID for the OP, plus a reference to Delegation Evidence (DE). The DE MUST include a vetting credential for the OP. If the OP is using a phone number allocated to the AP, the DE also MUST include a TNAlloc credential issued by the AP to the OP, delegating the right to use the AP's phone number. If the APE includes a brand credential, then the DE MUST also include a brand proxy credential, proving that the OP not only can use the AP's allocated telephone number, but has AP's permission to project the AP's brand while doing so.

(TODO: diagram)

It is crucial that this message-specific signature come from the OP, not the OSP or any other party. The OP can generate this signature in its on-prem or cloud PBX, using keys that it controls. It is also crucial that the distinction between OP and AP be transparent, with the relationship proved by strong evidence that the AP can create or revoke easily, in a self-service manner.

(TODO: finish Verifiers that are capable of understanding delegations It is also vital that verification algorithms This allows  software, is crucial, and is different from alternative approaches. The fact that the the , that closes the attestation gap in existing approaches. Closing this gap is only practical because of the chaining and disclosure features of ACDCs.)

#### Vetting credential
A vetting credential is a targeted credential that enumerates the formal and legal attributes of a unique legal entity. It MUST include a legal identifier that makes the entity unique in its home jurisdiction, and it MUST include an AID for the legal entity. This AID is unique globally. It is called a vetting credential because it MUST be issued according to a documented vetting process that offers formal assurance that is is only issued with accurate information, and only to the AP it describes. A vetting credential confers the privilege of acting with the associated legal identity if and only if the bearer can prove their identity as issuee via a digital signature.

A vetting credential MUST include a JL to a credential that qualifies the issuer as a party trusted to do vetting. This linked credential that qualifies the issuer of the vetting credential MAY contain a JL that qualifies its own issuer, and such JLs MAY be repeated through as many layers as desired. In VVP, the reference type of a vetting credential is an LE vLEI. This implies both a schema and a governance framework. Other vetting credential types are possible, but they MUST be true credentials that meet the normative requirements here. They MUST NOT be bearer tokens.

(TODO: sample vetting credential)

#### Brand credential
A brand credential is a targeted credential that enumerates brand properties such as a brand name and a logo. It MUST be issued to an AP as a legal entity, but it does not enumerate the formal and legal attributes of the AP; rather, it enumerates properties that would be meaningful to a TP who's deciding whether to take a phone call. It confers on its issue the right to use the described brand.

(TODO: sample brand credential)

This credential MUST be issued according to a documented brand research process that offers formal assurance that it is only issued with accurate information, and only to an AP that has the right to use the described brand. A single AP MAY have multiple brand credentials (e.g., Coca Cola Holdings Germany, Gmbh holds a brand credential for Coke and for Sprite), and rights to use the same brand MAY be conferred to on multiple APs (Coca Cola Holdings Germany, Gmbh and Coca Cola Canada, Ltd may both possess brand credentials for Sprite). A brand credential MUST contain a JL to a vetting credential, that shows that the right to use the brand was evaluated only after using a vetting credential to prove the identity of the issuee.

#### TNAlloc credential
A TNAlloc credential is a targeted credential that confers on its issuee the right to control how one or more telephone numbers are used. Regulators issue TNAlloc credentials to range holders, who issue them to TNUs. TNUs often play the AP role in VVP. If an AP delegates RTU to a proxy (e.g., an employee or call center), the AP MUST also issue a TNAlloc credential to the proxy, to confer the RTU. With each successive reallocation, the set of numbers in the new TNAlloc credential gets smaller. Except for TNAlloc credentials issued by regulators, all TNAlloc credentials MUST contain a JL to a parent TNAlloc credential, having a bigger set of numbers that includes those in the current credential. This JL in a child credential documents the fact that the child's issuer possessed an equal or broader RTU, from which the subset RTU in child credential derives.

#### Brand proxy credential
A brand proxy credential confers on an OP the right to project the brand of an AP when making phone calls, subject to a carefully selected set of constraints. This is different from the simple RTU conferred by TNAlloc. Without a brand proxy credential, a call center could make calls on behalf of an AP, using the AP's allocated phone number, but would be forced to do so under its own name or brand, because it lacks evidence that the AP intended anything different. If an AP intends for phone calls to be made by a proxy, and wants the proxy to project the AP's brand, the AP MUST issue this credential.

## Curating
The evidence that's available in today's telecom ecosystems resembles some of the evidence described above, in concept. However, it has gaps, and its format is fragile. Furthermore, if it is organized for discovery, it is typically assembled into large, centralized registries at a regional or national level. These registries become a trusted third party, which defeats some of the purpose of creating decentralized and independently verifiable evidence in the first place. Sharing such evidence across jurisdiction boundaries requires regulatory compatibility and bilateral agreements. Sharing at scale is impractical at best, if not downright illegal.

How evidence is issued, propagated, and reference is therefore an important concern for this specification.

### Vetting
The job of vetting legal entities (which includes APs, but also OPs) and issuing vetting credentials is performed by a Legal Entity Vetter (LEV). VVP places few requirements on such vetters, other than the ones already listed for vetting credentials themselves. Vetting credentials do not need to expire, and in fact ACDCs and AIDs facilitate much longer lifecycles than certificates; prophylactic key rotation is recommended but creates no reason to rotate evidence. However, the LEV MUST agree to revoke vetting credentials in a timely manner if the legal status of an entity changes, or if data in a vetting credential becomes invalid.

### Brand information
The job of analyzing the brand assets of a legal entity and issuing brand credentials is performed by a Brand Vetter (BV). A BV MAY be an LEV, and MAY issue both types of credentials after a composite analysis. However, the credentials themselves MUST NOT use a combined schema, the credentials MUST have independent lifecycles, and the assurances associated with each credential type MUST remain independent. A BV MUST verify the canonical properties of a brand, but it MUST do more than this: it MUST issue the brand credential to the AID of an issuee that is also the issuee of a vetting credential that already exists, and it MUST verify that the legal entity in the vetting credential has a right to use the brand in question. This link MUST NOT be based on mere weak evidence such as an observation that the legal entity's name and the brand name have some or all words in common, or the fact that a single person requested both credentials. Further, the BV MUST agree to revoke brand credentials in a timely manner if the associated vetting credential is revoked, if the legal entity's right to use the brand changes, or if characteristics of the brand evolve.

### TN allocations
Regulators issue TNAlloc credentials to range holders, and range holders issue them to TNUs, and TNUs MAY issue them to a delegate. If aggregators or other intemediaries hold an RTU in the eyes of a regulator, then intermediate TNAlloc credentials MUST be created to track that RTU as part of the chain. On the other hand, if TNUs acquire telephone numbers through aggregators, but regulators do not consider aggregators to hold allocations, then aggregators MUST work with range holders to assure that the appropriate TNAlloc credentials are issued to the TNUs.

### Brand proxy
APs issue brand proxy credentials to OPs.

Vetting and brand credentials may require large databases of metadata about organizations and brands, but how such systems work is out of scope. Once credentials are issued, they are an independent source of truth. No party has to return to the operators of such databases to validate anything.

Issuance and revocation of all other credentials is self-service at every level, and does not require any large, centralized system.

(TODO: finish. The AIDs of issuers SHOULD be linked to witnesses...)

## Citing

(TODO: OP calls PBX to initiate call, provides message-specific signature. Provide details and example)

## Verifying

(TODO: Verifier has a cache that was populated by monitoring the witnesses of known issuer AIDs. (If a new issuer is encountered, it can look up the witnesses and begin monitoring.) Etc.

Also discuss deploying in a hybrid mode to extend S/S.)

# Security Considerations

TODO Security

# Privacy

TODO Discuss graduated disclosure, compact versus expanded ACDCs, etc.


# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.

