# Default Report Skeleton

This is the default format used by the skill. It follows a Description → Impact → PoC → Recommendations flow with a vulnerability type field.

To use this skeleton, no changes are needed — it is the active format out of the box.

## Skeleton

````markdown
### {Title}

**Severity:** {Critical | High | Medium | Low}

## Description

{Paragraph 1: Context — how the feature works and who uses it. No vulnerability yet.}
{Paragraph 2: The problem — what goes wrong and the root cause.}
{Paragraph 3 (optional): Bridge to impact — what this means in practice.}

```{solidity|rust}
{Code snippet — only if essential to understanding. Prefer inline code references.}
```

## Impact

{2-4 sentences. Concrete consequences. Quantify what is lost or gained. State who is affected.}

## Scenario PoC

{Step-by-step scenario using Alice/Bob. One sentence per step. Only for Critical/High/Medium.}

1. Alice does X.
2. Bob does Y.
3. Result: Z happens because of the vulnerability.
4. Outcome: Alice loses N tokens / Bob gains unauthorized access.

## Recommended Mitigation:

It is recommended to {specific fix} which will {why it works}.

```diff
  {surrounding context line}
+ {line to add}
- {line to remove}
  {surrounding context line}
```

```

## Section Rules

| Section            | Required                  | Notes                                                                                   |
| ------------------ | ------------------------- | --------------------------------------------------------------------------------------- |
| Title              | Always                    | `{Actor} Can {Impact} {Affected Party}` — every word capitalized, no ending punctuation |
| Severity           | Always                    | Must come from the Likelihood × Impact matrix                                           |
| Vulnerability Type | Always                    | Use categories from `references/vulnerability-classes/`                                 |
| Description        | Always                    | Context → Problem → Impact bridge. Max 4 lines per paragraph                            |
| Impact             | Always                    | Concrete, quantified consequences. 2-4 sentences                                        |
| PoC                | Critical/High/Medium only | Alice/Bob scenario. One sentence per step. Omit for Low                                 |
| Recommendations    | Always                    | "It is recommended to..." + diff block                                                  |
```
````
