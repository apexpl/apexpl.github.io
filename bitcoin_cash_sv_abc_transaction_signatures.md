# Signing Bitcoin Cash / SV / ABC Transactionswith (SIGHASH_FORKID

I have a good deal of experience with the bitcoin protocol, but was recently tasked with developing a system for Bitcoin SV, which changes the way transactions are signed.  There unfortunately isn't much in the way of documentation on this except [this page on Github](https://github.com/bitcoincashorg/bitcoincash.org/blob/master/spec/replay-protected-sighash.md), and needless to say, it took longer than I would have liked to get signing and sending to work.  I hope this guide serves useful for anyone developing wallet software that sends via the Bitcoin Cash / SV / ABC networks.

This guide assumes you have working knowledge of the bitcoin protocol already.  To keep things simple, all examples will be in hex format, so if you're working with binary just use your favorite languages functions to translate hex into binary.  A huge shout out to the developers of the [Bitcash Python project](https://github.com/sporestack/bitcash), as their codebase is how I managed to learn this.


## The Short and Skinny

To sign a BTC transaction, you do the following:

1. Create the tx with blank inputs, and append '01000000' to it for signing purposes.
2. Iterate through the inputs, add the sig script (redeem for multisig, public key otherwise) to the input.
3. Create double SHA256 hash of the tx, sign it.  Use the resulting signature to create the DER signature, and place within the appropriate place in the tx for that input.
4. Done, and broadcast tx.

This changes with BCH as follows:

- The end of the DER signature within BTC transactions is '01'.  This needs to be changed to '41', so it has the 0x40 SIGHASH_FORKID.
- The way the hash you need to sign is generated is completely different, so totally disregard how it's generated within BTC transactions.  You no longer use the "blank" tx, nor do you even use public keys / redeem scripts.  See next section for details.
- Everything else stays the same.  Sign the hash, encode the DER signature, place it within the tx at the same location for each input, and broadcast the tx.


## Generating the SHA256 Hash

Although I can't claim to know why, the hash that needs to be signed for BCH transactions is a double SHA256 hash of a concatenated string with the following elements:


Element | Type | Notes
------------- |------------- |------------- 
Version | int32_t | Will be "02000000".
Prev Outputs Hash | Double SHA256 hash | Create a concatenated string that consists of the out points of all inputs within the tx being signed.  Each out point is (reversed txid (char32) + vin (uint32_t).  Double hash the resulting string via SHA256.
Sequences Hash | Double SHA256 hash | A concatenated string of the sequences (generally 'ffffffff') of all inputs within the tx being signed.  Double hash the resulting string via SHA256.
TxID | char(32) | Reversed txid of the input being signed.
Vin | uint32_t | The vin of the input being signed.
Sig Script Length | var_int | The length of the signature script of the input being signed.
Sig Script | uchar() | The signature script of the input being signed.
Input Amount | int64_t | The amount of the input being signed.
Input Sequence | uint32_t | The sequence of the input being signed, generally 'ffffffff'.
Outputs Hash | Double SHA256 hash | A concatenated string of all outputs within the tx being signed.  Each output consists of (amount (int64_t) + scriptsig_length (var_int) + scriptsig (uchar)).  Double hash the resulting string via SHA256.
Locktime | uint32_t | The locktime of the tx being signed.
Hash Type | uint32_t | The hash type.  For BCH, this will always be "41000000" (0x01 + 0x40).

Create a concatenated string with the above elements, then double hash the string via SHA256.  This is the hash that must be signed with the 
private key, which is then placed within the input of the tx at the same location as always.


## DER Signature

This guide assumes you already know how to encode the DER signature, and will not go into detail on it.  As you know, a DER signature looks like:

Element | Note
------------- |------------- 
&nbsp; | 0x30 to signify a DER signature.
length | The length of the entire DER signature.
&nbsp; | 0x02 to signify beginning of R.
R Length | The length of the R value.
R | The R value of the signature.
&nbsp; | 0x02 to signify the beginning of S.
S Length | The length of the S value.
S | The S value of the signature.
Hash Type | The hash type, for BTC transactions this will be 0x01.  For BCH transactions though, this changes to 0x41.

The DER signature is still generated the exact same way as with BTC transactions, the only difference being the end of the 
DER signature changes from 0x01 to 0x41.  That's it.


## Example

For an example, we will be signing a transaction with the following inputs and outputs:

~~~
{
    "inputs": [
        {
            "txid": "66829a9aba4d6548893af42ad81d445fab035dc5b123fc199da7d8d487da8b1d",
            "vin": 0,
            "amount": 0.001,
            "address": "mphVs2gGCHhZ1eW24CtLkYVLmj3yZtBA6p",
            "sequence": "ffffffff"
        },
        {
            "txid": "1be1a6f5873ca1e656b999b07ee0e74c7e1efbc4d6a973174e0953692445a16e",
            "vin": 1,
            "amount": 0.001,
            "address": "mvcKh1GYs234FqvQK23MvQ6FRd6gwBQHao",
            "sequence": "ffffffff"
        }
    ],
    "outputs": [
        {
            "address": "n18WhospQpmfNnKZYQe8PM7sRhM3Xm2mk9",
            "amount": 0.0015,
            "sigscript": "76a914d72400b0f606548e1a618b150388cfac371f243c88ac"
        },
        {
            "address": "mgDt5TD4yaC3nAexFC45D5gp4mxKKrPNG8",
            "amount": 0.000203,
            "sigscript": "76a91407bd88b6baf048869b5a8f41e479e470271f776988ac"
        }
    ]
}
~~~


We will generate the necessary SHA256 hash that needs to be signed for the second input.  The values of the elements from the above table in hex format will become:

Element | Hex Value
------------- |------------- 
Version | 02000000
Prev Outputs Hash | 5717eac77c25c5bdb9372361cecbc1b53a5d57c046722073cddc51ba6a9caa36
Sequences Hash | 752adad0a7b9ceca853768aebb6965eca126a62965f698a0c1bc43d83db632ad
TxID | 6ea145246953094e1773a9d6c4fb1e7e4ce7e07eb099b956e6a13c87f5a6e11b
Vin | 01000000
Sig Script Length | 19
Sig Script | 76a914a58dd3a9b590600967b6b9523e3adfca8744523a88ac
Input Amount | a086010000000000
Input Sequence | ffffffff
Outputs Hash | 96c5528b809d169f4b136505909f4571b709415d5361d14912dd93ada29e7625
Locktime | 00000000
Hash Type | 41000000

Some notes regarding the above values.

- The 'Prev Outputs Hash' is a double SHA256 hash of the string (txid1 + vin1 + txid2 + vin2): `1d8bda87d4d8a79d19fc23b1c55d03ab5f441dd82af43a8948654dba9a9a8266000000006ea145246953094e1773a9d6c4fb1e7e4ce7e07eb099b956e6a13c87f5a6e11b01000000`
- The 'Sequences Hash' is a double SHA256 hash of the string: `ffffffffffffffff`.
- The 'Outputs Hash' is a double SHA256 hash of all outputs within the tx, which is: `f0490200000000001976a914d72400b0f606548e1a618b150388cfac371f243c88ac4c4f0000000000001976a91407bd88b6baf048869b5a8f41e479e470271f776988ac`

You then generate a double SHA256 hash of the above large string, and as an end result will have:

`92550cb93bc172642437be5c4ef234edddcee133b74dbe83cd00a462f6e71b49`

This is the hash that needs to be signed with the private key, and included within the tx for the input.


## About the Author

Matt Dizak is a software entrepreneur with 19 years of high paced experience within the online software industry, and is the 
creator of Apex, an open source PHP based software platform at [https://apexpl.io/](https://apexpl.io/).  Please feel free to 
follow me on [Twitter](https://twitter.com/ApexPlatform), [Youtube](https://www.youtube.com/channel/UCXl-chb10kVE47aln-xUmog), or feel free to e-mail me 
directly anytime at matt.dizak@gmail.com.





