# Writing Style Guide

Rules and patterns for writing clear, professional vulnerability reports.

## Sentence-Level Rules

- Never start a sentence with "This" without a noun. Bad: "This allows..." → Good: "This flaw allows..."
- Never use filler: "It should be noted that", "It is important to", "It is worth mentioning"
- Every sentence must carry new information. If you can delete a sentence without losing meaning, delete it.
- Use active voice. Bad: "The funds can be stolen by the attacker." → Good: "The attacker can steal the funds."
- Prefer concrete over abstract. Bad: "This has negative consequences." → Good: "The attacker drains the liquidity pool."

## Paragraph Structure

- Maximum 4 lines per paragraph.
- No extra blank lines between paragraphs within a section.
- First sentence of each paragraph sets context for what follows.

## Code References

- Standalone functions: `foo()`
- Contract/program functions: `Contract.foo()` or `program::foo()`
- Variables and fields: `balanceOf`, `total_supply`
- Use single-line inline code. Only use fenced code blocks for vulnerable snippets essential to understanding.

## Quantifying Impact

Prefer specific language over vague:
- Bad: "funds are at risk"
- Good: "all staked funds are at risk"
- Better: "the entire TVL (~$5M at current value) can be drained in a single transaction"

When exact amounts are unknown, describe the scope:
- "all users who deposited before the upgrade"
- "any vault with more than 0 balance"
- "the protocol's fee accumulator"

## Scenario Writing (PoC)

- Use Alice, Bob, Carol, Dave as actor names.
- Each step is one sentence.
- Steps must be sequential — each builds on the previous.
- Include specific amounts (e.g., "100 USDC", not "some tokens").
- Final step states the outcome and who loses what.

## Diff Blocks

- Always use fenced diff syntax with `+` and `-` markers.
- Show enough surrounding context (1-2 unchanged lines) for the reader to locate the change.
- Diffs must be implementable — not pseudocode.
- If multiple changes in different functions/files, use separate diff blocks with a comment indicating location.

## Words to Avoid

| Avoid | Use Instead |
|---|---|
| "It should be noted" | (delete, just state the fact) |
| "It is important to" | (delete, just state the fact) |
| "Basically" | (delete) |
| "Simply" | (delete) |
| "Obviously" | (delete) |
| "In order to" | "To" |
| "At this point in time" | "Now" |
| "Due to the fact that" | "Because" |
| "A non-trivial amount" | Specify the amount or say "significant" |
