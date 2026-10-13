# Smart Contract Audit Archive

Real audit examples from my smart contract security work. Every report in `reports/` is a genuine audit against a deployed, publicly-verifiable contract — not a synthetic demo. Addresses are real; anyone can check the findings against the actual deployed bytecode on Etherscan.

See [`INDEX.md`](./INDEX.md) for the categorized breakdown of the 50-contract scan this archive is drawn from.

## Why real contracts, not synthetic examples

A report against a toy contract proves I can write Markdown. A report against a real 2018-era contract, with line-numbered findings that hold up when you open the actual verified source on Etherscan, proves something else. The three reports here include:

- A case where the finding **held up** under manual verification (access-control logic bug, confirmed against the real source).
- A case where a tool's initial "reentrancy" flag was investigated and correctly scoped as **low risk**, with the reasoning shown.
- A case where an initial "steal all funds" claim was investigated with a **live Foundry mainnet-fork test** and found to be a **false positive** — included specifically because showing you how a false positive gets caught is more useful than hiding it.

## Methodology (same for every report in this archive)

1. Static analysis: Slither, Aderyn, Mythril, Semgrep (Solidity-scoped), Heimdall for unverified bytecode.
2. Cross-validation: a finding only reaches HIGH confidence if corroborated by a second tool or a manual read of the real source.
3. Manual verification for anything claiming fund theft, full-drain, or access-control bypass — including Foundry fork tests against mainnet state when the claim is material.
4. Noise filtering: detector-level style findings (`naming-convention`, `name-reused`, `solc-version`, etc.) are excluded from severity counts — they're not vulnerabilities.

---

Audited by Nicolás Talero — Founder & Lead Security Auditor @ Digital Nova Solutions SAS
