# Severity Classification Guide

Use this reference when assigning severity to a finding. Severity is the intersection of **Likelihood** (how easy and probable the exploit is) and **Impact** (how much damage it causes).

## Matrix

| | Impact: High | Impact: Medium | Impact: Low |
|---|---|---|---|
| **Likelihood: High** | Critical | High | Medium |
| **Likelihood: Medium** | High | Medium | Low |
| **Likelihood: Low** | Medium | Low | Low |

---

## Severity Definitions

### Critical
- Easily exploitable by anyone, causing loss of assets or total undermining of the protocol's goal.
- A malicious actor can extract 20%+ of protocol value.
- Griefing attacks with wide-bearing implications (e.g., bricking withdrawals causing all users to lose deposited funds).
- Can be repeated to affect every user → Critical (not just High).
- Plays out in days, not months → Critical.

### High
- Funds directly or nearly directly at risk.
- Severe disruption of protocol functionality or availability.
- Not straightforward to exploit, but consequences are large.
- Attacks requiring significant capital that cannot be flash-loaned (TWAP manipulation, reference exchange manipulation).
- Significant griefing attacks — e.g., causing $500+/day in extra gas costs.
- DoS with wide implications on core protocol functionality.
- Affects one special user only (cannot be generalized) → High, not Critical.
- Takes months to play out → High, not Critical.

### Medium
- Funds indirectly at risk.
- Some disruption to functionality but not catastrophic.
- No significant impact, but not trivial.
- Significant impact but extremely rare likelihood.
- Rounding in the wrong direction that can be exploited over time.
- Centralization risks.
- If users behave correctly, does it still impact them noticeably? Yes → Medium+.

### Low
- Minor deviation from best practices.
- Gas optimizations, QA issues.
- Bugs with negligible or literally no impact.
- If users behave correctly, no noticeable impact → Low.

---

## Key Questions for Differentiating

### Critical vs High
- How guaranteed is the exploit? 100% certain profit → leaning Critical. ~70% → High.
- Does it affect all users significantly? → Probably Critical.
- Can it be repeated against every user? → Critical.
- Only applies to one special user? → High.
- Takes months to play out? → High. Days? → Critical.

### High vs Medium
- Rare but huge consequences → Likely still High.
- Rare with decent consequences → Probably Medium.
- Probable with huge consequences → High or Critical.
- Probable with decent consequences → High.
- Relies on frontrunning on a chain with no mempool? → High (or Medium).

### Medium vs Low
- If users behave correctly, does it impact them noticeably (besides gas)? Yes → Medium+. No → Low.
- Does it pose a risk of nontrivial exploit in the future? Yes → Medium. No → Low.
