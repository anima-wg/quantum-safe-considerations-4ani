---
title: "Quantum Safe (PQ) Considerations for Autonomic Network Infrastructure (ANI)"
abbrev: "ani-qs"
category: std

docname: draft-richardson-anima-quantum-safe-considerations-4ani-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Operations and Management"
workgroup: "Autonomic Networking Integrated Model and Approach"
keyword:
 - agenticAI
 - ANI
 - BRSKI
 - ACP
venue:
  group: "Autonomic Networking Integrated Model and Approach"
  type: "Working Group"
  mail: "anima@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/anima/"
  github: "anima-wg/quantum-safe-considerations-4ani"
  latest: "https://anima-wg.github.io/quantum-safe-considerations-4ani/draft-richardson-anima-quantum-safe-considerations-4ani.html"

author:
 -
- ins: M. Richardson
  name: Michael C. Richardson
  role: editor
  org: Sandelman Software
  email:
  - mcr+ietf@sandelman.ca
  - https://orcid.org/0000-0002-0773-8388
  uri: https://www.sandelman.ca/

normative:
  RFC8995: BRSKI
  RFC8994: ACP
  RFC8366bis: draft-ietf-anima-rfc8366bis

informative:
  RFC8366:


--- abstract

The imminent arrival of a Cryptographically Relevant Quantum Computer (CRQC) makes algorithms such as RSA, ECDSA and EdDSA vulnerable to attack.  A transition to Quantum-Safe (PQ) algorithms is occurring.

This document provides specific requirements (Mandatory to Implement) for Autonomic Network Infrastructure
(ANI/ACP) and AgenticAI manufacturers and operators to be able to seamlessly transition to Quantum Safe algorithms.

--- middle

# Introduction

THIS IS SUBJECT TO MORE WORK

{{?RFC9958}} there is significant concern that current public key schemes like RSA and ECC will be compromised with a few years of publication of this document by a Cryptographically Relevant Quantum Computer (CRQC).

In order to plan for this possibility it is necessary to switch to quantum-safe algorithms.

Autonomic Control Plans and AgenticAI that make use of Autonomic Network Infrastructure consist of the following dependancies:

* each manufacturer has an 802.1AR Public Key Infrastructure (PKI) for the IDevID certificates

* each manufacturer to maintain a system of MASA for Voucher signing, with a trust anchor predistributed

* each manufacturer has to maintain a system for firmware signing

* each device has to be able to sign Voucher Requests

* each device has to be able to do TLS mutual-authentication

* each operator has to maintain an operational (Domain) Public Key Infrastructure (PKI) for LDevID

* each operator has to maintain a Registrar with a TLS end-point

* each device has to be able to do IPsec IKEv2 operations with a quantum-safe PK, and to do quantum-safe key agreement

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Voucher Requirements for Autonomic Control Plane

When ECDSA is supported, curves secp256r1 and secp384r1 SHOULD be supported.
When EdDSA is supported, curves Ed25519 and Ed448 SHOULD be supported.
When RSA is supported, key lengths between 2048 and 4096 bits SHOULD be supported.

ML-DSA {{!RFC9964}} SHOULD be supported for COSE and JOSE signed vouchers,
with {{!RFC9881}} used for CMS format voucher artifacts.

# PKI Requirements

Neither of these are hybrid algorithms and may not be suitable for use by a certification authority creating IDevID certificates.
For that use, {{?I-D.ietf-lamps-pq-composite-sigs}} SHOULD be supported.

The decision as to when to transition to quantum-safe algorithms is a manufacturer
decision.

Other choices are possible, but likely will result in Pledge devices being unable to onboard on networks until the Registrar involved implements those choices.
In such a situation, an automated, zero-touch onboarding solution may be impossible, but
it may still be possible to use mechanisms such as suggested in {{RFC8995, Section 7.2}} to onboard a device.

# TLS Requirements

For issue 6: the key agreement protocols within TLS and IPsec/IKE are the subject of documents such as {{?RFC9954}}, {{?RFC10024}}, {{?I-D.ietf-tls-mlkem}}, {{?I-D.ietf-ipsecme-ikev2-pqc-auth}}, and {{?I-D.ietf-ipsecme-ikev2-mlkem}}.

# IPsec Requirements

Where the significant risk comes from is in all the related infrastructure required.
An attack could be made upon any of the steps.  This includes:

1. the PKI used create and sign IDevID

2. the PKI used by the MASA to sign Vouchers

3. the algorithm used by the Pledge to sign Voucher Requests (the public key algorithm within the IDevID), and which is used for client-side certificate authentication in {{RFC8995}}

3. the PKI used by Registrar operator to authenticate the TLS end-point for the Registrar within {{RFC8995}}

4. the PKI used by Registrar operator to provision operational certificates (LDevID) to the device

5. the algorithms used in the operational certificates to authenticate the device for operational purposes

6. the key agreement protocols used by the TLS instances, or for creation of the ACP {{RFC8994}}

7. the PKI used by the manufacturer to authenticate firmware updates.

Issue 3 (above) is a concern for {{RFC8995}}, issue 5 is about the ability of {{RFC8995}} to use {{EST}} to provision quantum-safe algorithms.

For issue 6: the key agreement protocols within TLS and IPsec/IKE are the subject of documents such as {{?RFC9954}}, {{?RFC10024}}, {{?I-D.ietf-tls-mlkem}}, {{?I-D.ietf-ipsecme-ikev2-pqc-auth}}, and {{?I-D.ietf-ipsecme-ikev2-mlkem}}.

Issue 7, is the domain of {{?I-D.ietf-suit-mti}} which suggests the inclusion of the HSS-LMS for signing of firmware updates.

The transition of operational aspects (LDevID, IPsec) to quantum-safe algorithms requires that equipement firmware contain support for the new algorithms.
Once that support is present, there are significant, but manageable, issues for the operator managing the transition.  New firmware, new (hybrid) certificates issued via {{EST}} following an orderly renewal process, along with in-protocol mechanisms
being developed for TLS and IPsec/IKE to allow for an incremental transition.
All the above things are not in scope for this document.

The manufacturer of a device controls what firmware goes into it and when.
To first order, the manufacturer may switch to quantum-safe methods as soon as:

* it has updated its MASA to verify voucher requests with the new algorithm, and sign vouchers with the new algorithm

* it has switched its IDevID PKI to use a quantum-safe trust anchor, updating its qfactory device ID provisioning mechanism to use the new algorithm

* it has a quantum-safe TLS certificate for it's MASA internet visible end-point

* it has suitable firmware in the Pledge device that can use the new algorithms

A manufacturer can essentially treat all new devices produced as being part of a new product line.
That could include putting in a different MASA URL into the IDevID.
That allows for all quantum-safe operations to be done on a new platform removing any risk to any installed based.

The limiting factor for doing this is that the operator's Registrar must be ready to support the new algorithms.

This includes:

* answering provisional-TLS {{RFC8995}} requests with a quantum-safe certificate

* accepting quantum-safe client-certificates from new devices

* verification of the Pledge Voucher Request (PVR)

* connecting to the MASA using quantum-safe mutual TLS

* validating/auditing the resulting voucher

Of these steps, only the voucher operations involves Registrar application code,
the rest of this is "just" TLS upgrades necessary for quantum-safe operation.

It may be possible for manufacturers to provision IDevID using both traditional and quantum-safe versions.
In that case, the Pledge can look at the certificate presented by the Registrar, and if it is not quantum-safe, then it can use traditional algorithms.
For enterprise and ISP level equipment used in an {{RFC8994}} ACP there are no code space reasons to prevent this.
For IoT use cases, there may be code space, but also network capacity issues with the quantum-safe methods.
This is an evolving problem in the IoT space.



# Security Considerations

Transitions are hard.

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
