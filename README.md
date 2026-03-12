# 🛡️ vuln-report-writer-skill

> A Claude skill for writing professional smart contract vulnerability reports — fast, consistent, and ready for submission. Built by [Naman](https://github.com/namx05)

Built for security researchers, auditors, and bug bounty hunters who work with **Solidity** and **Rust/Solana** codebases.

---

## What This Skill Does

Writing audit findings is tedious. You know the bug, but formatting it into a clean, structured report takes time you'd rather spend finding the next one.

This skill turns your raw vulnerability description (or code snippet) into a complete, submission-ready finding with:

- A properly formatted title (`{Actor} Can {Impact} {Affected Party}`)
- Severity assigned using a Likelihood × Impact matrix
- Structured Description → Impact → PoC → Recommendations flow
- Scenario-based PoCs with Alice and Bob for Critical/High/Medium findings
- Concrete `diff` blocks showing the recommended fix
- Clean, consistent markdown every time

---

## Install & Run

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed and authenticated
- Or access to [Claude.ai](https://claude.ai) with skills support

### Installation

```bash
claude install-skill https://github.com/namx05/vuln-report-writer
```

That's it. The skill activates automatically when you ask Claude to write a vulnerability report.

### Usage

Describe the bug naturally. Here are some example prompts:

```
Write a finding for this reentrancy bug in the withdraw function
```

```
Format this as an audit report: the price oracle can be manipulated
because it reads spot price from a single DEX pool
```

```
I found a missing signer check in this Solana program — document it as a finding
```

You can also paste vulnerable code directly and ask Claude to analyze and document it.

### Trigger Phrases

The skill activates on phrases like:

- "write a finding" / "write an issue"
- "write a report" / "vulnerability report" / "audit finding"
- "format this bug" / "write this as a finding"
- "document this issue"

Or any context where you're describing a smart contract security issue and want it formatted professionally.

---

## Example Output

```markdown
### Attacker Can Steal Deposited Funds By Calling Withdraw With Arbitrary Token ID

**Severity:** Critical

**Vulnerability Type:** Missing Access Control

## Description

The `Vault` contract allows users to deposit ERC-20 tokens and receive a token ID
representing their position. Users call `Vault.withdraw()` with their token ID to
reclaim their deposited tokens.
However, `Vault.withdraw()` does not verify that `msg.sender` is the owner of the
provided token ID. Any user can pass any valid token ID and receive the underlying
tokens.

## Impact

All deposited funds in the Vault are at risk. An attacker can steal 100% of the
total value locked by iterating through active token IDs.

## PoC

1. Alice deposits 1000 USDC into the Vault and receives token ID 42.
2. Bob calls `Vault.withdraw(42)` passing Alice's token ID.
3. The contract transfers 1000 USDC to Bob without checking ownership.
4. Alice's position is destroyed and her funds are gone.

## Recommendations

It is recommended to add an ownership check in the `withdraw()` function to ensure
only the token owner can withdraw the underlying assets.
```

---

## Severity Matrix

The skill uses a Likelihood × Impact matrix to assign severity:

|                        | Impact: High | Impact: Medium | Impact: Low |
| ---------------------- | ------------ | -------------- | ----------- |
| **Likelihood: High**   | Critical     | High           | Medium      |
| **Likelihood: Medium** | High         | Medium         | Low         |
| **Likelihood: Low**    | Medium       | Low            | Low         |

### Quick Reference

| Severity     | What It Means                                                                 |
| ------------ | ----------------------------------------------------------------------------- |
| **Critical** | Anyone can exploit it. Protocol-ending. 20%+ value at risk.                   |
| **High**     | Funds directly at risk. Significant disruption. Hard but high-reward exploit. |
| **Medium**   | Indirect risk. Potential future exploit. Centralization concerns.             |
| **Low**      | Best-practice deviation. QA. Gas optimization. Negligible impact.             |

See [`references/severity-guide.md`](references/severity-guide.md) for the full classification rules including how to differentiate between adjacent severities.

---

## Report Structure

The skill uses **swappable report skeletons** — you can change the output format by editing one line in [`assets/templates/FORMATS.md`](assets/templates/FORMATS.md).

### Shipped Formats

**Default** (`report-skeletons/default.md`) — used out of the box:

```
### {Actor} Can {Impact} {Affected Party}
**Severity:** {Critical | High | Medium | Low}
**Vulnerability Type:** {Category}
## Description       — Context → Problem → Bridge to impact
## Impact            — Concrete consequences, quantified where possible
## PoC              — Step-by-step scenario (Critical/High/Medium only)
## Recommendations  — Specific fix with diff block
```

**CodeHawks** (`report-skeletons/codehawks.md`) — for competitive audits:

```
### {Actor} Can {Impact} {Affected Party}
**Severity:** {Critical | High | Medium | Low}
## Summary           — 2-3 sentence elevator pitch of the issue
## Vulnerability Details — Full technical breakdown with code
## Impact            — Concrete consequences
## Tools Used        — Manual Review, Foundry, Slither, etc.
## Recommendations   — Specific fix with diff block
```

### Switching Formats

Open `assets/templates/FORMATS.md` and change the active format line:

```
**Current active format:** `codehawks.md`
```

### Creating Your Own Format

1. Create a new `.md` file in `assets/templates/report-skeletons/` (e.g., `sherlock.md`)
2. Define your skeleton structure with `{placeholder}` fields
3. Add a section rules table explaining each section
4. Set it as active in `FORMATS.md`

See [`assets/templates/FORMATS.md`](assets/templates/FORMATS.md) for the full guide on creating custom skeletons.

---

## Supported Ecosystems

- **Solidity** — EVM-based smart contracts (Ethereum, Polygon, Arbitrum, Base, etc.)
- **Rust / Anchor** — Solana programs

---

## Contributing

Contributions are open and very much appreciated! This skill was built by auditors, for auditors — and your real-world experience makes it better.

### How to Contribute

1. **Fork** this repository
2. **Create a branch** for your change (`git checkout -b feature/my-improvement`)
3. **Make your changes** following the structure described below
4. **Test** by installing your fork as a skill and running a few prompts
5. **Open a Pull Request** with a clear description of what you changed and why

### Areas You Can Improve

Here are specific ways to contribute, organized by the part of the skill they touch:

#### 🧬 New Vulnerability Classes (`references/vulnerability-classes/`)

The vulnerability class files define the categories used in the `Vulnerability Type` field. You can:

- **Add new categories** to the existing Solidity or Rust/Solana files (e.g., new DeFi-specific patterns like sandwich attacks, JIT liquidity exploits, or governance manipulation vectors)
- **Add entirely new ecosystem files** for chains not yet covered:
  - `cosmwasm.md` — CosmWasm / Cosmos smart contracts
  - `move.md` — Aptos / Sui Move modules
  - `cairo.md` — Starknet Cairo contracts
  - `vyper.md` — Vyper contracts on EVM
  - `ink.md` — ink! contracts on Polkadot/Substrate
- **Refine existing categories** with more specific sub-types or clearer descriptions

#### 📝 Example Findings (`references/examples/`)

Example findings are what calibrate the tone, depth, and structure of generated reports. You can:

- **Add more examples** for edge cases (flash loan attacks, cross-chain bridge exploits, governance attacks, MEV-related findings)
- **Add examples for new ecosystems** matching any new vulnerability class files you create
- **Improve existing examples** with more realistic code snippets, better PoC scenarios, or clearer recommendations
- **Add examples for specific DeFi primitives** (AMMs, lending protocols, yield aggregators, perpetual DEXs, restaking)

#### ⚖️ Severity Classification (`references/severity-guide.md`)

The severity guide drives how the skill assigns Critical/High/Medium/Low. You can:

- **Add more differentiation heuristics** — real examples of edge cases where severity is hard to call (e.g., "oracle manipulation that requires $10M capital" — is it High or Medium?)
- **Add protocol-specific severity guidance** — how severity changes based on protocol type (a rounding error in a lending protocol vs a simple vault has very different impact)
- **Improve the key questions** section with scenarios from real audits

#### ✍️ Writing Quality (`references/writing-style.md`)

The writing style guide controls sentence-level quality. You can:

- **Add more "avoid → use instead" patterns** from your own audit writing experience
- **Add ecosystem-specific conventions** (how Solana findings typically reference accounts vs how Solidity findings reference `msg.sender`)
- **Improve the quantifying impact guidance** with more specific patterns
- **Add examples of good vs bad sentences** pulled from real audit reports

#### 📋 Report Skeletons (`assets/templates/report-skeletons/`)

Report skeletons are the most community-friendly contribution — anyone can add a format they use. You can:

- **Add platform-specific skeletons** that match the exact submission format for:
  - `code4rena.md` — Code4rena contest submissions (with `[H-01]` / `[M-01]` labels, QA report format for Low/NC)
  - `sherlock.md` — Sherlock contest format (root-cause grouping, escalation structure)
  - `cantina.md` — Cantina reviews
  - `immunefi.md` — Immunefi bug bounty format (heavy emphasis on PoC and reproduction steps)
  - `hackerone.md` — HackerOne submission format
- **Add a full-audit-report skeleton** that wraps multiple findings into a complete document with executive summary, scope table, findings list, and severity breakdown
- **Add a gas-optimization skeleton** with a lighter format for Low-severity gas findings
- **Add an informational skeleton** for findings that don't follow the full severity structure
- **Improve existing skeletons** with better placeholder descriptions or section rules
- **Add section rule tables** to any skeleton that's missing them

Each skeleton file should follow the structure documented in [`FORMATS.md`](assets/templates/FORMATS.md): a header, the skeleton block, and a section rules table.

#### 📐 Universal Rules (`assets/templates/finding-template.md`)

The finding template contains rules that apply across all skeletons. You can:

- **Add more good/bad title examples** from real audit reports
- **Improve code reference conventions** for specific ecosystems
- **Add guidelines for diff blocks** in complex multi-file fixes

#### 🔧 Core Skill Logic (`SKILL.md`)

The main skill file orchestrates everything. You can:

- **Improve the "Before You Write" workflow** — add steps that help with common gotchas
- **Add support for batch findings** — processing multiple vulnerabilities from a single audit into a cohesive report
- **Add cross-referencing** between related findings (e.g., "See also Finding #3 which exploits the same root cause")
- **Improve code-context handling** — better instructions for when the user pastes an entire contract vs describes a bug verbally

#### 🌐 New Features

Bigger contributions that extend what the skill can do:

- **Multi-finding report generation** — a wrapper that combines individual findings into a full audit report with table of contents, severity breakdown, and executive summary
- **Automated severity suggestion** — heuristics that pre-classify severity based on vulnerability type and context before the user confirms
- **Platform-specific formatters** — output adapters that convert the standard finding into the exact format needed for Code4rena, Sherlock, Immunefi, Cantina, or HackerOne submissions
- **Comparison mode** — compare two versions of a contract and generate a finding based on the diff

### Contribution Guidelines

- Keep reference files focused. Each file should cover one topic well.
- Example findings must be realistic — based on patterns from real audits, not hypothetical toy examples.
- Report skeletons must include a `## Skeleton` block and a `## Section Rules` table. See existing skeletons for reference.
- Test your changes by installing the modified skill and generating 2-3 findings to verify the output quality.
- Follow the existing file naming conventions (`kebab-case.md`).
- If adding a new ecosystem, include both a vulnerability class file AND at least 3 example findings.
- If adding a new report skeleton, test it by setting it as active in `FORMATS.md` and generating findings in at least two severity levels.

### Reporting Issues

Found a bug, a misclassified severity, or a poorly worded example? [Open an issue](../../issues) with:

- What you expected vs what you got
- The prompt you used (if applicable)
- The ecosystem (Solidity / Rust / other)

---

## ⭐ Star This Repo

If this skill saves you time writing findings, consider giving it a star! It helps other auditors and researchers discover the tool — and motivates continued development.

---

## License

MIT — use it, fork it, improve it.
