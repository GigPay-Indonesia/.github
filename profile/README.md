Below is a fully rewritten, comprehensive, professional README focused strictly on how **Escrow + Treasury + Yield Aggregator** compose into one system.

The Thetanuts-specific track content has been removed.
All deployments include direct Blockscout links in structured tables.
There is a detailed section explaining how each deployed contract connects to the others.

You can paste this directly into your repository.

---

# GigPay Protocol

GigPay is a treasury-first, onchain payment and escrow protocol designed for enterprise-grade payouts. It combines three core primitives into one coherent capital system:

1. A company treasury vault that acts as the capital root.
2. A deterministic escrow state machine for payout lifecycle enforcement.
3. An upgradeable ERC4626 yield aggregator that keeps idle capital productive.

Built for Base and optimized for IDRX and stablecoin payouts.

---

# 1. Executive Overview

Traditional payout systems treat each payment as an isolated transfer. Enterprises do not operate this way. They manage working capital, milestone approvals, vendor delays, reconciliation cycles, and liquidity buffers. Funds sit idle between events.

GigPay models payouts as a capital system rather than a transfer function.

The system works as follows:

* A company funds a treasury vault once.
* The treasury funds multiple escrow intents over time.
* Idle balances are optionally deployed into an ERC4626 yield vault.
* Each escrow has clear ownership of principal and yield.
* Settlement is finalized deterministically through escrow state transitions.

The result is a programmable treasury rail that is capital-efficient, auditable, and modular.

---

# 2. Architectural Layers

GigPay consists of three tightly integrated but logically separated layers.

## 2.1 Treasury Layer

The treasury is the capital root of the system.

Primary contract:

* `CompanyTreasuryVault`

Responsibilities:

* Hold company working capital (IDRX in the demo deployment).
* Provide funding for escrow intents.
* Route idle liquidity to the yield layer.
* Maintain operator-level access control.

Design principle:
The company does not re-fund per payout. It maintains a single onchain treasury balance and allocates from it.

---

## 2.2 Escrow Layer

The escrow layer governs payment lifecycle and settlement.

Primary contract:

* `GigPayEscrowCoreV2`

Escrow lifecycle:

1. Create
2. Fund
3. Submit
4. Release or Refund (terminal states)

Escrow properties:

* Deterministic state machine.
* Explicit custody of principal.
* Optional yield activation for long acceptance windows.
* Settlement is self-contained and final.

Design principle:
Escrow is the only component that finalizes payouts.

---

## 2.3 Yield Aggregator Layer (ERC4626)

The yield layer is a separate repository providing a UUPS-upgradeable ERC4626 vault used as the treasury yield buffer.

Primary contracts:

* `BaseVaultUpgradeable` (ERC4626 proxy)
* `VaultStorage`
* `StrategyRegistry`
* `RebalanceManager`
* `WithdrawManager`

Responsibilities:

* Deploy idle capital to strategies.
* Enforce debt caps.
* Perform oracle-assisted rebalancing.
* Guarantee deterministic withdrawals via a liquidation queue.
* Protect against share dilution via locked-profit accounting.

Design principle:
Yield complexity is isolated from payment logic. The escrow and treasury interact with the yield vault through a narrow adapter boundary.

---

# 3. Capital Ownership Model

The system enforces strict ownership boundaries.

While funds are in the treasury:

* Yield belongs to the treasury.

Once principal is committed to escrow:

* Escrow becomes the yield owner for that principal (if escrow yield is enabled).

Settlement rule:

* Treasury yield and escrow yield never mix.
* Yield accounting is finalized during escrow release/refund.
* Principal has exactly one yield owner at any time.

This eliminates cross-contamination between obligations and treasury earnings.

---

# 4. Smart Contract Deployments (Base Sepolia)

Network: Base Sepolia
Explorer: [https://base-sepolia.blockscout.com](https://base-sepolia.blockscout.com)

## 4.1 Core GigPay Contracts

| Contract                 | Address                                    | Blockscout                                                                                                                                                                       |
| ------------------------ | ------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| GigPayRegistry           | 0x32F8dF36b1e373A9E4C6b733386509D4da535a72 | [https://base-sepolia.blockscout.com/address/0x32F8dF36b1e373A9E4C6b733386509D4da535a72](https://base-sepolia.blockscout.com/address/0x32F8dF36b1e373A9E4C6b733386509D4da535a72) |
| GigPayEscrowCoreV2       | 0xd09177C97978f5c970ad25294681F5A51685c214 | [https://base-sepolia.blockscout.com/address/0xd09177C97978f5c970ad25294681F5A51685c214](https://base-sepolia.blockscout.com/address/0xd09177C97978f5c970ad25294681F5A51685c214) |
| CompanyTreasuryVault     | 0xcDfd5B882e8dF41b3EFc1897dAf759a10a7457B8 | [https://base-sepolia.blockscout.com/address/0xcDfd5B882e8dF41b3EFc1897dAf759a10a7457B8](https://base-sepolia.blockscout.com/address/0xcDfd5B882e8dF41b3EFc1897dAf759a10a7457B8) |
| YieldManagerV2           | 0x22c94123e60fA65D742a5872a45733154834E4b0 | [https://base-sepolia.blockscout.com/address/0x22c94123e60fA65D742a5872a45733154834E4b0](https://base-sepolia.blockscout.com/address/0x22c94123e60fA65D742a5872a45733154834E4b0) |
| TokenRegistry            | 0x3Ded1e4958315Dbfa44ffE38B763De5b17690C57 | [https://base-sepolia.blockscout.com/address/0x3Ded1e4958315Dbfa44ffE38B763De5b17690C57](https://base-sepolia.blockscout.com/address/0x3Ded1e4958315Dbfa44ffE38B763De5b17690C57) |
| SwapRouteRegistry        | 0xa85D9Cf90E1b8614DEEc04A955a486D5E43c3297 | [https://base-sepolia.blockscout.com/address/0xa85D9Cf90E1b8614DEEc04A955a486D5E43c3297](https://base-sepolia.blockscout.com/address/0xa85D9Cf90E1b8614DEEc04A955a486D5E43c3297) |
| CompositeSwapManager     | 0x2b7ca209bca4E0e15140857dc6F13ca17B50F50d | [https://base-sepolia.blockscout.com/address/0x2b7ca209bca4E0e15140857dc6F13ca17B50F50d](https://base-sepolia.blockscout.com/address/0x2b7ca209bca4E0e15140857dc6F13ca17B50F50d) |
| Yield Aggregator Adapter | 0x3AD2D40a516eCa9C34DcE24BefD4B8dF50641855 | [https://base-sepolia.blockscout.com/address/0x3AD2D40a516eCa9C34DcE24BefD4B8dF50641855](https://base-sepolia.blockscout.com/address/0x3AD2D40a516eCa9C34DcE24BefD4B8dF50641855) |

---

## 4.2 Yield Aggregator Contracts (Treasury Buffer)

| Contract             | Address                                    | Blockscout                                                                                                                                                                       |
| -------------------- | ------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ERC4626 Vault Proxy  | 0xCFB357fae5e5034cCfA0649b711a2897e685D14a | [https://base-sepolia.blockscout.com/address/0xCFB357fae5e5034cCfA0649b711a2897e685D14a](https://base-sepolia.blockscout.com/address/0xCFB357fae5e5034cCfA0649b711a2897e685D14a) |
| Vault Implementation | 0xEE3D9b2Aa9B91215C75Bfc29Cd006F626Cdcf574 | [https://base-sepolia.blockscout.com/address/0xEE3D9b2Aa9B91215C75Bfc29Cd006F626Cdcf574](https://base-sepolia.blockscout.com/address/0xEE3D9b2Aa9B91215C75Bfc29Cd006F626Cdcf574) |
| Yield Oracle         | 0x826bFa587332002f35fd0240285dEd6C134E95dB | [https://base-sepolia.blockscout.com/address/0x826bFa587332002f35fd0240285dEd6C134E95dB](https://base-sepolia.blockscout.com/address/0x826bFa587332002f35fd0240285dEd6C134E95dB) |

---

## 4.3 Asset Contracts

| Token  | Address                                    | Blockscout                                                                                                                                                                       |
| ------ | ------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| IDRX   | 0x20Abd5644f1f77155c63A8818C75540F770ae223 | [https://base-sepolia.blockscout.com/address/0x20Abd5644f1f77155c63A8818C75540F770ae223](https://base-sepolia.blockscout.com/address/0x20Abd5644f1f77155c63A8818C75540F770ae223) |
| USDC   | 0x44E7c97Ee6EC2B25145Acf350DBBf636615e198d | [https://base-sepolia.blockscout.com/address/0x44E7c97Ee6EC2B25145Acf350DBBf636615e198d](https://base-sepolia.blockscout.com/address/0x44E7c97Ee6EC2B25145Acf350DBBf636615e198d) |
| Faucet | 0x31d563850441a218F742237aF505fb7Fc4198bc7 | [https://base-sepolia.blockscout.com/address/0x31d563850441a218F742237aF505fb7Fc4198bc7](https://base-sepolia.blockscout.com/address/0x31d563850441a218F742237aF505fb7Fc4198bc7) |

---

# 5. How Contracts Connect to Each Other

This section explains the live wiring between deployed contracts.

## 5.1 Registry as the System Router

`GigPayRegistry` acts as the address resolver.

It stores:

* Active YieldManager
* Active SwapManager
* Other module pointers

Escrow and Treasury never hardcode module addresses.
They query the registry.

This enables safe module upgrades.

---

## 5.2 Treasury to Yield

Flow:

1. Treasury holds IDRX.
2. Treasury calls `YieldManagerV2`.
3. YieldManager deposits into ERC4626 vault via adapter.
4. ERC4626 vault allocates across strategies.

Connection path:

CompanyTreasuryVault
→ YieldManagerV2
→ Yield Aggregator Adapter
→ ERC4626 Vault Proxy

Treasury never interacts directly with strategy contracts.

---

## 5.3 Treasury to Escrow

Flow:

1. Operator creates intent via EscrowCore.
2. Treasury funds escrow.
3. Principal custody moves from treasury to escrow.

Connection path:

CompanyTreasuryVault
→ GigPayEscrowCoreV2

Escrow maintains internal accounting for principal.

---

## 5.4 Escrow to Yield

If escrow yield is enabled and acceptance window exceeds threshold:

1. Escrow calls YieldManagerV2.
2. Escrow principal is deposited into ERC4626 vault.
3. Yield accrues.
4. On release/refund, escrow withdraws principal + yield.
5. Escrow finalizes settlement.

Connection path:

GigPayEscrowCoreV2
→ YieldManagerV2
→ ERC4626 Vault

Escrow yield accounting is independent from treasury yield.

---

## 5.5 Swap Routing (Optional)

Escrow may release with swap.

Connection path:

GigPayEscrowCoreV2
→ CompositeSwapManager
→ SwapRouteRegistry

Swap layer is modular and replaceable without touching escrow logic.

---

# 6. Deterministic Settlement

Settlement is always executed via escrow terminal functions:

* `release`
* `releaseWithSwap`
* `refund`

At settlement:

* Principal amount is resolved.
* Yield attributable to escrow is resolved.
* SplitRouter distributes funds.
* State becomes terminal.

No offchain reconciliation is required.

---

# 7. Core Invariants

The composed system enforces:

* One principal has one yield owner at a time.
* Treasury yield and escrow yield are never mixed.
* Escrow settlement is self-contained.
* ERC4626 vault cannot exceed strategy debt caps.
* Withdrawals follow deterministic queue liquidation.
* Strategies cannot mutate vault storage.

If any invariant is violated, the architecture is considered broken.

---

# 8. Testing

```bash
forge test
forge test -vvv
```

Test categories:

* Unit tests per contract.
* Integration tests for Treasury → Escrow → Settlement flows.
* Invariant tests ensuring no stuck funds and accounting integrity.

Key integration tests:

* Flow.TreasuryToEscrow.Release.t.sol
* Flow.TreasuryToEscrow.Refund.t.sol
* Flow.LongAcceptanceWindow.EscrowYieldThenRelease.t.sol
* Flow.LongAcceptanceWindow.EscrowYieldThenRefund.t.sol
* Invariant.NoStuckFunds.t.sol

---

# 9. Development

Requirements:

* Foundry

```bash
forge install
forge build
forge test
```

Solidity version:
0.8.28
Optimizer enabled with 200 runs.

---

# 10. Status
The protocol contains production-grade modular contract architecture and extensive testing.

The yield layer is upgradeable and isolated from escrow logic.

