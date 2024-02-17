## UTXO Set Compression
[PR #28193](https://github.com/bitcoin/bitcoin/pull/28193) is concerned with adding a unit test for a particular edge case for compression/decompression of P2PK outputs.

Bitcoin Core currently (at the time of writing) is capable of compressing (some types of) output scripts for UTXOs, such that the amount of storage needed to store those outputs is smaller (i.e. in Chainstate).  It appears to do this through a bit of implicit and explicit marking.  Not sure of the history here, but this technique and the smaller SPK of SegWit transactions seem to be "cousins" in a sense.

[compressor.h](https://github.com/bitcoin/bitcoin/blob/master/src/compressor.h#L43-L63)
```cpp
/** Compact serializer for scripts.
 *
 *  It detects common cases and encodes them much more efficiently.
 *  3 special cases are defined:
 *  * Pay to pubkey hash (encoded as 21 bytes)
 *  * Pay to script hash (encoded as 21 bytes)
 *  * Pay to pubkey starting with 0x02, 0x03 or 0x04 (encoded as 33 bytes)
 *
 *  Other scripts up to 121 bytes require 1 byte + script length. Above
 *  that, scripts up to 16505 bytes require 2 bytes + script length.
 */
struct ScriptCompression
```

Amounts can also be compressed with `AmountCompression` since it's usually the case that amounts are far smaller than the max amount size (e.g. extra zeros are redundant).

`CompressScript()` does SPK compression by calling helper functions (e.g. `IsToKeyID`, `IsToScriptID`, `IsToPubKey`) to both check that an SPK follows a very specific pattern (e.g. 25 bytes for P2PKH, `OP_DUP OP_HASH160 <20 bytes> < 20-byte PKH> OP_EQUALVERIFY OP_CHECKSIG`), and extract out the critical piece needed for the compressed storage (in this example, the 20-byte PKH).

`DecompressScript()` reconstructs the full scripts from the provided explicit marking:
 - `0x00` is associated with P2PKH, and the 20 bytes provided in `in` will be part of a reconstructed P2PKH script (e.g. `OP_DUP OP_HASH160 <20 bytes to follow> <20 byte PKH> OP_EQUALVERIFY OP_CHECKSIG`)
 - `0x01` is associated with P2SH
 - `0x02`/`0x03` denote P2PK, where the public key in the output should be stored as an SEC compressed pubkey (i.e. 33 bytes)
 - `0x04` indicates P2PK where the pubkey is in SEC uncompressed (i.e. 65 bytes).  `0x04` indicates that the even Y value should be used when decompressing, and `0x05` indicates that the odd Y value should be used.

 The PR was concerned with ensuring that P2PK compression and decompression properly handled situations where the X value of the pubkey wasn't on the secp256k1 curve (and therefore there wouldn't be a corresponding Y value).  As an aside, these outputs aren't spendable (barring something wild like DLog or ECDSA being broken, but more thought would need to be explored there).

