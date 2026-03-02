# NexusAudit — Fix Loop Guide (Phase 8)

## Fix Type Quick Reference

| Type | Files changed | New logic? | Risk | NexusAgil process |
|---|---|---|---|---|
| FAST-FIX | 1-2 | No | Low | Direct — no Story File |
| HU-MINOR | 3-5 | Simple | Medium | HU_APPROVED → Story File → sub-agent |
| HU-MAJOR | Many | Complex | High | Full F0→SDD→SPEC→F3→AR→F4 |
| KNOWN-LIMITATION | 0 | N/A | Deferred | Document + Linear issue |

---

## What is the Fix Loop?

Most audit methodologies stop at the report. NexusAudit goes one step further:
findings become verified fixes, not just recommendations.

The fix loop is what separates "we found bugs" from "we proved the bugs are gone."

---

## The Three-Layer Test Validation

After every fix, run tests in this exact order:

### Layer 1 — Compile
```bash
forge build
```
Must have 0 errors. If it fails, the fix broke the contract structure.

### Layer 2 — Regression (original tests)
```bash
forge test --match-contract [OriginalTestContract]
```
All must pass. If any fail, the fix introduced a regression — it changed behavior
that was correct before.

### Layer 3 — PoC Inversion
```bash
forge test --match-contract NexusAuditValidation
```
The attack PoC tests must now FAIL (the attack no longer works).
The FIXED regression tests must PASS (correct behavior confirmed).

**All 3 layers green = finding CLOSED.**

---

## How to Invert a PoC Test

Original PoC (attack works — test PASSES):
```solidity
function test_NA_M01_performUpkeep_Blocks_EmergencyExit() public {
    // ... setup ...
    vm.prank(payer);
    vm.expectRevert("WasiAI: operator still active");
    marketplace.emergencyWithdrawKey(KEY_ID);
    // Test passes = attack works = emergency exit blocked
}
```

Inverted PoC (fix applied — attack FAILS, correct behavior works):
```solidity
function test_NA_M01_FIXED_performUpkeep_NoLongerBlocksEmergencyExit() public {
    // Same setup...
    vm.prank(payer);
    marketplace.emergencyWithdrawKey(KEY_ID);  // Must NOT revert
    assertEq(usdc.balanceOf(payer), 1_000_000, "User recovered funds");
}
```

Keep both tests in the file. The original (renamed) documents what was broken.
The FIXED version documents that it's now correct.

---

## Known Limitation Pattern

When a finding cannot be fixed in the current sprint:

```solidity
/**
 * KNOWN LIMITATION — NA-H01
 * The insolvency risk between keyBalances and earnings pools
 * requires architectural redesign (separate contracts for each flow).
 * Scheduled for v2. This test remains active — attack is still possible.
 */
function test_NA_H01_Insolvency_KeyBalances_vs_Earnings() public {
    // ... original attack test, unchanged ...
    // This PASSES intentionally — documents the known limitation
}
```

---

## Fix Quality Rules

**Minimal fix:** Change only what the finding requires. Do not refactor surrounding code.

**Wrong:**
```solidity
// Finding: missing zero-check on setOperator
// Fix attempt: rewrote entire access control system
```

**Right:**
```solidity
// Finding: missing zero-check on setOperator
// Fix: one require() added at the top of the function
require(operator != address(0), "WasiAI: zero operator");
```

**Surgical beats thorough.** Each fix should be reviewable in isolation.

---

## Fix Loop Summary Table

Use this in the final report:

| Finding | Severity | Fix Applied | Layer 1 | Layer 2 | Layer 3 | Status |
|---|---|---|---|---|---|---|
| NA-M01 | MEDIUM | Remove lastOperatorActivity from performUpkeep | ✅ | ✅ | ✅ | CLOSED |
| NA-H02 | HIGH | require(amount == pricePerCall) | ✅ | ✅ | ✅ | CLOSED |
| NA-H01 | HIGH | None (architectural) | N/A | N/A | N/A | KNOWN LIMITATION |

---

## Real-World Example (WasiAI Sprint 9)

**Before fix loop:**
- 15 PoC tests PASSING (15 attacks confirmed executable)
- Contract deployed on Fuji testnet

**After fix loop:**
- 7 PoC tests INVERTED (7 attacks now revert correctly)
- 7 FIXED regression tests PASSING
- 2 KNOWN LIMITATIONS documented (NA-H01 insolvency, NA-M03 fee sandwich)
- 78 total tests, 0 failures
- Contract v7 ready for mainnet consideration

This is the evidence that replaces an external audit when budget is not available.
Not as exhaustive — but empirical, reproducible, and honest.
