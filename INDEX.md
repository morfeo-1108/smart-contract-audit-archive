# Index — 50 Contracts Scanned

Aggregate results from a full pass through the audit pipeline (Slither + Aderyn + Mythril + Semgrep + Heimdall, with cross-validation). Numbers below are **after** filtering out tool-noise detectors (`name-reused`, `naming-convention`, `solc-version`, and similar style-only findings) that don't represent real vulnerabilities — a report that doesn't do this filtering is reporting tool artifacts, not security findings.

## Summary

| | Count |
|---|---|
| Contracts scanned | 50 |
| **With ≥1 Critical/High finding** (real, after noise filter) | **29** |
| With real findings capped at Medium severity | 11 |
| With real findings capped at Low/Informational | 7 |
| Clean — zero real findings at any severity | 3 |

Every row above links to a real, individual report in [`reports/full-scan/`](./reports/full-scan/) — 50 files, one per contract, each showing exactly what was found and what was filtered as tool-noise. Click through and check any number against the actual report.

A clean result isn't a failure of the audit — a well-written, access-controlled, non-reentrant contract with checked transfers *should* come back clean.

## Finding categories observed across the 29 flagged contracts

| Category | Occurrences | Real-world impact |
|---|---|---|
| Unchecked ERC20 transfer return value | 13 | Silent failure on non-standard tokens (e.g. USDT-style) can desync internal accounting |
| Integer arithmetic (pre-0.8, no SafeMath) | 10 | Overflow/underflow risk depends on reachability — verified case-by-case, not assumed |
| ETH locked with no withdraw path | 5 | The exact pattern this archive's fund-recovery research targets |
| Hardcoded secret or address | 4 | Mixed: some are benign immutable references, some are real centralization risk — verified individually |
| Reentrancy: state change after external call | 3 (cross-validated) + 3 (`reentrancy-eth`) | Severity depends heavily on whether the call uses gas-limited `.transfer()`/`.send()` or unbounded `.call{value}()` — see `reports/etherToken-reentrancy-review.md` for a worked example of scoping this correctly |
| `abi.encodePacked()` hash collision | 3 | Real when used with multiple dynamic-length arguments in a signature/proof scheme |
| `arbitrary-send-eth` | 2 | Function sends ETH to an address influenced by caller input without sufficient access control |
| Weak PRNG | 2 | On-chain randomness derived from predictable values (`block.timestamp`, `blockhash`) |
| Uninitialized state | 2 | Storage variable read before being set anywhere in the constructor/init path |
| Delegatecall to user-controlled address | 1 | Full contract takeover if reachable without access control |

## Worked-example reports (deep-dive methodology)

- [`reports/icerockmining-access-control-review.md`](./reports/icerockmining-access-control-review.md) — access-control modifier bug, confirmed against real source
- [`reports/etherToken-reentrancy-review.md`](./reports/etherToken-reentrancy-review.md) — reentrancy flag investigated and correctly scoped as low risk
- [`reports/f2m-false-positive-case-study.md`](./reports/f2m-false-positive-case-study.md) — "steal all funds" claim, investigated, confirmed false positive via Foundry fork test

## Full scan — all 50 contracts

- [`reports/full-scan/README.md`](./reports/full-scan/README.md) — one real report per contract, sorted by findings. Every number in the summary table above traces back to one of these 50 files.

---

Audited by Nicolás Talero Riaño — Founder & Lead Security Auditor @ Digital Nova Solutions SAS
