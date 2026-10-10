# Support for Output Descriptors in Megacoin Core

Since Megacoin Core v1.9.9 / v1.10.0 base, there is support for Output Descriptors in the
`scantxoutset` RPC call. This is a simple language which can be used to
describe collections of output scripts.

This document describes the language. For the specifics on usage for scanning
the UTXO set, see the `scantxoutset` RPC help.

## Features

Output descriptors currently support:
- Pay-to-pubkey scripts (P2PK), through the `pk` function.
- Pay-to-pubkey-hash scripts (P2PKH), through the `pkh` function.
- Pay-to-witness-pubkey-hash scripts (P2WPKH), through the `wpkh` function.
- Pay-to-script-hash scripts (P2SH), through the `sh` function.
- Pay-to-witness-script-hash scripts (P2WSH), through the `wsh` function.
- Multisig scripts, through the `multi` function.
- Any type of supported address through the `addr` function.
- Raw hex scripts through the `raw` function.
- Public keys (compressed and uncompressed) in hex notation, or BIP32 extended pubkeys with derivation paths.

## Examples

- `pk(0279be667ef9dcbbac55a06295ce870b07029bfcdb2dce28d959f2815b16f81798)` describes a P2PK output with the specified public key.
- `pkh(02c6047f9441ed7d6d3045406e95c07cd85c778e4b8cef3ca7abac09b95c709ee5)` describes a P2PKH output with the specified public key.
- `wpkh(02f9308a019258c31049344f85f89d5229b531c845836f99b08601f113bce036f9)` describes a P2WPKH output with the specified public key.
- `sh(wpkh(03fff97bd5755eeea420453a14355235d382f6472f8568a18b2f057a1460297556))` describes a P2SH-P2WPKH output with the specified public key.
- `combo(0279be667ef9dcbbac55a06295ce870b07029bfcdb2dce28d959f2815b16f81798)` describes any P2PK, P2PKH, P2WPKH, or P2SH-P2WPKH output with the specified public key.
- `sh(wsh(pkh(02e493dbf1c10d80f3581e4904930b1404cc6c13900ee0758474fa94abe8c4cd13)))` describes a P2SH-P2WSH-P2PKH output with the specified public key.
- `multi(1,022f8bde4d1a07209355b4a7250a5c5128e88b84bddc619ab7cba8d569b240efe4,025cbdf0646e5db4eaa398f365f2ea7a0e3d419b7e0330e39ce92bddedcac4f9bc)` describes a bare *1-of-2* multisig output.
- `sh(multi(2,022f01e5e15cca351daff3843fb70f3c2f0a1bdd05e5af888a67784ef3e10a2a01,03acd484e2f0c7f65309ad178a9f559abde09796974c57e714c35f110dfc27ccbe))` describes a P2SH *2-of-2* multisig output.
- `wsh(multi(2,03a0434d9e47f3c86235477c7b1ae6ae5d3442d49b1943c2b752a68e2a47e247c7,03774ae7f858a9411e5ef4246b70c65aac5649980be5c17891bbec17895da008cb,03d01115d548e7561b15c38f004d734633687cf4419620095bc5b0f47070afe85a))` describes a P2WSH *2-of-3* multisig output.

## Reference

Descriptors consist of several types of expressions. The top level expression is always a `SCRIPT`.

`SCRIPT` expressions:
- `sh(SCRIPT)` (top level only): P2SH embed the argument.
- `wsh(SCRIPT)` (not inside another 'wsh'): P2WSH embed the argument.
- `pk(KEY)` (anywhere): P2PK output for the given public key.
- `pkh(KEY)` (anywhere): P2PKH output for the given public key.
- `wpkh(KEY)` (not inside `wsh`): P2WPKH output for the given compressed pubkey.
- `combo(KEY)` (top level only): an alias for the collection of `pk(KEY)` and `pkh(KEY)`. If the key is compressed, it also includes `wpkh(KEY)` and `sh(wpkh(KEY))`.
- `multi(k,KEY_1,KEY_2,...,KEY_n)` (anywhere): k-of-n multisig script.
- `addr(ADDR)` (top level only): the script which ADDR expands to.
- `raw(HEX)` (top level only): the script whose hex encoding is HEX.

`KEY` expressions:
- Hex encoded public keys (66 characters starting with `02` or `03`, or 130 characters starting with `04`).
- WIF encoded private keys.
- `xpub` encoded extended public key or `xprv` encoded private key.

`ADDR` expressions are any type of supported address:
- P2PKH addresses (base58, starting with `M...`).
- P2SH addresses (base58).
- Segwit addresses (bech32, starting with `mex1...`).
