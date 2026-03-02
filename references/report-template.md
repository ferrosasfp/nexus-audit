# NexusAudit — Report Template

```markdown
# NexusAudit Report — [Project Name]
**Auditor:** NexusAudit (powered by San)  
**Date:** YYYY-MM-DD  
**Contract:** [filename] — [N] lines  
**Methodology:** NexusAudit v1.0  
**Confidence system:** CONFIRMED | LIKELY | THEORETICAL

---

## Executive Summary

[2-3 sentences: what the contract does, overall security posture, top risk]

**Insurability Verdict:** [INSURABLE | CONDITIONALLY INSURABLE | NOT INSURABLE]

---

## Findings

### [SEVERITY]-[N]: [Title]

| Field | Value |
|---|---|
| Severity | CRITICAL / HIGH / MEDIUM / LOW / INFO |
| Confidence | CONFIRMED / LIKELY / THEORETICAL |
| Location | `file.sol:LineN` |
| Pattern | P-XX (from vulnerability library) |

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

---

## Anti-Hallucination Validation

For each finding, document:
- [ ] Line numbers verified by re-reading source ✅/❌
- [ ] Attack path executable without unrealistic assumptions ✅/❌
- [ ] Severity matches definition table ✅/❌
- [ ] Finding not already mitigated by existing code ✅/❌

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

| Finding | Severity | Fix Type | NexusAgil Process | PoC Inverted | Status |
|---|---|---|---|---|---|
| NA-XX | HIGH | FAST-FIX | Direct | ✅ | CLOSED |
| NA-XX | MEDIUM | HU-MINOR | Story File | ✅ | CLOSED |
| NA-XX | HIGH | HU-MAJOR | Full pipeline | ⏳ | IN PROGRESS |
| NA-XX | HIGH | KNOWN-LIMITATION | Follow-up issue | N/A | DEFERRED |

**Fix Types:**
- **FAST-FIX** — 1-2 files, surgical, no new logic → execute directly
- **HU-MINOR** — 3-5 files, simple new logic → NexusAgil Story File + sub-agent
- **HU-MAJOR** — architectural, high risk → NexusAgil full pipeline F0→SDD→F3→AR→F4
- **KNOWN-LIMITATION** — cannot fix this sprint → document + follow-up Linear issue
```
