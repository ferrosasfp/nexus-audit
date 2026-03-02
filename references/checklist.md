# NexusAudit — Static Analysis Checklist

Run every item. Mark: ✅ SAFE | ⚠️ REVIEW | 🔴 FINDING

## Reentrancy
- [ ] All functions that transfer ETH/tokens use ReentrancyGuard or CEI pattern
- [ ] No state changes AFTER external calls (without guard)
- [ ] Cross-function reentrancy: can function A re-enter function B sharing state?

## Arithmetic
- [ ] Solidity version >= 0.8.0 (overflow protection built-in)
- [ ] Are there `unchecked{}` blocks? If yes, are they safe?
- [ ] Division before multiplication (precision loss)?
- [ ] Rounding: does rounding always favor the protocol, never the attacker?

## Access Control
- [ ] Every sensitive function has appropriate modifier
- [ ] `address(0)` checked in all setters that accept addresses
- [ ] Owner/operator transfer is two-step (Ownable2Step)
- [ ] No function callable by address(0) as msg.sender

## External Calls
- [ ] Return values from external calls checked
- [ ] SafeERC20 used for all ERC-20 interactions
- [ ] External contracts are known/trusted/immutable
- [ ] Is there a risk of `address.call` returning false silently?

## Integer Edge Cases
- [ ] What happens when amount = 0? (division by zero, empty loops)
- [ ] What happens when amount = type(uint256).max?
- [ ] What happens when arrays are empty?
- [ ] What happens when arrays have length 1 vs. length 1000?

## Events
- [ ] All state changes emit events
- [ ] Events emitted AFTER state changes (not before)
- [ ] Events contain enough data to reconstruct state off-chain

## Initialization
- [ ] Constructor sets all critical state variables
- [ ] No uninitialized storage pointers
- [ ] If upgradeable: `_disableInitializers()` in constructor

## Logic
- [ ] Are there functions that can be called in wrong order?
- [ ] Are there time-based conditions? Can block.timestamp be manipulated? (±15s on mainnet)
- [ ] Are there off-by-one errors in comparisons (> vs >=)?
- [ ] Dead code or unreachable branches?

## ERC Compliance
- [ ] ERC-20: transfer/transferFrom return bool handled
- [ ] ERC-3009: signature validation parameters complete
- [ ] Chainlink Automation: checkUpkeep view, performUpkeep callable by anyone safely

## Gas & Loops
- [ ] No unbounded loops over user-controlled arrays
- [ ] Storage reads inside loops cached to memory
- [ ] No string operations inside loops
- [ ] Batch operations have size limits

## Centralization
- [ ] Document all privileged roles and their powers
- [ ] Worst-case: what can each role do if malicious?
- [ ] Are there timelocks on critical operations?
- [ ] Is there a pause mechanism?
