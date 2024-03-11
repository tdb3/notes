## Mempool.dat v1/v2 Format

Current as of 6dda050865e298e2e3d5f23a8435981d1e017dbe.

Unless the user specifies `-persistmempool=0` when starting bitcoind, the mempool will be saved to `mempool.dat` on stop (and `savemempool` RPC).

[PR 28207](https://github.com/bitcoin/bitcoin/pull/28207) added a new v2 mempool file format.

In [DumpMempool()](https://github.com/bitcoin/bitcoin/blob/6dda050865e298e2e3d5f23a8435981d1e017dbe/src/kernel/mempool_persist.cpp#L149), the file will start with the 64-bit int version (either `MEMPOOL_DUMP_VERSION_NO_XOR_KEY` or `MEMPOOL_DUMP_VERSION` when `persistmempoolv1=1` or not set, respectively).

The new version includes an XOR key to help protect against outside programs that accidentally mess with mempool.dat.

