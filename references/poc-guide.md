# NexusAudit — PoC + Fuzz Test Guide (Phase 6A & 6B)

## Purpose

Every CONFIRMED finding needs a Foundry test that proves the attack is executable.
A passing test = empirical evidence. No test = the finding is LIKELY at best.

> **v2.0:** Phase 6 is now split into 6A (deterministic PoCs) and 6B (fuzzing + invariant testing).

---

## Phase 6A — Deterministic PoC Tests

### Test Structure

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

### The Failure Protocol

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

### What Makes a Good PoC

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

### One Test Per Finding

Each finding gets its own test function. Never bundle two findings in one test —
if it fails, you won't know which finding caused it.

### PoC Test File Convention

Name the file: `[ContractName]_NexusAudit.t.sol`
All PoC tests in one file per audited contract.

### After All Tests Pass

Update the report: change `LIKELY` → `CONFIRMED` for each passing test.
Include test name in the finding: `PoC: test_NA_M01_performUpkeep_Blocks_EmergencyExit ✅`

---

## Phase 6B — Fuzzing & Invariant Testing

### Why Fuzzing?

Deterministic PoCs prove ONE specific attack path. Fuzzing discovers UNKNOWN paths
by testing with thousands of random inputs. The most expensive DeFi exploits in history
(Euler $197M, Beanstalk $182M, Curve $70M) involved edge cases no human anticipated.

### Step 1: Define Protocol Invariants

From Phase 3 W5 (Invariant Verification Lens), translate each invariant into a
Foundry invariant test:

```solidity
// File: [ContractName]_NexusAudit_Invariants.t.sol

contract ContractInvariantTest is Test {
    ContractUnderTest target;
    InvariantHandler handler;

    function setUp() public {
        target = new ContractUnderTest();
        handler = new InvariantHandler(target);

        // Focus fuzzer on meaningful functions
        targetContract(address(handler));
    }

    /// @dev Protocol must always be solvent
    function invariant_solvency() public view {
        assertLe(
            target.totalUserBalances(),
            token.balanceOf(address(target)),
            "INVARIANT BROKEN: contract is insolvent"
        );
    }

    /// @dev Fee can never exceed maximum
    function invariant_feeWithinBounds() public view {
        assertLe(
            target.platformFeeBps(),
            target.MAX_FEE(),
            "INVARIANT BROKEN: fee exceeds maximum"
        );
    }

    /// @dev Total supply accounting must be consistent
    function invariant_supplyConsistency() public view {
        uint256 sumOfBalances = _sumAllBalances();
        assertEq(
            sumOfBalances,
            target.totalSupply(),
            "INVARIANT BROKEN: balances don't sum to totalSupply"
        );
    }
}
```

### Step 2: Create a Handler Contract

The handler constrains fuzz inputs to realistic ranges:

```solidity
contract InvariantHandler is Test {
    ContractUnderTest target;

    constructor(ContractUnderTest _target) {
        target = _target;
    }

    function deposit(uint256 amount) external {
        // Bound to realistic range
        amount = bound(amount, 1, 1_000_000e6); // 1 to 1M USDC

        deal(address(usdc), msg.sender, amount);
        vm.prank(msg.sender);
        target.deposit(amount);
    }

    function withdraw(uint256 amount) external {
        uint256 balance = target.balanceOf(msg.sender);
        if (balance == 0) return; // skip if no balance
        amount = bound(amount, 1, balance);

        vm.prank(msg.sender);
        target.withdraw(amount);
    }
}
```

### Step 3: Run Invariant Tests

```bash
# Minimum 1000 runs for meaningful coverage
forge test --match-test invariant_ --fuzz-runs 1000

# Deep run for pre-mainnet (slower but more thorough)
forge test --match-test invariant_ --fuzz-runs 10000 --fuzz-seed 42
```

**If an invariant breaks:**
1. Foundry shows the exact call sequence that broke it
2. Create a new deterministic PoC reproducing the sequence
3. Classify as new CONFIRMED finding
4. Add to Phase 7 report

### Step 4: Fuzz Existing PoCs

For each Phase 6A finding, create a fuzz variant to test edge cases:

```solidity
/// @dev Fuzz variant of NA-H02 — test with variable amounts
function test_fuzz_NA_H02_arbitraryAmount(uint256 amount) public {
    // Bound to valid range
    amount = bound(amount, 1, type(uint128).max);

    // Same attack path as deterministic PoC but with random amount
    vm.prank(attacker);
    target.deposit(amount);

    // Assert invariant holds (or breaks) for any amount
    assertEq(
        target.balanceOf(attacker),
        amount,
        "Balance should match deposit"
    );
}

/// @dev Fuzz variant — test fee boundaries
function test_fuzz_feeBoundary(uint16 feeBps) public {
    feeBps = uint16(bound(feeBps, 0, 10000)); // 0% to 100%

    vm.prank(owner);
    target.setFee(feeBps);

    // Verify fee is correctly applied at any value
    uint256 deposit = 1_000_000e6;
    uint256 expectedFee = (deposit * feeBps) / 10000;
    uint256 expectedNet = deposit - expectedFee;

    vm.prank(user);
    target.deposit(deposit);

    assertEq(target.balanceOf(user), expectedNet);
}
```

### Step 5: Run Full Fuzz Suite

```bash
# Run all fuzz tests
forge test --match-test test_fuzz_ --fuzz-runs 500

# Run everything (PoCs + fuzz + invariants)
forge test --match-contract NexusAudit --fuzz-runs 1000
```

### Fuzz Test File Convention

| File | Contents |
|---|---|
| `[Contract]_NexusAudit.t.sol` | Phase 6A — deterministic PoCs |
| `[Contract]_NexusAudit_Invariants.t.sol` | Phase 6B — invariant tests |
| `[Contract]_NexusAudit_Fuzz.t.sol` | Phase 6B — fuzz variants of PoCs |

### Common Invariant Patterns

| Invariant Type | Template |
|---|---|
| Solvency | `sum(user_balances) <= token.balanceOf(contract)` |
| Supply consistency | `sum(individual_balances) == totalSupply` |
| Monotonic counter | `counter_after >= counter_before` |
| Fee bounds | `fee <= MAX_FEE` always |
| Access control | `only authorized role can change state X` |
| One-time use | `used IDs cannot be reused` |
| Non-negative | `balance >= 0` for all users |

### When to Skip Phase 6B

Phase 6B is recommended but optional for:
- INFO-level findings (no security impact)
- Contracts with <100 lines and no token interactions
- Gas-only optimizations (P-15)

Phase 6B is REQUIRED for:
- Any contract handling user funds
- CRITICAL or HIGH findings
- Contracts going to mainnet
