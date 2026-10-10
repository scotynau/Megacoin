Megacoin Core version 1.10.0 is now available from:

  <https://github.com/LIMXTEC/Megacoin/releases/tag/v1.10.0>

This is a major release of Megacoin Core, introducing a soft-fork for Masternode payment enforcement, security enhancements, compatibility with modern compiler toolchains (GCC 11–14, Boost 1.76+), and updated build dependencies.

Please report bugs using the issue tracker at GitHub:

  <https://github.com/LIMXTEC/Megacoin/issues>

How to Upgrade
==============

If you are running an older version (1.9.9.x or earlier), shut it down completely. Wait until it has completely shut down, then replace your binaries with the new `megacoind`, `megacoin-qt`, `megacoin-cli`, and `megacoin-tx` executables or run the installer (Windows).

Compatibility
==============

Megacoin Core is tested on multiple operating systems including Linux (x86_64, AArch64), Windows 7+ (64-bit), and macOS.

Notable Changes in 1.10.0
=========================

Masternode Payment Enforcement Soft-Fork (BIP9)
-----------------------------------------------
- Implemented a BIP9 soft-fork mechanism for strict Masternode payment enforcement.
- Nodes enforce valid Masternode payment targets in block templates upon soft-fork activation.
- Peers sending blocks with invalid Masternode payments are banned with DoS score 100.
- Spent Masternodes are automatically purged on block tip updates to maintain accurate network state.

Consensus & Feature Flags
-------------------------
- SegWit and CSV soft-forks are configured as ALWAYS_ACTIVE with no timeout bans, ensuring perpetual availability of SegWit and Bech32 address support (`mex1...`).
- Fixed `setBlockIndexCandidates` sanitization in `RewindBlockIndex` to prevent `nChainTx` assertion crashes during chain reorganizations.

Toolchain & Dependency Updates
------------------------------
- Modern C++ Compiler Support: Resolved compilation issues with GCC 11, GCC 12, GCC 13, and GCC 14.
- Boost 1.76+ Compatibility: Updated signals2 syntax, added missing `<deque>` and `<array>` includes.
- Depends System Overhaul:
  - Boost updated to 1.76.0.
  - ZeroMQ updated to 4.3.4.
  - Qt updated to 5.9.6 with MinGW `tagTOUCHINPUT` redefinition fixes and `fix_numeric_limits` patch.
  - BDB patched for AArch64 mutex preprocessor syntax.
  - Added `libbcrypt` dependency and `gmtime_s` fixes for MinGW Windows builds.
  - Fixed `xcb_proto` staging on Python 3.12+ build environments.

Network & Seed Nodes
--------------------
- Updated seed node list and DNS seeds for faster peer discovery.
- MiniUPnPc library updated to 2.2.8.

Credits
=======

Thanks to all contributors who helped test and develop Megacoin Core 1.10.0.
