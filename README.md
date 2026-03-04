# NexusAudit v2.0

> AI-powered smart contract audit methodology with empirical proof — no hallucinations, only confirmed findings.

---

## What is NexusAudit?

NexusAudit is a smart contract security audit methodology that combines the best techniques from Trail of Bits, Code4rena, Sherlock, and OpenZeppelin — then adds one rule that changes everything:

> **Every finding must be proven with a passing Foundry test before it can be reported as CONFIRMED.**

No test = no CONFIRMED finding. Maximum confidence is LIKELY.

---

## What makes it different

| Standard AI Audit | NexusAudit v2.0 |
|---|---|
| Reports "could potentially" issues | Only reports empirically proven issues |
| Manual checklist only | Automated tools (Slither/Aderyn) + manual review |
| Deterministic tests only | PoC tests + fuzzing + invariant testing |
| Findings live in a PDF | Findings become inverted tests that prove the attack no longer works |
| Generic threat model (STRIDE) | Smart-contract-native threat model (TRACE) |
| Stops at the report | Continues through verified fixes with 4-layer validation |
| One methodology | 6 specialized warden lenses + TRACE + Sherlock economic model |
| No finding quality metric | Cross-Validation Score (0-5) per finding |
| No fix guidance | Fix Type classification (FAST-FIX / HU-MINOR / HU-MAJOR) |
| 15 vulnerability patterns | 22 patterns with real-world exploit references |

---

## The 8-Phase Methodology

```
Phase 0  — Read everything first (contract + tests + docs)
Phase 1  — Threat Model (TRACE — Trust, Reentrancy, Access, Calculation, External)
Phase 2A — Automated Static Analysis (Slither + Aderyn)
Phase 2B — Manual Checklist (40+ items for business logic + semantic issues)
Phase 3  — Warden Lenses (6 specializations)
           W1: Access Control
           W2: Economic Attacks
           W3: ERC Standards Compliance
           W4: Gas & Architecture
           W5: Invariant Verification
           W6: Upgrade Safety (conditional — if proxy pattern detected)
Phase 4  — Economic Security (Sherlock model + insurability verdict)
Phase 5  — Anti-Hallucination Validation + Cross-Validation Score
Phase 6A — PoC Tests (deterministic — attack must PASS to be CONFIRMED)
Phase 6B — Fuzz + Invariant Tests (property-based — discover unknown edge cases)
Phase 7  — Report with Fix Type classification + CV Score
Phase 8  — Fix Loop (attack must FAIL after fix — 4-layer validation)
```

---

## TRACE Threat Model (v2.0)

Smart-contract-native threat model replacing STRIDE:

| Vector | What it checks |
|---|---|
| **T** — Trust Boundaries | Who can call what? Are trust assumptions validated? |
| **R** — Reentrancy & State | CEI pattern? Cross-contract reentrancy? Read-only reentrancy? |
| **A** — Access & Authorization | Missing modifiers? Zero address? Ownership transfer? |
| **C** — Calculation & Math | Precision loss? Rounding direction? Overflow in unchecked? |
| **E** — External Dependencies | Oracle manipulation? Token behavior? Flash loan vectors? |

> STRIDE is available as optional secondary framework for off-chain-heavy contracts.

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
6. Proxy risks — only flag if contract actually uses proxy pattern
7. Oracle manipulation — only flag if oracle is on-chain and manipulable in same transaction

---

## Confidence Levels

| Level | Evidence required |
|---|---|
| **CONFIRMED** | Passing Foundry PoC test |
| **LIKELY** | Code cited + attack path written, no test yet |
| **THEORETICAL** | Requires multiple unlikely conditions |

## Cross-Validation Score (v2.0)

Every finding scored 0-5 before proceeding to PoC:

| Score | Action |
|---|---|
| 4-5 | → Phase 6A — write PoC test |
| 2-3 | → Document as LIKELY — PoC optional |
| 0-1 | → DROP — insufficient evidence |

---

## The Fix Loop

The part that makes NexusAudit different from every other audit:

```
Bug found (Phase 6A PoC PASSES — attack works)
    ↓
Fix written
    ↓
PoC inverted (attack now FAILS with correct revert)
    ↓
FIXED regression test written (correct behavior PASSES)
    ↓
4-layer validation:
  Layer 1: forge build — 0 errors
  Layer 2: Original tests — no regressions
  Layer 3: Inverted PoC — attack reverts correctly
  Layer 4: Invariant tests — protocol invariants still hold
    ↓
Finding CLOSED
```

---

## Tooling (All Free & Open Source)

| Tool | Purpose | Required? |
|---|---|---|
| Foundry | Testing, fuzzing, invariants | Required |
| Slither | Static analysis | Required |
| Aderyn | Secondary static analyzer | Recommended |
| Halmos | Symbolic execution | Optional |
| Semgrep | Custom pattern matching | Optional |

| Reference DB | Purpose |
|---|---|
| Solodit | Verified audit findings |
| Rekt | Real exploit post-mortems |
| DeFiHackLabs | Reproduce real attacks |

---

## Vulnerability Patterns (22 total)

| ID | Pattern | Category |
|---|---|---|
| P-01 | Reentrancy | State |
| P-02 | Access Control Missing | Access |
| P-03 | Unchecked Return Value | External |
| P-04 | Fee-on-Transfer Incompatibility | External |
| P-05 | Front-running | Economic |
| P-06 | Timestamp Manipulation | Logic |
| P-07 | Integer Overflow (pre-0.8) | Math |
| P-08 | DoS via Gas Limit | Gas |
| P-09 | Centralization / Role Abuse | Access |
| P-10 | Missing Two-Step Ownership | Access |
| P-11 | Event Before State Update | Logic |
| P-12 | Invariant Violation | State |
| P-13 | Emergency Exit Bypass | Logic |
| P-14 | No Circuit Breaker | Architecture |
| P-15 | String as Mapping Key | Gas |
| P-16 | Signature Malleability | **NEW v2.0** |
| P-17 | Proxy/Upgrade Vulnerabilities | **NEW v2.0** |
| P-18 | ERC-4626 Vault Inflation | **NEW v2.0** |
| P-19 | Precision Loss in Math | **NEW v2.0** |
| P-20 | Flash Loan Attack Vectors | **NEW v2.0** |
| P-21 | Returnbomb / Gas Griefing | **NEW v2.0** |
| P-22 | Cross-Contract Reentrancy | **NEW v2.0** |

All patterns include code fingerprints + real-world exploit references.

---

## Real-World Results (WasiAI Marketplace)

NexusAudit was developed and validated on a production smart contract — WasiAI Marketplace on Avalanche.

**Audit results:**
- 16 findings identified
- 15 confirmed via PoC tests (0 false positives after validation)
- 7 findings fixed in Sprint 9 with inverted PoC tests
- 2 known limitations documented honestly
- 78 total tests, 0 failures after fix loop

---

## Skill Structure

```
nexus-audit/
├── SKILL.md                           — Core methodology v2.0 + execution order
└── references/
    ├── methodology.md                 — TRACE + STRIDE, 6 warden lenses, Sherlock model, CV Score
    ├── checklist.md                   — Automated (Slither/Aderyn) + 40+ manual checks
    ├── vulnerability-patterns.md      — 22 patterns with code fingerprints + real exploits
    ├── report-template.md             — Report format with TRACE, CV Score, invariant results
    ├── poc-guide.md                   — PoC tests + fuzzing + invariant testing guide
    └── fix-loop.md                    — Fix Type classification + 4-layer validation
```

---

## Usage

NexusAudit is designed to run as a Claude Code skill. Load `SKILL.md` and follow the 8-phase methodology on any Solidity contract.

```bash
# Phase 2A: Run automated analysis
slither . --json slither-report.json
aderyn

# Phase 6A: Run PoC tests
forge test --match-contract [YourContract]NexusAudit

# Phase 6B: Run fuzzing + invariant tests
forge test --match-test invariant_ --fuzz-runs 1000
forge test --match-test test_fuzz_ --fuzz-runs 500

# Phase 8: Verify fixes (4-layer validation)
forge build                                              # Layer 1
forge test --match-contract [OriginalTests]              # Layer 2
forge test --match-contract NexusAuditValidation         # Layer 3
forge test --match-test invariant_ --fuzz-runs 1000      # Layer 4
```

---

## Changelog

### v2.0 (2026-03-04)
- **TRACE threat model** replaces STRIDE as primary (STRIDE kept as optional)
- **Phase 2 split**: 2A (automated: Slither + Aderyn) + 2B (manual checklist)
- **Phase 6 split**: 6A (deterministic PoCs) + 6B (fuzzing + invariant testing)
- **W6 Upgrade Safety Lens** added for proxy/upgradeable contracts
- **Cross-Validation Score** (0-5) added to Phase 5
- **7 new vulnerability patterns** (P-16 to P-22) with real-world exploit references
- **Real-world exploit references** added to all 22 patterns
- **4-layer validation** (added Layer 4: invariant regression)
- **Tooling section** with required/recommended free tools

### v1.0
- Initial release with 8-phase methodology
- 15 vulnerability patterns
- STRIDE threat model
- 3-layer fix validation

---

## License

MIT

---

*NexusAudit was built by San (AI assistant) and Fernando Rosas during the development of WasiAI — a Web3 AI agent marketplace on Avalanche.*
