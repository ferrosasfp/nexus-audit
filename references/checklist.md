# NexusAudit — Static Analysis Checklist

## Phase 2A — Automated Static Analysis (Run First)

### Required Tools
```bash
# Slither (Trail of Bits) — primary static analyzer
slither . --json slither-report.json

# Aderyn (Cyfrin) — secondary static analyzer
aderyn
```

### Processing Automated Results
For EACH tool finding, classify:

| Classification | Meaning | Action |
|---|---|---|
| VALID | Real vulnerability confirmed in code | → Phase 6A (write PoC) |
| FALSE-POSITIVE | Tool flagged but code is safe (explain why) | → Document reasoning, skip |
| NOT-APPLICABLE | Valid pattern but doesn't apply here (e.g., fee-on-transfer for USDC-only) | → Document reasoning, skip |

### Slither Common Detectors to Watch
| Detector | Severity | What it finds |
|---|---|---|
| `reentrancy-eth` | HIGH | ETH-based reentrancy |
| `reentrancy-no-eth` | MEDIUM | State-only reentrancy |
| `uninitialized-state` | HIGH | Uninitialized storage variables |
| `arbitrary-send-eth` | HIGH | Unconstrained ETH transfers |
| `controlled-delegatecall` | HIGH | User-controlled delegatecall |
| `suicidal` | HIGH | Unprotected selfdestruct |
| `locked-ether` | MEDIUM | Contract receives ETH but can't withdraw |
| `divide-before-multiply` | MEDIUM | Precision loss |
| `unchecked-transfer` | HIGH | Missing return value check |
| `missing-zero-check` | LOW | Missing address(0) validation |

---

## Phase 2B — Manual Checklist (Run After Automated)

Run every item. Mark: ✅ SAFE | ⚠️ REVIEW | 🔴 FINDING

Focus on what automated tools CANNOT detect: business logic, semantic errors, design flaws.

### Reentrancy
- [ ] All functions that transfer ETH/tokens use ReentrancyGuard or CEI pattern
- [ ] No state changes AFTER external calls (without guard)
- [ ] Cross-function reentrancy: can function A re-enter function B sharing state?
- [ ] Cross-contract reentrancy: can external contract re-enter via callback?
- [ ] Read-only reentrancy: can view functions return inconsistent state during execution?

### Arithmetic
- [ ] Solidity version >= 0.8.0 (overflow protection built-in)
- [ ] Are there `unchecked{}` blocks? If yes, are they safe?
- [ ] Division before multiplication (precision loss)?
- [ ] Rounding: does rounding always favor the protocol, never the attacker?
- [ ] Fixed-point math: correct scaling factors used?
- [ ] MulDiv ordering: `(a * b) / c` vs `a * (b / c)` — which is safer?

### Access Control
- [ ] Every sensitive function has appropriate modifier
- [ ] `address(0)` checked in all setters that accept addresses
- [ ] Owner/operator transfer is two-step (Ownable2Step)
- [ ] No function callable by address(0) as msg.sender
- [ ] Initializer functions protected against re-initialization

### External Calls
- [ ] Return values from external calls checked
- [ ] SafeERC20 used for all ERC-20 interactions
- [ ] External contracts are known/trusted/immutable
- [ ] Is there a risk of `address.call` returning false silently?
- [ ] Returndata from low-level calls bounded (returnbomb protection)?
- [ ] Gas forwarded in low-level calls controlled?

### Integer Edge Cases
- [ ] What happens when amount = 0? (division by zero, empty loops)
- [ ] What happens when amount = type(uint256).max?
- [ ] What happens when arrays are empty?
- [ ] What happens when arrays have length 1 vs. length 1000?

### Events
- [ ] All state changes emit events
- [ ] Events emitted AFTER state changes (not before)
- [ ] Events contain enough data to reconstruct state off-chain

### Initialization
- [ ] Constructor sets all critical state variables
- [ ] No uninitialized storage pointers
- [ ] If upgradeable: `_disableInitializers()` in constructor
- [ ] If upgradeable: `initializer` modifier on initialize function
- [ ] If upgradeable: storage gaps for future variables (`uint256[50] private __gap`)

### Logic
- [ ] Are there functions that can be called in wrong order?
- [ ] Are there time-based conditions? Can block.timestamp be manipulated? (±15s on mainnet)
- [ ] Are there off-by-one errors in comparisons (> vs >=)?
- [ ] Dead code or unreachable branches?
- [ ] State machine transitions: can state skip steps or go backwards?

### ERC Compliance
- [ ] ERC-20: transfer/transferFrom return bool handled
- [ ] ERC-20: approve race condition mitigated (if applicable)
- [ ] ERC-721: safeTransferFrom callback handled
- [ ] ERC-4626: first depositor / share inflation attack mitigated?
- [ ] ERC-3009: signature validation parameters complete
- [ ] ERC-712: domain separator includes chainId and contract address?
- [ ] Chainlink Automation: checkUpkeep view, performUpkeep callable by anyone safely

### Gas & Loops
- [ ] No unbounded loops over user-controlled arrays
- [ ] Storage reads inside loops cached to memory
- [ ] No string operations inside loops
- [ ] Batch operations have size limits
- [ ] No returnbomb risk from unbounded returndata copy

### Centralization
- [ ] Document all privileged roles and their powers
- [ ] Worst-case: what can each role do if malicious?
- [ ] Are there timelocks on critical operations?
- [ ] Is there a pause mechanism?
- [ ] Can a single role drain all funds?

### Proxy / Upgrade Safety (if applicable)
- [ ] Storage layout consistent between versions (`forge inspect`)
- [ ] No storage collision with proxy admin slots
- [ ] UUPS: `_authorizeUpgrade` properly access-controlled
- [ ] Implementation cannot be initialized directly
- [ ] No selfdestruct/delegatecall in implementation
- [ ] Gap variables present for inheritance chain
