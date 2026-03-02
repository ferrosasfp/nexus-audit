---
name: nexus-audit
description: >
  Professional smart contract security audit using the NexusAudit methodology.
  Use when auditing Solidity/EVM smart contracts for vulnerabilities, economic attacks,
  centralization risks, and standards compliance. Combines Trail of Bits (threat modeling),
  Code4rena (warden specialization), Sherlock (economic security), and OpenZeppelin (standards).
  Anti-hallucination enforced - every finding must cite exact line numbers and reproduce
  the attack vector step-by-step. Use for: (1) auditing a contract, (2) pre-mainnet review,
  (3) comparing findings against external audits.
---

# NexusAudit Methodology

## Core Principle: Anti-Hallucination First

**Never report a finding without:**
1. Citing the exact file + line number
2. Quoting the actual code snippet
3. Writing a step-by-step attack scenario (or proof it's a real issue)
4. A passing PoC test (Phase 6) — without it, max confidence is LIKELY, never CONFIRMED
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

Do NOT start Phase 1 until Phase 0 is complete.

### Phase 1 — Threat Model (STRIDE)
See `references/methodology.md` → STRIDE section.
Output: threat matrix with applicable/not-applicable per vector.

### Phase 2 — Static Analysis Simulation
See `references/checklist.md` → Static Checklist.
Go through every item. Mark each: ✅ SAFE | ⚠️ REVIEW | 🔴 FINDING

### Phase 3 — Warden Specialization
Run 5 specialized lenses in order. See `references/methodology.md` → Warden Lenses.
- W1: Access Control
- W2: Economic Attacks
- W3: ERC Standards Compliance
- W4: Gas & Architecture
- W5: Invariant Verification

### Phase 4 — Economic Security (Sherlock Model)
See `references/methodology.md` → Economic Security.
Answer: "Is this protocol insurable?" — what's the max loss scenario?

### Phase 5 — Anti-Hallucination Validation
For every FINDING found in phases 1-4:
- Re-read the cited lines
- Confirm the attack path is executable
- Downgrade CONFIRMED → THEORETICAL if attack requires unrealistic conditions
- Drop findings that don't survive re-examination

### Phase 6 — PoC Tests (Proof of Concept)
For every finding that survived Phase 5, write a Foundry test:
- Test name: `test_[FINDING_ID]_[ShortDescription]()`
- Test must PASS for the finding to be CONFIRMED
- If test FAILS: diagnose — is the bug in the test or in the finding?
  - Bug in test → fix test, rerun
  - Bug in finding → downgrade to LIKELY or drop
- See `references/poc-guide.md` for test structure

**Rule:** Without a passing PoC → finding stays LIKELY, never CONFIRMED.

### Phase 7 — Report Generation
See `references/report-template.md` for output format.
Findings ordered by severity: CRITICAL → HIGH → MEDIUM → LOW → INFO
Each finding must show: confidence level + PoC test status (pass/fail/pending)

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
NexusAudit Phase 6: PoC PASSES (bug confirmed)
         ↓
Phase 7: Report with Fix Type assigned
         ↓
Phase 8: NexusAgil handoff
  FAST-FIX  → fix + invert + 3-layer validation
  HU-MINOR  → Linear + Story File + sub-agent + 3-layer validation
  HU-MAJOR  → Full NexusAgil pipeline + 3-layer validation
         ↓
PoC inverted: attack FAILS → finding CLOSED
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

---

## Quick Reference

- Detailed methodology phases → `references/methodology.md`
- Static analysis checklist → `references/checklist.md`
- Vulnerability pattern library → `references/vulnerability-patterns.md`
- Report output template → `references/report-template.md`
- PoC test guide → `references/poc-guide.md`
- Fix loop guide → `references/fix-loop.md`
