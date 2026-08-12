---
title: "Post-Quantum Traditional (PQ/T) Hybrid PKI Authentication with separate certificates in the Internet Key Exchange Version 2 (IKEv2)"
abbrev: "IKEv2 PQ/T Auth with Separate Certs"
category: std

docname: draft-hu-ipsecme-pqt-hybrid-auth-with-sep-cert-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: sec
workgroup: ipsecme
keyword:
 - Post-Quantum
 - Hybrid Authentication
 - IKEv2
 - Separate Certificates
venue:
  group: WG
  type: Working Group
  mail: ipsec@ietf.org
  arch: https://mailarchive.ietf.org/arch/browse/ipsec/
  github: USER/REPO
  latest: https://example.com/LATEST

author:
 -
    fullname: Jun Hu
    organization: Nokia
    email: jun.hu@nokia.com
    country: United States of America
 -
    fullname: Yasufumi Morioka
    organization: NTT DOCOMO, INC.
    email: yasufumi.morioka.dt@nttdocomo.com
    country: Japan
 -
    fullname: Guilin Wang
    organization: Huawei
    email: Wang.Guilin@huawei.com
    country: Singapore
 -
    fullname: Tirumaleswar Reddy
    organization: Nokia
    city: Bangalore
    region: Karnataka
    country: India
    email: "k.tirumaleswar_reddy@nokia.com"
 -
    ins: S. Fluhrer
    name: Scott Fluhrer
    org: Cisco Systems
    email: sfluhrer@cisco.com


normative:
  I-D.ietf-lamps-pq-composite-sigs:
  I-D.ietf-pquip-hybrid-signature-spectrums:
  RFC9763:
  RFC9881:
  RFC7296:
  RFC7427:
  RFC9593:
  X.690:
    title: "Information Technology - ASN.1 encoding rules: Specification of Basic Encoding Rules (BER), Canonical Encoding Rules (CER) and Distinguished Encoding Rules (DER)"
    seriesinfo:
      ISO/IEC: 8825-1:2021 (E)
      ITU-T: Recommendation X.690
    date: Feb.2021

informative:
  ML-DSA:
    title: Module-Lattice-Based Digital Signature Standard
    date: Aug.2023
    seriesinfo:
      NIST: FIPS-204
    target: https://csrc.nist.gov/pubs/fips/204/final

  ML-KEM:
    title: Module-Lattice-Based Key-Encapsulation Mechanism Standard
    date: Aug.2023
    seriesinfo:
      NIST: FIPS-203
    target: https://csrc.nist.gov/pubs/fips/203/final

  RFC8784:
  I-D.ietf-ipsecme-ikev2-mlkem:
  I-D.reddy-ipsecme-pqt-hybrid-auth:

--- abstract

A Cryptographically Relevant Quantum Computer (CRQC) would break the
traditional asymmetric algorithms, such as RSA and ECDSA, that are widely
used to authenticate the Internet Key Exchange Protocol Version 2 (IKEv2).
Post-quantum digital signature algorithms are intended to resist such a
computer, but these new algorithms and their implementations require time to
mature.  This document specifies a post-quantum traditional (PQ/T) hybrid PKI
authentication method for IKEv2 in which each peer uses two separate
certificates: one containing a post-quantum public key and one containing a
traditional public key.  Authentication remains secure as long as at least one
of the component signature algorithms remains secure.

--- middle

# Change Log

## Changes in -00

* Initial version, this draft is the type-2 setup split from draft-hu-ipsecme-pqt-hybrid-auth.

# Introduction

A Cryptographically Relevant Quantum Computer (CRQC) could break traditional
asymmetric cryptographic algorithms, such as RSA and ECDSA, that are widely
deployed for IKEv2 authentication.  Post-quantum cryptographic (PQC) digital
signature algorithms, including ML-DSA {{ML-DSA}}, are intended to resist a
CRQC.  However, potential flaws in new algorithm specifications and
implementations mean that relying only on a new PQC algorithm presents risk
until the algorithm and its implementations have matured.  The motivation for
hybrid authentication is discussed further in
{{Section 1.2 of I-D.ietf-pquip-hybrid-signature-spectrums}}.

This document specifies PQ/T hybrid digital signature authentication for
IKEv2.  It combines a PQC signature algorithm and a traditional signature
algorithm so that authentication remains secure as long as at least one
component algorithm remains secure.

There are two types of setup:

1. Each peer has one certificate which contains a single composite key as described in {{I-D.ietf-lamps-pq-composite-sigs}}.

2. Each peer has two separate certificates: one certificate contains
the PQC public key and the other contains the traditional public key.

This document specifies mechanism for type-2, while {{I-D.reddy-ipsecme-pqt-hybrid-auth}} address type-1;

The mechanism is a general framework for combinations of PQC and traditional
algorithms.  Combinations containing ML-DSA and traditional algorithms are
examples of this framework.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

Cryptographically Relevant Quantum Computer (CRQC):
: A quantum computer capable of breaking real-world cryptographic systems.

Post-Quantum Cryptographic (PQC) algorithm:
: An asymmetric cryptographic algorithm believed to be secure against a CRQC.

Traditional cryptographic algorithm:
: An existing asymmetric cryptographic algorithm, such as RSA or ECDSA, that
  could be broken by a CRQC.

PQC certificate:
: The certificate containing thepublic key of PQC algorithm.

Traditional certificate:
: The certificate containing the public key of traditonal algorigthm.

# IKEv2 Key Exchange

This document does not change the IKEv2 key exchange process.  When used with
PQ/T hybrid authentication, the key exchange also needs to be CRQC resistant.  For
example, deployments can use a Post-quantum Preshared Key (PPK) as defined in
{{RFC8784}}, or a hybrid key exchange that includes a PQC algorithm such as
ML-KEM {{ML-KEM}} through the multiple key exchange mechanism defined in
{{I-D.ietf-ipsecme-ikev2-mlkem}}.  The authentication method specified here is
independent of the selected key exchange.

# Exchanges

The example in {{hybrid-auth-figure}} illustrates hybrid authentication using
a PPK during key exchange.  Other quantum-resistant key exchanges can be used.

~~~~~~~~~~~
Initiator                         Responder
-------------------------------------------------------------------
HDR, SAi1, KEi, Ni,
          N(USE_PPK) -->
                  <--  HDR, SAr1, KEr, Nr, [CERTREQ, CERTREQ],
                           N(USE_PPK), N(SUPPORTED_AUTH_METHODS)

HDR, SK {IDi, CERT, CERT, [CERTREQ, CERTREQ],
        [IDr,] AUTH, SAi2, TSi, TSr,
        N(PPK_IDENTITY, PPK_ID),
        N(SUPPORTED_AUTH_METHODS)} -->
                            <--  HDR, SK {IDr, CERT, CERT,
                                      [N(PPK_IDENTITY)], AUTH}
~~~~~~~~~~~
{: #hybrid-auth-figure title="Hybrid Authentication Exchange with Separate Certificates"}

The exchange proceeds as follows:

1. The responder announces its supported hybrid authentication combinations
   in a SUPPORTED_AUTH_METHODS notification in the IKE_SA_INIT response.  Each
   combination identifies a PQC algorithm, a traditional algorithm, and a
   pre-hash algorithm.
2. The initiator selects one announced combination, generates the AUTH payload,
   and sends the corresponding PQC and traditional certificates in two CERT
   payloads.  The initiator includes its own supported combinations in a
   SUPPORTED_AUTH_METHODS notification in the IKE_AUTH request.
3. The responder selects one combination announced by the initiator, generates
   the AUTH payload, and sends the corresponding PQC and traditional
   certificates in two CERT payloads in the IKE_AUTH response.

## Announcement
{: #announcement}

Support for this hybrid authentication method is announced using the
multi-octet SUPPORTED_AUTH_METHODS format defined in {{RFC9593}}.  Each
announcement contains:

* the new IANA-assigned AUTH_METHOD value requested by this document
* a Cert Link field
* a composite signature AlgorithmIdentifier

If the Cert Link field contains a non-zero value N, the method is intended for
use with the N-th and (N+1)-th trust anchor CA references from the Certificate
Request payloads, as described in {{certreq}}.

The AlgorithmIdentifier is a variable-length ASN.1 object encoded using
Distinguished Encoding Rules (DER) {{X.690}}.  It identifies a combination of following:

* a PQC signature algorithm, such as id-ML-DSA-44;
* a traditional signature algorithm, such as id-RSASSA-PSS; and
* a pre-hash algorithm, such as id-sha256.

For example, a system supporting ML-DSA-44 with RSA-2048-PSS and ML-DSA-44
with ECDSA P-256 includes two multi-octet announcements:

* the new AUTH_METHOD value with AlgorithmIdentifier
  id-MLDSA44-RSA2048-PSS-SHA256; and
* the new AUTH_METHOD value with AlgorithmIdentifier
  id-MLDSA44-ECDSA-P256-SHA256.

~~~~~~~~~~~
                         1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |  Length (>3)  | NEW_AUTH_VAL  |   Cert Link   |               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+               +
    |                                                               |
    ~          id-MLDSA44-RSA2048-PSS-SHA256 (AlgorithmIdentifier)  ~
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |  Length (>3)  | NEW_AUTH_VAL  |   Cert Link   |               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+               +
    |                                                               |
    ~          id-MLDSA44-ECDSA-P256-SHA256 (AlgorithmIdentifier)   ~
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~~~~~~~~
{: #sam-payload title="Example SUPPORTED_AUTH_METHODS Announcements"}

This document does not change the multi-octet announcement format in
{{RFC9593}}.  The only new protocol element is the AUTH_METHOD value requested
for authentication with separate certificates.

## Algorithm Combinations and Identifiers
This document uses algorithm combinations and identifiers specified in {{Section 6 of I-D.ietf-lamps-pq-composite-sigs}}, other combinations could be specified in future.


## Certificate Request
{: #certreq}

A peer performing this hybrid authentication method MAY send two CERTREQ
payloads.  The first payload contains the hash of the CA public key for the CA
that issued the PQC certificate.  The immediately following payload contains
the hash of the CA public key for the CA that issued the traditional
certificate.  Each CERTREQ uses the hash-of-CA-public-key format defined in
{{Section 3.7 of RFC7296}}.

When an announcement's Cert Link field is non-zero, its value identifies the
first of the two consecutive CA references: the PQC certificate CA is the N-th
reference and the traditional certificate CA is the (N+1)-th reference.

## AUTH and CERT Payloads
{: #auth-cert}

The IKEv2 AUTH payload has the format defined in
{{Section 3.8 of RFC7296}}:

~~~~~~~~~~~
                            1                   2                   3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      | Next Payload  |C|  RESERVED   |         Payload Length        |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      | Auth Method   |                RESERVED                       |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                                                               |
      ~                      Authentication Data                      ~
      |                                                               |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~~~~~~~~
{: #rfc7296-auth title="IKEv2 AUTH Payload"}

The Auth Method field contains the new IANA-assigned value requested by this
document.  The Authentication Data field uses the format defined in
{{Section 3 of RFC7427}}:

~~~~~~~~~~~
                           1                   2                   3
       0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      | ASN.1 Length  | AlgorithmIdentifier ASN.1 object              |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                                                               |
      ~        AlgorithmIdentifier ASN.1 object continuing            ~
      |                                                               |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                                                               |
      ~                         Signature Value                       ~
      |                                                               |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~~~~~~~~
{: #hybrid-auth-data title="Authentication Data in the Hybrid AUTH Payload"}

The AlgorithmIdentifier MUST match the selected combination previously
announced by the peer.  The Signature Value is generated as described in
{{signing}}.

The sender MUST place its signing PQC certificate in the first CERT payload and
its signing traditional certificate in the immediately following CERT payload.

### Signing
{: #signing}

The sender generates the hybrid AUTH payload as follows:

1. Select the composite signature AlgorithmIdentifier agreed through
   SUPPORTED_AUTH_METHODS.
2. Combine the PQC private key and traditional private key using the
   SerializePrivateKey operation defined in
   {{Section 4.2 of I-D.ietf-lamps-pq-composite-sigs}}.
3. Run the Sign operation defined in
   {{Section 3.2 of I-D.ietf-lamps-pq-composite-sigs}} with:

   * `sk`: the output of SerializePrivateKey;
   * `M`: InitiatorSignedOctets or ResponderSignedOctets, as appropriate, from
     {{Section 2.15 of RFC7296}}; and
   * `ctx`: the ASCII encoding of "IKEv2-PQT-Hybrid-Auth" (21 octets, with no
     null terminator).

4. Serialize the resulting CompositeSignatureValue as defined in
   {{Section 4.3 of I-D.ietf-lamps-pq-composite-sigs}} and place the result in
   the Signature Value field.

{{Section 6 of RFC9881}} defines three options for storing an ML-DSA private
key.  When ML-DSA is a component algorithm, the implementation MUST use an
option that includes the seed because the Sign operation in
{{I-D.ietf-lamps-pq-composite-sigs}} requires the seed.

The composite signature specification uses a pre-hash algorithm with the pure
mode of ML-DSA, rather than HashML-DSA.  See
{{Section 2.1 of I-D.ietf-lamps-pq-composite-sigs}} for the rationale.

### RelatedCertificate
{: #related-certificate}

Either signing certificate MAY contain the RelatedCertificate extension
defined in {{RFC9763}}.  If either certificate contains this extension, the
receiver MUST verify the extension according to {{Section 4.2 of RFC9763}}.
Failure to verify the extension MUST cause authentication to fail.

### Verification
{: #verification}

The receiver verifies the hybrid AUTH payload as follows:

1. Verify that the Auth Method is the value assigned for the authentication
   method specified by this document.
2. Verify that the AlgorithmIdentifier matches a combination previously
   announced in the sender's SUPPORTED_AUTH_METHODS notification.  If it does
   not match, reject the exchange with AUTHENTICATION_FAILED.
3. Obtain the PQC public key from the first CERT payload and the traditional
   public key from the immediately following CERT payload.  Validate both
   certificate paths according to the local policy and the requirements of
   {{RFC7296}}.
4. If either certificate contains a RelatedCertificate extension, verify it as
   specified in {{related-certificate}}.
5. Construct the composite public key using the SerializePublicKey operation
   defined in {{Section 4.1 of I-D.ietf-lamps-pq-composite-sigs}}.
6. Run the Verify operation defined in
   {{Section 3.3 of I-D.ietf-lamps-pq-composite-sigs}} with:

   * `pk`: the output of SerializePublicKey;
   * `M`: InitiatorSignedOctets or ResponderSignedOctets, as appropriate, from
     {{Section 2.15 of RFC7296}};
   * `ctx`: the ASCII encoding of "IKEv2-PQT-Hybrid-Auth" (21 octets, with no
     null terminator); and
   * `sig`: the Signature Value from the Authentication Data field.

7. If Verify returns failure, reject the IKE_AUTH exchange with
   AUTHENTICATION_FAILED.

# Security Considerations

The security of PQ/T hybrid authentication is discussed in
{{I-D.ietf-pquip-hybrid-signature-spectrums}}.  This document also uses
mechanisms defined in {{I-D.ietf-lamps-pq-composite-sigs}}, {{RFC7427}},
{{RFC9593}}, and {{RFC9763}}; the security considerations of those documents
apply.

A component key used by this hybrid authentication method MUST NOT be reused
for any other purpose, including authentication with only that component
algorithm.  Key reuse could invalidate the security properties expected from
the composite signature construction.

Both certificate paths MUST be validated.  Accepting a valid path for only one
component certificate would reduce the authentication method to a
single-algorithm method.  When a RelatedCertificate extension is present, its
verification binds the two certificates and prevents an attacker from
substituting an unrelated component certificate.

## Downgrade Attack Prevention

The IKE_SA_INIT exchange is not integrity-protected.  An active on-path
attacker can modify or remove a SUPPORTED_AUTH_METHODS notification in that
exchange.  If the notification is removed from the responder's IKE_SA_INIT
response, an initiator supporting both hybrid and non-hybrid authentication
might otherwise fall back to traditional-only authentication.

A system configured to require mutual hybrid authentication for a peer MUST
reject a SUPPORTED_AUTH_METHODS notification that does not contain an
acceptable hybrid combination.  It also MUST reject an IKE_AUTH exchange in
which the peer's AUTH payload does not use the Auth Method specified by this
document or does not use an AlgorithmIdentifier that was previously announced.

# IANA Considerations

This document requests a new value in the "IKEv2 Authentication Method"
subregistry of the "Internet Key Exchange Version 2 (IKEv2) Parameters"
registry:

| Value | Method | Reference |
| --- | --- | --- |
| TBD1 | PQ/T Hybrid Digital Signature with Separate Certificates | This document |

TBD1 is used in the Auth Method field of the IKEv2 AUTH payload and as the
AUTH_METHOD value in a multi-octet SUPPORTED_AUTH_METHODS announcement.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
