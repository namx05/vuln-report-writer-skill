# Finding Template — Universal Rules

These rules apply to ALL report skeletons regardless of which format is active. They govern the title, severity, and general structure of every finding.

## Title Format

Pattern: `{Actor} Can {Impact} {Affected Party}`

- Every word capitalized.
- Never end with punctuation.
- The actor should be specific (Attacker, Malicious Validator, Any User) not generic ("Someone").
- The impact should describe the action (Drain, Steal, Bypass, Overwrite, Brick, Inflate) not the bug class (Reentrancy, Missing Check).

### Good Titles

- `Attacker Can Drain Staking Rewards From Any Depositor`
- `Malicious Validator Can Bypass Slashing Mechanism In Consensus Module`
- `Anyone Can Overwrite User Profiles Due To Missing PDA Seed Uniqueness`
- `First Depositor Can Inflate Share Price And Steal Subsequent Deposits`
- `Liquidator Can Seize Healthy Positions By Manipulating Oracle Price`

### Bad Titles

- `Reentrancy Bug` — no actor, no impact, no affected party
- `Missing Access Control in withdraw()` — no actor, no consequence described
- `Potential Issue With Token Transfers.` — vague, ends with punctuation
- `Vulnerability in the Protocol` — says nothing specific
- `Users Can Lose Funds` — too generic, which users? how?

## Severity

Always assigned using the Likelihood × Impact matrix from `references/severity-guide.md`. Never assign severity based on gut feeling alone.

## Code References (All Formats)

- Standalone functions: `foo()`
- Contract/program functions: `Contract.foo()` or `program::foo()`
- Variables and fields: `balanceOf`, `total_supply`
- Prefer single-line inline code over multi-line blocks
- Multi-line code blocks only for vulnerable snippets essential to understanding

## Diff Blocks (All Formats)

- Always use fenced diff syntax with `+` and `-` markers
- Show 1-2 surrounding unchanged lines for context
- Must be implementable code, not pseudocode
- Multiple changes in different files get separate diff blocks with a location comment
