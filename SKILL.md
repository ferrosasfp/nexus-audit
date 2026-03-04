---
name: nexus-audit
description: >
  Professional smart contract security audit using the NexusAudit methodology v2.0.
  Use when auditing Solidity/EVM smart contracts for vulnerabilities, economic attacks,
  centralization risks, and standards compliance. Combines Trail of Bits (threat modeling),
  Code4rena (warden specialization), Sherlock (economic security), and OpenZeppelin (standards).
  Anti-hallucination enforced - every finding must cite exact line numbers and reproduce
  the attack vector step-by-step. Uses automated tools (Slither, Aderyn) + manual review.
  Includes fuzzing and invariant testing for empirical validation.
  Use for: (1) auditing a contract, (2) pre-mainnet review,
  (3) comparing findings against external audits.
---

# NexusAudit Methodology v2.0

## Core Principle: Anti-Hallucination First

**Never report a finding without:**
1. Citing the exact file + line number
2. Quoting the actual code snippet
3. Writing a step-by-step attack scenario (or proof it's a real issue)
4. A passing PoC test (Phase 6A) — without it, max confidence is LIKELY, never CONFIRMED
5. Marking confidence: `CONFIRMED` | `LIKELY` | `THEORETICAL`

If you cannot do all 5 → do NOT mark as CONFIRMED. Downgrade or drop it.

**Confidence levels:**
| Level | Evidence required |
|---|---|
| CONFIRMED | Passing PoC Forge test |
| LIKELY | Code cited + attack path written, no test yet |
| THEORETICAL | Requires multiple unlikely conditions |

---

## Execution Order

### Phase 0 — Read Everything First
Before any analysis:
- Read the full contract source (every line)
- Read all tests
- Read any audit package / architecture docs
- List all: state variables, roles, modifiers, external calls, events
- Identify: proxy patterns, inheritance chain, external dependencies

Do NOT start Phase 1 until Phase 0 is complete.

### Phase 1 — Threat Model (TRACE)
See `references/methodology.md` → TRACE section.
Apply the on-chain threat model: **T**rust, **R**eentrancy, **A**ccess, **C**alculation, **E**xternal.
Output: threat matrix with applicable/not-applicable per vector + specific code locations.

> **Note:** STRIDE (traditional) is available as optional secondary framework in `references/methodology.md`.
> TRACE is preferred because it maps directly to smart contract vulnerability categories.

### Phase 2 — Static Analysis (Automated + Manual)

#### Phase 2A — Automated Static Analysis (Obligatorio)
Run real tools before manual review:
1. `slither . --json slither-report.json` — automated vulnerability detection
2. `aderyn` (if available) — secondary static analyzer
3. For each tool finding, classify: `VALID` | `FALSE-POSITIVE` | `NOT-APPLICABLE`
4. VALID findings feed directly into Phase 6A for PoC development

#### Phase 2B — Manual Checklist (Complementario)
See `references/checklist.md` → Static Checklist.
Go through every item. Mark each: ✅ SAFE | ⚠️ REVIEW | 🔴 FINDING

> **Why both?** Slither detects mechanical issues in seconds. The manual checklist catches
> business logic, semantic errors, and design flaws that tools cannot detect.

### Phase 3 — Warden Specialization
Run 6 specialized lenses in order. See `references/methodology.md` → Warden Lenses.
- W1: Access Control
- W2: Economic Attacks
- W3: ERC Standards Compliance
- W4: Gas & Architecture
- W5: Invariant Verification
- W6: Upgrade Safety (if contract uses proxies — see below)

#### W6 — Upgrade Safety Lens (conditional)
Apply ONLY if the contract uses any proxy/upgrade pattern (UUPS, Transparent, Beacon, Diamond).
If no proxy → skip W6 and document "N/A — no proxy pattern detected."

Checklist:
- [ ] `initializer` used instead of `constructor`?
- [ ] `_disableInitializers()` in constructor of implementation contract?
- [ ] Storage layout documented and consistent across versions?
- [ ] No storage slot collision between implementation versions?
- [ ] UUPS: `_authorizeUpgrade` has proper access control?
- [ ] No `selfdestruct` or arbitrary `delegatecall` in implementation?
- [ ] Immutable variables stored in code, not in storage slots?
- [ ] Run: `forge inspect ContractName storage-layout` to verify

### Phase 4 — Economic Security (Sherlock Model)
See `references/methodology.md` → Economic Security.
Answer: "Is this protocol insurable?" — what's the max loss scenario?

### Phase 5 — Anti-Hallucination Validation with Cross-Validation Score
For every FINDING found in phases 1-4, apply the Cross-Validation Score:

**Score each finding 0-5:**
| Points | Criterion |
|---|---|
| +1 | Exact line number cited and verified by re-reading source |
| +1 | Code snippet matches current contract code (not stale) |
| +1 | Attack path has ≤ 3 steps (realistically executable) |
| +1 | Does NOT require multiple unlikely conditions simultaneously |
| +1 | Real-world precedent exists (Solodit, Rekt, Immunefi database) |

**Action based on score:**
| Score | Action |
|---|---|
| 4-5 | Proceed to Phase 6A — high confidence finding |
| 2-3 | Document as LIKELY — PoC optional but recommended |
| 0-1 | DROP — insufficient evidence, do not report |

### Phase 6A — PoC Tests (Proof of Concept)
For every finding that survived Phase 5, write a Foundry test:
- Test name: `test_[FINDING_ID]_[ShortDescription]()`
- Test must PASS for the finding to be CONFIRMED
- If test FAILS: diagnose — is the bug in the test or in the finding?
  - Bug in test → fix test, rerun
  - Bug in finding → downgrade to LIKELY or drop
- See `references/poc-guide.md` for test structure

**Rule:** Without a passing PoC → finding stays LIKELY, never CONFIRMED.

### Phase 6B — Fuzz + Invariant Testing (NEW)
After deterministic PoCs, run property-based testing:

#### Step 1: Define Protocol Invariants
From Phase 3 W5, express invariants as Foundry tests:
```solidity
function invariant_solvency() public {
    assertLe(
        contract.totalUserBalances(),
        token.balanceOf(address(contract)),
        "Invariant: contract must always be solvent"
    );
}
```

#### Step 2: Run Invariant Tests
```bash
forge test --match-test invariant_ --fuzz-runs 1000
```
Minimum 1000 runs. If an invariant breaks → new CONFIRMED finding automatically.

#### Step 3: Fuzz Existing PoCs
For each Phase 6A finding, add a fuzz variant:
```solidity
function test_fuzz_NA_H02_arbitraryAmount(uint256 amount) public {
    vm.assume(amount > 0 && amount <= type(uint128).max);
    // ... test with variable input
}
```

#### Step 4: Run Full Fuzz Suite
```bash
forge test --match-test test_fuzz_ --fuzz-runs 500
```
Any failure → new finding or expanded severity of existing finding.

See `references/poc-guide.md` → Fuzzing & Invariant Testing section.

### Phase 7 — Report Generation
See `references/report-template.md` for output format.
Findings ordered by severity: CRITICAL → HIGH → MEDIUM → LOW → INFO
Each finding must show: confidence level + PoC test status + Cross-Validation Score

### Phase 8 — Fix Classification + NexusAgil Handoff

Before writing any fix, classify each CONFIRMED finding using the Fix Type system.
This determines which NexusAgil process to follow.

**Fix Type classification:**

| Type | Criteria | NexusAgil Process |
|---|---|---|
| FAST-FIX | 1-2 files, surgical change, no new logic | Execute directly, no Story File |
| HU-MINOR | 3-5 files, simple new logic, low risk | HU_APPROVED → Story File → Sub-agent → QA |
| HU-MAJOR | Architectural, multiple contracts, high risk | Full F0→SDD→SPEC_APPROVED→F3→AR→F4 |
| KNOWN-LIMITATION | Cannot fix this sprint (redesign needed) | Document + create follow-up issue |

**Classification examples:**
- `require(operator != address(0))` → FAST-FIX
- Remove one line from performUpkeep → FAST-FIX
- Add Pausable inheritance → HU-MINOR
- Migrate Ownable → Ownable2Step → HU-MINOR
- Separate keyBalances/earnings into two contracts → HU-MAJOR
- Timelock on fee changes → HU-MAJOR

**Fix Loop execution (per Fix Type):**

FAST-FIX:
1. Write fix directly (no Story File)
2. Invert PoC test (attack must FAIL with correct revert)
3. Write FIXED regression test (correct behavior PASS)
4. Run 3-layer validation → CLOSED

HU-MINOR / HU-MAJOR:
1. Create Linear issue with finding details + Fix Type
2. Follow NexusAgil pipeline (Story File → sub-agent → tests → commit)
3. Phase 8 Fix Loop applied at end of NexusAgil F3 step
4. QA validates inverted PoC tests → CLOSED

KNOWN-LIMITATION:
1. Document in report with rationale
2. Keep original PoC test active (honest — attack still possible)
3. Create follow-up Linear issue for future sprint
4. Status: OPEN / DEFERRED

**The complete cycle:**
```
NexusAudit Phase 6A: PoC PASSES (bug confirmed)
NexusAudit Phase 6B: Invariant/Fuzz testing (additional bugs found)
         ↓
Phase 7: Report with Fix Type + Cross-Validation Score
         ↓
Phase 8: NexusAgil handoff
  FAST-FIX  → fix + invert + 3-layer validation
  HU-MINOR  → Linear + Story File + sub-agent + 3-layer validation
  HU-MAJOR  → Full NexusAgil pipeline + 3-layer validation
         ↓
PoC inverted: attack FAILS → finding CLOSED
Invariant tests: still passing → no regressions
```

See `references/fix-loop.md` for detailed execution guide.

---

## Severity Definitions

| Level | Definition |
|---|---|
| CRITICAL | Direct loss of funds, no preconditions |
| HIGH | Loss of funds with one condition (e.g., compromised role) |
| MEDIUM | Protocol malfunction, no direct fund loss OR fund loss with 2+ conditions |
| LOW | Best practice violation, minor risk |
| INFO | Architecture suggestion, no security impact |

---

## Anti-Hallucination Rules

1. **No "could potentially"** without a concrete attack path
2. **No severity escalation** because it "sounds bad" — follow the definition table
3. **External call risks** — only flag if the external contract is untrusted or mutable
4. **Reentrancy** — only flag if there's an actual state change AFTER an external call without ReentrancyGuard
5. **Overflow** — only flag if Solidity <0.8.0 or unchecked{} blocks are present
6. **Centralization** — document as INFO unless there's a direct fund loss path
7. **Proxy risks** — only flag if contract actually uses proxy pattern (check W6)
8. **Oracle manipulation** — only flag if oracle is on-chain and manipulable in same transaction

---

## Tooling Requirements

### Required (Free, Open Source)
| Tool | Purpose | Install |
|---|---|---|
| Slither | Static analysis | `pip3 install slither-analyzer` |
| Foundry | Testing, fuzzing, invariants | `curl -L https://foundry.paradigm.xyz \| bash` |

### Recommended (Free, Open Source)
| Tool | Purpose | Install |
|---|---|---|
| Aderyn | Secondary static analyzer | `cargo install aderyn` |
| Halmos | Symbolic execution | `pip3 install halmos` |
| Semgrep | Custom pattern matching | `pip3 install semgrep` |

### Reference Databases (Free)
| Resource | Purpose | URL |
|---|---|---|
| Solodit | Verified audit findings | solodit.xyz |
| Rekt | Real exploit post-mortems | rekt.news |
| DeFi Hacks | Reproduce real attacks | github.com/SunWeb3Sec/DeFiHackLabs |

---

## Quick Reference

- Detailed methodology phases → `references/methodology.md`
- Static analysis checklist → `references/checklist.md`
- Vulnerability pattern library (P-01 to P-22) → `references/vulnerability-patterns.md`
- Report output template → `references/report-template.md`
- PoC + Fuzzing test guide → `references/poc-guide.md`
- Fix loop guide → `references/fix-loop.md`
