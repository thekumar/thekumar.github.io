---
layout: post
title:  "Open sourcing `@yaki-inc/crypto`"
date:   2024-02-02 23:00:00 -0700
categories: cryptography AEAD encryption e2ee
mermaid: true
---

End to end encryption is at the core of how I'm building privacy and security into [Datayaki][datayaki]. And
with the availability of solid battle tested encryption libraries like [`libsodium`][ls] and [`tweetnacl`][tn],
implementing end-to-end encryption properly into your NodeJS and Typescript packages has become very
accessible...

> ...Except it hasn't!

There is Symmetric encryption where a single secret key is used for both encrypting and decrypting
data, Public/Private key based Asymmetric encryption where you encrypt with one of two keys and
decrypt with its complementary key. And there is Diffie Hellman Key Exchange where two parties can
independently compute and arrive at the same shared symmetric key knowing just the public key of the
other party.

There's so much security built into these encryption algorithms. Yet, the primary packages that are
widely used come with one major security risk...

> Developer Error!

`tweetnacl` is a fast and correct implementation of cryptographic primitives, yet all the keys in
the library are typed as `string`s.

While this makes it very easily portable and light weight, it also makes it very easy to introduce
an unintentional bug like exposing a PrivateKey in a keypair when you meant to share the PublicKey.
Since they are both essentially string of bytes, it is easy to mistakenly send one for the other,
and have it be exposed in some server's access log on the internet for all eternity. Having all the
keys be strings also makes it hard to debug and easy to introduce bugs in your code that lead to
sneaky bugs. For instance, passing in a symmetric key in place of a signing key, or vice versa.

# Announcing the availability of `@yaki-inc/crypto`

Yaki's crypto library is a strongly-typed and easy-to-understand wrapper around the battle tested
tweetnacl. It's [open sourced][gh] under MIT license.

You can read more about it on NPM at [@yaki-inc/crypto][yc]. But, the gist of it is,
it's strongly typed. A PublicKey and PrivateKey are two different types. And an EncryptionPublicKey
is different from a SigningPublicKey.

It also comes with a super easy to use AEAD helper class that lets you do symmetric, asymmetric, and
seal/unseal operations and still enforce type restrictions on the resulting `EncryptedDatagram`s.
The library also introduces **YEP** protocol, or the **Yaki Encrypted Permissions** protocol. This
is a way to implement access controls by requiring cryptographic proofs for permission grants,
instead of relying on simple boolean flags that any database admin can go tinker with.

# Datayaki uses it!

Open sourcing was both done to give back something to the community of developers and builders out
there, as well as to openly showcase the secure foundations upon which Datayaki's security is built.

Zero trust is the best way to guarantee trust in Datayaki.

[datayaki]https://www.datayaki.com
[yc]https://www.npmjs.com/package/@yaki-inc/crypto
[gh]https://www.github.com/yaki-inc/crypto
[tn]https://www.npmjs.com/package/tweetnacl
[ls]https://www.npmjs.com/package/libsodium