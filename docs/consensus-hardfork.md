> Long-form analysis referenced from issue #33. The issue itself is the short version.

# Consensus Hard Fork Proposal: Bounded Emergency Difficulty & Strict Masternode Payment Enforcement

**Status:** Long-form analysis. The issue is the short version.

---

## Part 1: Bounded Emergency Difficulty Adjustment

### Problem 1

Megacoin's mainnet is 100% proof-of-work: masternodes do not mine, they only vote on node
payments, governance, InstantSend and PrivateSend. The difficulty retarget only runs when a
block is actually found. If network hash power drops or disappears, no block is ever produced
and the chain freezes permanently with no recovery path.

### Expected behavior

A bounded, time-triggered emergency difficulty adjustment: after a sustained period with no
blocks, required work drops gradually and with a hard floor — enough for the chain to resume,
not enough to make low-difficulty mining attractive or exploitable.

### Current behavior

- `fPowAllowMinDifficultyBlocks = false` on mainnet (`src/chainparams.cpp:114`).
- The previous "dead lock protection" was deliberately removed in `255f91ff` ("Remove KGW",
  2020-05-08); the block remains commented out in `src/pow.cpp:25-49`.
- The V2 retarget (`src/pow.cpp:163-269`, active since fork2 = block 21000) adjusts difficulty
  every 64 blocks (`DifficultyAdjustmentIntervalV2() = nPowTargetTimespanV2 / spacing`, where
  `nPowTargetTimespanV2 = 160 * 60` and mainnet spacing = 150 s, `src/chainparams.cpp:112-113`),
  clamped to `1/1.15 .. 1.15×`, and only when a block exists. It cannot act on a zero-activity
  chain: no block, no retarget.
- Consensus check is exact-equality (`src/validation.cpp:3512`:
  `block.nBits != GetNextWorkRequired(...)`), so any change to the difficulty algorithm is a
  **hard fork** and must activate at a fixed height that all nodes and miners upgrade to in
  advance.

### Why the removed code cannot be restored verbatim

`src/pow.cpp:25-49` has three properties that make a straight restore unsafe:

1. **Instant drop to the floor**: with Megacoin's current difficulty regime (bits `0x1d00xxxx`
   vs powLimit `0x1d00ffff`, i.e. difficulty ~1.2-1.8 against a floor of ~1.0), the old code's
   own clamp `if (fPowAllowMinDifficultyBlocks || (bnPowLimit >> nHalvings) < bnBits)
   bnBits = bnPowLimit;` triggers on the *first* halving: after any sustained stall the
   difficulty jumps **directly to the proof-of-work limit** in a single step, and stays there
   for at least a full V2 window (64 blocks) once mining resumes. The "hundreds of halvings to
   near-zero difficulty" scenario only exists when difficulty sits far above the floor; the
   realistic failure mode on mainnet today is an abrupt, whole-window floor storm.
2. **Timestamp-triggered**: it compares `pblock->GetBlockTime()` against the previous block's
   time directly. Miners control the timestamp within MTP bounds, so backdated timestamps can
   trigger the drop on demand and keep mining easy blocks (timewarp).
3. **Dead dependency**: it references `DarkGravityWave()`, whose definition no longer exists
   anywhere in the tree (only the commented call survives at `src/pow.cpp:26`), and
   `ALGO_ACTIVE_COUNT` (`src/primitives/block.h:31`). Restoring the block means
   **reimplementing DarkGravityWave**, not merely redeclaring a constant.

### Observed evidence (mainnet, October 2026)

- Block production degrades periodically down to 6-15 blocks/hour.
- Gaps between consecutive blocks in a 60-block sample: 848 s, 964 s, 993 s, 1243 s, 1399 s.
  Five gaps near or above 15-23 minutes in one short window.
- Difficulty oscillates between ~1.18 and ~1.80 while network hashrate fluctuates between
  ~10 and ~50 MH/s (single dominant pool; when it reduces hash, production collapses).

**Consequence for the trigger choice:** a 20-minute trigger would already fire several times
per thousand blocks in today's *normal* degraded regime, turning the emergency path into the
default operating mode and gluing difficulty to the floor. The trigger must be set above the
worst observed downtime gaps, not below them.

### Proposal 1

Add a bounded emergency adjustment to `GetNextWorkRequired`, activated by a fixed hard-fork
height (testnet first, all nodes and miners upgraded in advance):

- **Trigger**: no block for a fixed period measured against the validated chain/index time,
  not the raw candidate timestamp. Recommended baseline: **45-60 minutes** (18-24× the 150 s
  spacing) — well above the ~23-minute worst gap observed in October 2026, so the emergency
  path stays an emergency. Exact value is an open question (see below).
- **Drop bounded per block**: one step (e.g. 1/1.15 relative to the last normal target) per
  triggered block, with a hard floor: never below the proof-of-work limit, and no more than N
  halvings from the last normal target.
- **No backdated timestamps**: the candidate header time must not be older than the previous
  block's time (already enforced) and the emergency path must not accept a time older than
  median-time-past.
- The normal V2 retarget stays untouched below the fork height.

---

## Part 2: Strict Masternode Payment Enforcement

### Problem 2

Under the current implementation (`src/masternode-payments.cpp`), masternode payment checks
rely on `CMasternodePayments::IsTransactionValid()`. If fewer than 6 P2P vote signatures
(`MNPAYMENTS_SIGNATURES_REQUIRED = 6`) exist for a block height, `IsTransactionValid()`
returns `true` ("approve whichever is the longest chain").

Because P2P vote broadcasting is inactive in my v1.10.0 network, the 6-vote threshold is never
reached. As a result, miners can bypass masternode payments entirely and claim 100% of the
coinbase reward (at this moment 1.40625 MEC instead of splitting 0.703125 MEC with the masternode). Upgraded
nodes currently accept these 100%-miner blocks as valid.

### Expected behavior

Masternode payment enforcement should be mandatory at consensus level once activated: every valid
block MUST contain a masternode payout output of `GetMasternodePayment()` (50% of the block value,
i.e., 0.703125 MEC) paid to the legitimate masternode payee determined by the local queue fallback,
regardless of P2P vote count. Any block omitting the masternode payout must be rejected by consensus.

### Proposal 2

At the same hard-fork height:

1. Modify `IsBlockPayeeValid()` / `IsTransactionValid()` in `src/masternode-payments.cpp`: when
   `IsMasternodePaymentEnforced()` is active, remove the "return true if < 6 votes" escape hatch.
2. Require that the coinbase transaction contains a valid output of at least `GetMasternodePayment()`
   matching the queue-selected masternode payee (`GetNextMasternodeInQueueForPayment()`).
3. Reject any block that attempts to allocate 100% of the coinbase to the miner.

---

## Why these must be consensus-level (and not a spork)

- **Emergency Difficulty**: Sporks cannot rescue a frozen chain: without blocks there is no block
  propagation and no way to distribute a spork activation.
- **Strict Masternode Payments**: Relaxing vote quorum requirements to reject non-paying miner
  blocks causes upgraded nodes to reject blocks that legacy nodes accept, creating a chain split if
  miners do not upgrade. Therefore, both changes must be bundled into a single coordinated Hard Fork.

## Coordination note & P2P Protocol Bump

My current v1.10.0 release ships the masternode-payment enforcement soft fork
that was merged upstream (height-based plan at `consensus.nMasternodeEnforcementHeight
= 2200000`, and a BIP9 `mn_enforcement` deployment, bit 2, active on mainnet since block
2096000). Any difficulty & enforcement hard fork must be announced and coordinated with that line
so the network does not end up with two live consensus rule sets during the transition.

### Protocol Version Bump (`PROTOCOL_VERSION 70007`)

To prevent network partitioning and peer poisoning between upgraded and legacy nodes during this Hard Fork:
- The Hard Fork release must bump `PROTOCOL_VERSION` from `70006` to `70007` (and enforce `MIN_PEER_PROTO_VERSION = 70007` at the fork height via `SPORK_MEGACOIN_16_MIN_PEER_PROTO_VERSION` or hard-coded consensus gate).
- This ensures upgraded nodes disconnect and reject P2P connections from unupgraded `70006` nodes that do not enforce the new rules, guaranteeing a clean network split at the fork height.

Additionally, existing reorg protection (`src/validation.cpp:3668-3669`, checkpoint +
`nlastValidPowHashHeight`) already bounds deep reorganizations below the last validated POW-hash
height — the "subsidy windfall on deep reorg" concern above that line remains valid, but the
actual blast radius should be stated against the current checkpoint value on master.

## Environment

- Repository: LIMXTEC/Megacoin, mainnet `master`, post v1.10.0 re-release.
- Removal commit: `255f91ff31f2fcad5e6cf379596031e4c01a9a11` ("Remove KGW", 2020-05-08).
- Rebuild/upload commit: `b3cfc58a779a53768e6360afc3b971355a70b392` ("Upload Rebuild Megacoin",
  2019-12-26).
- V2 retarget: `src/pow.cpp:163-269`; disabled KGW block: `src/pow.cpp:25-49`; consensus check:
  `src/validation.cpp:3512`; min-difficulty flag: `src/chainparams.cpp:114` (mainnet `false`).
- Masternode enforcement logic: `src/masternode-payments.cpp` (`IsBlockPayeeValid`, `IsTransactionValid`).

## Alternatives

- **A. Restore KGW verbatim**: minimal diff, but brings the instant floor-drop and timewarp risks
  above, and still requires reimplementing `DarkGravityWave`; not recommended without a cap.
- **B. Bounded restoration + Strict MN Payment Enforcement (proposal above)**: comprehensive, safe.
- **C. Do nothing**: document the risks; the network stays one permanent outage away from death and
  vulnerable to miner reward hijacking.

## Open questions for the maintainer

1. Preferred difficulty mechanism: B (recommended) or A-with-cap?
2. Trigger threshold (45-60 min?) and drop floor (max halvings / ratio per block)?
3. Consensus enforcement mechanism for masternode payees (queue fallback vs deterministic list)?
4. Protocol version bump (`70007`) transition schedule and minimum P2P peer version enforcement?
5. Activation height and testnet plan for the combined Hard Fork?
6. Coordination path with the v1.10.0 enforcement line already in the released code?