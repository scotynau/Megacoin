Megacoin Core version 1.10.0 is now available from:

  <https://megacoin-mec.cc>

This major release introduces enforced Masternode block rewards, automatic peer banning for invalid payments, active collateral spent purging, perpetual SegWit/CSV rules, updated seed nodes, and C++17/Boost 1.76+ build compatibility.

Release Updated
---------------

**This release was modified and its binaries were replaced.** The originally
published `v1.10.0` artifacts contained a bug in block verification that could
cause a node to crash (`RewindBlockIndex` / `nChainTx` assertion failure) or to
mishandle valid blocks during initial block download (IBD) and chain reindex.

All platform artifacts (Linux x86_64, Windows, and Linux AArch64) and the
official Docker image have been rebuilt from the fixed source and re-uploaded.
Users who downloaded `v1.10.0` before this update should re-download and replace
their binaries.

Summary of Changes
------------------

### Masternode Enforcement
- **Enforced Payment Rules (BIP9 Soft-Fork)**: Blocks that do not pay the valid winner determined by the Masternode network are rejected once activated.
- **Banning Misbehaving Peers**: Nodes submitting blocks with invalid Masternode payments trigger a DoS 100 ban score.
- **Spent Collateral Purging**: Masternodes whose 4,200 MEC collateral inputs are spent are immediately removed from the active Masternode list.

### Consensus & Deployment
- **Perpetual SegWit & CSV**: Set SegWit (BIP141/143/147) and CSV (BIP68/112/113) deployments to ALWAYS_ACTIVE (`startTime = 0`, `timeout = 999999999999ULL`).

### Build System & Toolchain
- **Modern Compiler Support**: Fixed compilation issues under GCC 11, 12, 13, 14, and recent Clang toolchains.
- **Boost 1.76+ Compatibility**: Updated dependencies and build scripts for Boost 1.76+.
- **MinGW & Depends Updates**: Updated ZeroMQ (4.3.4), Qt (5.9.6), and MinGW build configurations.

### Network
- **Seed Nodes**: Updated DNS seeds and hardcoded bootstrap nodes for improved P2P connectivity.
