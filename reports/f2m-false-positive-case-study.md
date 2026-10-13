# Case Study — A "Steal All ETH" Finding That Didn't Hold Up

**Target:** `0xF211128cC6d925A3a328647Cf78b322b51429C53` ("F2m")
**Function in question:** `swapToken()`, lines 824–837
**Initial claim (LLM review layer):** "Attacker can steal all ETH from F2m" — High severity

This case study exists because showing how a false positive gets caught is more useful to a prospective client than hiding it ever happened.

## The initial finding

The LLM review flagged `swapToken()` as allowing an attacker to redirect the contract's entire ETH balance to an address of their choosing, with no access control stopping it. Taken at face value, that's a Critical/High finding and the kind of thing that ends up in a report headline.

## Verification steps taken before accepting the finding

1. **Read the real deployed source directly from Etherscan** rather than relying on the LLM's paraphrase of the function. The destination address referenced in `swapToken()` is set via an admin-controlled setter elsewhere in the contract — it is **not** a parameter the caller supplies, and not derived from any caller-controlled input.
2. **Checked who can call the setter** that configures that destination — gated by an owner check.
3. Because the claim was material (full fund drain) and the static read left some ambiguity about execution order, we did not stop at source reading: we **forked Ethereum mainnet with Foundry/Anvil at the current block** and attempted to actually execute the drain path as an unprivileged attacker, against live contract state.

```
Fork: Ethereum mainnet @ current block
Actor: unprivileged EOA, no special permissions
Action attempted: call swapToken() to redirect contract ETH to attacker-controlled address
Result: destination resolves to the admin-set address regardless of caller —
        attacker-controlled redirection is not possible.
```

## Why this matters

An audit pipeline that reports every LLM-flagged "steal all funds" claim without verification will eventually put a false Critical in front of a client — and the cost of that isn't just an awkward correction, it's the client making a real decision (freezing a contract, panicking stakeholders, paying for incident response) based on something that was never true.

The finding was downgraded to `FALSE_POSITIVE_AUTO` / confidence LOW after this review, with the destination-is-fixed reasoning recorded against it. It did not ship as a Critical finding.

## What this demonstrates about the review process

- LLM-flagged findings with "steal/drain/theft" language get an automatic secondary check against the contract's real access-control modifiers before they're trusted.
- When a claim is material enough (full fund drain, in this case), static reading of source is treated as necessary but not sufficient — a live fork test against real state is the actual bar for confirming or rejecting it.
- A finding that doesn't survive this process gets reported as a false positive with the reasoning shown, not quietly deleted.
