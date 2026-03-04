# NexusAudit — Detailed Methodology

## TRACE Threat Model (Primary — Smart Contract Native)

TRACE is designed specifically for smart contract security. Each letter maps directly
to a category of on-chain vulnerabilities.

### T — Trust Boundaries
**Question:** Who can call what? Are trust assumptions validated in code?

| Check | What to look for |
|---|---|
| Role enumeration | List every role (owner, operator, admin, user, anyone) |
| Trust assumptions | What does the contract assume about each caller? |
| Cross-contract trust | Does contract A trust contract B's state without verification? |
| Trust validation | Are trust assumptions enforced by modifiers/requires? |
| Privilege escalation | Can a lower role acquire higher role capabilities? |

**Output:** Trust boundary diagram mapping roles → functions → state changes.

### R — Reentrancy & State Consistency
**Question:** Can state become inconsistent during or after external calls?

| Check | What to look for |
|---|---|
| CEI pattern | Checks-Effects-Interactions order respected? |
| Cross-function reentrancy | Can function A re-enter function B sharing state? |
| Cross-contract reentrancy | Can external contract re-enter via callback? |
| Read-only reentrancy | Can view functions return stale data during execution? |
| ReentrancyGuard | Present on all functions with external calls + state changes? |
| State consistency | Is state always consistent between external calls? |

**Output:** Matrix of functions × reentrancy risk (SAFE / GUARDED / VULNERABLE).

### A — Access & Authorization
**Question:** Are all state-changing functions properly gated?

| Check | What to look for |
|---|---|
| Missing modifiers | State-changing functions without access control |
| Zero address | Can address(0) be set in critical roles? |
| Ownership transfer | One-step (risky) vs two-step (safe)? |
| Function visibility | Public functions that should be internal/private? |
| Initializer protection | `_disableInitializers()` in implementation constructors? |
| Self-destruct | Can anyone trigger selfdestruct/delegatecall? |

**Output:** Function-level access control matrix.

### C — Calculation & Math
**Question:** Can mathematical operations produce incorrect or exploitable results?

| Check | What to look for |
|---|---|
| Precision loss | Division before multiplication? |
| Rounding direction | Does rounding favor protocol or attacker? |
| Overflow in unchecked | `unchecked{}` blocks with user-controlled inputs? |
| Edge cases | amount=0, amount=MAX, empty arrays? |
| Fee calculations | What happens at fee=0%? fee=100%? |
| Share price manipulation | First depositor attacks in vault patterns? |
| Fixed-point math | Correct scaling factors and decimal handling? |

**Output:** List of mathematical operations with risk assessment.

### E — External Dependencies
**Question:** What happens when external systems behave unexpectedly?

| Check | What to look for |
|---|---|
| Oracle manipulation | Can oracle price be manipulated in same transaction? |
| Token behavior | Fee-on-transfer, rebasing, pausable, blocklisting? |
| Composability risks | Flash loan vectors? MEV/sandwich exposure? |
| Upgrade risks | Can external dependency upgrade and break assumptions? |
| Chain-specific | Block time, gas limits, precompile availability? |
| Fallback behavior | What happens when external call fails/reverts? |

**Output:** External dependency matrix with risk ratings.

### TRACE Output Format
```
| TRACE Vector | Applicable | Location | Risk Level | Notes |
|---|---|---|---|---|
| T — Trust Boundaries | YES/NO | file:line | HIGH/MED/LOW | ... |
| R — Reentrancy & State | YES/NO | file:line | HIGH/MED/LOW | ... |
| A — Access & Authorization | YES/NO | file:line | HIGH/MED/LOW | ... |
| C — Calculation & Math | YES/NO | file:line | HIGH/MED/LOW | ... |
| E — External Dependencies | YES/NO | file:line | HIGH/MED/LOW | ... |
```

---

## STRIDE Threat Model (Optional — Traditional Security)

> **When to use STRIDE:** As a secondary framework when auditing contracts that interact
> heavily with off-chain systems (oracles, keepers, APIs) where traditional threat
> categories add value. TRACE is always the primary model.

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

> **Note on STRIDE limitations for smart contracts:**
> - Repudiation is largely irrelevant — all transactions are recorded on-chain
> - Info Disclosure has different meaning — all on-chain data is public by design
> - TRACE's vectors (Trust, Reentrancy, Calculation, External) map more directly to real exploits

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
- Can flash loans amplify any attack vector?
- Is the contract exposed to MEV/sandwich attacks?

### W3 — ERC Standards Lens
Questions:
- What ERC standards does the contract use? (ERC-20, ERC-721, ERC-1155, ERC-4626, ERC-3009, etc.)
- Are external interfaces declared locally vs. using OZ standard?
- What if the external token changes behavior? (USDC upgrade risk)
- Are return values from external calls checked?
- Does `SafeERC20` cover all token interactions?
- ERC-4626: is first depositor attack mitigated?
- ERC-712: domain separator correctly implemented?

### W4 — Gas & Architecture Lens
Questions:
- Are there unbounded loops? What's the realistic max iteration count?
- Are storage reads cached in memory inside loops?
- Are mappings using expensive key types (string) when bytes32 would work?
- Is there unnecessary on-chain storage of off-chain data?
- Is the contract doing too much? Should it be split?
- Are events emitted before or after state changes?
- Are there dead code paths / unused variables?
- Returnbomb risk: is returndata from low-level calls bounded?
- Can calldata size cause excessive gas consumption?

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

**NEW in v2.0:** Invariants defined here MUST be tested in Phase 6B using Foundry
invariant tests, not just conceptually documented.

### W6 — Upgrade Safety Lens (Conditional)
**Apply ONLY if the contract uses proxy/upgrade patterns.**

Checklist:
- [ ] `initializer` used instead of `constructor`?
- [ ] `_disableInitializers()` in constructor of implementation?
- [ ] Storage layout documented and verified with `forge inspect`?
- [ ] No storage slot collision between versions?
- [ ] UUPS: `_authorizeUpgrade` has proper access control (onlyOwner)?
- [ ] No `selfdestruct` in implementation (pre-Dencun concern)?
- [ ] No arbitrary `delegatecall` in implementation?
- [ ] Immutable variables stored in bytecode, not storage?
- [ ] Initializer cannot be called twice (reentrancy in initialize)?
- [ ] Transparent proxy: admin cannot call implementation functions?
- [ ] Beacon: beacon owner properly secured?

**Verification command:**
```bash
forge inspect ContractName storage-layout
# Compare output between implementation versions
```

**Common proxy vulnerabilities:**
| Pattern | Risk | Example |
|---|---|---|
| Uninitialized proxy | CRITICAL | Parity wallet ($300M frozen) |
| Storage collision | HIGH | Audius governance hack |
| Missing _authorizeUpgrade | CRITICAL | Anyone can upgrade |
| selfdestruct in impl | CRITICAL | Implementation destroyed |
| Gap variables missing | MEDIUM | Future upgrade storage collision |

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

---

## Cross-Validation Score (Phase 5)

Every finding receives a score before proceeding to Phase 6:

| Points | Criterion |
|---|---|
| +1 | Exact line number cited and verified by re-reading source |
| +1 | Code snippet matches current contract code |
| +1 | Attack path ≤ 3 steps (realistically executable) |
| +1 | Does NOT require multiple unlikely simultaneous conditions |
| +1 | Real-world precedent exists (Solodit/Rekt/Immunefi) |

| Score | Action |
|---|---|
| 4-5 | → Phase 6A (PoC) — high confidence |
| 2-3 | → Document as LIKELY — PoC optional |
| 0-1 | → DROP — do not report |
