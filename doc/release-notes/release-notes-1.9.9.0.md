Megacoin Core version 1.9.9.0 is now available from:

  <https://megacoin-mec.cc>

This is a major release introducing the updated Megacoin Core codebase rebased on Bitcoin Core 0.17 base, introducing the Mega-Mec hashing algorithm and Masternodes system.

Features & Changes
------------------
- **Codebase Rebase**: Rebased core codebase onto Bitcoin Core 0.17 architecture.
- **Mega-Mec Hashing**: Implemented custom Mega-Mec hashing algorithm (`mega-mec.h`).
- **Masternodes**: Introduced full Masternode network capabilities with 4,200 MEC collateral requirements.
- **Bitcore Diffshield**: Added Bitcore Diffshield difficulty adjustment algorithm.
- **OP_RETURN Data Limit**: Extended OP_RETURN payload capacity to 220 bytes.
- **SegWit & Bech32**: Prepared SegWit network support with native `mex1...` Bech32 address formatting.
- **CI / Automation**: Configured continuous integration with Travis CI.
