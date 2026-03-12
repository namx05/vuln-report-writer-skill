# Example Findings — Rust / Solana

These are reference examples of well-written findings for Solana programs. Read these when writing a new finding to calibrate tone, structure, and depth.

---

## Example 1: Critical — Missing Signer Check

### Anyone Can Withdraw Vault Funds Due To Missing Authority Validation

**Severity:** Critical

**Vulnerability Type:** Missing Signer Authorization

## Description
The `withdraw` instruction in the Vault program transfers SOL from the vault PDA to a specified recipient. The instruction accepts an `authority` account that is supposed to be the vault owner.
However, the instruction does not verify that the `authority` account is a signer. An attacker can pass the real authority's public key as a non-signing account and set their own address as the recipient. The PDA signature is derived from the authority key alone, so the CPI transfer succeeds without the authority ever signing the transaction.

## Impact
All SOL held in any vault can be stolen by anyone. The attacker only needs to know the authority's public key, which is typically derivable or publicly visible on-chain. No capital or special access is required.

## PoC
1. Alice creates a vault with 500 SOL, setting herself as authority.
2. Bob constructs a `withdraw` instruction passing Alice's pubkey as `authority` (non-signer) and his own address as `recipient`.
3. The PDA derivation succeeds because it only depends on Alice's pubkey. The CPI transfer sends 500 SOL to Bob.
4. Alice's vault is drained with no way to recover the funds.

## Recommendations
It is recommended to enforce that the `authority` account is a signer, which will prevent unauthorized withdrawals.

```diff
  #[derive(Accounts)]
  pub struct Withdraw<'info> {
-     pub authority: AccountInfo<'info>,
+     #[account(signer)]
+     pub authority: Signer<'info>,
      #[account(mut, seeds = [b"vault", authority.key().as_ref()], bump)]
      pub vault: Account<'info, Vault>,
  }
```

---

## Example 2: High — Type Cosplay / Account Confusion

### Attacker Can Pass Fake Pool Account To Drain Swap Reserves

**Severity:** High

**Vulnerability Type:** Type Cosplay

## Description
The `swap` instruction in the DEX program reads pool state from a passed-in account to determine the exchange rate. It deserializes the account data using `Pool::try_from_slice()`.
The instruction does not verify the account's discriminator or that it is owned by the DEX program. An attacker can create a fake account with crafted data that mimics the `Pool` struct layout but with a manipulated exchange rate, then pass it as the pool account.

## Impact
An attacker can execute swaps at an artificially favorable rate, draining real reserves from the pool into their own token account. The attack requires deploying a fake account with matching data layout but costs only the account rent.

## PoC
1. The real pool holds 10,000 USDC and 10,000 TOKEN at a 1:1 rate.
2. Bob creates a fake account with data that deserializes as a `Pool` with a 1:1000 rate.
3. Bob calls `swap` with the fake pool account, exchanging 10 TOKEN for 10,000 USDC.
4. The real reserves are transferred to Bob because the CPI uses the real token vaults.

## Recommendations
It is recommended to validate the account discriminator and program ownership before deserialization. Using Anchor's `Account<'info, Pool>` type handles both checks automatically.

```diff
  #[derive(Accounts)]
  pub struct Swap<'info> {
-     pub pool: AccountInfo<'info>,
+     #[account(
+         has_one = token_vault_a,
+         has_one = token_vault_b,
+     )]
+     pub pool: Account<'info, Pool>,
  }
```

---

## Example 3: Medium — PDA Seed Collision

### Different Users Can Derive The Same PDA Causing Account Overwrite

**Severity:** Medium

**Vulnerability Type:** PDA Seed Collision

## Description
The `create_profile` instruction derives a PDA using `seeds = [b"profile", name.as_bytes()]` where `name` is a user-provided string. The instruction uses `init` so the account is created on first call.
The problem is that two different users can provide the same name string. The first user creates the account successfully. Any subsequent user with the same name will fail to create their profile because the PDA already exists. More critically, if `init_if_needed` is used instead, the second user's data silently overwrites the first user's profile.

## Impact
With `init`, users can grief others by front-running popular profile names. With `init_if_needed`, user data can be overwritten. The fix is straightforward but the impact depends on how the profile data is used downstream.

## PoC
1. Alice calls `create_profile` with name "alice" — the PDA is created and stores her data.
2. Bob calls `create_profile` with name "alice" — if using `init_if_needed`, Alice's profile is overwritten with Bob's data.
3. Any downstream logic reading Alice's profile now sees Bob's data.

## Recommendations
It is recommended to include the user's public key in the PDA seeds which will guarantee uniqueness per user regardless of the name chosen.

```diff
  #[account(
      init,
      payer = user,
-     seeds = [b"profile", name.as_bytes()],
+     seeds = [b"profile", user.key().as_ref(), name.as_bytes()],
      bump,
      space = Profile::SIZE,
  )]
  pub profile: Account<'info, Profile>,
```
