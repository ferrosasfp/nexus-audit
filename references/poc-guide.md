# NexusAudit — PoC Test Guide (Phase 6)

## Purpose

Every CONFIRMED finding needs a Foundry test that proves the attack is executable.
A passing test = empirical evidence. No test = the finding is LIKELY at best.

## Test Structure

```solidity
/**
 * TEST [FINDING-ID] — [Title]
 * Hypothesis: [one sentence describing the attack]
 *
 * If test PASSES → finding REAL (attack executable)
 * If test FAILS  → investigate: bug in test OR finding is wrong
 */
function test_[FINDING_ID]_[Description]() public {
    // ARRANGE: set up the preconditions
    // ACT: execute the attack
    // ASSERT: prove the impact happened
    // LOGS: emit useful debug info with emit log_named_uint(...)
}
```

## The Failure Protocol

When a PoC test fails, do NOT immediately discard the finding. Follow this:

1. **Read the revert reason** — what exactly failed?
2. **Check the assertion** — was the arithmetic in `assertEq` correct?
3. **Check the attack path** — did you miss a step?
4. **Check the contract** — does the contract already mitigate this?

Only after all 4 checks → decide: fix test OR downgrade finding.

**Real example (WasiAI NA-H02):**
- Test failed: `900001 != 899999`
- The contract DID accept arbitrary amounts (visible in trace)
- The bug was in the expected value calculation
- Verdict: fix test, finding survives as CONFIRMED

## What Makes a Good PoC

**Good:** Minimal setup, clear attack path, meaningful assert
```solidity
// GOOD: proves treasury got more than it should
assertGt(usdc.balanceOf(treasury), expectedWithOriginalFee);
```

**Bad:** Over-complicated setup that obscures the attack
```solidity
// BAD: 50 lines of setup before 1 line of attack
```

**Good:** Logs that tell the story
```solidity
emit log_named_uint("Fee at deposit (bps)", feeAtDeposit);      // 1000
emit log_named_uint("Fee at settle (bps)",  feeAtSettle);        // 3000
emit log_named_uint("User expected to pay", expectedFee);        // 10000
emit log_named_uint("User actually paid",   actualFee);          // 30000
```

## One Test Per Finding

Each finding gets its own test function. Never bundle two findings in one test —
if it fails, you won't know which finding caused it.

## PoC Test File Convention

Name the file: `[ContractName]_NexusAudit.t.sol`
All PoC tests in one file per audited contract.

## After All Tests Pass

Update the report: change `LIKELY` → `CONFIRMED` for each passing test.
Include test name in the finding: `PoC: test_NA_M01_performUpkeep_Blocks_EmergencyExit ✅`
