---
tags: [Standards]
description: This file summarizes the type and kind of cryptography used by Avalanche at the network and blockchain layers.
sidebar_label: Cryptography
pagination_label: Cryptographic Primitives
---

# Cryptographic Primitives

Avalanche uses a variety of cryptographic primitives for its different
functions. This file summarizes the type and kind of cryptography used at the
network and blockchain layers.

## Cryptography in the Network Layer

Avalanche uses Transport Layer Security, TLS, to protect node-to-node
communications from eavesdroppers. TLS combines the practicality of public-key
cryptography with the efficiency of symmetric-key cryptography. This has
resulted in TLS becoming the standard for internet communication. Whereas most
classical consensus protocols employ public-key cryptography to prove receipt of
messages to third parties, the novel Snow\* consensus family does not require
such proofs. This enables Avalanche to employ TLS in authenticating stakers and
eliminates the need for costly public-key cryptography for signing network
messages.

### TLS Certificates

Avalanche does not rely on any centralized third-parties, and in particular, it
does not use certificates issued by third-party authenticators. All certificates
used within the network layer to identify endpoints are self-signed, thus
creating a self-sovereign identity layer. No third parties are ever involved.

### TLS Addresses

To avoid posting the full TLS certificate to the Platform chain, the certificate
is first hashed. For consistency, Avalanche employs the same hashing mechanism
for the TLS certificates as is used in Bitcoin. Namely, the DER representation
of the certificate is hashed with sha256, and the result is then hashed with
ripemd160 to yield a 20-byte identifier for stakers.

This 20-byte identifier is represented by "NodeID-" followed by the data’s
[CB58](https://support.avalabs.org/en/articles/4587395-what-is-cb58) encoded
string.

## Cryptography in the Avalanche Virtual Machine

The Avalanche virtual machine uses elliptic curve cryptography, specifically
`secp256k1`, for its signatures on the blockchain.

This 32-byte identifier is represented by "PrivateKey-" followed by the data’s
[CB58](https://support.avalabs.org/en/articles/4587395-what-is-cb58) encoded
string.

### Secp256k1 Addresses

Avalanche is not prescriptive about addressing schemes, choosing to instead
leave addressing up to each blockchain.

The addressing scheme of the X-Chain and the P-Chain relies on secp256k1.
Avalanche follows a similar approach as Bitcoin and hashes the ECDSA public key.
The 33-byte compressed representation of the public key is hashed with sha256
**once**. The result is then hashed with ripemd160 to yield a 20-byte address.

Avalanche uses the convention `chainID-address` to specify which chain an
address exists on. `chainID` may be replaced with an alias of the chain. When
transmitting information through external applications, the CB58 convention is
required.

### Bech32

Addresses on the X-Chain and P-Chain use the
[Bech32](http://support.avalabs.org/en/articles/4587392-what-is-bech32) standard
outlined in [BIP 0173](https://en.bitcoin.it/wiki/BIP_0173). There are four
parts to a Bech32 address scheme. In order of appearance:

- A human-readable part (HRP). On Mainnet this is `avax`.
- The number `1`, which separates the HRP from the address and error correction code.
- A base-32 encoded string representing the 20 byte address.
- A 6-character base-32 encoded error correction code.

Additionally, an Avalanche address is prefixed with the alias of the chain it
exists on, followed by a dash. For example, X-Chain addresses are prefixed with
`X-`.

The following regular expression matches addresses on the X-Chain, P-Chain and
C-Chain for Mainnet, Fuji and localhost. Note that all valid Avalanche addresses
will match this regular expression, but some strings that are not valid
Avalanche addresses may match this regular expression.

```text
^([XPC]|[a-km-zA-HJ-NP-Z1-9]{36,72})-[a-zA-Z]{1,83}1[qpzry9x8gf2tvdw0s3jn54khce6mua7l]{38}$
```

Read more about Avalanche's [addressing scheme](https://support.avalabs.org/en/articles/4596397-what-is-an-address).

### Secp256k1 Recoverable Signatures

Recoverable signatures are stored as the 65-byte **`[R || S || V]`** where
**`V`** is 0 or 1 to allow quick public key recoverability. **`S`** must be in
the lower half of the possible range to prevent signature malleability. Before
signing a message, the message is hashed using sha256.

### Secp256k1 Example

Suppose Rick and Morty are setting up a secure communication channel. Morty
creates a new public-private key pair.

Private Key: `0x98cb077f972feb0481f1d894f272c6a1e3c15e272a1658ff716444f465200070`

Public Key (33-byte compressed): `0x02b33c917f2f6103448d7feb42614037d05928433cb25e78f01a825aa829bb3c27`

Because of Rick’s infinite wisdom, he doesn’t trust himself with carrying around
Morty’s public key, so he only asks for Morty’s address. Morty follows the
instructions, SHA256’s his public key, and then ripemd160’s that result to
produce an address.

SHA256(Public Key): `0x28d7670d71667e93ff586f664937f52828e6290068fa2a37782045bffa7b0d2f`

Address: `0xe8777f38c88ca153a6fdc25942176d2bf5491b89`

Morty is quite confused because a public key should be safe to be public
knowledge. Rick belches and explains that hashing the public key protects the
private key owner from potential future security flaws in elliptic curve
cryptography. In the event cryptography is broken and a private key can be
derived from a public key, users can transfer their funds to an address that has
never signed a transaction before, preventing their funds from being compromised
by an attacker. This enables coin owners to be protected while the cryptography
is upgraded across the clients.

Later, once Morty has learned more about Rick’s backstory, Morty attempts to
send Rick a message. Morty knows that Rick will only read the message if he can
verify it was from him, so he signs the message with his private key.

Message: `0x68656c702049276d207472617070656420696e206120636f6d7075746572`

Message Hash: `0x912800c29d554fb9cdce579c0abba991165bbbc8bfec9622481d01e0b3e4b7da`

Message Signature: `0xb52aa0535c5c48268d843bd65395623d2462016325a86f09420c81f142578e121d11bd368b88ca6de4179a007e6abe0e8d0be1a6a4485def8f9e02957d3d72da01`

Morty was never seen again.

## Signed Messages

A standard for interoperable generic signed messages based on the Bitcoin Script
format and Ethereum format.

```text
sign(sha256(length(prefix) + prefix + length(message) + message))
```

The prefix is simply the string `\x1AAvalanche Signed Message:\n`, where `0x1A`
is the length of the prefix text and `length(message)` is an
[integer](/reference/standards/serialization-primitives.md#integer) of the message size.

### Gantt Pre-Image Specification

```text
+---------------+-----------+------------------------------+
| prefix        : [26]byte  |                     26 bytes |
+---------------+-----------+------------------------------+
| messageLength : int       |                      4 bytes |
+---------------+-----------+------------------------------+
| message       : []byte    |          size(message) bytes |
+---------------+-----------+------------------------------+
                            |       26 + 4 + size(message) |
                            +------------------------------+
```

### Example

As an example we will sign the message "Through consensus to the stars"

```text
// prefix size: 26 bytes
0x1a
// prefix: Avalanche Signed Message:\n
0x41 0x76 0x61 0x6c 0x61 0x6e 0x63 0x68 0x65 0x20 0x53 0x69 0x67 0x6e 0x65 0x64 0x20 0x4d 0x65 0x73 0x73 0x61 0x67 0x65 0x3a 0x0a
// msg size: 30 bytes
0x00 0x00 0x00 0x1e
// msg: Through consensus to the stars
54 68 72 6f 75 67 68 20 63 6f 6e 73 65 6e 73 75 73 20 74 6f 20 74 68 65 20 73 74 61 72 73
```

After hashing with `sha256` and signing the pre-image we return the value
[cb58](https://support.avalabs.org/en/articles/4587395-what-is-cb58) encoded:
`4Eb2zAHF4JjZFJmp4usSokTGqq9mEGwVMY2WZzzCmu657SNFZhndsiS8TvL32n3bexd8emUwiXs8XqKjhqzvoRFvghnvSN`.
Here's an example using [Core web](https://core.app/tools/signing-tools/sign/).

:::info
A full guide on how to sign messages with Core web can be found 
[here](https://support.avax.network/en/articles/7206948-core-web-how-do-i-use-the-signing-tools).
:::

![Sign message](/img/signed-message.png)

## Cryptography in Ethereum Virtual Machine

Avalanche nodes support the full Ethereum Virtual Machine (EVM) and precisely
duplicate all of the cryptographic constructs used in Ethereum. This includes
the Keccak hash function and the other mechanisms used for cryptographic
security in the EVM.

## Cryptography in Other Virtual Machines

Since Avalanche is an extensible platform, we expect that people will add
additional cryptographic primitives to the system over time.
