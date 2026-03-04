# NexusAudit — Report Template

```markdown
# NexusAudit Report — [Project Name]
**Auditor:** NexusAudit (powered by [Name])
**Date:** YYYY-MM-DD
**Contract:** [filename] — [N] lines
**Methodology:** NexusAudit v2.0
**Confidence system:** CONFIRMED | LIKELY | THEORETICAL
**Threat model:** TRACE (Trust, Reentrancy, Access, Calculation, External)

---

## Executive Summary

[2-3 sentences: what the contract does, overall security posture, top risk]

**Insurability Verdict:** [INSURABLE | CONDITIONALLY INSURABLE | NOT INSURABLE]

---

## Automated Analysis Results

### Slither
| Detector | Count | Valid | False Positive | N/A |
|---|---|---|---|---|
| reentrancy-eth | X | X | X | X |
| ... | ... | ... | ... | ... |

### Aderyn (if used)
| Finding | Valid | False Positive | N/A |
|---|---|---|---|

---

## TRACE Threat Model Summary

| TRACE Vector | Applicable | Location | Risk Level | Notes |
|---|---|---|---|---|
| T — Trust Boundaries | YES/NO | file:line | HIGH/MED/LOW | ... |
| R — Reentrancy & State | YES/NO | file:line | HIGH/MED/LOW | ... |
| A — Access & Authorization | YES/NO | file:line | HIGH/MED/LOW | ... |
| C — Calculation & Math | YES/NO | file:line | HIGH/MED/LOW | ... |
| E — External Dependencies | YES/NO | file:line | HIGH/MED/LOW | ... |

---

## Findings

### [SEVERITY]-[N]: [Title]

| Field | Value |
|---|---|
| Severity | CRITICAL / HIGH / MEDIUM / LOW / INFO |
| Confidence | CONFIRMED / LIKELY / THEORETICAL |
| Cross-Validation Score | [0-5] |
| Location | `file.sol:LineN` |
| Pattern | P-XX (from vulnerability library) |
| PoC Test | `test_NA_XX_Description` ✅/❌/⏳ |

**Code:**
\`\`\`solidity
// exact snippet from the contract
\`\`\`

**Attack Path:**
1. Step 1
2. Step 2
3. Result: [impact]

**Impact:** [what happens]
**Likelihood:** [how realistic]
**Recommendation:** [concrete fix]
**Real-world precedent:** [if any — from Solodit/Rekt/Immunefi]

---

## Anti-Hallucination Validation

For each finding, document:
- [ ] Line numbers verified by re-reading source ✅/❌
- [ ] Attack path executable without unrealistic assumptions ✅/❌
- [ ] Severity matches definition table ✅/❌
- [ ] Finding not already mitigated by existing code ✅/❌
- [ ] Cross-Validation Score >= 2 ✅/❌

---

## Phase 6B — Invariant & Fuzz Testing Results

### Invariant Tests
| Invariant | Runs | Result | Notes |
|---|---|---|---|
| Solvency | 1000 | ✅ PASS / 🔴 BROKEN | ... |
| Supply consistency | 1000 | ✅ PASS / 🔴 BROKEN | ... |
| Fee bounds | 1000 | ✅ PASS / 🔴 BROKEN | ... |

### Fuzz Tests
| Finding | Fuzz Variant | Runs | Result |
|---|---|---|---|
| NA-H02 | test_fuzz_NA_H02_arbitraryAmount | 500 | ✅ / 🔴 |

---

## Comparison vs. Prior Audits

| Finding | This Audit | Prior Audit | Match? | Notes |
|---|---|---|---|---|

---

## Checklist Summary

| Category | Items | ✅ Safe | ⚠️ Review | 🔴 Finding |
|---|---|---|---|---|

---

## Recommendations Priority

| Priority | Issue | Effort |
|---|---|---|
| P0 — Mainnet blocker | | |
| P1 — High priority | | |
| P2 — Recommended | | |
| P3 — Nice to have | | |

---

## Phase 8 — NexusAgil Fix Classification

| Finding | Severity | CV Score | Fix Type | NexusAgil Process | Layer 1-4 | Status |
|---|---|---|---|---|---|---|
| NA-XX | HIGH | 5 | FAST-FIX | Direct | ✅✅✅✅ | CLOSED |
| NA-XX | MEDIUM | 4 | HU-MINOR | Story File | ✅✅✅✅ | CLOSED |
| NA-XX | HIGH | 5 | HU-MAJOR | Full pipeline | ⏳ | IN PROGRESS |
| NA-XX | HIGH | 3 | KNOWN-LIMITATION | Follow-up issue | N/A | DEFERRED |

**Fix Types:**
- **FAST-FIX** — 1-2 files, surgical, no new logic → execute directly
- **HU-MINOR** — 3-5 files, simple new logic → NexusAgil Story File + sub-agent
- **HU-MAJOR** — architectural, high risk → NexusAgil full pipeline F0→SDD→F3→AR→F4
- **KNOWN-LIMITATION** — cannot fix this sprint → document + follow-up Linear issue

**Validation Layers:**
- Layer 1: Compile (`forge build` — zero errors)
- Layer 2: Regression (original tests pass)
- Layer 3: PoC Inversion (attack fails, fix passes)
- Layer 4: Invariant Regression (protocol invariants hold)
```
