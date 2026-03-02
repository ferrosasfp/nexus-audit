# NexusAudit

> AI-powered smart contract audit methodology with empirical proof — no hallucinations, only confirmed findings.

---

## What is NexusAudit?

NexusAudit is a smart contract security audit methodology that combines the best techniques from Trail of Bits, Code4rena, Sherlock, and OpenZeppelin — then adds one rule that changes everything:

> **Every finding must be proven with a passing Foundry test before it can be reported as CONFIRMED.**

No test = no CONFIRMED finding. Maximum confidence is LIKELY.

---

## What makes it different

| Standard AI Audit | NexusAudit |
|---|---|
| Reports "could potentially" issues | Only reports empirically proven issues |
| Findings live in a PDF | Findings become inverted tests that prove the attack no longer works |
| Stops at the report | Continues through verified fixes |
| One methodology | 5 specialized warden lenses + STRIDE + Sherlock economic model |
| No fix guidance | Fix Type classification (FAST-FIX / HU-MINOR / HU-MAJOR) |

---

## The 8-Phase Methodology

```
Phase 0  — Read everything first (contract + tests + docs)
Phase 1  — Threat Model (STRIDE)
Phase 2  — Static Analysis Checklist (40+ items)
Phase 3  — Warden Lenses (5 specializations)
           W1: Access Control
           W2: Economic Attacks
           W3: ERC Standards Compliance
           W4: Gas & Architecture
           W5: Invariant Verification
Phase 4  — Economic Security (Sherlock model + insurability verdict)
Phase 5  — Anti-Hallucination Validation (findings that don't survive are dropped)
Phase 6  — PoC Tests (attack must PASS to be CONFIRMED)
Phase 7  — Report with Fix Type classification
Phase 8  — Fix Loop (attack must FAIL after fix — NexusAgil handoff)
```

---

## Fix Type Classification

Every confirmed finding is classified before fixing:

| Type | Criteria | Process |
|---|---|---|
| **FAST-FIX** | 1-2 files, surgical, no new logic | Execute directly |
| **HU-MINOR** | 3-5 files, simple new logic | Story File + sub-agent |
| **HU-MAJOR** | Architectural, high risk | Full development pipeline |
| **KNOWN-LIMITATION** | Cannot fix this sprint | Document + follow-up issue |

---

## Anti-Hallucination Rules

1. No "could potentially" without a concrete attack path
2. No severity escalation because it "sounds bad"
3. Reentrancy — only flag if there's a real state change after external call without ReentrancyGuard
4. Overflow — only flag if Solidity <0.8.0 or unchecked{} blocks present
5. Centralization — INFO only, unless there's a direct fund loss path

---

## Confidence Levels

| Level | Evidence required |
|---|---|
| **CONFIRMED** | Passing Foundry PoC test |
| **LIKELY** | Code cited + attack path written, no test yet |
| **THEORETICAL** | Requires multiple unlikely conditions |

---

## The Fix Loop

The part that makes NexusAudit different from every other audit:

```
Bug found (Phase 6 PoC PASSES — attack works)
    ↓
Fix written
    ↓
PoC inverted (attack now FAILS with correct revert)
    ↓
FIXED regression test written (correct behavior PASSES)
    ↓
3-layer validation:
  Layer 1: forge build — 0 errors
  Layer 2: Original tests — no regressions
  Layer 3: Inverted PoC — attack reverts correctly
    ↓
Finding CLOSED
```

---

## Real-World Results (WasiAI Marketplace)

NexusAudit was developed and validated on a production smart contract — WasiAI Marketplace on Avalanche.

**Audit results:**
- 16 findings identified
- 15 confirmed via PoC tests (0 false positives after validation)
- 7 findings fixed in Sprint 9 with inverted PoC tests
- 2 known limitations documented honestly
- 78 total tests, 0 failures after fix loop

**Comparison vs. simulated audits from Trail of Bits, Code4rena, Sherlock, OpenZeppelin:**
- 15/16 findings matched across all methodologies
- 1 finding not detected (transferAgent — classified as UX, not security)
- 2 potential findings discarded before reporting (survived Phase 5 validation)

---

## Skill Structure

```
nexus-audit/
├── SKILL.md                           — Core methodology + execution order
└── references/
    ├── methodology.md                 — STRIDE, warden lenses, Sherlock model
    ├── checklist.md                   — 40+ static analysis checks
    ├── vulnerability-patterns.md      — 15 known patterns with code fingerprints
    ├── report-template.md             — Standardized report format
    ├── poc-guide.md                   — How to write PoC tests
    └── fix-loop.md                    — Fix Type classification + 3-layer validation
```

---

## Usage

NexusAudit is designed to run as an OpenClaw skill. Load `SKILL.md` and follow the 8-phase methodology on any Solidity contract.

```bash
# Run audit on a contract
# Phase 0: read the contract
# Phase 1-5: analysis
# Phase 6: write PoC tests
forge test --match-contract [YourContract]NexusAudit

# Phase 8: verify fixes
forge test --match-contract [YourContract]NexusAudit
# All attack tests should now FAIL (revert correctly)
# All FIXED tests should PASS
```

---

## License

MIT

---

*NexusAudit was built by San (AI assistant) and Fernando Rosas during the development of WasiAI — a Web3 AI agent marketplace on Avalanche.*
