# Example Findings — Solidity

These are reference examples of well-written findings. Read these when writing a new finding to calibrate tone, structure, and depth.

---

## Example 1: Critical — Missing Access Control

### Attacker Can Drain All Deposited Funds By Calling Withdraw With Any Token ID

**Severity:** Critical

**Vulnerability Type:** Missing Access Control

## Description
The `Vault` contract allows users to deposit ERC-20 tokens and receive a token ID representing their position. Users call `Vault.withdraw()` with their token ID to reclaim their deposited tokens.
However, `Vault.withdraw()` does not verify that `msg.sender` is the owner of the provided token ID. Any user can pass any valid token ID and receive the underlying tokens.
This means an attacker can enumerate all active token IDs and drain every depositor's funds in a single transaction per position.

## Impact
All deposited funds in the Vault are at risk. An attacker can steal 100% of the total value locked by iterating through active token IDs. No special permissions or capital are required.

## PoC
1. Alice deposits 1000 USDC into the Vault and receives token ID 42.
2. Bob calls `Vault.withdraw(42)` passing Alice's token ID.
3. The contract transfers 1000 USDC to Bob without checking ownership.
4. Alice's position is destroyed and her funds are irrecoverable.

## Recommendations
It is recommended to add an ownership check in the `withdraw()` function which will ensure only the token owner can withdraw the underlying assets.

```diff
  function withdraw(uint256 tokenId) external {
+     require(msg.sender == ownerOf(tokenId), "Not token owner");
      uint256 amount = deposits[tokenId];
      delete deposits[tokenId];
      token.transfer(msg.sender, amount);
  }
```

---

## Example 2: High — Reentrancy

### Malicious Token Receiver Can Re-Enter Claim Function And Double-Spend Rewards

**Severity:** High

**Vulnerability Type:** Reentrancy

## Description
The `RewardDistributor` contract lets users claim accrued rewards by calling `RewardDistributor.claim()`. The function calculates the pending reward, transfers it via an ERC-721 safe transfer, and then sets the user's claimed balance to the current total.
The problem is that `_safeMint()` triggers the `onERC721Received` callback on the recipient before the claimed balance is updated. A contract recipient can re-enter `claim()` during this callback and receive rewards a second time since the state still reflects the pre-claim balance.

## Impact
An attacker deploying a malicious receiver contract can drain the entire reward pool by repeatedly re-entering `claim()`. The attack requires deploying a contract but no special permissions or capital.

## PoC
1. Alice deploys a malicious contract that implements `onERC721Received` to call `RewardDistributor.claim()` again.
2. Alice calls `claim()` through her contract. The function calculates 500 tokens owed.
3. During the `_safeMint()` callback, Alice's contract re-enters `claim()` and receives another 500 tokens.
4. The re-entrancy repeats until the reward pool is drained.

## Recommendations
It is recommended to follow the checks-effects-interactions pattern by updating the claimed balance before making the external call. This removes the stale state that enables re-entrancy.

```diff
  function claim() external {
      uint256 pending = _calculateReward(msg.sender);
+     claimedBalance[msg.sender] = totalAccrued[msg.sender];
      _safeMint(msg.sender, pending);
-     claimedBalance[msg.sender] = totalAccrued[msg.sender];
  }
```

---

## Example 3: Medium — Rounding Error

### Protocol Loses Value Over Time Due To Rounding In Favor Of Withdrawers

**Severity:** Medium

**Vulnerability Type:** Precision Loss / Rounding Error

## Description
The `Pool` contract calculates a user's share of the pool on withdrawal using `Pool.withdraw()`. The formula divides the user's shares by the total supply and multiplies by the pool balance: `amount = (shares * balance) / totalSupply`.
Due to Solidity's integer division, this truncates in favor of the withdrawer when the result is not exact. Over many withdrawals, the protocol loses dust amounts per transaction. While each individual loss is small, the cumulative effect grows with transaction volume.

## Impact
The protocol slowly leaks value to withdrawers through rounding. On a high-volume pool processing thousands of withdrawals per day, the cumulative loss can become material over weeks to months.

## PoC
1. The pool holds 1,000,000 USDC with 999,999 total shares.
2. Bob holds 1 share and calls `withdraw()`.
3. The calculation yields `(1 * 1000000) / 999999 = 1` — Bob receives 1 USDC instead of ~1.000001 USDC.
4. Repeated across thousands of users, the pool balance diverges from what share-holders are collectively owed.

## Recommendations
It is recommended to round down in favor of the protocol (not the withdrawer) by using `mulDiv` with rounding-down behavior. This ensures the protocol never pays out more than its fair share.

```diff
- uint256 amount = (shares * balance) / totalSupply;
+ uint256 amount = Math.mulDiv(shares, balance, totalSupply, Math.Rounding.Down);
```

---

## Example 4: Low — Missing Event Emission

### Protocol Does Not Emit Event On Fee Update

**Severity:** Low

**Vulnerability Type:** Best Practice Violation

## Description
The `FeeManager.setFee()` function allows the admin to update the protocol fee percentage. The function correctly validates the new fee is within acceptable bounds and updates the state variable.
However, no event is emitted after the fee change. Off-chain monitoring systems and indexers have no way to detect fee changes without polling the contract state directly.

## Impact
Off-chain integrations and monitoring dashboards cannot track fee changes in real time. This is a best-practice deviation with no direct financial impact.

## Recommendations
It is recommended to emit an event after updating the fee which will allow off-chain systems to track changes efficiently.

```diff
+ event FeeUpdated(uint256 oldFee, uint256 newFee);
+
  function setFee(uint256 newFee) external onlyAdmin {
      require(newFee <= MAX_FEE, "Fee too high");
+     emit FeeUpdated(fee, newFee);
      fee = newFee;
  }
```
