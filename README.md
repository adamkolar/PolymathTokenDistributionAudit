# Polymath distribution contract + token contract
The purpose of the contract is to distribute polymath tokens between multiple owners and contracts according to this schedule:

* Presale supply - 240M (no vesting no cliff)
* Founder - 150M (4 year vest, 1 year cliff)
* Airdrop - 10M (no vesting, no cliff)
* Advisor - 25M (no vesting, 7 month cliff)
* Bonus - 80M (4 year vest, 1 year cliff) assigned to presale investors proportional to how much they purchased
* Reserve - 495M (4 year vest, 6 month cliff)

Each 'vest' drips out a linear amount over time from the `startTime` until the cliff.

# Contracts under review

 * [PolyDistribution.sol](./contracts/PolyDistribution.sol)
 * [PolyToken.sol](./contracts/PolyToken.sol)
 * [Ownable.sol](./contracts/Ownable.sol)

# List of issues

## 1. Due to mistake in a check, owner can transfer all tokens to an arbitrary address at any time

### PolyDistribution.sol / line 113

```javascript
    require(_token != address(this));
```

Supposed intent behind this check in the `refundTokens` function is to prevent owner from using the function to transfer POLY tokens, but instead of the POLY token address, the address of the distribution contract is used, so the check has no practical effect. The correct version of the code is:

```javascript
    require(_token != address(POLY));
```

## 2. Log variables are not indexed

### PolyDistribution.sol / line 41, 42

```javascript
  event LogNewAllocation(address _recipient, uint8 _fromSupply, uint256 _totalAllocated, uint256 _grandTotalAllocated);
  event LogPolyClaimed(address _recipient, uint8 _fromSupply, uint256 _amountClaimed, uint256 _totalAllocated, uint256 _grandTotalClaimed);
```

To make the log entries searchable, _rectipient and _fromSupply variables should be indexed (see: https://solidity.readthedocs.io/en/develop/contracts.html#events). The correct code should read:

```javascript
  event LogNewAllocation(address indexed _recipient, uint8 indexed _fromSupply, uint256 _totalAllocated, uint256 _grandTotalAllocated);
  event LogPolyClaimed(address indexed _recipient, uint8 indexed _fromSupply, uint256 _amountClaimed, uint256 _totalAllocated, uint256 _grandTotalClaimed);
```

# Other suggestions

At the time of the audit, the distribution schedule was not clear from the code comments, the readability of the code could be also improved if cliff and vesting times would be stored in named constants.


