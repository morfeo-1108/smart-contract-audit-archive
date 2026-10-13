# Audit Report — EtherToken (WETH-style wrapper)

**Target:** `0xc0829421C1d260BD3cB3E0F06cfE2D52db2cE315` (Ethereum mainnet)
**Deployed:** 2018 · Solidity `v0.4.16` · Source verified on Etherscan
**Scope:** Full contract — Bancor-pattern ETH wrapper (`deposit()`/`withdraw()`, `Utils`/`Owned`/`ERC20Token`/`TokenHolder` base contracts)
**Tools:** Slither, Aderyn, Mythril, Semgrep, Solodit pattern match, automated review with static cross-validation

## Summary

| Severity | Count |
|---|---|
| Critical | 0 |
| High | 0 |
| Medium | 5 |
| Low/Informational | 43 |

This is a worked example of **scoping a reentrancy flag correctly instead of defaulting to High severity because the word "reentrancy" appears.**

## Finding: reentrancy in `withdrawTo()` — scoped as Low (not High)

```solidity
function withdrawTo(address _to, uint256 _amount)
    public
    protectSafe
{
    balanceOf[msg.sender] = safeSub(balanceOf[msg.sender], _amount);
    _to.transfer(_amount);                      // external call
    Destruction(_amount);                        // event emitted AFTER
    Transfer(msg.sender, this, _amount);          // event emitted AFTER
}
```

Slither flags this as `reentrancy-eth` because state-unrelated events fire after an external call. On manual review, the actual balance deduction (`balanceOf[msg.sender] = ...`) happens **before** the external call — the checks-effects-interactions pattern is respected for the value that matters. The only things that happen after the call are **event emissions**, which carry no exploitable state. Additionally, `.transfer()` caps the forwarded gas at 2300, which is below what's needed for any meaningful reentrant call on current mainnet gas costs.

**Why this matters for how we report findings:** a less careful pass would report this as "High — reentrancy vulnerability" because the pattern superficially matches the checks-effects-interactions violation shape. Reading the actual order of operations (balance update vs. event emission) changes the correct severity from High to Low. We report the Low.

## Finding: `deposit()` does not emit `Transfer()` (flagged by the review layer, LOW confidence, under review)

Flagged by the automated review layer, not corroborated by static tools. This is a real ERC20-compliance gap (some indexers/integrations expect a `Transfer(0x0, user, amount)` event on mint-equivalent operations) but not a security vulnerability — funds are correctly tracked internally regardless of the missing event. Reported at LOW confidence per our grounding policy: findings from the review layer alone without static-tool corroboration don't get promoted to HIGH without manual confirmation.

## Verdict

No exploitable vulnerability identified. The contract correctly implements checks-effects-interactions for the value-affecting state change; the reentrancy *pattern* flagged by automated tooling does not correspond to a real exploit path once the actual order of operations is read. ETH held by the contract (27.15 ETH at time of review) is the aggregate of individual wrapped balances — not trapped or recoverable funds.

---

Audited by Nicolás Talero — Founder & Lead Security Auditor @ Digital Nova Solutions SAS
