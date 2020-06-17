---
title: The OPAQUE Asymmetric PAKE Protocol
abbrev: OPAQUE
docname: draft-irtf-cfrg-opaque-latest
date:
category: info

ipr: trust200902
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: H. Krawczyk
    name: Hugo Krawczyk
    organization: Algorand Foundation
    email: hugokraw@gmail.com
 -
    ins: C. A. Wood
    name: Christopher A. Wood
    organization: Cloudflare
    email: caw@heapingbits.net

informative:

  Boyen09:
    title: "HPAKE: Password authentication secure against cross-site user
impersonation"
    author:
      -
        ins: X. Boyen
        name: Xavier Boyen

    seriesinfo: Cryptology and Network Security (CANS)
    date: 2009

  BG04:
    title: The static Diffie-Hellman problem
    author:
      -
        ins: D. Brown
        name: Daniel R. L. Brown
      -
        ins: R. Galant
        name: Robert P. Galant

    seriesinfo: http://eprint.iacr.org/2004/306
    date: 2004

  Canetti01:
    title: "Universally composable security: A new paradigm for cryptographic
protocols"
    author:
      -
        ins: R. Canetti
        name: Ran Canetti

    seriesinfo: IEEE Symposium on Foundations of Computer Science (FOCS)
    date: 2001

  Cheon06:
    title: Security analysis of the strong Diffie-Hellman problem
    author:
      -
        ins: J. H. Cheon
        name: Jung Hee Cheon

    seriesinfo: Euroctypt 2006
    date: 2006

  FK00:
    title: Server-assisted generation of a strong secret from a password
    author:
      -
        ins: W. Ford
        name: Warwick Ford
      -
        ins: B. S. Kaliski, Jr
        name: Burton S. Kaliski, Jr

    seriesinfo: WETICE
    date: 2000

  GMR06:
    title: "A method for making password-based
key exchange resilient to server compromise"

    author:
      -
        ins: C. Gentry
        name: Craig Gentry
      -
        ins: P. MacKenzie
        name: Phil MacKenzie
      -
        ins: Z, Ramzan
        name: Zulfikar Ramzan

    seriesinfo: CRYPTO
    date: 2006

  AuCPace:
    title: "AuCPace: Efficient verifier-based PAKE protocol tailored for the IIoT"
    author:
      -
        ins: B. Haase
        name: Bjorn Haase
      -
        ins: B. Labrique
        name: Benoit Labrique

    seriesinfo: http://eprint.iacr.org/2018/286
    date: 2018

  I-D.ietf-tls-exported-authenticator:
  I-D.barnes-tls-pake:
  I-D.irtf-cfrg-hash-to-curve:
  I-D.irtf-cfrg-voprf:
  I-D.sullivan-tls-opaque:
  I-D.ietf-tls-esni:
  I-D.ietf-tls-semistatic-dh:

  OPAQUE:
    title: "OPAQUE: An Asymmetric PAKE Protocol Secure Against Pre-Computation
    Attacks"
    # see the quotes above? Needed because of the embedded colon.
    author:
      -
        ins: S. Jarecki
        name: Stanislaw Jarecki
      -
        ins: H. Krawczyk
        name: Hugo Krawczyk
      -
        ins: J. Xu
        name: Jiayu Xu
    seriesinfo: Eurocrypt
    date: 2018

  Blinding:
    title: "Multiplicative DH-OPRF and Its Applications to Password Protocols"
    author:
      -
        ins: S. Jarecki
        name: Stanislaw Jarecki
      -
        ins: H. Krawczyk
        name: Hugo Krawczyk
      -
        ins: J. Xu
        name: Jiayu Xu
    seriesinfo: Manuscript
    date: 2020

  JKKX16:
    title: "Highly-efficient and composable
password-protected secret sharing (or: how to protect your bitcoin wallet
online)"
    # see the quotes above? Needed because of the embedded colon.
    author:
      -
        ins: S. Jarecki
        name: Stanislaw Jarecki
      -
        ins: A. Kiayias
        name: Aggelos Kiayias
      -
        ins: H. Krawczyk
        name: Hugo Krawczyk
      -
        ins: J. Xu
        name: Jiayu Xu
    seriesinfo: IEEE European Symposium on Security and Privacy
    date: 2016


  SIGMA:
    title: "SIGMA: The SIGn-and-MAc approach to authenticated Diffie-Hellman and its use in the IKE protocols"
    author:
      -
        ins: H. Krawczyk
        name: Hugo Krawczyk

    seriesinfo: CRYPTO
    date: 2003

  HMQV:
    title: "HMQV: A high-performance secure Diffie-Hellman protocol"
    author:
      -
        ins: H. Krawczyk
        name: Hugo Krawczyk

    seriesinfo: CRYPTO
    date: 2005

  SPAKE2plus:
    title: "Security Analysis of SPAKE2+"
    author:
      -
        ins: V. Shoup
        name: Victor Shoup

    seriesinfo: http://eprint.iacr.org/2020/313
    date: 2020

  SIGNAL:
    title: "Signal recommended cryptographic algorithms"
    seriesinfo: https://signal.org/docs/specifications/doubleratchet/#recommended-cryptographic-algorithms
    date: 2016

  RFC2945:
  RFC5869:
  RFC8018:
  RFC8125:
  RFC8446:

--- abstract

This draft describes the OPAQUE protocol, a secure asymmetric
password authenticated key exchange (aPAKE) that supports mutual
authentication in a client-server setting without reliance on PKI and
with security against pre-computation attacks upon server compromise.
Prior aPAKE protocols either did not use salt or transmitted the salt
in the clear, which allows targeted pre-computed dictionary attacks.
Jarecki et al. (Eurocrypt 2018) proved OPAQUE security in a strong and
universally composable formal model of aPAKE security. In addition,
the protocol provides forward secrecy and the ability to hide the
password from the server, even during password registration.

Strong security, versatility through modularity, good performance,
and an array of additional features make OPAQUE a natural candidate
for practical use and for adoption as a standard. To this end, this
draft presents several instantiations of OPAQUE and ways of
integrating OPAQUE with TLS.

This draft presents a high-level description of OPAQUE highlighting
its components and modular design. It also provides the basis for a
specification for standardization but a detailed specification ready
for implementation is beyond the current scope of this document
(which may be expanded in future revisions or done separately).

--- middle

# Introduction {#intro}

<!-- Remember this can be used for comments -->

Password authentication is the prevalent form of authentication in
the web and in most other applications. In the most common
implementation, a user authenticates to a server by sending its user
id and password to the server over a TLS connection. This makes
the password vulnerable to server mishandling, including accidentally
logging the password or storing it in cleartext in a database. Server
compromise resulting in access to these plaintext passwords is not an
uncommon security incident, even among security-conscious companies.
Moreover, plaintext password authentication over TLS is also vulnerable
to TLS failures, including many forms of PKI attacks, certificate
mishandling, termination outside the security perimeter, visibility
to middle boxes, and more.

Asymmetric (or augmented) Password Authenticated Key Exchange (aPAKE)
protocols are designed to provide password authentication and
mutually authenticated key exchange without relying on PKI (except
during user/password registration) and without disclosing passwords
to servers or other entities other than the client machine. A secure
aPAKE should provide the best possible security for a password
protocol. Namely, it should only be open to inevitable attacks, such as
online impersonation attempts with guessed user passwords and offline
dictionary attacks upon the compromise of a server and leakage of its
password file. In the latter case, the attacker learns a mapping of
a user's password under a one-way function and uses such a mapping to
validate potential guesses for the password. Crucially important is
for the password protocol to use an unpredictable one-way mapping.
Otherwise, the attacker can pre-compute a deterministic list of mapped
passwords leading to almost instantaneous leakage of passwords upon
server compromise.

Despite the existence of multiple designs for
(PKI-free) aPAKE protocols, none of these protocols are secure against
pre-computation attacks. In particular, none of these protocols can
use the standard technique against pre-computation that combines
_secret_ random values ("salt") into the one-way password mappings.
Either these protocols do not use salt at all or, if they do, they
transmit the salt from server to user in the clear, hence losing the
secrecy of the salt and its defense against pre-computation. Furthermore,
transmitting the salt may require additional protocol messages.

This draft describes OPAQUE, a PKI-free secure aPAKE that is
secure against pre-computation attacks and capable of using a secret
salt. Jarecki et al. {{OPAQUE}} recently proved the security of OPAQUE
in a strong aPAKE model that ensures security against pre-computation attacks
and is formulated in the Universal Composability (UC) framework {{Canetti01}}
under the random oracle model. In contrast, very few aPAKE protocols have
been proven formally, and those proven were analyzed in a weak
security model that allows for pre-computation attacks (e.g.,
{{GMR06}}). This is not just a formal issue: these protocols are
actually vulnerable to such attacks. This includes protocols that have recent
analyses in the UC model such as AuCPace {{AuCPace}} and SPAKE2+ {{SPAKE2plus}}.
We note that as shown in {{OPAQUE}}, these protocols, and any aPAKE
in the model from {{GMR06}}, can be converted into an aPAKE secure against
pre-computation attacks at the expense of an additional OPRF execution.

It is worth noting that the currently most deployed (PKI-free) aPAKE is
SRP {{?RFC2945}}, which is open to pre-computation attacks, and is inefficient
relative to OPAQUE. Moreover, SRP requires a ring as it mixes addition and
multiplication operations, and thus does not work over plain elliptic curves.
OPAQUE is therefore a suitable replacement.

OPAQUE's design builds on a line of work initiated in the seminal
paper of Ford and Kaliski {{FK00}} and is based on the HPAKE protocol
of Xavier Boyen {{Boyen09}} and the (1,1)-PPSS protocol from Jarecki
et al. {{JKKX16}}. None of these papers considered security against
pre-computation attacks or presented a proof of aPAKE security
(not even in a weak model).

In addition to its proven resistance to pre-computation attacks,
OPAQUE's security features include forward secrecy (essential for
protecting past communications in case of password leakage) and the
ability to hide the password from the server - even during password
registration. Moreover, good performance and an array of additional
features make OPAQUE a natural candidate for practical use and for
adoption as a standard. Such features include the ability to increase
the difficulty of offline dictionary attacks via iterated hashing
or other hardening schemes, and offloading these operations to the
client (that also helps against online guessing attacks); extensibility of
the protocol to support storage and
retrieval of user's secrets solely based on a password; and being
amenable to a multi-server distributed implementation where offline
dictionary attacks are not possible without breaking into a threshold
of servers (such a distributed solution requires no change or awareness
on the client side relative to a single-server implementation).

OPAQUE is defined and proven as the composition of two
functionalities: An Oblivious PRF (OPRF) and a key-exchange protocol.
It can be seen as a "compiler" for transforming any key-exchange
protocol (with KCI security and forward secrecy - see below)
into a secure aPAKE
protocol. In OPAQUE, the user stores a secret private key at the
server during password registration and retrieves this key each time
it needs to authenticate to the server. The OPRF security properties
ensure that only the correct password can unlock the private key
while at the same time avoiding potential offline guessing attacks.
This general composability property provides great flexibility and
enables a variety of OPAQUE instantiations, from optimized
performance to integration with TLS. The latter aspect is of prime
importance as the use of OPAQUE with TLS constitutes a major security
improvement relative to the standard password-over-TLS practice.
At the same
time, the combination with TLS builds OPAQUE as a fully functional
secure communications protocol and can help provide privacy to
account information sent by the user to the server prior to authentication.

The KCI property required from KE protocols for use with OPAQUE
states that knowledge of a party's private key does not allow an attacker
to impersonate others to that party. This is an important security
property achieved by most public-key based KE protocols, including
protocols that use signatures or public key encryption for
authentication. It is also a property of many implicitly
authenticated protocols (e.g., HMQV) but not all of them. We also note that
key exchange protocols based on shared keys do not satisfy the KCI
requirement, hence they are not considered in the OPAQUE setting.
We note that KCI is needed to ensure a crucial property of OPAQUE: even upon
compromise of the server, the attacker cannot impersonate the user to the
server without first running an exhaustive dictionary attack.
Another essential requirement from KE protocols for use in OPAQUE is to
provide forward secrecy (against active attackers).

This draft presents a high-level description of OPAQUE, highlighting
its components and modular design. It also provides the basis for a
specification for standardization, but a detailed specification ready
for implementation is beyond the current scope of this document
(which may be expanded in future revisions or done separately).

We describe OPAQUE with a specific instantiation of the OPRF component
over elliptic curves and with a few KE schemes, including the HMQV {{HMQV}},
3DH {{SIGNAL}} and SIGMA {{SIGMA}} protocols.
We also present several strategies for
integrating OPAQUE with TLS 1.3 {{RFC8446}} offering different tradeoffs
between simplicity, performance and user privacy.  In general, the modularity
of OPAQUE's design makes it easy to integrate with additional key-exchange
protocols, e.g., IKEv2.

The computational cost of OPAQUE is determined by the cost of the OPRF,
the cost of a regular Diffie-Hellman exchange, and the cost of
authenticating such exchange. In our elliptic-curve implementation of
the OPRF, the cost for the client is two exponentiations (one or two
of which can be fixed base) and one hashing-into-curve operation
{{I-D.irtf-cfrg-hash-to-curve}}; for the server, it is just one
exponentiation. The cost of a Diffie-Hellman exchange is as usual two
exponentiations per party (one of which is fixed-base). Finally, the
cost of authentication per party depends on the specific KE protocol:
it is just 1/6 of an exponentiation with HMQV, two exponentiations for 3DH,
and it is one signature generation and verification in the case of SIGMA and
TLS 1.3.
These instantiations preserve the number of messages in the underlying KE
protocol except in one of the TLS instantiations where user privacy may
require an additional round trip.

## Requirements Notation

{::boilerplate bcp14}

## Notation

The following terms are used throughout this document to describe the
operations, roles, and behaviors of OPAQUE:

- Client (U): Entity which has knowledge of a password and wishes to authenticate.
- Server (S): Entity which authenticates clients using passwords.
- (skX, pkX): A key pair used in role X; skX is the private key and pkX is
the public key. For example, (skU, pkU) refers to the U's private and public key.
- pk(skX): The public key corresponding to private key skX.
- concat(x0, ..., xN): Concatenation of byte strings.
  `concat(0x01, 0x0203, 0x040506) = 0x010203040506`.
- random(n): Generate a random byte string of length `n` bytes.
- zero(n): An all-zero byte string of length `n` bytes. `zero(4) = 0x00000000` and
  `zero(0)` is the empty byte string.

Except if said otherwise, random choices in this specification refer to
drawing with uniform distribution from a given set (i.e., "random" is short
for "uniformly random"). Random choices can be replaced with fresh outputs from
a cryptographically strong pseudorandom generator, according to the requirements
in {{!RFC4086}}, or pseudorandom function.

The name OPAQUE: A homonym of O-PAKE where O is for Oblivious
(the name OPAKE was taken).

# Cryptographic Protocol and Algorithm Dependencies

OPAQUE relies on the following protocols and primitives:

- Oblivious Pseudorandom Function (OPRF):
  - GenerateKeyPair(): This function generates a public and private key for the OPRF group.
  - Blind(x): Convert input `x` into an element of the OPRF group, randomize it
    by some value `r`, producing `M`, and output (`r`, `M`).
  - Evaluate(k, M): Compute the scalar multiplication `k*M` and output the
    result `Z`.
  - Unblind(r, Z): Remove randomizer `r` from `Z`, yielding output `N`.
  - Finalize(x, N, dst): Compute the OPRF output using input `x`, `N`, and domain
    separation tag `dst`.
  - Serialize(x): Encode the OPRF group element x as a fixed-length byte string
    `enc`. The size of `enc` is determined by the underlying OPRF group.
  - Deserialize(enc): Decode a byte string `enc` into an OPRF group element `x`,
    or produce an error of `enc` is an invalid encoding. This is the inverse
    of Encode, i.e., `x = Deserialize(Serialize(x))`.

- Random-Key Robustness Authenticated Encryption (RKR-AEAD): <!-- EnvU = AuthEnc(RwdU; skU, pkU, pkS) -->
  - Seal(k, n, aad, pt): Encrypt plaintext `pt` with additional
    associated data `aad` using the key `k` of length `Nk` and nonce `n` of length
    `Nn`, producing a ciphertext `ct` and tag `t`.

  - Open(k, n, aad, ct): Decrypt ciphertext `ct` with tag `t` and additional associated data
    `aad` using the key `k` of length `Nk` and nonce `n` of length `Nn`, producing
    plaintext `pt` on success or an error on failure.

- Memory Hard Function (MHF):
  - Harden(msg, params): Repeatedly apply a memory hard function with parameters
    `params` to strengthen the input `msg` against offline dictionary attacks.

## DH-OPRF {#SecOprf}

OPAQUE uses DH-OPRF instantiations from {{I-D.irtf-cfrg-voprf}}. It supports
the following ciphersuites:

- OPRF-curve25519_XMD:SHA-512_ELL2_RO_
- OPRF-curve448_XMD:SHA-512_ELL2_RO_
- OPRF-P256_XMD:SHA-256_SSWU_RO_
- OPRF-P384_XMD:SHA-512_SSWU_RO_
- OPRF-P521_XMD:SHA-512_SSWU_RO_

The output of the OPRF Finalize() operation may be hardened using a MHF F.
This greatly increases the cost of an offline attack upon the compromise of
the password file at the server. Supported MHFs include Argon2 {{?I-D.irtf-cfrg-argon2}},
scrypt {{?RFC7914}}, and PBKDF2 {{?RFC2898}}. with suitable parameter choices
public values or set at the time of password registration and stored at the server.
In this case, the server communicates these parameters to the user during OPAQUE
executions together with the second OPRF message.

## CTR+HMAC RKR-AEAD {#SecEnvU}

A RKR-AEAD algorithm is one in which, given a pair of random AEAD keys it is infeasible
to create an authenticated ciphertext that successfully decrypts under both keys.
Not all AEAD schemes satisfy this property, and this includes AES-GCM.
Therefore, we specify a scheme for implementing RKR-AEAD based on counter-mode
encryption and HMAC, hereby referred to as CTR+HMAC, using AES-256 and HMAC-SHA256.

The algorithm works by encrypting the plaintext with the input nonce, concatenating
the resulting ciphertext to the nonce, computing an HMAC tag over the
resulting string, and outputting the (nonce, ciphertext, tag) tuple.

We first define the function `DeriveKeys` used for key derivation. In particular,
it extracts and expands the input AEAD key `k` into two keys: an authentication key and
encryption key of length `La` and `Le`, respectively.
As specified, `La = Le = 32`, `Nn = 8`, and `Nk = 32`.

~~~
DeriveKeys(k)

- La, length of the authentication key in octets
- Le, length of the encryption key in octets

Input:
- k, a key of length Nk

Output:
- Ka, a key of length La
- Ke, a key of length Le

Steps:
1. uniform_bytes = HKDF-Expand(key=k, info="EnvU", Length=La+Le)
2. Ka = uniform_bytes[0:La]
3. Ke = uniform_bytes[La:]
5. Output (Ka, Ke)
~~~

We then define the Seal and Open implementations using AES-CTR and HMAC, in conjunction
with `DeriveKeys`. For completeness, we specify AES-CTR in Appendix {{AppCTR}}.
CTR+HMAC uses AES-256 in counter mode with key EncKey and an initial counter value
defined as the concatenation of the random `Nn`-byte nonce provided as input
to Seal and an eight-byte encoding of of 1.

~~~
Seal(k, n, aad, pt)

Input:
- k, an opaque key of Nk bytes
- n, an opaque nonce of length Nn bytes
- aad, additional associated data to authenticate
- pt, plaintext to encrypt

Output:
- ct, a ciphertext value of length equal to pt
- t, an authentication tag

Steps:
1. (Ka, Ke) = DeriveKeys(k)
2. IV = concat(n, zero(7), [0x01])
3. ct = AES-CTR(Ke, IV, pt)
4. t = HMAC(Ka, concat(ct, aad))
5. Output (ct, t)
~~~

Open is implemented similar to Seal, as defined below.

~~~
Open(k, n, aad, ct, t)

Input:
- k, an opaque key of Nk bytes
- n, an opaque nonce of length Nn bytes
- aad, additional associated data to authenticate
- ct, ciphertext to decrypt
- t, ciphertext authentication tag

Output:
- pt, decrypted plaintext, or an error upon failure

Steps:
1. (Ka, Ke) = DeriveKeys(k)
2. IV = concat(n, zero(7), [0x01])
3. pt = AES-CTR(Ke, IV, ct)
4. t' = HMAC(Ka, concat(ct, aad))
5. If CT_EQUAL(t, t'), output pt
6. Else, output error
~~~

[[TODO: If an RFC defining this mode exists, we should refer to it instead.
The mode is defined in [RFC3686] but in the context of IPsec's ESP, so having
a distilled version as in the Appendix may be worthwhile, particularly as we
use a different initial value (the above RFC assumes a given IV which we do
not have here).]]

# OPAQUE Protocol {#protocol}

OPAQUE consists of two stages: registration and authenticated key exchange.
In the first stage, a client stores its encrypted credentials on the server.
In the second stage, a client obtains those credentials and subsequently uses
them as input to an authenticated key exchange (AKE) protocol.

Both registration and authenticated key exchange stages require running an OPRF protocol.
The latter stage additionally requires running a mutually-authenticated
key-exchange protocol KE using credentials recovered after the OPRF protocol completes.
(The key-exchange protocol MUST satisfy the KCI requirement discussed in {{intro}}.)
Specification of the key-exchange protocol is out of scope for this document.

We first define the core OPAQUE protocol based on any OPRF, RKR-AEAD, and MHF functions.
{{instantiations}} describes specific instantiations of OPAQUE using various
AKE protocols, including: HMQV, 3DH, and SIGMA-I. {{I-D.sullivan-tls-opaque}}
discusses integration with TLS 1.3 {{RFC8446}}.

## Protocol messages {#protocol-messages}

The OPAQUE protocol runs the OPRF protocol in two stages: registration and
authenticated key exchange. A client and server exchange protocol messages in
executing these stages. This section specifies the structure of these protocol
messages using TLS notation (see {{RFC8446}}, Section 3).

~~~
enum {
    registration_request(1),
    registration_response(2),
    registration_upload(3),
    cred_request(4),
    cred_response(5),
    (255)
} ProtocolMessageType;

struct {
    ProtocolMessageType msg_type;    /* protocol message type */
    uint24 length;                   /* remaining bytes in message */
    select (ProtocolMessage.msg_type) {
        case registration_request: RegistrationRequest;
        case registration_response: RegistrationResponse;
        case registration_upload: RegistrationUpload;
        case cred_request: CredentialRequest;
        case cred_response: CredentialResponse;
    };
} ProtocolMessage;
~~~

Additionally, OPAQUE makes use of an additional structure `Credentials` to store
user (client) credentials. It is structured as follows. Each public and
private key value is an opaque byte string, specific to the AKE protocol in
which OPAQUE is instantiated. For example, if used as raw public keys for
TLS 1.3 {{?RFC8446}}, they may be RSA, DSA, or ECDSA keys as per {{?RFC7250}}.

~~~
struct {
  opaque skU<1..2^16-1>;
  opaque pkU<0..2^16-1>; // optional
  opaque pkS<1..2^16-1>;
  opaque idU<0..2^16-1>; // optional
  opaque idS<0..2^16-1>; // optional
} Credentials;
~~~

skU
: An encoded private key.

pkU
: An encoded public key corresponding to private_key. This field is optional.

pkS
: An encoded public key corresponding to the server.

idU
: The user identity. This is an application-specific value, e.g., an e-mail
address or normal account name. This field is optional.

idS
: An identity of the server. This is typically a domain name, e.g., example.com.
See {{SecIdentities}} for information about this identity. This field is optional.

## Offline registration stage {#SecPasReg}

Registration is executed between a user U (running on a client machine) and a
server S. It is assumed the server can identify the user and the client can
authenticate the server during this registration phase. This is the only part
in OPAQUE that requires an authenticated channel, either physical, out-of-band,
PKI-based, etc. This section describes the registration flow, message encoding,
and helper functions. Moreover, it is assumed the user has a key pair (skU, pkU)
that it wishes to register. These may be randomly generated for the account,
or may be keys located in persistent storage, such as a hardware token.

To begin, U chooses password PwdU, and S chooses its own pair of private-public
keys skS and pkS for use with protocol KE. S can use the same pair of keys with
multiple users. These steps can happen offline, i.e., before the registration phase.
Once complete, the registration process proceeds as follows:

~~~
 Client (IdU, PwdU, skU, pkU)                 Server (skS, pkS)
  -----------------------------------------------------------------
   request, metadata = CreateRegistrationRequest(IdU, PwdU)

                                   request
                              ----------------->

            (response, kU) = CreateRegistrationResponse(request, pkS)

                                   response
                              <-----------------

    record = FinalizeRequest(IdU, PwdU, metadata, request, response)

                                    record
                              ------------------>

                                             StoreUserRecord(record)
~~~

### Registration messages

~~~
struct {
    opaque context<0..2^8-1>;
    opaque id<0..2^16-1>;
    opaque data<1..2^16-1>;
} RegistrationRequest;
~~~

context
: An opaque string which identifies the registration request and which will be
echoed in the response. Clients use this to tie requests and responses together.

id
: An opaque string carrying the client account information, if available.

data
: An encoded element in the OPRF group. See {{I-D.irtf-cfrg-voprf}} for a
description of this encoding.

~~~
struct {
    opaque data_blind<1..2^16-1>;
} RequestMetadata;
~~~

data_blind
: An encoded OPRF scalar element. See {{I-D.irtf-cfrg-voprf}} for a
description of this encoding.

~~~
struct {
    opaque context<0..2^8-1>;
    opaque data<0..2^16-1>;
    opaque pkS<0..2^16-1>;
} RegistrationResponse;
~~~

context
: An opaque string matching the context provided in the corresponding registration
request, or empty string if one was not provided.

data
: An encoded element in the OPRF group. See {{I-D.irtf-cfrg-voprf}} for a
description of this encoding.

pkS
: An encoded public key that will be used for the online authenticated key exchange stage.

~~~
struct {
    opaque context<0..2^8-1>;
    opaque envelope<0..2^16-1>;
    opaque pkU<0..2^16-1>;
} RegistrationUpload;
~~~

context
: An opaque string matching the context set in the corresponding registration
request.

envelope
: An encryption of a Credentials structure. The encryption procedure is defined in {{SecEnvU}}.

pkU
: An encoded public key, matching the public key contained within the encrypted
envelope.

### Registration functions

#### CreateRegistrationRequest

~~~
CreateRegistrationRequest(IdU, PwdU)

Input:
- IdU, an opaque byte string containing the user's identity
- PwdU, an opaque byte string containing the user's password

Output:
- request, a RegistrationRequest structure
- metadata, a RequestMetadata structure

Steps:
1. context = GetUnusedContext()
2. (r, M) = Blind(PwdU)
3. data = Serialize(M)
4. Create RegistrationRequest request with (context, IdU, data)
5. Create RequestMetadata metadata with Serialize(r)
6. Output (request, metadata)
~~~

#### CreateRegistrationResponse

~~~
CreateRegistrationResponse(request, pkS)

Input:
- request, a RegistrationRequest structure
- pkS, the server's public KE key

Output:
- response, a RegistrationResponse structure
- kU, Per-user OPRF key

Steps:
1. (kU, pkU) = GenerateKeyPair() # pkU is ignored
2. M = Deserialize(request.data)
3. Z = Evaluate(kU, M)
4. data = Z.encode()
5. Create RegistrationResponse response with (request.context, data)
6. Output (response, kU)
~~~

#### FinalizeRequest

~~~
FinalizeRequest(IdU, PwdU, skU, metadata, request, response)

Parameters:
- params, the MHF parameters established out of band

Input:
- IdU, an opaque byte string containing the user's identity
- PwdU, an opaque byte string containing the user's password
- metadata, a RequestMetadata structure
- request, a RegistrationRequest structure
- response, a RegistrationResponse structure

Output:
- upload, a RegistrationUpload structure
- exporter_key, an additional key

Steps:
1. Z = Deserialize(response.data)
2. N = Unblind(input.data_blind, Z)
3. y = Finalize(PwdU, N, "RFCXXXX")
4. RwdU = Harden(y, params)
5. credentials_key = HKDF(salt=0, IKM=RwdU, "CredentialsKey", Nk)
6. exporter_key = HKDF(salt=0, IKM=RwdU, "ExporterKey", Nk)
7. n = random(Nn) # generate a fresh nonce randomly
8. Credentials C with (skU=skU, pkS=response.pkS)
9. (ct, t) = Seal(credentials_key, n, "", C)
10. EnvU = concat(n, ct, t)
11. Create RegistrationUpload upload with envelope value EnvU
11. Output (upload, exporter_key)
~~~

[[RFC editor: please change "RFCXXXX" to the correct number before publication.]]

The application may optionally include pk(skU), IdS, and IdU in the Credentials
structure C at step 8.

The server identity `IdS` comes from context. For example, if registering with
a server within the context of a TLS connection, the identity might be the
server domain name.

See {{exporter-usage}} for details about the output exporter_key usage.

#### StoreUserRecord

The StoreUserRecord function stores the tuple (EnvU, pkS, skS, pkU, kU),
where EnvU and pkU are obtained from the input RegistrationUpload message in
a record associated with the user's account IdU. If skS and pkS are used for
multiple users, the server can store these values separately and omit them from
the user's record.

## Online authenticated key exchange stage

After registration, the user (through a client machine) and server run the
authenticated key exchange stage of the OPAQUE protocol. This stage is composed of a concurrent
OPRF and key exchange flow. At the end, the client proves the user's knowledge
of the password, and both client and server agree on a mutually authenticated shared
secret key. This section describes the message flow, encoding, and helper functions
used in this stage.

~~~
 Client (IdU, PwdU)                           Server (skS, pkS)
  -----------------------------------------------------------------
   request, metadata = CreateCredentialRequest(IdU, PwdU)

                                   request
                              ----------------->

              (response, pkU) = CreateCredentialResponse(request)

                                   response
                              <-----------------

    creds = RecoverCredentials(PwdU, metadata, request, response)

                               (AKE with creds)
                              <================>
~~~

The protocol messages below do not include the AKE protocol. Instead, OPAQUE
assumes the client and server run the AKE using the credentials recovered after
the OPRF protocol completes.

Note also that the authenticate stage can run the OPRF and AKE protocols concurrently
with interleaved and combined messages (while preserving the internal ordering of
messages in each protocol). In all cases, the client needs to obtain EnvU and
RwdU (i.e., complete the OPRF protocol) before it can use its own private key
skU and the server's public key pkS in the run of KE. See {{instantiations}}
for examples of this integration.

### Authenticated key exchange messages

~~~
struct {
    opaque context<0..2^8-1>;
    opaque id<0..2^16-1>;
    opaque data<1..2^16-1>;
} CredentialRequest;
~~~

context
: An opaque string which identifies the credential request and which will be
echoed in the response. Clients use this to tie requests and responses together.

id
: An opaque string carrying the client account information, if available. If absent,
the server is assumed to have some way of ascertaining the client account information
out of band.

data
: An encoded element in the OPRF group. See {{I-D.irtf-cfrg-voprf}} for a
description of this encoding.

~~~
struct {
    opaque context<0..2^8-1>;
    opaque data<1..2^16-1>;
    opaque envelope<1..2^16-1>;
} CredentialResponse;
~~~

context
: An opaque string matching the context provided in the corresponding credential
request, or empty string if one was not provided.

data
: An encoded element in the OPRF group. See {{I-D.irtf-cfrg-voprf}} for a
description of this encoding.

envelope
: An encryption of an Credentials. The encryption procedure is defined in {{SecEnvU}}.

### Authenticated key exchange functions

#### CreateCredentialRequest(IdU, PwdU)

~~~
CreateCredentialRequest(IdU, PwdU)

Input:
- IdU, an opaque byte string containing the user's identity
- PwdU, an opaque byte string containing the user's password

Output:
- request, an CredentialRequest structure
- metadata, a RequestMetadata structure

Steps:
1. context = GetUnusedContext()
2. (r, M) = Blind(PwdU)
3. data = Serialize(M)
4. Create CredentialRequest request with (context, IdU, data)
5. Create RequestMetadata metadata with Serialize(r)
6. Output (request, metadata)
~~~

#### CreateCredentialResponse(request)

~~~
CreateCredentialResponse(request)

Input:
- request, an CredentialRequest structure

Output:
- response, a RegistrationUpload structure
- pkU, Public key of the user

Steps:
1. (kU, EnvU, pkU) = LookupUserRecord(request.id)
2. M = Deserialize(request.data)
3. Z = Evaluate(kU, M)
4. data = Z.encode()
5. Create CredentialResponse response with (request.context, data, EnvU)
6. Output (response, pkU)
~~~

#### RecoverCredentials(IdU, PwdU, metadata, request, response)

~~~
RecoverCredentials(PwdU, metadata, request, response)

Parameters:
- params, the MHF parameters established out of band

Input:
- PwdU, an opaque byte string containing the user's password
- metadata, a RequestMetadata structure
- request, a RegistrationRequest structure
- response, a RegistrationResponse structure

Output:
- C, a Credentials structure
- exporter_key, an additional key

Steps:
1. Z = Deserialize(response.data)
2. N = Unblind(input.data_blind, Z)
3. y = Finalize(PwdU, N, "RFCXXXX")
4. RwdU = Harden(y, params)
5. credentials_key = HKDF(salt=0, IKM=RwdU, "CredentialsKey", Nk)
6. exporter_key = HKDF(salt=0, IKM=RwdU, "ExporterKey", Nk)
7. (n, ct, t) = response.envelope
8. C = Open(credentials_key, n, "", ct, t)
9. Output C, exporter_key
~~~

[[RFC editor: please change "RFCXXXX" to the correct number before publication.]]

## Exporter Keys {#exporter-usage}

In addition to Credentials, OPAQUE outputs an exporter_key that may be used for additional
application-specific purposes. For example, one might expand the use of OPAQUE with a
credential-retrieval functionality that is separate from the contents of the Credentials
structure.

## AKE Execution and Party Identities {#SecIdentities}

The AKE protocol is run as part of the online authenticated key exchange
flow described above. The AKE MUST authenticate the OPAQUE transcript, which
consists of the encoded `request` and `response` messages exchanged during the
OPRF computation and credential fetch flow.

Also, authenticated key-exchange protocols generate keys that need to be uniquely
and verifiably bound to a pair of identities. In the case of OPAQUE, those identities
correspond to IdU and IdS. Thus, it is essential for the parties to agree on such
identities, including an agreed bit representation of these identities as needed.

Applications may have different policies about how and when identities are
determined. A natural approach is to tie IdU to the identity the server uses
to fetch EnvU (hence determined during password registration) and to tie IdS
to the server identity used by the client to initiate an offline password
registration or online authenticated key exchange session.
IdS and IdU can also be part of EnvU or be tied to the
parties' public keys. In principle, it is possible that identities change
across different sessions as long as there is a policy that can establish if
the identity is acceptable or not to the peer. However, we note that the
public keys of both the server and the user must always be those defined at
time of password registration.

# OPAQUE Instantiations {#instantiations}

We present several instantiations of OPAQUE using DH-OPRF
and different KE protocols. For the sake of concreteness we focus on
KE protocols consisting of three messages, denoted KE1, KE2, KE3, and
such that KE1 and KE2 include DH values sent by user and server,
respectively, and KE3 provides explicit user authentication.
As shown in {{OPAQUE}}, OPAQUE cannot use less than three messages
so the 3-message instantiations presented here are optimal in terms
of number of messages.
On the other hand, there is no impediment of using OPAQUE with protocols with
more than 3 messages as in the case of IKEv2 (or the underlying SIGMA-R
protocol {{SIGMA}}).

OPAQUE generic outline with 3-message KE:

- C to S: IdU, alpha=H'(PwdU)^r, KE1
- S to C: beta=alpha^kU, EnvU, KE2
- C to S: KE3

Key derivation and other details of the protocol are specified by the
KE scheme. We do note that by the results in {{OPAQUE}}, KE2 and KE3 should
include authentication of the OPRF messages (or at least of the value alpha)
for binding between the OPRF run and the KE session.

Next, we present three instantiations of OPAQUE - with HMQV, 3DH and SIGMA-I.
In {{SecTls13}} we discuss integration with TLS 1.3 {{RFC8446}}.

## Instantiation of OPAQUE with HMQV and 3DH {#SecHmqv}

The integration of OPAQUE with HMQV {{HMQV}} leads to the most
efficient instantiation of OPAQUE in terms of exponentiations count.
Performance is close to optimal due to the low cost of authentication in
HMQV: Just 1/6 of an exponentiation for each party over the cost of a regular
DH exchange. However, HMQV is encumbered by an IBM patent, hence we also
present OPAQUE with 3DH which only differs in the key derivation function
at the cost of an extra exponentiation (and less resilience to the compromise
of ephemeral exponents). We note that 3DH serves as a basis for the
key-exchange protocol of {{SIGNAL}}.

Importantly, many other protocols follow a similar format with differences
mainly in the key derivation function. This includes the Noise family of
protocols. Extensions may also apply to KEM-based KE protocols as in many
post-quantum candidates.

The private and public keys of the parties in these examples are
Diffie-Hellman keys, namely, pkU=g^skU and pkS=g^skS.

Specification/implementation details that are specific to the choice of group G
will be adapted from the corresponding standards for different elliptic curves.

PROTOCOL MESSAGES. OPAQUE with HMQV and OPAQUE with 3DH comprises:

- KE1 = OPRF1, nonceU, info1*, IdU*, epkU
- KE2 = OPRF2, EnvU, nonceS, info2*, epkS, Einfo2*, Mac(Km2; transcript2),
- KE3 = info3*, Einfo3*, Mac(Km3; transcript3)}

where:

- \* denotes optional elements;

- OPRF1, OPRF2 denote the DH-OPRF values alpha, beta sent by user and server,
respectively, as defined in {{SecOprf}};

- EnvU is the OPAQUE's envelope stored by the server containing keying
information for the client to run the AKE with the server;

- nonceU, nonceS are fresh random nonces chosen by client and server,
respectively;

- info1, info2, info3 denote optional application-specific information sent in
the clear (e.g., they can include parameter negotiation, parameters for a
hardening function, etc.);

- Einfo2, Einfo3 denotes optional application-specific information sent
encrypted under keys Ke2, Ke3 defined below;

- IdU is the user's identity used by the server to fetch the corresponding user
record, including EnvU, OPRF key, etc. (it can be omitted from message KE1 if
the information is available to the server in some other way);

- IdS, the server's identity, is not shown explicitly, it can be part of an info
field (encrypted or not), part of EnvU, or can be known from other context
(see {{SecIdentities}}); it is used crucially for key derivation (see below);

- epkU, epkS are Diffie-Hellman ephemeral public keys chosen by user and
server, respectively;

- transcript2 includes the concatenation of the values OPRF1, nonceU, info1*, IdU*, epkU,
OPRF2, EnvU, nonceS, info2*, epkS, Einfo2*;

- transcript3 includes the concatenation of all elements in transcript2 followed by
info3*, Einfo3*;

Notes:

- The explicit concatenation of elements under transcript2 and transcript3 can be
 replaced with hashed values of these elements, or their combinations, using
 a collision-resistant hash (e.g., as in the transcript-hash of TLS 1.3).

- The inclusion of the values OPRF1 and OPRF2 under transcript2 is needed for
 binding the OPRF execution to that of the KE session. On the other hand,
 including EnvU in transcript2 is not mandatory.

- The ephemeral keys epkU, epkS, can be exchanged prior to the above 3
 messages, e.g., when running these protocols under TLS 1.3.

KEY DERIVATION. The above protocol requires MAC keys Km2, Km3, and optional
encryption keys Ke2, Ke3, as well as generating a session key SK which is the
AKE output for protecting subsequent traffic (or for generating further key
material). Key derivation uses HKDF [RFC5869] with a combination of the parties static
and ephemeral private-public key pairs and the parties' identities IdU, IdS.
See {{SecIdentities}}.

~~~
SK, Km2, Km3, Ke2, Ke3 = HKDF(salt=0, IKM, info, L)
~~~

where L is the sum of lengths of SK, Km2, Km3, Ke2, Ke3, and SK
gets the most significant bytes of the HKDF output, Km2 the next bytes, etc.

Values IKM and info are defined for each protocol:

For HMQV:

 - info = "HMQV keys" \| nonceU \| nonceS \| IdU \| IdS

 - IKM = Khmqv

   where Khmqv is computed:

   - by the client:  Khmqv = (epkS \* pkS^s)^{eskU + u\*skU}

   - by the server:  Khmqv = (epkU \* pkU^u)^{eskS + u\*skS}

   and u = H(epkU \| "user" \| info) and s = H(epkS \| "srvr" \| info).

<!--     (u and s are often denoted d and e)   -->

FOR 3DH:

 - info = "3DH keys" \| nonceU \| nonceS \| IdU \| IdS

 - IKM = K3dh

   where K3dh is the concatenation of three DH values computed:

   - by the client:  K3dh = epkS^eskU \| pkS^eskU \| epkS^skU

   - by the server:  K3dh = epkU^eskS \| epkU^skS \| pkU^eskS

## Instantiation of OPAQUE with SIGMA-I {#SecSigma}

We show how OPAQUE is built around the 3-message SIGMA-I protocol {{SIGMA}}.
This is an example of a signature-based protocol and also serves
as a basis for integration of OPAQUE with TLS 1.3, as the latter follows the
design of SIGMA-I (see {{SecTls13}}.
This specification can be extended to the 4-message SIGMA-R protocol as used
in IKEv2.

PROTOCOL MESSAGES. OPAQUE with SIGMA-I comprises:

- KE1 = OPRF1, nonceU, info1*, IdU*, epkU
- KE2 = OPRF2, EnvU, nonceS, info2*, epkS, Einfo2*,
       Sign(skS; transcript2-), Mac(Km2; IdS),
- KE3 = info3*, Einfo3*, Sign(skU; transcript3-), Mac(Km3; IdU)}

See explanation of fields above. In addition, for the signed material,
transcript2- is defined similarly to transcript2, however if transcript2 includes
information that identifies the user, such information can be eliminated in
transcript2- (this is advised if signing user's identification information by
the server is deemed to have adverse privacy consequences).
Similarly, transcript3- is defined as transcript3 with server identification
information removed if so desired.

KEY DERIVATION. Key in SIGMA-I are derived as

~~~
SK, Km2, Km3, Ke2, Ke3 = HKDF(salt=0, IKM, info, L)
~~~

 where

 - L is the sum of lengths of SK, Km2, Km3, Ke2, Ke3, and SK gets the most
   significant bytes of the HKDF output, Km2 the next bytes, etc.,

 - info = "SIGMA-I keys" \| nonceU \| nonceS \| IdU \| IdS

 - IKM = Ksigma

   and Ksigma is computed:

   - by the client: Ksigma = epkS^eskU

   - by the server: Ksigma = epkU^eskS

## Integrating OPAQUE with TLS 1.3 {#SecTls13}

This section is intended as a discussion of ways to integrate OPAQUE with
TLS 1.3.
Precise protocol details are left for a future separate specification.
A very preliminary draft is {{I-D.sullivan-tls-opaque}}.

As stated in {{intro}}, the security of the standard password-over-TLS
mechanism for password authentication suffers from its essential reliance on
PKI and the exposure of passwords to the server (and possibly others) upon
TLS decryption. Integrating OPAQUE with TLS removes these vulnerabilities
while at the same time it armors TLS itself against PKI failures. Such
integration also benefits OPAQUE by leveraging the standardized negotiation
and record-layer security of TLS. Furthermore, TLS offers an initial
PKI-authenticated channel to protect the privacy of account information such
as user name transmitted between client and server.

If one is willing to forgo protection of user account information
transmitted between user and server, integrating OPAQUE with TLS
1.3 is relatively straightforward and follows essentially the same approach
as with SIGMA-I in {{SecSigma}}. Specifically, one reuses the Diffie-Hellman
exchange from TLS and uses the user's private key skU retrieved from
the server as a signature key for TLS client authentication. The
integrated protocol will have as its first message the TLS's Client
Hello augmented with user account information and with the DH-OPRF first
message (the value alpha).
The server's response includes the regular TLS 1.3 second flight
augmented with the second OPRF message which includes the values beta
and EnvU. For its TLS signature, the server uses the private key
skS whose corresponding public key pkS is authenticated as part of
the user envelope EnvU (there is no need to send a regular TLS certificate in
this case). Finally, the third flight consists of the standard client
Finish message with client authentication where the client's signature
is produced with the user's private key skU retrieved from EnvU and
verified by the server using public key pkU.

The above scheme is depicted in {{TLS3Flows}} where the sign + indicates
fields added by OPAQUE, and OPRF1, OPRF2 denote
the two DH-OPRF messages. Other messages in the figure are the same
as in TLS 1.3. Notation {...} indicates encryption under handshake
keys. Note that ServerSignature and ClientSignature are performed
with the private keys defined by OPAQUE and they replace signatures
by traditional TLS certificates.

<!--   The ~~~ (any number of ~ larger than 2?)
       defines a figure; it seems equivalent to enclosing within
      <figure><artwork>   </artwork></figure>
-->

~~~
        Client                                               Server

        ClientHello
        key_share
        + userid + OPRF1        -------->
                                                        ServerHello
                                                          key_share
                                                {+    OPRF2 + EnvU}
                                                  {ServerSignature}
                                <--------          {ServerFinished}

        {ClientSignature}
        {ClientFinished}        -------->
~~~~~~
{: #TLS3Flows title="Integration of OPAQUE in TLS 1.3 (no userid confidentiality)"}


Note that in order to send OPRF1 in the first message, the client needs to
know the DH group the server uses for OPRF, or it needs to "guess" it.
This issue already appears in TLS 1.3 where the client needs to guess the
key_share group and it should be handled similarly in OPAQUE
(e.g., the client may try one or more groups in its first message).

Protection of user's account information can be added through TLS
1.3 pre-shared/resumption mechanisms where the account information
appended to the ClientHello message would be encrypted under the
pre-shared key.

When a resumable session or pre-shared key between the client and the
server do not exist, user account protection requires a server
certificate. One option that does not add round trips is to use a
mechanism similar to the proposed ESNI extension {{I-D.ietf-tls-esni}}
or a semi-static TLS exchange as in {{I-D.ietf-tls-semistatic-dh}}.
Without such extensions, one would run a TLS 1.3 handshake augmented with
the two first OPAQUE messages interleaved between the second and third
flight of the regular TLS handshake. That is, the protocol consists
of five flights as follows: (i) A regular 2-flight 1-RTT handshake to
produce handshake traffic keys authenticated by the server's TLS
certificate; (ii) two OPAQUE messages that include user identification
information, the DH-OPRF messages exchanged between client and
server, and the retrieved EnvU, all encrypted under the handshake
traffic keys (thus providing privacy to user account information);
(iii) the TLS 1.3 client authentication flight where client
authentication uses the user's private signature key skU retrieved
from the server in step (ii).

Note that server authentication in step (i) uses TLS certificates hence
PKI is used for user account privacy but not for user authentication
or other purposes. (In some applications, PKI may be trusted also for server
authentication in which case server authentication through OPAQUE may be
forgone).
In OPAQUE the server authenticates
using the private key skS whose corresponding
public key pkS is sent to the user as part of EnvU. There are
two options: If pkS is the same as the public key the server used in
the 1-RTT authentication (step (i)) then there is no need for further
authentication.
<!-- In this case, the client gets assurance from the authenticated
EnvU, not (only) from the PKI certificates. -->
Otherwise, the server needs to
send a signature under skS that is piggybacked to the second OPAQUE
message in (ii). In this case, the signature would cover the running
transcript hash as is standard in TLS 1.3. The client signature in
the last message also covers the transcript hash including the
regular handshake and OPAQUE messages.

The described scheme is depicted in {{TLS5Flows}}. Please refer to the text
before {{TLS3Flows}} describing notation. Note the asterisk in the
ServerSignature message. This indicates that this message is optional
as it is used only if the server's key pkS in OPAQUE is different
than the one in the server's certificate (transmitted in the second
protocol flight).

~~~~
        Client                                               Server

        ClientHello
        key_share               -------->
                                                         ServerHello
                                                           key_share
                                                       {Certificate}
                                                 {CertificateVerify}
                                <--------           {ServerFinished}

        {+ userid + OPRF1}      -------->

                                                    {+ OPRF2 + EnvU}
                                <--------       {+ ServerSignature*}

        {ClientSignature}
        {ClientFinished}        -------->
~~~~~~
{: #TLS5Flows title="Integration of OPAQUE in TLS 1.3 (with userid confidentiality)"}


We note that the above approaches for integration of OPAQUE with TLS
may benefit from the post-handshake client authentication mechanism
of TLS 1.3 and the exported authenticators from
{{I-D.ietf-tls-exported-authenticator}}. Also, formatting of messages
and negotiation information suggested in {{I-D.barnes-tls-pake}} can
be used in the OPAQUE setting.

# Security considerations

This is an early draft presenting the OPAQUE concept and its
potential instantiations. More precise details and
security considerations will be provided in future drafts. We note
that the security of OPAQUE is formally proved in {{OPAQUE}} under a
strong model of aPAKE security assuming the security of the OPRF
function and of the underlying key-exchange protocol. In turn, the
security of DH-OPRF is proven in the random oracle model under the
One-More Diffie-Hellman assumption {{JKKX16}}.

Best practices regarding implementation of cryptographic schemes
apply to OPAQUE. Particular care needs to be given to the
implementation of the OPRF regarding testing group membership and
avoiding timing and other side channel leakage in the hash-to-curve
mapping. Drafts {{I-D.irtf-cfrg-hash-to-curve}} and
{{I-D.irtf-cfrg-voprf}} have detailed instantiation and
implementation guidance.

While one can expect the practical security of the OPRF function
(namely, the hardness of computing the function without knowing the
key) to be in the order of computing discrete logarithms or solving
Diffie-Hellman, Brown and Gallant [BG04] and Cheon [Cheon06] show an
attack that slightly improves on generic attacks. For the case that
q-1 or q+1, where q is the order of the group G, has a t-bit divisor,
they show an attack that calls the OPRF on 2^t chosen inputs and
reduces security by t/2 bits, i.e., it can find the OPRF key in time
2^{q/2-t/2} and 2^{q/2-t/2} memory. For typical curves, the attack
requires an infeasible number of calls and/or results in insignificant
security loss (\*). Moreover, in the OPAQUE application, these
attacks are completely impractical as the number of calls to the function
translates to an equal number of failed authentication attempts by a
_single_ user. For example, one would need a billion impersonation attempts
to reduce security by 15 bits and a trillion to reduce it by 20 bits - and
most curves will not even allow for such attacks in the first place
(note that this theoretical loss of security is with respect to computing
discrete logarithms, not in reducing the password strength).

(\*) Some examples (courtesy of Dan Brown): For P-384, 2^90 calls reduce
security from 192 to 147 bits; for NIST P-256 the options are 6-bit
reduction with 2153 OPRF calls, about 14 bit reduction with 187 million
calls and 20 bits with a trillion calls. For Curve25519, attacks are
completely infeasible (require over 2^100 calls) but its twist form allows
an attack with 25759 calls that reduces security by 7 bits and one with
117223 calls that reduces security by 8.4 bits.

Note on user authentication vs. authenticated key exchange. OPAQUE
provides PAKE (password-based authenticated key exchange)
functionality in the client-server setting. While in the case of user
identification, focus is often on the authentication part, we stress
that the key exchange element is not less crucial. Indeed, in most
cases user authentication is performed to enforce some policy, and
the key exchange part is essential for binding this enforcement to
the authentication step. Skipping the key exchange part is analogous
to carefully checking a visitor's credential at the door and then
leaving the door open for others to enter freely.

This draft complies with the requirements for PAKE protocols set forth in
{{RFC8125}}.

## Envelope considerations

Note (rationale of CTRBASE): The nonce used in defining CTRBASE is needed,
for example, for the case where a user registers the same password
repeatedly, choosing a fresh skU each time while the value of the server's
OPRF key kU stays fixed. This results in the same encryption key but
different plaintexts which requires a changing nonce. Eight bytes are more
than enough for this.

Note (using GCM): Can one replace AES-CTR with GCM-AES for encrypting AEenv?
Yes, as long as one keeps the HMAC authentication. As said, any secure
encryption can be used for encrypting AEenv. However, GCM also produces an
authentication tag that is not needed here. As a result, using GCM may tempt
someone to drop the HMAC authentication which would be insecure since
standalone GCM is not random-key robust. For this reason it may be better not
to replace plain AES-CTR with GCM or any other authenticated encryption.

Note (storage/communication efficient authentication-only EnvU):
It is possible to dispense with encryption in the construction of EnvU to
obtain a shorter EnvU (resulting in less storage at the server and less
communication from server to client). The idea is to derive skU from RwdU.
However, for cases where skU is not a random string of a given length, we
define a more general procedure. Namely, what's derived from RwdU is a random
seed used as an input to a key generation procedure that generates the pair
(skU, pkU). In this case, AEenv is empty and AOenv contains pkS. The
random key generation seed is defined as
HKDF-Expand(KdKey; info="KG seed", L)
where L is the required seed length. We note that in this encryption-less
scheme, the authentication still needs to be random-key robust which HMAC
satisfies.

<!--
Mention advantage of avoidable equivocable encryption? Still needs equivocable
authentication, but that one gets by modeling HMAC as programmable RO - check.
-->

To further minimize storage space, the server can derive per-user OPRF keys
kU from a single global secret key, and it can use the same pair
(skS,pkS) for all users. In this case, the per-user OPAQUE storage
consists of pkU and HMAC(Khmac; pkS), a total of 64-byte overhead with a
256-bit curve and hash. EnvU communicated to the user is of the same length,
consisting of pkS and HMAC(Khmac; pkS).

<!-- Can provide AuCPace paper (sec 7.7) as reference to importance of small
EnvU (for settings where storage and/or communication is expensive) -->

## User enumeration {#SecEnumeration}

User enumeration refers to attacks where the attacker tries to learn
whether a given user identity is registered with a server. Preventing
such attack requires the server to act with unknown user identities
in a way that is indistinguishable from its behavior with existing
users. Here we suggest a way to implement such defense, namely, a way for
simulating the values beta and EnvU for non-existing users.
Note that if the same pair of user identity IdU and value alpha is received
twice by the server, the response needs to be the same in both cases (since
this would be the case for real users).
For protection against this attack, one would apply the encryption function in
the construction of EnvU ({{SecEnvU}}) to all the key material in EnvU,
namely, AOenv will be empty.
The server S will have two keys MK, MK' for a PRF f
(this refers to a regular PRF such as HMAC or CMAC).
Upon receiving a pair of user identity IdU and value alpha for a non-existing
user IdU, S computes kU=f(MK; IdU) and kU'=f(MK'; IdU) and responds with
values beta=alpha^kU and EnvU, where the latter is computed as follows.
RwdU is set to kU' and AEenv is set to the all-zero string (of the
length of a regular EnvU plaintext). Care needs to be taken to avoid side
channel leakage (e.g., timing) from helping differentiate these
operations from a regular server response.
The above requires changes to the server-side implementation but not to the
protocol itself or the client side.

There is one form of leakage that the above allows and whose prevention would
require a change in OPAQUE.
Note that an attacker that tests a IdU (and same alpha) twice and receives
different responses can conclude that either the user registered with the
service between these two activations or that the user was registered before
but changed its password in between the activations (assuming the server
changes kU at the time of a password change). In any case, this
indicates that IdU is a registered user at the time of the second activation.
To conceal this information, S can implement the derivation of kU
as kU=f(MK; IdU) also for registered users. Hiding changes in EnvU, however,
requires a change in the protocol. Instead of sending EnvU as is,
S would send an encryption of EnvU under a key that the user derives from the
OPRF result (similarly to RwdU) and that S stores during password
registration. During the authenticated key exchange stage, the user will derive
this key from the OPRF result, will use it to decrypt EnvU, and continue with the
regular protocol. If S uses a randomized encryption, the encrypted EnvU will look
each time as a fresh random string, hence S can simulate the encrypted EnvU also
for non-existing users.

Note that the first case above does not change the protocol so its
implementation is a server's decision (the client side is not changed).
The second case, requires changes on the client side so it changes OPAQUE
itself.

[[TODO: Should this variant be documented/standardized?]

## Password salt and storage implications

In OPAQUE, the OPRF key acts as the secret salt value that ensures the infeasibility
of pre-computation attacks. No extra salt value is needed. Also, clients never
disclose their password to the server, even during registration. Note that this
does not prevent a malicious server from conducting a dictionary attack on inputs
provided by the client. OPAQUE assumes the server is honest, and only guarantees
safeguards against parties who may later compromise the server and any stored
user account information.

Some applications may require learning the user's password for enforcing password
rules. Doing so invalidates this important security property of OPAQUE and is
NOT RECOMMENDED. Applications should move such checks to the client. Note that
limited checks at the server are possible to implement, e.g., detecting repeated
passwords.

# IANA Considerations

TODO

--- back

# Counter mode encryption {#AppCTR}

We define counter mode encryption to be used with EnvU ({{SecEnvU}}).
The specification is based on [RFC3686] with a different initial value of
CTRBLK. The description refers to AES but it applies to any block cipher
(with its corresponding block size).

Let PT be the plaintext to be encrypted and CTRBASE a 128-bit initial value
(see {{SecEnvU}} for the OPAQUE-specific CTRBASE value).
Partition PT into 128-bit blocks PT = PT[1] PT[2] ... PT[n] where the
final block can be shorter than 128 bits. To compute the ciphertext
CT, each block PT[i] is XORed with a block KS[i] of a key stream KS
obtained by applying AES to a 128-bit counter CTRBLK initialized to CTRBASE
and incremented for each block KS[i]. The last value KS[n] is truncated,
if necessary, to the length of PT[n]. The ciphertext CT includes n+1
blocks defined as CT[0]=CTRBASE and CT[i]=PT[i] xor KS[i], for i=1,...,n,
with the final block possibly shorter than 128 bits.

The encryption of n plaintext blocks can be summarized as:

~~~
  CT[0] := CTRBASE
  CTRBLK := CTRBASE
  FOR i := 1 to n-1 DO
    CT[i] := PT[i] XOR AES(CTRBLK)
    CTRBLK := CTRBLK + 1
  END
  CT[n] := PT[n] XOR TRUNC(AES(CTRBLK))
~~~

The AES() function performs AES encryption with key EncKey.
The TRUNC() function truncates the output of the AES encrypt
operation to the same length as the final plaintext block, returning
the most significant bits.

Decryption is similar. The decryption of ciphertext CT= CT[0] ... CT[n]
summarized as:

~~~
  CTRBLK := C[0]
  FOR i := 1 to n-1 DO
    PT[i] := CT[i] XOR AES(CTRBLK)
    CTRBLK := CTRBLK + 1
  END
  PT[n] := CT[n] XOR TRUNC(AES(CTRBLK))
~~~

# Acknowledgments

The OPAQUE protocol and its analysis is joint work of the author with Stas
Jarecki and Jiayu Xu. We are indebted to the OPAQUE reviewers during CFRG's
aPAKE selection process, particularly Julia Hesse and Bjorn Tackmann.
This draft has benefited from comments by multiple people. Special thanks
to Richard Barnes, Dan Brown, Eric Crockett, Paul Grubbs, Fredrik Kuivinen,
Kevin Lewi, Payman Mohassel, Jason Resch, Nick Sullivan.
