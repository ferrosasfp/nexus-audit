# NexusAudit — Detailed Methodology

## STRIDE Threat Model

For each vector, answer: applicable? if yes, where exactly?

| Threat | Question to answer |
|---|---|
| Spoofing | Can an attacker impersonate a privileged role? Is identity verified only by msg.sender? |
| Tampering | Can state be altered by an unauthorized party? Are there indirect state modification paths? |
| Repudiation | Is there a complete audit trail? Can actions be denied? |
| Info Disclosure | Are sensitive values exposed that shouldn't be? |
| Denial of Service | Can an attacker block critical functions (settlement, withdrawal)? |
| Elevation of Privilege | Can a lower-privilege role acquire higher privileges? |

**Output format:**
```
| STRIDE Vector | Applicable | Location | Notes |
```

---

## Warden Lenses

### W1 — Access Control Lens
Questions to answer for EVERY function:
- Who can call this? Is the modifier correct?
- Can the caller be address(0)?
- Is there a missing authorization on any state change?
- Can roles be transferred to bad addresses?
- Are there functions with no modifier that should have one?
- Is there a two-step transfer for critical ownership?

### W2 — Economic Attack Lens
Questions:
- Can an attacker front-run any transaction profitably?
- Can a privileged role extract value (fee manipulation, sandwich)?
- Is there a price oracle? Can it be manipulated?
- What happens if fee = 100%? Fee = 0%?
- Can dust amounts cause rounding issues that accumulate?
- Is there a timelock on critical parameter changes?
- Can a user force the contract into an unprofitable state?

### W3 — ERC Standards Lens
Questions:
- What ERC standards does the contract use? (ERC-20, ERC-721, ERC-3009, etc.)
- Are external interfaces declared locally vs. using OZ standard?
- What if the external token changes behavior? (USDC upgrade risk)
- Are return values from external calls checked?
- Does `SafeERC20` cover all token interactions?

### W4 — Gas & Architecture Lens
Questions:
- Are there unbounded loops? What's the realistic max iteration count?
- Are storage reads cached in memory inside loops?
- Are mappings using expensive key types (string) when bytes32 would work?
- Is there unnecessary on-chain storage of off-chain data?
- Is the contract doing too much? Should it be split?
- Are events emitted before or after state changes?
- Are there dead code paths / unused variables?

### W5 — Invariant Verification Lens
Define the protocol's invariants, then try to break each:

**How to define invariants:**
1. List all state variables that track value (balances, totals)
2. For each: write `sum(X) == Y` type relationships
3. Check if any function can violate those relationships

**Common invariants to check:**
- `sum(earnings) + sum(keyBalances) <= usdc.balanceOf(contract)` (solvency)
- `totalVolume` monotonically increases
- `platformFeeBps <= MAX_FEE` always
- Once a paymentId is used, it can never be used again
- keyBalance can never go negative

---

## Economic Security (Sherlock Model)

Answer these 5 questions:

**1. What is the maximum loss scenario?**
Who can drain the contract? Under what conditions? How fast?

**2. What privileged roles exist and what can they do?**
Map each role to its maximum extractable value.

**3. Is there a trustless exit for users?**
Can users always recover their funds without operator permission?

**4. Are parameter changes bounded and time-locked?**
Can critical parameters change instantly to adversarial values?

**5. Insurability verdict:**
- INSURABLE: No single actor can drain funds unilaterally
- CONDITIONALLY INSURABLE: With multisig/timelock on operator
- NOT INSURABLE: Single hot wallet can drain all funds

---

## Confidence Levels

**CONFIRMED** — Attack path executable in a test with no external dependencies
**LIKELY** — Attack path requires a specific condition (e.g., privileged role acting maliciously) that is realistic
**THEORETICAL** — Attack path requires multiple unlikely conditions; document but don't escalate severity
