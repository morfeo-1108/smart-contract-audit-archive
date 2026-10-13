# Audit Report — IceRockMining Token Contract

**Target:** `0xC16b542ff490e01fcc0DC58a60e1EFdc3e357cA6` (Ethereum mainnet)
**Deployed:** 2018 · Solidity `^0.4.18` · Source verified on Etherscan
**Scope:** Full contract (custom ERC20-style token with dividend distribution and a custom `itMaps` storage library)
**Tools:** Slither, Aderyn, Mythril, Semgrep, Solodit pattern match, automated review with static cross-validation

## Summary

| Severity | Count | Notes |
|---|---|---|
| Critical | 0 | |
| High | 7 | All investigated manually — see below for which hold up and which don't |
| Medium | 23 | Mostly accounting/arithmetic, standard for pre-SafeMath era |
| Low/Informational | 43 | Style, naming, dead code — not included in this summary's severity weighting |

## Finding: broken `onlyOwner` modifier (confirmed real)

```solidity
modifier onlyOwner {
    if (msg.sender == IceRockMiningAddress) {
        _;
    }
}
```

Flagged by Slither (`incorrect-modifier`, High confidence) as not executing `_;` or reverting on all paths. Confirmed by reading the real source: if the caller is not the owner, the modifier simply **does not execute the function body** — but it also does not revert. The call returns successfully having done nothing.

**Impact, after manual review:** this is a correctness bug, not a privilege-escalation vulnerability. A non-owner calling an `onlyOwner` function gets silent no-op behavior instead of either (a) the expected side effect or (b) a clear revert. It does **not** allow a non-owner to execute owner-gated logic. Downgraded from "access control bypass" (the review layer's initial framing) to **logic correctness defect** after verification.

## Finding: "owner can drain all funds" (verified, correctly scoped)

```solidity
function withdraw(uint amount) public onlyOwner {
    IceRockMiningAddress.transfer(amount);
}
```

The automated review flagged this as "owner can drain all funds" (LOW confidence, flagged `needs_review`). Verified: this is the contract's own owner-withdrawal function, gated by the (broken-but-not-bypassable) modifier above. It is centralization risk — the same category as "has no timelock" — not an externally exploitable vulnerability. Reported as a centralization/governance finding, not a security vulnerability.

## Finding: reentrancy in `transferAllDividends()` (scoped as Low — correctly, not inflated)

```solidity
function transferAllDividends() external {
    for (...) {
        tokenHolder.transfer(approvedDividends[tokenHolder]); // external call
        // ...
        approvedDividends[tokenHolder] = 0;                   // state write AFTER the call
    }
}
```

Classic checks-effects-interactions violation — but the external call uses gas-limited `.transfer()` (2300 gas stipend), which is insufficient for a meaningful nested call back into the contract. Real risk here is **not** fund drain via reentrancy; it's **denial-of-service**: a single token holder whose fallback reverts (or simply isn't payable) can revert the entire loop, blocking dividend distribution for every other holder. Reported as Low/DoS, not High/fund-theft — matching Slither's own severity assignment, not inflating it.

## Verdict

0 Critical, 0 confirmed High-severity exploitable vulnerabilities after manual review. The real findings are code-quality and centralization-risk issues typical of 2018-era Solidity (no SafeMath, custom storage library with unchecked return values, broken-but-safe access modifier). No path for an external attacker to drain contract funds was identified.

---

Audited by Nicolás Talero — Founder & Lead Security Auditor @ Digital Nova Solutions SAS
