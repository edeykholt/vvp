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
  github: "dhh1128/vvp"
  latest: "https://dhh1128.github.io/vvp/draft-hardman-verifiable-voice-protocol.html"

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
  RFC8224:
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
  RFC6350:
  RFC7095:
  RCD-DRAFT: I-D.ietf-sipcore-callinfo-rcd
  RCD-PASSPORT: I-D.ietf-stir-passport-rcd
  SD-JWT-DRAFT: I-D.ietf-oauth-selective-disclosure-jwt
  ATIS-1000074:
    target: https://atis.org/resources/signature-based-handling-of-asserted-information-using-tokens-shaken-atis-1000074-e/
    title: "Signature-Based Handling of Asserted Information Using toKENs (SHAKEN)"
    author:
      org: Alliance for Telecommunications Industry Solutions
    date: Feb 2019
  CTIA-BCID:
    target: https://api.ctia.org/wp-content/uploads/2022/11/Branded-Calling-Best-Practices.pdf
    title: "Branded Calling ID Best Practices"
    author:
      org: CTIA
    date: Nov 2022
  W3C-DID: W3C.REC-did-core-20220719
  W3C-VC: W3C.REC-vc-data-model-20220303

--- abstract

Verifiable Voice Protocol (VVP) authenticates and authorizes organizations and individuals that make telephone calls, eliminating trust gaps that scammers exploit. Like STIR, SHAKEN, RCD, BCID, and related technologies, VVP binds strong cryptographic evidence to a SIP INVITE, and verifies this evidence downstream. However, VVP builds from different technical and governance assumptions, with different backing evidence. This unique foundation allows VVP to cross jurisdictional boundaries easily and robustly. It also makes VVP simpler, more decentralized, cheaper to deploy and maintain, more private, more scalable, and higher assurance than alternatives. VVP can plug gaps or build bridges between other approaches. For example, it can justify an A attestation in SHAKEN when a call originates outside a regulated certificate ecosystem. Where other approaches are not deployed, VVP also works well as a standalone mechanism.

--- middle

# Introduction
When we get phone calls, we want to know who's calling, and why. Annoying strangers abuse expectations far too often.

Regulators are wisely mandating countermeasures. This creates heavy burdens. Vetting every possible caller is ambitious. Monitoring all certificates and cryptographic keys is harder. Operating a central registry, especially across jurisdictional boundaries, is harder still. Regulatory constraints on data movement, shifting standards, variety in PSTNs and system interconnects, and many other factors further complexify.

Yet the inherent limit in today's protections is even more fundamental: a pernicious gap between the party accountable for a call, and the party that provides evidence about who's calling. A decade ago, terminating service providers told us who's calling in a probabilistic way, based on unreliable metadata. More recently, approaches like STIR/SHAKEN move evidence closer to the origin, and add tamper detection. However, the gap persists. The signer of a SHAKEN passport is the originating service provider, not the caller, and we have no evidence to close this gap. If the originating service provider lacks reputation (e.g, because it operates outside the regulated jurisdiction), we have few good options. A high percentage of STIR/SHAKEN traffic does not receive an A attestation, and even the traffic that does receive it may receive it on imperfect evidence.

In theory, this gap could be eliminated by perfect authentication and authorization between callers and OSPs, for every phone call: simply refuse to connect calls that do not match an ideal profile for trust. However, this is impractical. Organization X might hire Call Center Y to call on their behalf, using X's reputation/brand and X's phone number. In such a situation, X is the relevant answer to the "who's calling" question, but Y is the originator of the call -- and there's no good way to guarantee that everyone understands that relationship. In addition, the market demands wholesalers/aggregators of phone numbers, and is characterized by complex system layers and interconnects, that further challenge transparent accountability.

VVP solves these problems by applying two crucial innovations.

## Evidence format
VVP uses an evidence format called *authentic chained data container*s (*ACDC*s) -- {{TOIP-ACDC}}. The chaining feature in ACDCs is safer, more powerful, and easier to maintain than the one in X509 certificates {{RFC5280}}. It is also capable of modeling nuanced delegations such as the one between Organization X and Call Center Y. This eliminates the gap between accountable party and caller. Given X's formal approval, Y can sign a call on behalf of X, using a number allocated to X, and using X's brand, without impersonating X, and they can prove to any OSP or any other party, in any jurisdiction, that they have the right to do so.

The evidence that Y cites can be built and maintained by X and Y, doesn't get stale or require periodic reissuance, and doesn't need to be published in a central registry.

Further, when such evidence is filtered through suballocations or crosses jurisdictional boundaries, it can be reused, or linked and transformed, without altering its robustness or efficiency. ACDCs verify data back to a root through arbitrarily long and complex chains of issuers, whereas formats like W3C Verifiable Credentials {{W3C-VC}} and SD-JWTs {{SD-JWT-DRAFT}} require direct trust in the proximate issuer.

The synergies of these properties mean that ACDCs can be permanent, flexible, robust, low-maintenance, and lossless rather than lossy in delegation. In VVP, no third party has to guess who's accountable for a call; the accountable party is transparently and provably accountable, period. (Notwithstanding this transparency, ACDCs support a form of pseudonymity and graduated disclosure that satisfies vital privacy and data processing constraints. See {{<privacy}}.)

## Vetting refinements
Although VVP operates with governance frameworks such as SHAKEN {{ATIS-1000074}}, it allows for a dramatic upgrade of at least one key component: the foundational vetting mechanism. The evidence format used by VVP is also the format used by the Verifiable Legal Entity Identifier (vLEI) standardized in {{ISO-17442-3}}. vLEIs implement a KYC approach established by the Regulatory Oversight Committee of the G20, based on the LEI {{ISO-17442-1}} that's globally required in high-security, high-regulation, cross-border banking.

To be clear, VVP does not *require* that vLEIs be used for vetting. However, by choosing an evidence format that is high-precision and lossless enough to accommodate vLEIs, VVP lets telecom ecosystems opt in, either wholly or partially, to trust bases that are already adopted, and that are not limited to any particular jurisdiction or to the telecom industry. It thus offers two-way, easy bridges between identity in phone calls and identity in financial, legal, technical, logistic, regulatory, web, email, and social media contexts. If VVP chose a weaker evidence format like X509 or SD-JWT or W3C VCs, higher vetting standards in some jurisdictions or use cases would not be an option.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Overview

Fundamentally, VVP requires a caller to assemble a dossier ({{<dossier}}) of stable evidence that proves identity and authorization. This is done once or occasionally, in advance, as a configuration precondition. Then, for each call, an ephemeral STIR-compatible VVP PASSporT ({{<passport}}) is generated that cites the preconfigured dossier. Verifiers check the dossier, including realtime revocation status, to make decisions.

## Roles

The following roles require careful definition. Their meaning in VVP may not match casual usage.

### Allocation Holder
An *allocation holder* (*AH*) controls how a telephone number is used, in the eyes of a regulator. Typically, range holders are AHs, and so are the direct *telephone number users* (*TNUs*). TNUs are customers of range holders -- enterprises and consumers. Range holders are service providers; although they are AHs, they may not be TNUs for all the phone numbers in their allocated ranges.

It is possible for an ecosystem to include other parties as AHs (e.g., wholesalers, aggregators). However, some regulators dislike this outcome, and prefer that such parties broker allocations without actually holding the allocations as intermediaries.

### Terminating Party {#TP}
For a given phone call, the *terminating party* (*TP*) receives the call. These can be individual consumers or organizations. The direct service provider of the TP is the *terminating service provider* (*TSP*).

### Originating Party {#OP}
An *originating party* (*OP*) controls the first *session border controller* (*SBC*) that processes a call, and therefore cites the evidence that authenticates and authorizes it.

It may be tempting to equate the OP with "the caller", but this equivalence lacks nuance and doesn't always hold. In a VVP context, it is more accurate to say that the OP creates a SIP INVITE with explicit, provable authorization from the party accountable for calls on the originating phone number.

It may also be tempting to associate the OP with an organizational identity like "Company X". While this is not wrong, and is in fact used in high-level descriptions in this specification, in its most careful definition, the cryptographic identity of an OP is more narrow. Tt typically corresponds to a single service operated by the IT department within (or at the behest of) Company X, rather than to Company X generically. This narrowness limits cybersecurity risk, because a single service operated by Company X needs far fewer privileges than the company as a whole. Failing to narrow identity appropriately creates vulnerabilities in some alternative approaches. The evidence securing VVP MUST therefore prove a valid relationship between the OP's narrow identity and the broader legal entities that stakeholders naturally conceive.

The direct service provider of an OP is its *originating service provider* (*OSP*). For a given phone call, there may be complexity between the hardware that begins a call and the SBC of the OP -- and there may also be many layers, boundaries, and transitions between OSP and TSP.

### Accountable Party {#AP}
For a given call, the *accountable party* (*AP*) is the organization or individual that has the right to use the source telephone number, according to the associated regulator. When a TP asks, "Who's calling?", it is almost always the AP that they want to identify; thus, the AP is "the caller", as far as the TP concerned. APs MAY operate their own SBCs and therefore be OPs. However, APs may also use a UCaaS provider that makes this relationship indirect. Going further, a business can hire a call center, and delegate to the call center the right to use its phone number. In such a case, the business is the AP, but the call center is the OP that makes calls on its behalf.

### Verifier
A *verifier* is a party that wants to know who's calling, and why, and that evaluates the answers to these questions by examining formal evidence. TPs, TSPs, OSPs, government regulators, law enforcement doing lawful intercept, auditors, and even APs or OPs can be verifiers. Each may need to see different views of the evidence about a particular phone call, and it may be impossible to comply with various regulations unless these views are kept distinct -- yet each wants similar and compatible assurance.

In addition to checking the validity of cryptographic evidence, the verifier role in VVP MAY also consider how that evidence matches business rules and external conditions. For example, a verifier might begin its analysis by deciding whether Call Center Y has the right, in the abstract, to make calls on behalf of Organization X using a given telephone number. However, VVP evidence allows a verifier to go further: it can also consider whether Y is allowed to exercise this right at the particular time of day when a call occurs, or in the jurisdiction of a particular TP, given the business purpose asserted in a particular call.

## Lifecycle
VVP depends on three interrelated activities with evidence:

* Curating
* Citing
* Verifying

Chronologically, evidence must be curated before it can be cited or verified. In addition, some vulnerabilities in existing approaches occur because evidence requirements are too loose. Therefore, understanding the nature of backing evidence, and how that evidence is created and maintained, is a crucial consideration for VVP. This specification includes normative statements about evidence.

However, curating does not occur in realtime during phone calls. Citing and verifying are the heart of VVP, and implementers will probably approach VVP from the standpoint of SIP flows. Therefore, we defer the question of curation. Where not-yet-explained evidence concepts are used, inline references allow easy cross-reference to formal definitions.

# Citing
A call secured by VVP begins when the OP ({{<OP}}) generates a new VVP PASSporT that complies with STIR {{RFC8224}} requirements. The passport is a compact-serialized JWT {{RFC7519}} that appears in an `Identity` header in a SIP INVITE. In its JSON-serialized form, a typical VVP PASSporT might look like this:

~~~ json
{
  "header": {
    "alg": "EdDSA",
    "typ": "JWT",
    "ppt": "passport",
    "kid": "https://agentsrus.net/oobi/EMCYrQqWyRLAYqMLYv_qm-qP7eKN81Wmjyz5nXQvYLYa/agent/EAxBDJkpA0rEjUG8vJrMdZKw8YL63r_7zYUMDrZMf1Wx",
  }
  "payload": {
    "orig": {"tn": ["+17035550001"]},
    "dest": {"tn": ["+15715550000"]},
    "card": ["NICKNAME:Examples-R-Us", "CHATBOT:https://example.com/chatwithus",
      "LOGO;HASH=EK2r6EnDXre2pecTBO8s99j4OtNaaDIhVyr7uGugDhmp;VALUE=URI:https://example.com/logo64x48.png"],
    "call-reason": "schedule next appointment",
    "evd": "https://acme.com/dossier.cesr",
    "origId": "e0ac7b44-1fc3-4794-8edd-34b83c018fe9",
    "iat": 1699840000,
    "exp": 1699840030,
    "jti": "70664125-c88d-49d6-b66f-0510c20fc3a6"
  }
}
~~~

The semantics of the fields are:

* `alg` *(required)* MUST be "EdDSA". Standardizing on one scheme prevents jurisdictions with incompatible or weaker cryptography. The RSA, HMAC, and ES256 algorithms MUST NOT be used. (This choice is motivated by compatibility with the vLEI and its associated ACDC ecosystem, which depends on the Montgomery-to-Edwards transformation.)
* `typ` *(required)* MUST be "JWT".
* `ppt` *(required)* Per STIR, MUST be "passport".
* `kid` *(required)* MUST be the OOBI of an AID {{<aid}} controlled by the OP {{<OP}}. Typically this is NOT the AID that identifies the OP as a legal entity, but rather an AID controlled by software running on or invoked by the SBC operated by the OP. (The AID that identifies the OP as a legal entity may be controlled by a multisig scheme and thus require multiple humans to create a signature. The AID for `kid` MUST be singlesig and, in the common case where it is not the legal entity AID, MUST have a delegate relationship with the legal entity AID, proved through Delegate Evidence {{<DE}}.)
* `orig` *(required)* MUST conform to SHAKEN requirements, with the additional constraint that only one phone number is allowed. Although the containing SIP INVITE may allow multiple originating phone numbers, only one can be tied to evidence evaluated by verifiers.
* `dest` *(required)* MUST conform to SHAKEN requirements.
* `evd` *(required)* MUST be the OOBI of a bespoke ACDC (the dossier, {{<dossier}}) that constitutes a verifiable data graph of all evidence justifying belief in the identity and authorization of the AP, the OP, and any relevant delegations. An OOBI is a special URL that returns IANA content-type `application/json+cesr`. Such a URL can be hosted on any convenient web server, and is analogous to the `x5u` header in X509 contexts. See below for details.
* `origId` *(optional)* Follows SHAKEN semantics.
* `iat` *(required)* Follows standard JWT semantics.
* `jti` *(required)* Follows standard JWT semantics.

# Verifying

When a verifier encounters a VVP passport, they use the following algorithm to verify:

1. Confirm that the `orig` and `dest` fields match contextual observations and other SIP metadata.
1. Extract the `kid` field.
1. Fetch the key state for the OP from the OOBI. (Caches may be used to optimize this, as long as they meet the freshness standards of the verifier.)
1. Use the public key of the OP to verify that the signature on the passport is valid. On success, the verifier knows that the OP is at least making a testable assertion and can be held accountable for it.
1. Extract the `evd` field, which references the dossier ({{<dossier}}) that constitutes backing evidence.
1. Check to see whether the dossier has already been fully validated. Since dossiers are highly stable, caching dossier validations is recommended.
1. If the dossier requires full validation, perform it. Validation includes checking the signatures on each ACDC in the dossier's data graph against the key state of the issuer at the time the issuance occurred. Key state is proved by the KEL {{<KEL}}, and checked against independent witnesses. (Subsequent key rotations do not invalidate this analysis, which is a vital improvement over certificate-based evidence.) Validation also includes comparing data structure and values against the declared schema, plus a full traversal of all chained CVD {{<cvd}}, back to the root of trust for each artifact. The correct relationships among evidence artifacts is checked (e.g., proving that the issuer of one piece is the issuee of another piece).
1. Check to see whether the revocation status of the dossier has been tested recently enough to satisfy the verifier's freshness standards. If no, check for revocations anywhere in the data graph of the dossier. Revocations are not the same as key rotations. They can be checked much more quickly than doing a full validation.
1. Assuming that the dossier is valid and has no breakages due to revocation, confirm that the OP is authorized to make the phone call. If there is no DE ({{<DE}}), the OP must be the issuee of the vetting credential; otherwise, the OP must be the issuee of a delegated signing credential for which the issuer is the AP.
1. Extract the `orig` field and compare it to the TNAlloc credential {{<tnalloc-credential}} cited in the dossier {{<dossier}} to confirm that the AP {{<AP}} (or, if OP is not equal to AP and OP is using their own number, the OP {{<OP}}) has the right to call on this number.
1. If the passport includes non-null values for the optional `card` or `goal` fields, extract that information and check that the brand attributes claimed for the call are justified by the brand credential {{<brand-credential}} in the dossier.
1. Check any business logic. For example, if the delegated signer credential says that the OP can only call on behalf of the AP during certain hours, or in certain geos, check those attributes of the call.

The complete algorithm listed above is quite rigorous. With no caches, it may take a second or two, much like a full validation of a certificate chain. However, because ACDC-based evidence is indexed by SAID {{<said}}, any given ACDC only needs to be validated once in its lifespan, which persists across key rotations and may last as long as an AP {{<AP}} uses a phone number -- years or decades. This allows very aggressive caching. And since the same dossier is used to justify many outbound calls -- perhaps thousands or millions of calls, for busy call centers -- and many dossiers will reference the same issuers and issuees and their associated key states and KELs {{<KEL}}, caching will produce huge benefits.

Furthermore, because SAIDs and their associated data (including links to other nodes in a data graph) have a tamper-evident relationship, any party can perform validation and compile their results, then share the data with verifiers that want to do less work. Validators like this are not oracles, because consumers of such data need not trust shared results blindly. They can always directly recompute some or all of it from a passport, to catch deception. However, they can do this lazily or occasionally, per their preferred balance of risk/effort.

*In toto*, these characteristics mean that no centralized registry is required in any given ecosystem. Data can be fetched directly from its source, with consent, across jurisdictional boundaries. Simple opportunistic, uncoordinated reuse (e.g., in or across the datacenters of TSPs {{<TP}}) will arise spontaneously and will dramatically improve the scale and efficiency of the system.

TODO: talk about second-level caching, of revocation status.

# Curating
The evidence that's available in today's telecom ecosystems resembles some of the evidence described here, in concept. However, it has gaps, and its format is fragile. Furthermore, if it is organized for discovery, it is typically assembled into large, centralized registries at a regional or national level. These registries become a trusted third party, which defeats some of the purpose of creating decentralized and independently verifiable evidence in the first place. Sharing such evidence across jurisdiction boundaries requires regulatory compatibility and bilateral agreements. Sharing at scale is impractical at best, if not illegal.

How evidence is issued, propagated, and reference is therefore an important concern for this specification.

## Activities

### Vetting entity identity
The job of vetting legal entities (which includes APs {{<AP}}, but also OPs {{<OP}}) and issuing vetting credentials {{<vetting-credential}} is performed by a *legal entity vetter* (*LEV*). VVP places few requirements on such vetters, other than the ones already listed for vetting credentials themselves. Vetting credentials do not need to expire, and in fact ACDCs and AIDs facilitate much longer lifecycles than certificates; prophylactic key rotation is recommended but creates no reason to rotate evidence. However, the LEV MUST agree to revoke vetting credentials in a timely manner if the legal status of an entity changes, or if data in a vetting credential becomes invalid.

### Vetting brand
The job of analyzing the brand assets of a legal entity and issuing brand credentials {{<brand-credential}} is performed by a *brand vetter* (*BV*). A BV MAY be an LEV, and MAY issue both types of credentials after a composite analysis. However, the credentials themselves MUST NOT use a combined schema, the credentials MUST have independent lifecycles, and the assurances associated with each credential type MUST remain independent. A BV MUST verify the canonical properties of a brand, but it MUST do more than this: it MUST issue the brand credential to the AID {{<aid}} of an issuee that is also the issuee of a vetting credential that already exists, and it MUST verify that the legal entity in the vetting credential has a right to use the brand in question. This link MUST NOT be based on mere weak evidence such as an observation that the legal entity's name and the brand name have some or all words in common, or the fact that a single person requested both credentials. Further, the BV MUST agree to revoke brand credentials in a timely manner if the associated vetting credential is revoked, if the legal entity's right to use the brand changes, or if characteristics of the brand evolve.

### Allocating TNs
Regulators issue TNAlloc credentials {{<tnalloc-credential}} to range holders, and range holders issue them to TNUs {{<allocation-holder}}, and TNUs MAY issue them to a delegate such as a call center. If aggregators or other intemediaries hold an RTU in the eyes of a regulator, then intermediate TNAlloc credentials MUST be created to track that RTU as part of the chain. On the other hand, if TNUs acquire telephone numbers through aggregators, but regulators do not consider aggregators to hold allocations, then aggregators MUST work with range holders to assure that the appropriate TNAlloc credentials are issued to the TNUs.

### Granting brand proxy rights
APs {{<AP}} issue brand proxy credentials {{<brand-proxy-credential}} to OPs {{<OP}}, giving them the right to use the AP's brand. Without this credential, the OP only has the right to use the AP's phone number.

Vetting and brand credentials may require large databases of metadata about organizations and brands, but how such systems work is out of scope. The credentials themselves contain all necessary information, and once credentials are issued, they constitute an independent source of truth. No party has to return to the operators of such databases to validate anything.

Issuance and revocation of all other credentials is self-service at every level, and does not require any large, centralized system.

(TODO: finish. The AIDs of issuers SHOULD be linked to witnesses...)

### Revoking
TODO

### Witnessing and watching
TODO

## General evidence categories
The term "credential" has a fuzzy meaning in casual conversation. However, understanding how evidence is built from credentials in VVP requires considerably more precision. We will start from lower-level concepts.

### SAID
A *self-addressing identifier* (*SAID*) is the hash of a canonicalized JSON object, encoded in self-describing CESR format {{TOIP-CESR}}. The raw bytes from the digest function are left-padded to the nearest 24-bit boundary, transformed to base64url {{RFC4648}}, and prefixed with a character that tells which digest function was used.

An example of a SAID is `E81Wmjyz5nXMCYrQqWyRLAYeKNQvYLYqMLYv_qm-qP7a`, and a regex that matches all valid SAIDs is: `[EFGHI][-_\w]{43}|0[DEFG][-_\w]{86}`. The `E` prefix in the example indicates that it is a Blake3-256 hash.

SAIDs are evidence that hashed data has not changed. They can also function like a reference, hyperlink, or placeholder for the data that was hashed to produce them (though they are more similar to URNs than to URLs {{RFC3986}}, since they contain no location information).

### Signature
A digital signature over arbitrary data D constitutes evidence that the signer processed D with a signing function that took D and the signer's private key as inputs: `signature = sign(D, privkey)`. The evidence can be verified by checking that the signature is bound to D and the public key of the signer: `valid = verify(signature, D, pubkey)`. Assuming that the signer has not lost unique control of the private key, and that cryptography is appropriately strong, we are justified in the belief that the signer must have taken deliberate action that required seeing an unmodified D in its entirety.

The assumption that a signer has control over their private keys may often be true (or at least believed, by the signer) at the time a signature is created. However, after key compromise, an attacker can create and sign evidence that purports to come from the current or an earlier time period, unless signatures are anchored to a data source that detects anachronisms. Lack of attention to this detail undermines the security of many credential schemes, including in telecom. VVP explicitly addresses this concern.

### AID
An *autonomic identifier* (*AID*) is a short string that can be resolved to one or more cryptographic keys at a specific version of their key state. Using cryptographic keys, a party can prove it is the controller of an AID by creating digital signatures. AIDs are like W3C DIDs {{W3C-DID}}}, and can be transformed into DIDs. The information required to resolve an AID to its cryptographic keys is communicated through a special form of URI called an *out-of-band invitation* (*OOBI*). An OOBI points to an HTTP resource that returns IANA content-type `application/json+cesr`; it is somewhat analogous to a combination of the `kid` and `x5u` constructs in many JWTs. AIDs and OOBIs are defined in the KERI spec {{TOIP-KERI}}.

An example of an AID is `EMCYrQqWyRLAYqMLYv_qm-qP7eKN81Wmjyz5nXQvYLYa`. AIDs are created by calculating the hash of the identifier's inception event data structure; they therefore match the same regex as SAIDs in general. An example of an OOBI is `https://agentsrus.net/oobi/EMCYrQqWyRLAYqMLYv_qm-qP7eKN81Wmjyz5nXQvYLYa/agent/EAxBDJkpA0rEjUG8vJrMdZKw8YL63r_7zYUMDrZMf1Wx`. Note the same `EMCY...` IN both strings. Many constructs in KERI may have OOBIs, but when OOBIs are associated with AIDs, such OOBIs always contain their associated AID as the first URL segment that matches the AID regex. They point either to an agent or a witness that provides verifiable state information for the AID.

AIDs possess several security properties that are not guaranteed by DIDs in general (self-certification, support for prerotation and multisig, support for witnesses, cooperative delegation, and so forth). These properties are vital to some of VVP's goals for high assurance.

### Signatures over SAIDs
Since neither a SAID value nor the data it hashes can be changed without breaking the correspondence between them, signing over a SAID is equivalent, in how it commits the signer to content and provides tamper evidence, to signing over the data that the SAID hashes. Since SAIDs can function as placeholders for JSON objects, a SAID can represent such an object in a larger data structure. And since SAIDs can function as a reference without making a claim about location, it is possible to combine these properties to achieve some indirections in evidence that are crucial in privacy and regulatory compliance.

VVP uses SAIDs and digital signatures as primitive forms of evidence.

### X509 certificates
VVP does not depend on X509 certificates {{RFC5280}} for any of its evidence. However, if deployed in a hybrid mode, it MAY be used beside alternative mechanisms that are certificate-based. In such cases, self-signed certificates that never expire might suffice to tick certificate boxes, while drastically simplifying the burden of maintaining accurate, unexpired, unrevoked views of authorizations and reflecting that knowledge in certificates. This is because deep authorization analysis flows through VVP's more rich and flexible evidence chain.

### PASSporT
VVP emits and verifies a STIR PASSporT {{RFC8225}}. This is a form of evidence suitable for evaluation during the brief interval when a call is being initiated, and it is carefully backed by evidence with a longer lifespan ({{<dossier}}). Conceptually, VVP's version is similar to a SHAKEN passport {{RFC8588}}. It MAY also reference brand-related evidence, allowing it to play an additional role similar to the RCD passport {{RCD-PASSPORT}}.

Neither VVP's backing evidence nor its passport depends on a certificate authority ecosystem. The passport MUST be secured by an EdDSA digital signature {{RFC8032}}, {{FIPS186-4}}, rather than the signature variants preferred by the other passport types. Instead of including granular fields in the claims of its JWT, the VVP passport cites a rich data graph of evidence by referencing the SAID of that data graph. This indirection and its implications are discussed below.

<figure>
<name>SHAKEN PASSporT compared to VVP PASSporT</name>
<artset>
  <artwork type="svg" name="fig1.svg">
    <svg xmlns="http://www.w3.org/2000/svg" version="1.1" height="240" width="528" viewBox="0 0 528 240" class="diagram" text-anchor="middle" font-family="monospace" font-size="13px" stroke-linecap="round">
    <path d="M 8,32 L 8,192" fill="none" stroke="black"/>
    <path d="M 200,32 L 200,192" fill="none" stroke="black"/>
    <path d="M 232,64 L 232,224" fill="none" stroke="black"/>
    <path d="M 296,32 L 296,176" fill="none" stroke="black"/>
    <path d="M 488,32 L 488,176" fill="none" stroke="black"/>
    <path d="M 520,144 L 520,224" fill="none" stroke="black"/>
    <path d="M 8,32 L 200,32" fill="none" stroke="black"/>
    <path d="M 296,32 L 488,32" fill="none" stroke="black"/>
    <path d="M 192,64 L 232,64" fill="none" stroke="black"/>
    <path d="M 472,144 L 520,144" fill="none" stroke="black"/>
    <path d="M 296,176 L 488,176" fill="none" stroke="black"/>
    <path d="M 8,192 L 200,192" fill="none" stroke="black"/>
    <path d="M 160,224 L 232,224" fill="none" stroke="black"/>
    <path d="M 488,224 L 520,224" fill="none" stroke="black"/>
    <polygon class="arrowhead" points="496,224 484,218.4 484,229.6" fill="black" transform="rotate(180,488,224)"/>
    <polygon class="arrowhead" points="168,224 156,218.4 156,229.6" fill="black" transform="rotate(180,160,224)"/>
    <circle cx="464" cy="144" r="6" class="opendot" fill="white" stroke="black"/>
    <g class="text">
    <text x="104" y="20">SHAKEN PASSporT</text>
    <text x="396" y="20">VVP PASSporT</text>
    <text x="56" y="52">protected</text>
    <text x="344" y="52">protected</text>
    <text x="108" y="68">kid: pubkey of OSP</text>
    <text x="380" y="68">kid: AID of OP</text>
    <text x="48" y="84">payload</text>
    <text x="336" y="84">payload</text>
    <text x="48" y="100">iat</text>
    <text x="336" y="100">iat</text>
    <text x="52" y="116">orig</text>
    <text x="340" y="116">orig</text>
    <text x="52" y="132">dest</text>
    <text x="340" y="132">dest</text>
    <text x="60" y="148">attest</text>
    <text x="392" y="148">evd (SAID = ref t</text>
    <text x="92" y="164">...more claims</text>
    <text x="368" y="164">signature of OP</text>
    <text x="84" y="180">signature of OSP</text>
    <text x="92" y="228">pubkey in cert</text>
    <text x="388" y="228">data graph of evidence</text>
    </g>
    </svg>
  </artwork>
  <artwork type="ascii-art" name="fig1.txt">
<![CDATA[
     SHAKEN PASSporT                       VVP PASSporT
+-----------------------+           +-----------------------+
| protected             |           | protected             |
|   kid: pubkey of OSP -+---+       |   kid: AID of OP      |
| payload               |   |       | payload               |
|   iat                 |   |       |   iat                 |
|   orig                |   |       |   orig                |
|   dest                |   |       |   dest                |
|   attest              |   |       |   card                |
|   ...more claims      |   |       |   evd (SAID = ref to)-+---+
| signature of OSP      |   |       | signature of OP       |   |
+-----------------------+   |       +-----------------------+   |
                            |                                   |
                            |                                   |
    pubkey in cert <--------+        data graph of evidence <---+
]]>
    </artwork>
  </artset>
</figure>

### ACDCs
Besides digital signatures and SAIDs, and the ephemeral passport, VVP uses long-lasting evidence in the ACDC format {{TOIP-ACDC}}. This is normalized, serialized data with an associated digital signature. Unlike X509 certificates, ACDCs are bound directly to the AIDs of their issuers and issuees, not to public keys of these parties. This has a radical effect on the lifecycle of evidence, because keys can be rotated without invalidating ACDCs.

### KELs {#KEL}
Unlike X509 certificates, JWTs {{RFC7519}}, and W3C Verifiable Credentials {{W3C-VC}}, signatures over ACDC data are not *contained inside* the ACDC; rather, they are *referenced by* the ACDC and *anchored in* a verifiable data structure called a *key event log* or *KEL* {{TOIP-KERI}}.

<figure>
<name>X509 compared to ACDC</name>
<artset>
  <artwork type="svg" name="fig2.svg">
    <svg xmlns="http://www.w3.org/2000/svg" version="1.1" height="304" width="456" viewBox="0 0 456 304" class="diagram" text-anchor="middle" font-family="monospace" font-size="13px" stroke-linecap="round">
    <path d="M 8,32 L 8,192" fill="none" stroke="black"/>
    <path d="M 200,32 L 200,192" fill="none" stroke="black"/>
    <path d="M 248,32 L 248,192" fill="none" stroke="black"/>
    <path d="M 248,240 L 248,288" fill="none" stroke="black"/>
    <path d="M 376,240 L 376,288" fill="none" stroke="black"/>
    <path d="M 432,64 L 432,272" fill="none" stroke="black"/>
    <path d="M 448,32 L 448,192" fill="none" stroke="black"/>
    <path d="M 8,32 L 200,32" fill="none" stroke="black"/>
    <path d="M 248,32 L 448,32" fill="none" stroke="black"/>
    <path d="M 400,64 L 432,64" fill="none" stroke="black"/>
    <path d="M 8,192 L 200,192" fill="none" stroke="black"/>
    <path d="M 248,192 L 448,192" fill="none" stroke="black"/>
    <path d="M 248,240 L 376,240" fill="none" stroke="black"/>
    <path d="M 376,272 L 432,272" fill="none" stroke="black"/>
    <path d="M 248,288 L 376,288" fill="none" stroke="black"/>
    <polygon class="arrowhead" points="408,64 396,58.4 396,69.6" fill="black" transform="rotate(180,400,64)"/>
    <g class="text">
    <text x="28" y="20">X509</text>
    <text x="268" y="20">ACDC</text>
    <text x="36" y="52">Data</text>
    <text x="304" y="52">v (version)</text>
    <text x="64" y="68">Version</text>
    <text x="324" y="68">d (SAID of item)</text>
    <text x="88" y="84">Serial Number</text>
    <text x="328" y="84">i (AID of issuer)</text>
    <text x="100" y="100">Issuer: DN of issuer</text>
    <text x="340" y="100">ri (status registry)</text>
    <text x="52" y="116">Validity</text>
    <text x="300" y="116">s (schema)</text>
    <text x="104" y="132">Subject: DN of issuee</text>
    <text x="316" y="132">a (attributes)</text>
    <text x="104" y="148">Sub PubKey Info: KeyX</text>
    <text x="336" y="148">i (AID of issuee)</text>
    <text x="60" y="164">Extensions</text>
    <text x="340" y="164">dt (issuance date)</text>
    <text x="56" y="180">Signature</text>
    <text x="292" y="180">...etc</text>
    <text x="256" y="228">KEL</text>
    <text x="312" y="260">signed anchor</text>
    <text x="292" y="276">for SAID</text>
    </g>
    </svg>
  </artwork>
  <artwork type="ascii-art" name="fig2.txt">
<![CDATA[
 X509                          ACDC
+-----------------------+     +------------------------+
| Data                  |     | v (version)            |
|   Version             |     | d (SAID of item) <---+ |
|   Serial Number       |     | i (AID of issuer)    | |
| Issuer: DN of issuer  |     | ri (status registry) | |
| Validity              |     | s (schema)           | |
| Subject: DN of issuee |     | a (attributes)       | |
| Sub PubKey Info: KeyX |     |  i (AID of issuee)   | |
| Extensions            |     |  dt (issuance date)  | |
| Signature             |     |  ...etc              | |
+-----------------------+     +----------------------+-+
                                                     |
                              KEL                    |
                              +---------------+      |
                              | signed anchor |      |
                              | for SAID      +------+
                              +---------------+
]]>
    </artwork>
  </artset>
</figure>

A KEL has some trust characteristics that resemble a blockchain. However, it is specific to one AID only (the AID of the issuer of the ACDC) and thus is not centralized. The KELs for two different AIDs need not (and typically do not) share any common storage or governance, and do not require coordination or administration. KELs thus suffer none of the performance and governance problems of blockchains, and incur none of blockchain's difficulties with regulatory requirements like data locality or right to be forgotten.

ACDCs can be freely converted between text and binary representations, and either type of representation can also be compacted or expanded to support nuanced disclosure goals. An ACDC is also uniquely identified by its SAID, which means that a SAID can take the place of a full ACDC in certain data structures and processes. None of these transformations invalidate the associated digital signatures, which means that any variant of a given ACDC is equivalently verifiable.

The revocation of a given ACDC can be detected via the witnesses declared in its issuer's KEL. Discovering, detecting, and reacting to such events can be very efficient. Any number of aggregated views can be built on demand, for any subset of an ecosystem's evidence, by any party. This requires no special authority or access, and does not create a central registry as a source of truth, since such views are tamper-evident and therefore can be served by untrusted parties. Further, different views of the evidence can contain or elide different fields of the evidence data, to address privacy, regulatory, and legal requirements.

### CVD
*Cryptographically verifiable data* (*CVD*) is data that's associated with a digital signature and a claim about who signed it. When CVD is an assertion, we make the additional assumption that the signer intends whatever the data asserts, since they took an affirmative action to create non-repudiable evidence that they processed it. CVD can be embodied in many formats, but in the context of VVP, all instances of CVD are ACDCs. When CVD references other CVD, the computer science term for the resulting data structure is a *data graph*.

### Credential
A *credential* is a special kind of CVD that asserts entitlements for its legitimate bearer -- *and only its bearer*. CVD that says Organization X exists with a particular ID number in government registers, and with a particular legal name, is not necessarily a credential. In order to be a credential, it would have to be associated with an assertion that its legitimate bearer -- *and only its bearer* -- is entitled to use the identity of Organization X. If signed data merely enumerates properties without conferring privileges on a specific party, it is just CVD. Many security gaps in existing solutions arise from conflating CVD and credentials.

ACDCs can embody any kind of CVD, not just credentials.

### Bearer token
VVP never uses bearer tokens, but we define them here to provide a contrast. A *bearer token* is a credential that satisfies binding requirements by a trivial test of possession -- like a movie ticket, the first party that presents the artifact to a verifier gets the privilege. Since Bearer Credentials can be stolen or copied, this is risky. JWTs, session cookies, and other artifacts in familiar identity technologies are often bearer tokens, even if they carry digital signatures. Although they can be protected to some degree by expiration dates and secure channels, these protections are imperfect. For example, TLS channels can be terminated and recreated at multiple places between call origination and delivery.

### Targeted credential
A *targeted credential* is CVD that identifies an issuee as the bearer, and that requires the issuee to prove their identity cryptographically (e.g., to produce a proper digital signature) in order to claim the associated privilege. All credentials in VVP are targeted credentials.

### JL
A *justifying link* (*JL*) is a reference, inside of one CVD, to another CVD that justifies what the first CVD is asserting. JLs can be SAIDs that identify other ACDCs. JLs are edges in an ACDC data graph.

## Specific artifacts

### PSS
Each voice call begins with a SIP INVITE, and in VVP, each SIP INVITE contains an `Identity` header that MUST contain a signature from the call's OP {{<OP}}. This *passpport-specific signature* (*PSS*) MUST be an Ed25519 signature serialized as CESR; it is NOT a JWS. The 64 raw bytes of the signature are left-padded to 66 bytes, then base64url-encoded. The `AA` at the front of the result is cut and replaced with `0B`, giving an 88-character string. A regex that matches the result is: `0B[-_\w]{86}`, and a sample value (with the middle elided) is: `0BNzaC1lZD...yRLAYeKNQvYx`.

The signature MUST be the result of running the EdDSA algorithm over input data that consists of the following ordered metadata about a call: the source phone number (`orig` claim in the JWT), the destination phone number (`dest` claim), an OOBI for the OP (`kid`), a timestamp (`iat`), optional brand information (`card` with value `null` if missing), optional `call-reason` (with value `null` if missing), optional `goal` (with value `null` if missing), and a reference to evidence (`evd`).

### Dossier
The `evd` field in the passport contains the OOBI ({{<aid}}) of an ACDC data graph called the *dossier*. The dossier is a compilation of all the permanent, backing evidence that justifies trust in the identity and authorization of the AP and OP. It is created and must be signed by the AP. It is CVD ({{<cvd}}) asserted to the world, not a credential ({{<credential}}) issued to a specific party.

### Accountable Party Evidence {#APE}
The dossier MUST include at least what is called *accountable party evidence* (*APE*).

APE consists of several credentials, explored in detail below. It MUST include a vetting credential for the AP. it MUST include a TNAlloc credential that proves RTU. Normally the RTU MUST be assigned to the AP; however, if a proxy is the OP and uses their own telephone number, the RTU MUST be assigned to the OP instead. If the AP intends to contextualize the call with a brand, it MUST include a brand credential for the AP. (In cases where callers are private individuals, "brand" maps to descriptive information about the individual, as imagined in mechanisms like VCard {{RFC6350}} or JCard {{RFC7095}}.) If no brand credential is present, verifiers MUST NOT impute a brand to the call on the basis of any VVP guarantees.

### Delegation Evidence {#DE}
When a private individual makes a call with VVP, they might be both the AP and the OP. In such cases, we expect their AID to be the recipient or issuee of all the APE, and no backing evidence beyond the APE may be necessary. However, in business contexts, it will almost always be true that the OP role is played by a delegate. In such conditions, evidence must also include proof that this indirection is valid. We call this *delegation evidence* (*DE*).

DE is nearly always required when the AP is an organization, because the cryptographic identifier for the organization as a legal entity is typically not the same as the cryptographic identifier for the organization's automated software that prepares SIP INVITEs. DE can thus distinguish between Acme Corporation in general, and software operated by Acme's IT department for the express purpose of signing voice traffic. The former has a vetting credential and legal accountability, and can act as the company to publish press releases, prepare invoices, spend money, and make attestations to regulators; the latter should only be able to sign outbound voice calls on Acme's behalf. Failing to make this distinction creates serious cybersecurity risks.

<figure>
<name>Sample evidence graph; OP kid could bind to APE or DE</name>
<artset>
  <artwork type="svg" name="fig3.svg">
    <svg xmlns="http://www.w3.org/2000/svg" version="1.1" height="528" width="512" viewBox="0 0 512 528" class="diagram" text-anchor="middle" font-family="monospace" font-size="13px" stroke-linecap="round">
    <path d="M 24,32 L 24,128" fill="none" stroke="black"/>
    <path d="M 24,240 L 24,272" fill="none" stroke="black"/>
    <path d="M 24,304 L 24,352" fill="none" stroke="black"/>
    <path d="M 24,400 L 24,432" fill="none" stroke="black"/>
    <path d="M 24,464 L 24,512" fill="none" stroke="black"/>
    <path d="M 128,192 L 128,208" fill="none" stroke="black"/>
    <path d="M 216,32 L 216,88" fill="none" stroke="black"/>
    <path d="M 216,104 L 216,128" fill="none" stroke="black"/>
    <path d="M 224,240 L 224,272" fill="none" stroke="black"/>
    <path d="M 224,304 L 224,352" fill="none" stroke="black"/>
    <path d="M 224,400 L 224,512" fill="none" stroke="black"/>
    <path d="M 248,192 L 248,416" fill="none" stroke="black"/>
    <path d="M 272,240 L 272,272" fill="none" stroke="black"/>
    <path d="M 272,304 L 272,336" fill="none" stroke="black"/>
    <path d="M 272,400 L 272,496" fill="none" stroke="black"/>
    <path d="M 288,496 L 288,512" fill="none" stroke="black"/>
    <path d="M 296,80 L 296,128" fill="none" stroke="black"/>
    <path d="M 320,128 L 320,144" fill="none" stroke="black"/>
    <path d="M 360,192 L 360,208" fill="none" stroke="black"/>
    <path d="M 400,80 L 400,128" fill="none" stroke="black"/>
    <path d="M 448,400 L 448,496" fill="none" stroke="black"/>
    <path d="M 464,240 L 464,336" fill="none" stroke="black"/>
    <path d="M 464,440 L 464,512" fill="none" stroke="black"/>
    <path d="M 488,112 L 488,144" fill="none" stroke="black"/>
    <path d="M 488,192 L 488,464" fill="none" stroke="black"/>
    <path d="M 24,32 L 216,32" fill="none" stroke="black"/>
    <path d="M 296,80 L 312,80" fill="none" stroke="black"/>
    <path d="M 360,80 L 400,80" fill="none" stroke="black"/>
    <path d="M 88,96 L 296,96" fill="none" stroke="black"/>
    <path d="M 400,112 L 488,112" fill="none" stroke="black"/>
    <path d="M 24,128 L 216,128" fill="none" stroke="black"/>
    <path d="M 296,128 L 400,128" fill="none" stroke="black"/>
    <path d="M 128,192 L 360,192" fill="none" stroke="black"/>
    <path d="M 24,240 L 224,240" fill="none" stroke="black"/>
    <path d="M 272,240 L 464,240" fill="none" stroke="black"/>
    <path d="M 272,336 L 464,336" fill="none" stroke="black"/>
    <path d="M 24,352 L 224,352" fill="none" stroke="black"/>
    <path d="M 24,400 L 224,400" fill="none" stroke="black"/>
    <path d="M 272,400 L 448,400" fill="none" stroke="black"/>
    <path d="M 224,416 L 248,416" fill="none" stroke="black"/>
    <path d="M 448,416 L 464,416" fill="none" stroke="black"/>
    <path d="M 448,432 L 488,432" fill="none" stroke="black"/>
    <path d="M 464,464 L 488,464" fill="none" stroke="black"/>
    <path d="M 272,496 L 448,496" fill="none" stroke="black"/>
    <path d="M 24,512 L 224,512" fill="none" stroke="black"/>
    <path d="M 288,512 L 464,512" fill="none" stroke="black"/>
    <circle cx="320" cy="80" r="6" class="opendot" fill="white" stroke="black"/>
    <g class="text">
    <text x="124" y="20">VVP PASSporT</text>
    <text x="72" y="52">protected</text>
    <text x="12" y="68">..</text>
    <text x="96" y="68">...kid: AID of OP</text>
    <text x="8" y="84">:</text>
    <text x="64" y="84">payload</text>
    <text x="340" y="84">f-OP</text>
    <text x="8" y="100">:</text>
    <text x="64" y="100">evd</text>
    <text x="336" y="100">SAID of</text>
    <text x="8" y="116">:</text>
    <text x="96" y="116">signature of OP</text>
    <text x="348" y="116">data graph</text>
    <text x="8" y="132">:</text>
    <text x="8" y="148">:</text>
    <text x="8" y="164">:</text>
    <text x="228" y="164">Accountable Party Evidence</text>
    <text x="424" y="164">Delegation Evidence</text>
    <text x="12" y="180">v?</text>
    <text x="312" y="180">(APE)</text>
    <text x="484" y="180">(DE)</text>
    <text x="100" y="228">vetting credential</text>
    <text x="348" y="228">TNAlloc credential</text>
    <text x="52" y="260">SAID</text>
    <text x="300" y="260">SAID</text>
    <text x="104" y="276">AID of issuer</text>
    <text x="352" y="276">AID of issuer</text>
    <text x="124" y="292">..:...AID of AP............:..</text>
    <text x="312" y="292">..:...AID of AP</text>
    <text x="8" y="308">:</text>
    <text x="92" y="308">legal name</text>
    <text x="344" y="308">TNAllocList</text>
    <text x="8" y="324">:</text>
    <text x="116" y="324">legal identifier</text>
    <text x="372" y="324">...more attributes</text>
    <text x="8" y="340">:</text>
    <text x="124" y="340">...more attributes</text>
    <text x="8" y="356">:</text>
    <text x="8" y="372">:</text>
    <text x="8" y="388">:</text>
    <text x="92" y="388">brand credential</text>
    <text x="340" y="388">more credentials</text>
    <text x="8" y="404">:</text>
    <text x="8" y="420">:</text>
    <text x="52" y="420">SAID</text>
    <text x="8" y="436">:</text>
    <text x="104" y="436">AID of issuer</text>
    <text x="360" y="436">e.g., delegate RTU,</text>
    <text x="64" y="452">:.:...AID of AP</text>
    <text x="352" y="452">vet for call ctr,</text>
    <text x="92" y="468">brand name</text>
    <text x="364" y="468">proxy right to brand</text>
    <text x="68" y="484">logo</text>
    <text x="124" y="500">...more attributes</text>
    </g>
    </svg>
  </artwork>
  <artwork type="ascii-art" name="fig3.txt">
<![CDATA[
         VVP PASSporT
  +-----------------------+
  | protected             |
......kid: AID of OP      |
: | payload               |         +--of-OP-----+
: |   evd --------------------------+ SAID of    |
: | signature of OP       |         | data graph +----------+
: +-----------------------+         +--+---------+          |
:                                      |                    |
:              Accountable Party Evidence  Delegation Evidence
v?                                  (APE)                 (DE)
               +--------------+--------+----+               |
               |              |             |               |
   vetting credential         |   TNAlloc credential        |
  +------------------------+  |  +-----------------------+  |
  | SAID                   |  |  | SAID                  |  |
  |   AID of issuer        |  |  |   AID of issuer       |  |
..:...AID of AP............:..|..:...AID of AP           |  |
: |   legal name           |  |  |   TNAllocList         |  |
: |   legal identifier     |  |  |   ...more attributes  |  |
: |   ...more attributes   |  |  +-----------------------+  |
: +------------------------+  |                             |
:                             |                             |
:  brand credential           |   more credentials          |
: +------------------------+  |  +---------------------+    |
: | SAID                   +--+  |                     +-+  |
: |   AID of issuer        |     | e.g., delegate RTU, +----+
:.:...AID of AP            |     | vet for call ctr,   | |  |
  |   brand name           |     | proxy right to brand| +--+
  |   logo                 |     |                     | |
  |   ...more attributes   |     +-+-------------------+ |
  +------------------------+       +---------------------+
]]>
    </artwork>
  </artset>
</figure>

Where DE exists, the APE will identify and authorize the AP, but the OOBI in the `kid` claim of the passport will identify the OP, and these two parties will be different. Therefore, the DE in the dossier MUST include a delegated signer credential that authorizes the OP to act on the AP's behalf and that stipulates the constraints that govern this delegation. It SHOULD also include a vetting credential for the OP, that proves the OP's identity. If the APE includes a brand credential, then the DE MUST also include a brand proxy credential, proving that the OP not only can use the AP's allocated telephone number, but has AP's permission to project the AP's brand while doing so.

The PASSporT-specific signature MUST come from the OP, not the OSP or any other party. The OP can generate this signature in its on-prem or cloud PBX, using keys that it controls. It is crucial that the distinction between OP and AP be transparent, with the relationship proved by strong evidence that the AP can create or revoke easily, in a self-service manner.

### Vetting credential
A vetting credential is a targeted credential that enumerates the formal and legal attributes of a unique legal entity. It MUST include a legal identifier that makes the entity unique in its home jurisdiction (e.g., an LEI), and it MUST include an AID for the legal entity as an AP. This AID is unique globally. The vetting credential is so called because it MUST be issued according to a documented vetting process that offers formal assurance that is is only issued with accurate information, and only to the entity it describes. A vetting credential confers the privilege of acting with the associated legal identity if and only if the bearer can prove their identity as issuee via a digital signature from the issuee's AID.

A vetting credential MUST include a JL to a credential that qualifies the issuer as a party trusted to do vetting. This linked credential that qualifies the issuer of the vetting credential MAY contain a JL that qualifies its own issuer, and such JLs MAY be repeated through as many layers as desired. In VVP, the reference type of a vetting credential is an LE vLEI. This implies both a schema and a governance framework. Other vetting credential types are possible, but they MUST be true credentials that meet the normative requirements here. They MUST NOT be bearer tokens.

(TODO: sample vetting credential)

### TNAlloc credential
A TNAlloc credential is a targeted credential that confers on its issuee the right to control how one or more telephone numbers are used. Regulators issue TNAlloc credentials to range holders, who issue them to TNUs. TNUs often play the AP role in VVP. If an AP delegates RTU to a proxy (e.g., an employee or call center), the AP MUST also issue a TNAlloc credential to the proxy, to confer the RTU. With each successive reallocation, the set of numbers in the new TNAlloc credential gets smaller. Except for TNAlloc credentials issued by regulators, all TNAlloc credentials MUST contain a JL to a parent TNAlloc credential, having a bigger set of numbers that includes those in the current credential. This JL in a child credential documents the fact that the child's issuer possessed an equal or broader RTU, from which the subset RTU in child credential derives.

### Brand credential
A brand credential is a targeted credential that enumerates brand properties such as a brand name, logo, chatbot URL, social media handle, and domain name. It MUST be issued to an AP as a legal entity, but it does not enumerate the formal and legal attributes of the AP; rather, it enumerates properties that would be meaningful to a TP who's deciding whether to take a phone call. It confers on its issuee the right to use the described brand by virtue of research conducted by the issuer (e.g., a trademark search).

(TODO: sample brand credential)

This credential MUST be issued according to a documented process that offers formal assurance that it is only issued with accurate information, and only to a legal entity that has the right to use the described brand. A single AP MAY have multiple brand credentials (e.g., a fictional corporation, `Amce Space Travel Deutschland, GmbH`, might hold brand credentials for both `Sky Ride` and for `Orbítame Latinoamérica`). Rights to use the same brand MAY be conferred on multiple APs (`Acme Space Travel Deutschland, Gmbh` and `Acme Holdings Canada, Ltd` may both possess brand credentials for `Sky Ride`). A brand credential MUST contain a JL to a vetting credential, that shows that the right to use the brand was evaluated only after using a vetting credential to prove the identity of the issuee.

### Brand proxy credential
A brand proxy credential confers on an OP the right to project the brand of an AP when making phone calls, subject to a carefully selected set of constraints. This is different from the simple RTU conferred by TNAlloc. Without a brand proxy credential, a call center could make calls on behalf of an AP, using the AP's allocated phone number, but would be forced to do so under its own name or brand, because it lacks evidence that the AP intended anything different. If an AP intends for phone calls to be made by a proxy, and wants the proxy to project the AP's brand, the AP MUST issue this credential.

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

